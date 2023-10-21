# PTM

## Prerequisites

### Environment

This project is developed on CentOS 8.5.2111. 
It should work on other Linux distributions as well.

This project requires a C++ compiler with C++20 support. 
It is known to work with GCC 12.2.0

### Library

The following libraries are required necessarily. 
- Intel OneAPI-OneTBB
- numa
- papi
- jemalloc

There are some unnecessary libraries. You have to remove them from CMakeLists.txt if you don't have.
- hwloc
- gtest
- gprof


## Build and Run

### Preparation
Before building process, you should execute a python script to detect
the information about architecture.
```shell
python3 arch_dect.py --pmem_dir=[path]
```
Where `[path]` denote the directory on PMEM for data storage.

Besides, you need to edit the file `src/main.cpp` to determine the configuration of 
components.

### Normal execution
Then enter the build directory to launch cmake and unix makefile
```shell
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j
./PTM
```

### Automatic batching execution
For automatic building and testing, you need to edit the file `auto_test.py` 
and execute it.
```shell
python3 auto_test.py
```

## Detail

- Design: [Design Principle](doc/DesignPattern.md)
- Workload: [Workload README](workload/README.md)

### Module Organization

There are four modules in this code:

- util
- concurrent_control
- storage_manager
- workload

The `util` module is independent, while the other are ONLY based on module `util`. 
This means you can move one of them to other programs only with util module.

Besides, the directories `include` and `src` are just coordinators among modules above in this project.

`graph` contains some test result and corresponding scripts for graph plotting.

Note that the directory `concurrentqueue` is an outer library.

### Component Configuration

- `WORKLOAD_DEFINED` type of workload
  - **TPCC**
  - **YCSB_A**
  - **YCSB_B**
  - **YCSB_C**
  - **YCSB_D**
  - **YCSB_E**
  - **YCSB_F**
- `INDEX_DEFINED` type of index
  - **HashMap**
  - **BPTree**
- `CONCURRENT_CONTROL_DEFINED` type of concurrent control
  - **OCC**
  - **TPC**
  - **TICTOC**
  - **MVCC**
- `TRANSACTION_MANAGER_DEFINED` type of transaction manager
- `STORAGE_MANAGER_DEFINED` type of storage manager

### Recorder Configuration

- `LOGGER_OUTPUT` : output destination of logger
  - **FILE** : output by file
  - **CONSOLE** : output by terminal console
- `LOGGER_OUTPUT_FILE_NAME` path of file for output if needed

### NVM Configuration

- `PWB` type of instruction for flushing
  - **CLWB**
  - **CLFLUSH**
  - **CLFLUSHOPT**


## Trick
To compare with other PTM which is tested on tricky benches, there are several
tricks should be applied.
### Workload
Mainly for YCSB.
- Decrease the number of field
- Decrease the size read/updated once
- Cancel the asymmetry load between read and write

### Concurrent Control
- When running under YCSB, cancel the read before update
- When running under YCSB, decrease the size reserved for read/write set
- Adjust the `MAX_RETRY_TIMES` for locking
- Adjust the `MAX_BATCH_NUM` and `TASK_HIGH_LEVEL` for delaying batching writes.

### Transaction Manager
- Execute new transaction rather than retrying.
- Use preload workload

### Util
- If use auto test, make MAX_TID controlled by MACRO

## CMake Configuration

To enable configuration by cmake, you need to add `-DAUTO_TEST=true` as
one of the params of cmake at first.


## Reference

- CRWWP SpinLock 
- [FAST_FAIR](https://github.com/DICL/FAST_FAIR) A concurrent B+ tree
- [moodycamel::ConcurrentQueue](https://github.com/cameron314/concurrentqueue) A scalable concurrent queue