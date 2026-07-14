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

# Repository Highlights

This repository goes beyond a textbook implementation of the HHL algorithm by developing and validating an end-to-end Quantum Linear System Algorithm (QLSA) pipeline for sparse graph-based problems.

Highlights include:

- **Complete QPE-based HHL workflow**, from sparse state preparation to quantum measurement.
- **Berry–Childs quantum-walk block encoding** of sparse Hermitian matrices, including compiler-generated walk operators.
- **Two distinct eigenvalue inversion architectures**, illustrating the trade-off between qubit count and circuit depth.
- **Modular notebook design**, allowing each component of the algorithmic pipeline to be developed and validated independently.
- **Sparse financial graph models** based on graph Laplacians and normalized graph diffusion operators.
- **Rigorous engineering of sparse-oracle data structures**, including careful management of garbage-space orthogonality for Berry–Childs state preparation.
- **Automatic precision analysis**, including principled selection of QPE precision qubits and Chebyshev approximation degree based on user-specified error thresholds.
- **Fast Möbius Transform** implementation for efficiently computing Boolean monomial coefficients used in direct Chebyshev rotation synthesis (Variant 2).
- **Extensive classical verification**, comparing intermediate and final quantum results against exact numerical linear algebra.
- **Highly modular implementation**, making it straightforward to replace or extend individual algorithmic components.

---

# Repository Structure

The repository is organized around **five primary notebooks**, supported by several smaller exploratory notebooks developed during the early stages of the project.

```text
                        Main Computational Pipeline

                    Notebook C
          Berry–Childs Block Encoding
                     │
                     │  produces W_eff_positive
                     ▼
              Notebook A (V1 / V2)
      Complete HHL Circuit Construction
                     │
                     ▼
              Notebook B (V1 / V2)
     Execution, Validation & Verification
```

The responsibilities of the main notebooks are summarized below.

| Notebook | Purpose |
|----------|---------|
| **Notebook C – Berry–Childs** | Constructs the Berry–Childs block encoding for the chosen sparse system matrix and generates the effective walk operator `W_eff_positive`, which is subsequently reused by the HHL notebooks. |
| **Notebook A – Version 1** | Develops the complete HHL architecture using the "Compute, then Rotate" eigenvalue inversion strategy (Variant 1). |
| **Notebook A – Version 2** | Develops the complete HHL architecture using the "Rotate while Computing" eigenvalue inversion strategy (Variant 2). |
| **Notebook B – Version 1** | Executes and validates Variant 1 against the corresponding classical solution, including intermediate verification steps and fidelity analysis. |
| **Notebook B – Version 2** | Executes and validates Variant 2 using the same validation framework, allowing direct comparison between the two architectures. |

In addition to these primary notebooks, the repository also contains several supporting notebooks that document earlier exploratory work and prototype implementations. These notebooks are not required for running the main workflow but provide useful background on the evolution of the project.

---

# Execution Order

The notebooks are intended to be executed in the following order.

## Step 1 — Construct the Berry–Childs Walk Operator

Run

```text
Notebook_C_Berry_Childs.ipynb
```

for the desired system matrix configuration (model, graph size, diffusion parameter, sparsity, minimum edge weight, etc.).

This notebook constructs the complete Berry–Childs block encoding and generates the effective positive walk operator

```text
W_eff_positive
```

which serves as the Hamiltonian simulation primitive used by the HHL implementations.

---

## Step 2 — Transfer the Walk Operator

After Notebook C finishes execution, copy the generated matrix

```text
W_eff_positive
```

into the definition of

```text
W_eff
```

in **Cell 43** of the desired Notebook B implementation (Version 1 or Version 2).

It is important that **Notebook C and Notebook B use identical problem parameters**, including for example

- graph model,
- graph size,
- diffusion model,
- diffusion parameter `α`,
- minimum edge weight `J_min`,
- sparsity,
- normalization strategy, and
- any other parameters defining the system matrix `A`.

Using inconsistent parameters will produce a walk operator that no longer corresponds to the matrix being solved.

---

## Step 3 — Execute the Desired HHL Variant

Choose one of the two HHL architectures.

### Variant 1

```text
Notebook_B_Version_1_HHL_Architecture_with_Validation.ipynb
```

Implements the **Compute, then Rotate** approach, where the polynomial approximation is first computed into an auxiliary register before preparing the HHL ancilla.

---

### Variant 2

```text
Notebook_B_Version_2_HHL_Architecture_with_Validation.ipynb
```

Implements the **Rotate while Computing** approach, where the polynomial evaluation and ancilla preparation are performed simultaneously using Boolean monomial synthesis and multi-controlled rotations.

---

## Step 4 — Validate the Solution

Each validation notebook performs a complete comparison against the corresponding classical solution, including verification of intermediate quantities wherever possible.

The validation workflow includes

- construction of the classical reference solution,
- comparison of predicted quantum amplitudes,
- fidelity calculations,
- probability analysis,
- circuit visualization, and
- consistency checks for the complete HHL pipeline.

This modular workflow allows the Berry–Childs block encoding to be generated once and reused across multiple HHL implementations, making it straightforward to compare different eigenvalue inversion architectures while keeping the underlying Hamiltonian representation identical.

# Main Notebooks

The repository is centered around five primary notebooks that together implement and validate a complete QPE-based Quantum Linear System Algorithm for graph-based financial risk diffusion. Each notebook focuses on a well-defined stage of the overall pipeline, making it possible to study, verify, or extend individual components independently.

---

## Notebook C — Berry–Childs Block Encoding

**File:** `Notebook_C_Berry_Childs.ipynb`

This notebook implements the Berry–Childs quantum-walk block encoding of the system matrix `A`, which forms the Hamiltonian simulation primitive used throughout the project.

Major responsibilities include

- constructing sparse position and value oracles,
- generating the Berry–Childs state preparation operator,
- engineering orthogonal garbage and failure states,
- implementing the reflection and swap operators defining the quantum walk,
- synthesizing the effective walk operator `W_eff_positive`, and
- validating the resulting block encoding against the original matrix.

The matrix `W_eff_positive` generated by this notebook is subsequently reused by both HHL implementations. Since its construction depends on the chosen system matrix, **Notebook C should be rerun whenever the defining parameters of `A` are modified.**

---

## Notebook A — HHL Architecture Development (Versions 1 & 2)

**Files**

- `Notebook_A_Version_1_HHL_Architecture.ipynb`
- `Notebook_A_Version_2_HHL_Architecture.ipynb`

These notebooks develop the complete HHL circuit architecture without emphasizing large-scale execution or numerical validation.

The primary objective is to construct and document each stage of the algorithm, including

- sparse state preparation,
- Quantum Phase Estimation,
- eigenvalue inversion,
- inverse Quantum Phase Estimation,
- ancilla post-selection,
- measurement circuits, and
- overall circuit composition.

The two versions differ only in their eigenvalue inversion strategy.

### Version 1 — Compute, then Rotate

The polynomial approximation is first evaluated into an auxiliary register through a reversible polynomial evaluation oracle. Controlled rotations are then applied using the encoded polynomial values before the auxiliary register is uncomputed.

This approach requires additional qubits but results in a comparatively simple ancilla preparation circuit.

### Version 2 — Rotate while Computing

Instead of explicitly storing polynomial values, this implementation directly synthesizes the required ancilla rotations using the Boolean monomial representation of the polynomial approximation.

This considerably reduces the number of qubits while increasing circuit depth through multi-controlled rotations.

---

## Notebook B — Execution & Validation (Versions 1 & 2)

**Files**

- `Notebook_B_Version_1_HHL_Architecture_with_Validation.ipynb`
- `Notebook_B_Version_2_HHL_Architecture_with_Validation.ipynb`

These notebooks provide the primary execution and verification framework for the repository.

Using the `W_eff_positive` matrix generated by Notebook C, they execute the complete HHL workflow for the selected eigenvalue inversion architecture and compare every major result against the corresponding classical computation.

The validation framework includes

- classical solution generation,
- execution of the complete HHL circuit,
- fidelity calculations,
- probability comparisons,
- intermediate consistency checks,
- visualization of important circuit components, and
- verification of the final quantum solution state.

These notebooks therefore represent the main reproducible implementations of the repository.

---

# Supporting Notebooks

Several smaller notebooks document exploratory work performed during the development of the project. Although they are not required for the primary workflow, they provide useful insight into individual algorithmic components and the evolution of the implementation.

---

## `Simplified_SV_State_Prep.ipynb`

Investigates a simplified implementation of the Shukla–Vedula sparse state preparation algorithm used for preparing sparse equal-superposition shock states.

This notebook was used to verify correctness of the state preparation strategy before integrating it into the complete HHL pipeline.

---

## `HHL_QPE_Toy.ipynb`

A small educational implementation of the textbook HHL algorithm for tiny matrices.

Its purpose is to illustrate the fundamental mechanics of

- Quantum Phase Estimation,
- controlled eigenvalue inversion,
- inverse QPE, and
- post-selection

before introducing the substantially more sophisticated Berry–Childs block encoding architecture.

---

## `HHL_QPE_Toy_Efficient.ipynb`

An exploratory notebook investigating efficient reversible arithmetic required for scalable eigenvalue inversion.

In particular, it explores numerical techniques for implementing reversible polynomial evaluation and arithmetic operations needed within Quantum Phase Estimation–based HHL algorithms.

Although many of these ideas later influenced the final implementation, the notebook itself was not completed after the project adopted two dedicated eigenvalue inversion architectures:

- **Variant 1:** reversible polynomial evaluation followed by controlled rotations ("Compute, then Rotate"), and
- **Variant 2:** direct Chebyshev rotation synthesis using the Boolean monomial basis ("Rotate while Computing").

The notebook is therefore retained primarily as documentation of the project's design evolution.

---

# Algorithmic Pipeline

The complete workflow implemented in this repository is summarized below.

```text
Graph-Based Financial Network
              │
              ▼
Construct Sparse System Matrix A
              │
              ▼
Prepare Sparse Shock State |b⟩
              │
              ▼
Berry–Childs Block Encoding
              │
              ▼
Quantum Phase Estimation (QPE)
              │
              ▼
Eigenvalue Inversion
   ├───────────────┐
   │               │
   ▼               ▼
Variant 1      Variant 2
Compute         Rotate
then Rotate     while Computing
   │               │
   └───────────────┘
              │
              ▼
Inverse QPE
              │
              ▼
Ancilla & Phase Register Post-selection
              │
              ▼
Quantum Solution State |x⟩
              │
              ▼
Classical Verification
              │
              ▼
Financial Risk Estimation
```

Each stage of this pipeline is implemented as an independent module, allowing individual components to be tested, validated, and improved without affecting the remainder of the workflow.

This modular organization also makes it straightforward to compare alternative implementations of the same algorithmic stage—for example, the two eigenvalue inversion architectures—while keeping the underlying Berry–Childs block encoding, state preparation, and validation framework unchanged.

# Key Implementation Features

Although the repository focuses on a single application—graph-based financial risk diffusion—it develops a number of reusable components that are broadly applicable to Quantum Linear System Algorithms.

---

## Berry–Childs Quantum-Walk Block Encoding

The repository implements the Berry–Childs quantum-walk construction for block encoding sparse Hermitian matrices.

The implementation includes

- sparse position and value oracles,
- Berry–Childs state preparation,
- reflection about the image of the isometry,
- quantum-walk operator synthesis,
- compiler-generated local unitary construction, and
- validation of the resulting effective walk operator.

Particular care has been taken to ensure that the generated walk operator faithfully represents the underlying sparse matrix while remaining compatible with the subsequent Quantum Phase Estimation stage.

---

## Sparse-Oracle Engineering

Constructing efficient sparse oracles is one of the most important practical aspects of scalable Hamiltonian simulation.

This repository develops sparse-oracle representations that allow efficient access to

- locations of non-zero matrix entries,
- corresponding matrix values,
- row normalization data, and
- auxiliary information required during Berry–Childs state preparation.

These oracle data structures are reused throughout the implementation, avoiding unnecessary recomputation.

---

## Orthogonality of Garbage and Failure States

Berry–Childs state preparation introduces auxiliary states outside the desired signal space.

The implementation carefully engineers these garbage and failure states so that they remain orthogonal to the computational subspace throughout the construction of the block encoding.

Special attention is given to the allocation of failure basis indices and the prevention of unwanted interference between garbage states, ensuring that the resulting quantum walk satisfies the theoretical assumptions of the Berry–Childs construction.

---

## Two Independent Eigenvalue Inversion Architectures

A central objective of this repository is to investigate two different strategies for implementing the HHL eigenvalue inversion stage.

**Variant 1 — Compute, then Rotate**

- reversible polynomial evaluation,
- basis encoding of polynomial values,
- controlled ancilla rotations, and
- reversible uncomputation.

This architecture uses more qubits but requires a comparatively simple rotation circuit.

**Variant 2 — Rotate while Computing**

- Chebyshev polynomial approximation,
- Boolean monomial decomposition,
- direct ancilla preparation, and
- multi-controlled rotation synthesis.

This architecture eliminates the polynomial register at the cost of a deeper quantum circuit.

Implementing both variants within a common validation framework allows their resource requirements and numerical behaviour to be compared directly.

---

## Fast Möbius Transform for Boolean Basis Synthesis

Variant 2 requires the polynomial approximation to be expressed in the Boolean monomial basis before it can be synthesized into parameterized quantum rotations.

Rather than computing these coefficients directly—which becomes prohibitively expensive as the number of variables increases—the implementation employs the **Fast Möbius Transform** to efficiently compute the Boolean basis coefficients required for direct Chebyshev rotation synthesis.

---

## Automatic Precision Selection

Several numerical parameters must be chosen carefully to achieve a desired solution accuracy.

The repository includes systematic procedures for selecting

- the number of Quantum Phase Estimation precision qubits,
- spectral scaling constants,
- Chebyshev approximation intervals,
- polynomial approximation degree, and
- numerical error tolerances.

These quantities are derived from the spectral properties of the system matrix rather than being chosen heuristically.

---

## Modular Architecture

The implementation is organized into independent modules corresponding to the major stages of the HHL algorithm.

This separation allows individual components—including state preparation, block encoding, eigenvalue inversion, and validation—to be developed, tested, and replaced independently while maintaining a consistent interface between successive stages of the pipeline.

---

# Validation Methodology

Every major component of the implementation is validated against an equivalent classical computation before being integrated into the complete HHL workflow.

Rather than verifying only the final solution state, the repository performs intermediate consistency checks throughout the algorithm, making it easier to identify and isolate implementation errors.

The validation framework includes

- verification of sparse graph construction,
- validation of state preparation circuits,
- comparison of the Berry–Childs block encoding with the target matrix,
- verification of Quantum Phase Estimation outputs,
- comparison of eigenvalue inversion behaviour,
- fidelity calculations between quantum and classical solution states,
- probability comparisons for measured observables, and
- consistency checks after inverse Quantum Phase Estimation and post-selection.

Whenever practical, numerical quantities produced by the quantum circuit are compared against exact results obtained from classical linear algebra.

This layered validation strategy substantially simplifies debugging while providing confidence that each stage of the implementation behaves as expected before being incorporated into the complete Quantum Linear System Algorithm.
