# Quantum Linear System Algorithms for Graph-Based Financial Risk Diffusion

This repository presents a modular implementation of **Quantum Linear System Algorithms (QLSAs)** for solving sparse linear systems arising from **graph-based financial risk diffusion**. The project investigates how the Harrow–Hassidim–Lloyd (HHL) algorithm can be adapted into a practical end-to-end workflow, beginning with a mathematical model of financial contagion and progressing through sparse state preparation, Berry–Childs block encoding, Quantum Phase Estimation (QPE), eigenvalue inversion, and classical validation.

Unlike many educational implementations that focus only on the textbook HHL algorithm, this repository emphasizes the complete engineering pipeline required to build a realistic quantum linear solver. Particular attention is given to modular circuit construction, sparse-oracle design, reversible arithmetic, precision analysis, and systematic validation against classical numerical solutions.

The implementation targets graph-based financial risk propagation as a representative application, although the techniques developed here are broadly applicable to sparse linear systems encountered in scientific computing, optimization, machine learning, and network analysis.

---

# Overview

Many large-scale computational problems reduce to solving a sparse system of linear equations

```text
Ax = b
```

where the matrix `A` is sparse, well-conditioned, and too large for conventional dense linear algebra techniques to remain practical. Quantum Linear System Algorithms aim to prepare a quantum state proportional to the solution vector,

```text
|x⟩ ∝ A⁻¹|b⟩
```

potentially offering asymptotic speedups under appropriate assumptions regarding sparsity, state preparation, and matrix simulation.

This repository studies these algorithms in the context of **graph-based financial risk diffusion**. A financial network is represented as a sparse weighted graph whose nodes correspond to institutions and whose edges represent financial exposure. When a small number of institutions experience financial distress, the resulting risk propagates throughout the network according to a graph diffusion model. Estimating the steady-state risk scores therefore reduces to solving a sparse linear system.

Rather than treating HHL as a standalone quantum algorithm, this project develops the complete computational pipeline required for such an application. The implementation includes

- formulation of graph-based diffusion models,
- efficient preparation of sparse quantum states,
- Berry–Childs quantum-walk block encoding,
- Quantum Phase Estimation (QPE),
- two distinct architectures for coherent eigenvalue inversion,
- post-selection and measurement procedures, and
- validation against classical numerical solvers.

The repository is organized as a collection of modular notebooks that progressively build each component of the algorithm while maintaining a clear separation between mathematical derivations, quantum circuit construction, and verification.

---

# Project Motivation

Financial systems naturally form sparse networks in which institutions are connected through loans, investments, guarantees, or other financial exposures. A localized increase in risk can propagate through these connections, affecting other institutions even when they were not initially distressed.

A common approach to modelling this phenomenon is through **graph diffusion**, where each institution is represented as a node in a weighted graph and the edge weights quantify the strength of financial interaction. The resulting steady-state risk scores can be obtained by solving sparse linear systems involving graph Laplacians or normalized graph operators.

These systems possess several properties that make them attractive candidates for Quantum Linear System Algorithms:

- the matrices are naturally sparse,
- the underlying graphs are often locally connected,
- the required observables are typically aggregate statistics rather than the full solution vector, and
- the mathematical models admit Hermitian formulations that are well suited to HHL.

The primary objective of this repository is therefore not simply to implement HHL, but to demonstrate how a modern quantum linear-system workflow can be engineered for a realistic application. Considerable effort has been devoted to designing reusable software components, carefully analyzing numerical precision requirements, constructing efficient sparse-oracle representations, and validating every stage of the quantum computation against its classical counterpart.

Although the experiments presented here are performed on statevector simulators, the overall software architecture has been designed to reflect the modular structure expected of scalable fault-tolerant quantum algorithms, making it straightforward to replace or improve individual components as newer quantum techniques become available.

## Contents

- QPE-based HHL implementation
- Simple sparse state preparation
- Sparse Hamiltonian simulation
- Qubitization experiments
- Financial graph construction
- Classical verification pipelines

## Technologies

- Python
- Qiskit
- NumPy
- SciPy
