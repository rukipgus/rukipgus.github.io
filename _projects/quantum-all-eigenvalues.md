---
layout: page
title: Quantum Algorithm for Finding All Eigenvalues of a Sparse Hermitian Matrix
description: Developed a full-spectrum quantum estimation scheme without prior eigenstate preparation using computational-basis sampling and QPE, and investigated sine-windowed QPE as a finite-precision enhancement. A later re-analysis revealed spectral resolution as a key bottleneck, shaping my current focus on quantum advantage under realistic resource constraints.
importance: 2
category: research
related_publications: false
---

**M.S. Thesis Research**

## Overview

My master's research investigated whether a quantum computer could estimate the **full spectrum of a large sparse Hermitian matrix without prior knowledge of its eigenvectors**.

Standard quantum phase estimation (QPE) can efficiently estimate an eigenvalue when a corresponding eigenstate is available. For full-spectrum reconstruction, however, those eigenvectors are generally unknown in advance. My approach was therefore to replace eigenstate preparation with easily preparable computational-basis states and recover the spectrum through repeated probabilistic sampling.

The project combined **Hamiltonian simulation, quantum phase estimation, computational-basis sampling, and window-function design**.

---

## Problem

Given an $N \times N$ sparse Hermitian matrix $H$, the goal is to estimate all of its eigenvalues

$$
\{\lambda_1,\lambda_2,\ldots,\lambda_N\},
$$

without assuming prior access to the corresponding eigenvectors.

A key difficulty is that conventional QPE starts from a state with nonzero overlap with the desired eigenstate. If the eigenvectors themselves are unknown, preparing such states for every eigenvalue becomes problematic.

This led to the central question of my thesis:

> **Can the entire spectrum of a sparse Hermitian matrix be reconstructed without explicitly preparing its eigenvectors?**

---

## Approach

Let

$$
H = \sum_{j=1}^{N} \lambda_j \lvert u_j\rangle \langle u_j\rvert
$$

be the spectral decomposition of the Hermitian matrix.

Instead of preparing an eigenstate $\lvert u_j\rangle$, I used computational-basis states $\lvert b\rangle$ as inputs:

$$
\lvert b\rangle = \sum_{j=1}^{N} \alpha_{bj} \lvert u_j\rangle
$$

Applying Hamiltonian evolution and QPE then produces eigenvalue estimates according to the overlap probabilities

$$
|\alpha_{bj}|^2.
$$

Because the computational basis and the eigenbasis are both orthonormal bases, averaging over computational-basis inputs distributes the total sampling weight across the complete set of eigenvectors.

Repeated measurements can therefore recover different eigenvalues without explicitly constructing their eigenstates. Full-spectrum reconstruction can be interpreted as a sampling process analogous to the **coupon collector problem**.

---

## Sine-Windowed Quantum Phase Estimation

A second part of the project focused on improving the finite-precision behavior of phase estimation.

Standard QPE uses a uniform superposition over the phase-estimation register. I instead investigated a **sine-windowed initial state**, assigning amplitudes according to a sinusoidal window before Hamiltonian evolution.

The motivation was to suppress spectral leakage away from the true eigenvalue. Compared with the conventional rectangular window, the sine window produces substantially faster decay of off-target probability.

In the asymptotic error regime, the leakage probability improves from approximately

$$
O\!\left(\frac{1}{\delta^2}\right)
$$

to

$$
O\!\left(\frac{1}{\delta^4}\right),
$$

where $\delta$ denotes the distance from the target phase in units of the QPE bin width.

I later examined how this improvement depends on the alignment between the true phase and the discrete QPE grid. To avoid relying on a favorable grid position, I swept the target phase across a full bin and evaluated both average- and worst-case success probabilities.

For the full **12-qubit LiH/STO-3G Hamiltonian**, sine-windowed QPE improved the grid-offset-averaged single-shot probability of estimating the ground-state energy within chemical accuracy from **88.4% to 96.6%** at $N_{\mathrm{EVAL}}=13$. The corresponding worst-case probability improved from **79.0% to 94.9%**.

{% include figure.liquid
   loading="eager"
   path="assets/img/projects/all-eigenvalues/sine_window_lih.png"
   class="img-fluid rounded z-depth-1"
   zoomable=true %}

*Single-shot chemical-accuracy success for conventional and sine-windowed QPE on the full 12-qubit LiH/STO-3G Hamiltonian. Results are shown both averaged over energy-grid offsets and for the worst-case offset.*

### Robustness under Implementation Errors

I then examined whether the idealized advantage of sine-windowed QPE survives imperfect Hamiltonian simulation and hardware noise.

With Trotterized evolution, the advantage was preserved when the systematic phase bias remained small relative to the QPE bin width. In the tested LiH active-space instances, the sine window retained its worst-case advantage for biases of approximately $0.14$ bin or less, whereas biases of $0.4$ bin or more could reverse the advantage.

Under calibration-based noise models from IBM Heron processors, the idealized sine-window advantage disappeared in the tested circuits. The observed difference was consistent in magnitude with the additional implementation overhead of the sine-window state preparation, including an extra ancilla and additional entangling gates.

These results showed that improved phase-estimation kernels do not automatically translate into improved hardware-level performance: the advantage depends on sufficiently accurate Hamiltonian evolution and sufficiently low implementation noise.

| Setting | Observation |
| --- | --- |
| Ideal QPE kernel, $r \in [1,2)$ | Sine window improves both average- and worst-case single-shot success |
| Full 12-qubit LiH, $N_{\mathrm{EVAL}}=13$ | Average: **88.4% → 96.6%**; worst case: **79.0% → 94.9%** |
| Trotter bias $\lesssim 0.14$ bin | Sine-window advantage preserved in the tested instances |
| Trotter bias $\gtrsim 0.4$ bin | Worst-case advantage can reverse |
| IBM Heron calibration-based noise | Idealized advantage disappears in the tested circuits |

---

## Initial Complexity Analysis

Under sparse-matrix oracle assumptions, I analyzed the cost of combining Hamiltonian simulation with the number of measurements required to observe the complete spectrum.

The resulting initial runtime estimate was

$$
\tilde{O}\!\left(Ns^2(\log N)^2\right),
$$

where $N$ is the matrix dimension and $s$ denotes matrix sparsity.

This analysis suggested that quantum spectrum reconstruction could substantially reduce the computational cost relative to conventional full diagonalization under the assumed oracle model.

However, this complexity estimate did not fully account for the physical cost required to **resolve increasingly dense eigenvalues**.

---

## Critical Re-evaluation: The Spectral-Resolution Bottleneck

I later revisited the algorithm from the perspective of finite spectral resolution.

If $N$ eigenvalues occupy a bounded spectral interval, the minimum spacing between neighboring eigenvalues can scale as $\Delta_{\mathrm{min}} = O(1/\mathrm{poly}(N))$ or become even smaller.

To distinguish two eigenvalues separated by $\Delta_{\mathrm{min}}$, phase estimation requires a precision $\epsilon < \Delta_{\mathrm{min}}$. The evolution time or query complexity required to achieve this precision scales at least inversely with $\epsilon$, giving a lower-bound scaling of $T = \Omega(1/\epsilon)$.

As a result, the dominant cost can shift from the **number of samples** required to collect the eigenvalues to the **resolution required for each individual sample**.

This observation showed that an oracle-level runtime analysis alone can substantially underestimate the physical resources required for full-spectrum reconstruction.

---

## Can Spectral Filtering Avoid This Bottleneck?

I also examined whether more sophisticated spectral transformations could circumvent the resolution problem.

One possibility is to construct a narrow spectral filter using polynomial methods such as QSVT. However, isolating spectral features separated by a small gap $\Delta$ requires increasingly high polynomial degree as the target transition becomes sharper.

Similarly, artificially stretching nearby eigenvalues requires a transformation with a rapidly varying slope within a narrow spectral interval. Polynomial approximation theory constrains how rapidly a bounded low-degree polynomial can vary.

These observations suggest that spectral transformations can redistribute approximation resources, but they do not eliminate the fundamental cost associated with resolving arbitrarily fine spectral structure.

---

## What I Learned

This project became important to my later research for a reason beyond the original algorithm itself.

My initial analysis focused primarily on asymptotic algorithmic complexity. Re-examining the method showed that the apparent advantage of full-spectrum reconstruction can be limited by the spectral resolution required for each phase-estimation sample.

The later window-function experiments reinforced the same lesson at the level of a single algorithmic primitive: an improvement that is clear under ideal phase estimation can be reduced or eliminated by Hamiltonian-simulation error and circuit-level noise.

Together, these analyses showed me that claims of quantum advantage must also account for

* spectral precision,
* Hamiltonian evolution time,
* oracle implementation,
* circuit depth,
* sampling complexity, and
* physically realizable resource constraints.

This experience motivated my current interest in designing and evaluating quantum algorithms under **realistic computational and hardware constraints**, rather than relying solely on abstract query-complexity improvements.

---

## Key Contributions

* Developed a full-spectrum quantum estimation framework that avoids prior eigenstate preparation.
* Formulated computational-basis sampling as a mechanism for probabilistically accessing the complete eigenvalue spectrum.
* Investigated sine-windowed QPE for suppressing spectral leakage and quantified its average- and worst-case finite-precision advantages across phase-grid offsets.
* Validated the sine-window improvement on the full 12-qubit LiH/STO-3G Hamiltonian and characterized Trotter error and calibration-based hardware noise as practical limits to the idealized advantage.
* Derived an initial sparse-oracle complexity estimate for full-spectrum recovery.
* Identified spectral resolution as a critical bottleneck omitted from the initial complexity analysis.
* Investigated the limitations of polynomial spectral filtering as a possible route around the resolution bottleneck.

---

## Research Topics

`Quantum Algorithms` · `Quantum Phase Estimation` · `Hamiltonian Simulation` · `Sparse Hermitian Matrices` · `Spectrum Estimation` · `Window Functions` · `Polynomial Approximation`
