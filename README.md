# Cache Coherence (MSI, MESI, MOESI) & NUMA Memory Access Performance

This repository contains the simulation scripts, C/OpenMP benchmark programs, and experimental results for evaluating hardware cache coherence protocols, false sharing penalties, and Non-Uniform Memory Access (NUMA) latency.

## 📋 Objective
- Study how snooping cache coherence protocols (**MSI**, **MESI**, and **MOESI**) maintain memory consistency across multi-core processors.
- Measure the performance degradation caused by **false sharing** when independent thread counters share the same 64-byte cache line versus residing on padded, separate cache lines.
- Evaluate **NUMA memory placement** performance by comparing local memory access (`--membind=0`) against remote interconnect memory access (`--membind=1`).

---

## 🛠️ Environment & Requirements
- **Virtualization:** VMware Workstation Pro
- **Operating System:** Ubuntu 22.04 LTS
- **Compiler:** GCC (`build-essential`) with OpenMP (`-fopenmp`)
- **System Tools:** `numactl`, `hwloc`, `python3`

---

## 📂 Repository Contents

| File | Description |
| :--- | :--- |
| `coherence_sim.py` | Python 3 snooping-cache simulator comparing bus transactions, invalidations, write-backs, and cache-to-cache transfers across MSI, MESI, and MOESI protocols. |
| `false_sharing.c` | C/OpenMP program demonstrating execution speed differences between adjacent counters (false sharing) and 64-byte padded counters. |
| `numa_sweep.c` | C/OpenMP memory bandwidth benchmark sweeping a 512 MB buffer to test local vs. remote NUMA node access latency. |
| `compare_numa.c` | Unified benchmark generating scalability tables across 1, 2, and 4 threads alongside NUMA memory sweep measurements. |
| `screenshots/` | Terminal output screenshots verifying compilation, simulation traces, NUMA topology, and timing runs. |

---

## 🚀 Compilation & Execution

### 1. Run the Cache Coherence Protocol Simulator
```bash
python3 coherence_sim.py 1000
```

### 2. Compile the C Benchmarks
```bash
gcc -O2 false_sharing.c -fopenmp -o false_sharing
gcc -O2 numa_sweep.c -fopenmp -o numa_sweep
gcc -O2 compare_numa.c -fopenmp -o compare_numa
```

### 3. Execute False Sharing Tests
```bash
# Adjacent counters (causes cache-line ping-ponging)
OMP_NUM_THREADS=2 ./false_sharing adjacent

# Padded counters (64-byte aligned, eliminates invalidations)
OMP_NUM_THREADS=2 ./false_sharing padded
```

### 4. Execute NUMA Memory Access Tests
First, verify your system's NUMA node topology:
```bash
numactl --hardware
```
Run local vs. remote memory tests:
```bash
# Local memory access (Threads on Node 0, RAM on Node 0)
OMP_NUM_THREADS=2 numactl --cpunodebind=0 --membind=0 ./numa_sweep

# Remote memory access (Threads on Node 0, RAM on Node 1)
OMP_NUM_THREADS=2 numactl --cpunodebind=0 --membind=1 ./numa_sweep
```

### 5. Run the Unified Comparison Benchmark
```bash
./compare_numa
numactl --cpunodebind=0 --membind=0 ./compare_numa
numactl --cpunodebind=0 --membind=1 ./compare_numa
```

---

## 📊 Key Observations
- **Protocol Efficiency:** **MESI** eliminates upgrade bus traffic on private data via silent `E -> M` transitions. **MOESI** eliminates forced memory write-backs on shared dirty lines via direct cache-to-cache (`O -> S`) transfers.
- **False Sharing Impact:** Adjacent counters suffer heavy invalidation traffic and fail to scale with additional threads. Padding counters to separate 64-byte cache lines drastically improves throughput and scalability.
- **NUMA Locality:** Keeping threads and their allocated memory on the same NUMA node avoids cross-interconnect latency penalties on multi-socket architectures.
