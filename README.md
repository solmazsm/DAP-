## Distance-Aware Pruned Graphs for Efficient ANN Search (Submitted to VLDB 2026)

> **Distance-Aware Pruned Graphs for Accurate and Efficient Approximate Nearest Neighbor Search**  
> *Submitted to VLDB 2026*
---

## Introduction

This repository provides the source code for **DAPG**, a novel graph-based Approximate Nearest Neighbor (ANN) indexing system introduced in the paper:
DAPG integrates a **Distance-Aware Pruning (DAP)** mechanism into proximity graph construction, enabling efficient ANN search with reduced edge redundancy and improved recall. It adapts pruning thresholds based on **local geometric statistics**, producing sparse yet high-quality graphs.


---

## Compilation

The code is implemented in **C++11** and supports parallelism using **OpenMP**. It can be compiled on both Linux and Windows.

### Linux

```bash
cd ./cppCode/DAPG
make
```

###  Windows

Use **Visual Studio 2019+** to import the project located in:

```
./cppCode/DAPG/src/
```

Make sure to enable OpenMP and C++11 support in the build settings.

## Usage

### Command Format

```bash
./dapg datasetName
```

- `datasetName`: The name of the dataset (e.g., `sift`, `mnist`, `audio`)

### Example

```bash
cd ./cppCode/DAPG
./dapg sift
```

This runs DAPG index construction and search on the `sift` dataset.

## Key Features

-  Distance-Aware Local Pruning: Adapts edge filtering based on local percentile distances.
-  Sparsity Control: Limits node degree while preserving connectivity in sparse regions.
-  Improved Recall–Latency Tradeoff: Reduces query time without degrading recall.
-  Compatible with LSH-based Pipelines: Easily integrates with existing LSH-APG or other ANN frameworks.

## Dataset Format

The expected input format is a binary file containing float vectors, structured as:

```
{int: float size in bytes}
{int: number of vectors}
{int: dimension}
{float[]: all vector values, stored sequentially}
```

### Example: `sift.data_new`

To use your dataset:

1. Convert it into the binary format shown above.
2. Rename it as `[datasetName].data_new`
3. Place it in: `./dataset/`

A sample dataset (e.g., `audio.data_new`) is already provided.

## Benchmark Datasets

We support and have tested DAPG on:

- [Audio](https://github.com/RSIA-LIESMARS-WHU/LSHBOX-sample-data)
- [SIFT1M](http://corpus-texmex.irisa.fr/)
- [Deep1M](https://www.cse.cuhk.edu.hk/systems/hash/gqr/dataset/deep1M.tar.gz)
- [MNIST](http://yann.lecun.com/exdb/mnist/)
- [SIFT100M](http://corpus-texmex.irisa.fr/)


Convert these into the `.data_new` format for compatibility.

## System Setup

Our experiments were conducted on both local and cloud-based environments to evaluate the efficiency and scalability of the DAPG system.

###  Local Workstation
- **Processor**: 13th Gen Intel® Core™ i9-13900HX (24 cores, 32 threads)
- **Base Frequency**: 2.2 GHz  
- **OS**: Ubuntu 20.04 LTS  
- **Precision**: `float32` for all vectors  
- **Implementation**: C++ with multi-threading via OpenMP  
- **Query Setup**: 104 queries per experiment, averaged over 5 independent runs

###  Microsoft Azure Virtual Machines

1. **Standard F32s v2**
   - **vCPUs**: 32  
   - **RAM**: 64 GiB  
   - **Dataset**: SIFT1M  
   - **Purpose**: Scalability evaluation

2. **Standard E64-32s v3 (High-Memory)**
   - **vCPUs**: 32 (Intel® Xeon® Platinum 8272CL)  
   - **RAM**: 432 GiB  
   - **Disk**: 1 TB Premium SSD  
   - **OS**: Ubuntu 22.04 LTS  
   - **Dataset**: SIFT100M  
   - **Purpose**: Large-scale indexing and ANN benchmarking

This high-memory configuration allowed for efficient scaling to large datasets, and multi-threaded execution ensured fast parallel processing during both index construction and query search.
``` 
/*
 * 
 * Advisor: Professor Dongfang Zhao
 * Author: Solmaz Seyed Monir  
 * Affiliation: Database Research Group, University of Washington  
 * Description: Implementation of DAPG (Distance-Aware Pruned Graphs)  
 * Paper: Submitted to VLDB 2026  
 ## cppCode/DAPG/main.cpp
 *  Loop over W and k: Hyperparameter tuning for DAPG evaluation  
 *  Dataset default: Automatic dataset selection with preprocessing  
 *  Graph construction: Percentile-based pruning implemented  
 *  algNameStream: Logging pruning threshold in algorithm name  
 *  uniqueResults map: Multi-metric tracking for recall, cost, time, and pruning  
 *  Summary log: Index size, indexing time, recall, and pruning recorded  
 *  Time-stamped run end: Log end timestamp for reproducibility  
 *
 * All modifications by: Solmaz Seyed Monir
 */
```

## Benchmark Logs

The file [`mnist_all_index_stats.txt`](cppCode/LSH-APG/mnist_all_index_stats.txt) contains **experimental results** produced by running the DAPG implementation on the MNIST dataset.

Each row logs detailed metrics:

- **Recall**: Top-k retrieval accuracy
- **Pruning**: Ratio of retained neighbors after DAP-based filtering
- **Time**, **Cost**, and additional performance indicators
- **Algorithm Name**: Includes pruning threshold information (e.g., `DAP_k10_th...`)

All data in this file was automatically logged during batch experiments executed using the modified `main()` function in `cppCode/LSH-APG/src`.

These results validate the effectiveness of our DAPG method, as submitted in the VLDB 2026 paper.


### Experimental Results Log

We provide experimental results produced by our DAPG system on the MNIST dataset.

- The file [`mnist_result.txt`](./cppCode/DAPG/indexes/mnist_result.txt) contains timestamped benchmark logs produced from running LSH-G with varying `ef`, `k`, and pruning thresholds.
- The header includes configuration details such as:  
  `k=20, probQ=0.9, L=2, K=18, T=24`

Each row records:
- `algName`: algorithm configuration with pruning threshold
- `k`: number of neighbors
- `ef`: search parameter
- `Time`: average query time (ms)
- `Recall`: search recall
- `Cost`, `CPQ1`, `CPQ2`: computation cost metrics
- `Pruning`: pruning ratio applied

These logs support reproducibility and provide a transparent view of DAPG’s performance characteristics.


## Citation
```bibtex
@article{
  author    = {Solmaz S. Monir and Dongfang Zhao},
  title     = {Distance-Aware Pruned Graphs for Accurate and Efficient Approximate Nearest Neighbor Search},
  journal   = {Proc. VLDB Endow.},
  year      = {2026},
  note      = {Submitted}
}
```

## Directory Structure

```
.
├── cppCode/
│   ├── DAPG/
│   │   ├── src/           # Core source code
│   │   ├── include/       # Headers
│   │   ├── Makefile
│   │   └── bin/           # Executable output
├── dataset/               # Contains .data_new binary datasets
├── scripts/               # Python tools (optional)
└── README.md
```

## Contact

For questions or contributions, please open an issue or contact the authors listed in the paper.
