# Gem5simulator
# CS223 Assignment - Gem5 Simulator Modification Report

## Introduction

In this project, we set out to explore and analyze cache behavior by modifying the **gem5 simulator**. Our primary goal was to gain fine-grained insight into how different workloads interact with the cache, especially focusing on **per-set access patterns**, **hit/miss statistics**, and **interval-based tracking**.

We worked with benchmarks from **CPU2017**, particularly `sjeng` and `parest` (our prof has assigned us these particular benchmarks), and examined how cache size influences performance. To do this, we enhanced the gem5 infrastructure by adding new data tracking mechanisms that could capture detailed statistics not available by default in gem5.

---

## Project Overview

Our work focused on implementing:
- **Per-set hit and miss counters**: To determine which cache sets are accessed more frequently.
- **Set access frequency tracking**: Using C++ maps to monitor which sets are "hot" during execution.
- **Interval-specific statistics**: Leveraging `curTick()` to isolate performance between time intervals.
- **Integration into stats**: Ensuring our new metrics were logged into gem5’s stats output.

We modified key files such as `CacheMemory.hh`, `CacheMemory.cc`, and SLICC protocol files (`MESI_Two_Level`) to integrate these capabilities without disrupting gem5’s original statistics collection.

By doing this, we were able to:
- Identify cache hotspots,
- Understand workload-specific cache behavior,
- Observe how increasing cache size affects hits, misses, and latency.

This hands-on work not only gave us a deep understanding of **cache architecture** and **memory hierarchies**, but also practical experience working with a **cycle-accurate simulator** at a low level.

---

## 1. Changes Made to Gem5 Simulator

### a. Setup and Installation
- Downloaded `gem5` and `scons` folders into a single directory.
- Followed installation instructions from the official tutorial.
- Modified paths for build to refer to custom folders, specifically in:
  - `BoolVec.hh`
  - `DataBlock.hh`
  - `Set.hh`
  - `TBETable.hh`
  - `TimerTable.hh`
  - `WriteMask.hh`
- Ran the "Hello World" program successfully.
- Downloaded `cpu2017` benchmark zip and followed these steps:
  - Moved `CPU-2017.py` to `gem5/configs/common/`
  - Updated the script with correct benchmark paths
  - Imported `CPU-2017.py` into `se.py`

### b. File Modifications and Purpose

#### `CacheMemory.hh`
- Declared variables in `stats` struct (vectors and scalars).
- Introduced C++ `map` to track set access.
- Added function declarations for:
  - Top 5 most accessed sets
  - Per-set hits and misses
  - Interval-specific hits/misses

#### `CacheMemory.cc`
- Implemented the above functions.
- Initialized vectors in `CacheMemory::init()`.
- Used `ADD_STAT` to register statistics.
- Verified correctness with simulations.

#### `RubySlicc_Types.sm`
- Declared new helper functions for hit/miss tracking.

#### `MESI_Two_Level-L1cache.sm` and `L2cache.sm`
- Invoked set access functions within action commands.
- Called hit/miss tracking only during respective events.

### c. Justifications and Impact
- Created dedicated functions for interval-based hit/miss tracking using `curTick()`.
- Introduced per-set access frequency tracking for analysis.
- Aimed to observe behavioral patterns and cache hotspots.
- Enhanced simulator’s ability to give fine-grained access statistics.
- Did not interfere with original stats like total hits/misses.

---

## 2. Experimental Data Collection

Structured data was collected for each benchmark individually and stored using the custom stat functions. Results included:
- Top 5 most frequently accessed sets
- Per-set hit and miss counts
- Changes observed across cache sizes

---

## 3. Analysis and Conclusion

### a. Key Findings
- **sjeng benchmark**: Uniform cache access. Doubling size had minimal impact on hit/miss count, but increased latency. Top accessed sets: 43 (C4_1), 107 (C4_2).
- **parest benchmark**: Non-uniform access. Doubling cache size reduced misses, improved hit rate. Notable access hotspots at sets 40 (C4_1), 104 (C4_2).

### b. Observations and Challenges
#### Observations:
- Cache performance varies based on access pattern and workload.
- Cache size affects latency and area, not always beneficial.
- Identified sets could benefit from specific optimizations (e.g. prefetching).

#### Challenges:
- Modifying gem5 while ensuring correctness was complex and time-consuming.
- Interpretation of hit/miss variations required careful control over variables.
- Stats integration into readable format was a non-trivial task.

#### Potential Improvements:
- Investigate prefetching or prioritization for frequently accessed sets.
- Reduce associativity to minimise conflict misses.
- Extend analysis to L2 or LLC caches.

---
