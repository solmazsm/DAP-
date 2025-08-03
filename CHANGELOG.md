

## Description: Implementation of DAPG (Distance-Aware Pruned Graphs)  
  Paper: Submitted to VLDB 2026  
 ## cppCode/DAPG/main.cpp
 *  Loop over W and k: Hyperparameter tuning for DAPG evaluation  
 *  Dataset default: Automatic dataset selection with preprocessing  
 *  Graph construction: Percentile-based pruning implemented  
 *  algNameStream: Logging pruning threshold in algorithm name  
 *  uniqueResults map: Multi-metric tracking for recall, cost, time, and pruning  
 *  Summary log: Index size, indexing time, recall, and pruning recorded  
 *  Time-stamped run end: Log end timestamp for reproducibility  
 
 All modifications by: Solmaz Seyed Monir




## [2025-08-01] – Local Percentile-Based Pruning Integration
### Key Features and Observations

#### Hybrid LSH + Graph Construction
- Integrated LSH-based candidate retrieval (`searchLSH`) with incremental graph building.
- Used `insertLSHRefine` to refine candidate neighbors before inserting them into the graph.

#### Distance-Aware Pruning (DAP)
- Introduced a percentile-based thresholding mechanism.
- For each node, computed the 80th percentile distance (τ_q) over LSH candidates.
- Inserted only neighbors with `dist < τ_q` to ensure sparsity and relevance.
- Exposed `last_threshold` for optional diagnostics or debugging.

####  Neighbor Management
- Supports two pruning strategies:
  - `chooseNN_div`: Ensures selected neighbors are both close and diverse (distance-based repulsion).
  - `chooseNN_simple`: Simplified version using max-heap filtering.

####  Parallel Construction
- Used `ParallelFor` to insert all nodes in parallel (except the first).
- Enabled thread safety via `std::shared_mutex` for concurrent graph updates.
- Fallbacks to `std::mutex` when C++17 is not available.

#### Serialization
- Graph (`linkLists`) and LSH hash tables are saved to and loaded from a binary format.
- Implemented in `save()` and the constructor `divGraph(Preprocess* prep, ...)`.

####  Search Procedure
- Two-stage hybrid search:
  1. LSH-based candidate generation (`searchLSH`).
  2. Graph-based search refinement (`bestFirstSearchInGraph`).
- Reflects an LSH-HNSW-style retrieval pipeline for improved accuracy and speed.

---



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

The experimental results presented here were produced by our DAPG system on the MNIST dataset to support reproducibility and performance evaluation.

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

### Parameter Settings

We evaluate DAP (Distance-Aware Pruning) with:

- `K = 18`, `L = 2`, `T = 24`, `T′ = 48`
- `W = 1.0`, `pC = 0.95`, `pQ = 0.9`, `efC = 80`

DAP applies **local dynamic pruning**, computing a threshold `τ_q` per node, unlike LSH-APG's global percentile pruning (`p = 95`). This results in better graph sparsity and recall-efficiency.

We tune `K = 18` (vs. LSH-APG's default `K = 16`) for fairness.

#### Baseline Settings

- **HNSW**: `M = 48`, `ef = 80`  
- **NSG**: `L = 40`, `R = 50`, `C = 500`  
- **HCNNG**: 10 iterations, cluster size ≤ 500  
- **DB-LSH**: `c = 1.5`, `K = 12`, `L = 5`

---

### Evaluation Setup

We evaluate DAP across a range of:

- `k ∈ {1, 10, 20, ..., 100}`
- `ef` values for query expansion

This allows robust analysis of recall and efficiency across diverse search settings

**Modified and Authored by:**  
**Solmaz Seyed Monir**, Ph.D. Student  
**Professor Dongfang Zhao**, Advisor  
Affiliation: Database Research Group, University of Washington  
Date: **2025-08-01**  
Affected Files: `divgraph.cpp`, `Node2`, `insertLSHRefine`, `README.md`
