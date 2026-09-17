# DQC Transpilation Overhead Analysis

> **An empirical study evaluating the impact of monolithic transpilation strategies (Qiskit vs. TKET) on Distributed Quantum Computing (DQC) inter-QPU entanglement overhead and partitioned circuit depth.**

## 📌 Project Overview
As monolithic Noisy Intermediate-Scale Quantum (NISQ) processors approach physical scaling limits, Distributed Quantum Computing (DQC) has emerged as a viable solution by networking modular Quantum Processing Units (QPUs). A critical challenge in DQC is mapping monolithic circuits across networked QPUs while minimizing the severe communication bottleneck caused by non-local routing (e-bit consumption).

While current DQC literature evaluates various partitioning algorithms based on e-bit reduction, the impact of **pre-compilation (monolithic transpilation)** on distributed structural overhead remains largely unexplored. This repository contains the complete experimental pipeline investigating whether the choice of monolithic transpiler—specifically **Qiskit vs. TKET**—directly affects the quality and structural overhead of DQC circuit partitioning.

## 🔬 Methodology

Our simulation-based benchmarking pipeline consists of three distinct phases to ensure a controlled, reproducible, and rigorous evaluation:

### Phase 1: Dataset Generation
Generated a controlled dataset of 15 target-independent quantum circuits utilizing the [Munich Quantum Toolkit (MQT) Bench](https://github.com/cda-tum/mqt-bench). 
*   **Algorithms:** Quantum Fourier Transform (QFT), Quantum Approximate Optimization Algorithm (QAOA), and Random Quantum Circuits.
*   **Scalability:** Circuits generated at sizes of 16, 32, 64, 80, and 96 qubits.

### Phase 2: Monolithic Transpilation
Unrolled circuits were decomposed to enforce identical starting conditions and compiled using:
*   **Qiskit** (Optimization Level 2)
*   **TKET** (Full Peephole Optimise, with `OpType.CX` enforced to match Qiskit's cost metric)
*   **Metrics Tracked:** 2-Qubit (CX) Gate Count, Circuit Depth, and Compilation Execution Time.

### Phase 3: DQC Partitioning Simulation
To strictly isolate the topological footprint of the monolithic compilers from secondary routing heuristics, DQC partitioning was abstracted into a pure graph-theoretic model using `NetworkX`. 
*   **Clustering Heuristic:** Recursive Kernighan-Lin Bisection (balanced 4-QPU mapping).
*   **Metrics Tracked:** Inter-QPU E-Bit Cost (cross-partition edge cuts) and Distributed Depth (incorporating strict latency penalties for remote operations).

## 🚀 Key Findings: The Depth Reversal Paradox
The most critical finding of this study is the inversion of circuit depth when transitioning from a monolithic to a distributed environment, particularly evident in the **QFT algorithm**.

1.  **Monolithic Level:** TKET aggressively optimized for circuit depth (664 vs. Qiskit's 758 at 96 qubits) but at the expense of introducing massive two-qubit gate inflation (4,290 CX gates vs. Qiskit's 2,478).
2.  **Distributed Level (E-Bits):** Qiskit's gate-efficient strategy yielded a massive communication advantage, consuming only 630 E-bits compared to TKET's 2,082 E-bits.
3.  **The Paradox:** When factoring in temporal penalties for remote teleportation, TKET's distributed depth skyrocketed to 6,910 layers, while Qiskit maintained a significantly shallower distributed depth of 2,648 layers.

**Conclusion:** Optimizing exclusively for monolithic circuit depth (as seen with TKET) becomes critically detrimental in DQC environments if it inflates two-qubit gate counts. For distributed architectures, **gate minimization (Qiskit) is significantly superior**, as it successfully suppresses exponentially expensive E-bit communication costs.

## 📂 Repository Structure

```text
DQC-Transpilation-Overhead-Analysis/
│
├── data/
│   ├── raw_circuits/              # Original target-independent MQT Bench .qasm files
│   └── optimized_circuits/        # Qiskit & TKET transpiled .qasm files
│
├── results/
│   ├── csv/                       # Generated metrics for Phase 2 & Phase 3
│   └── figures/                   # High-resolution plots of transpilation scaling
│
├── DQC_Compiler_Analysis.ipynb    # Master Jupyter Notebook containing the full pipeline
└── README.md
