---
layout: page
title: Quantum Algorithm for Finding All Eigenvalues of a Sparse Hermitian Matrix
description: Developed a full-spectrum quantum estimation method using randomly sampled computational-basis inputs and QPE without prior eigenstate preparation. After my master's degree, I revisited its minimum-gap resolution bottleneck, explored QSVT-based spectral transformations, and investigated finite-precision QPE and hardware constraints through simulations, noise analysis, and hands-on QPU experience.
importance: 2
category: research
related_publications: false
---

**M.S. Thesis Research and Subsequent Investigations**  
[Thesis PDF](/assets/pdf/Seongmuk_khang_master_thesis.pdf)

## Overview

My master's research addressed a basic question in quantum algorithms: **Can a quantum computer recover all eigenvalues of a large sparse Hermitian matrix without first preparing its eigenstates?**

I developed a sampling-based approach in which **one computational-basis state is randomly selected and prepared for each quantum phase estimation (QPE) run**. Repeating this process allows different eigenvalues to be collected probabilistically, without explicitly constructing their corresponding eigenvectors.

The initial analysis was encouraging under sparse-matrix oracle assumptions. After completing my master's degree, however, I revisited a missing resource cost: **the spectral precision needed to distinguish closely spaced eigenvalues**. I explored quantum singular value transformation (QSVT) as a possible way to reshape the spectrum, studied sine-windowed QPE to improve finite-precision estimation, and gained experience running quantum circuits on real quantum processing units (QPUs). These investigations shifted my focus from idealized query complexity toward quantum algorithms evaluated under realistic precision and hardware constraints.

---

## Master's Thesis: Problem and Sampling-Based Algorithm

### Problem

Given an $$N \times N$$ sparse Hermitian matrix $$H$$, the task is to estimate its eigenvalue spectrum

$$
\{\lambda_1,\lambda_2,\ldots,\lambda_N\}
$$

without assuming prior knowledge of its eigenvectors.

Conventional QPE estimates an eigenvalue when the input state has overlap with the associated eigenstate. Preparing a different eigenstate for every eigenvalue, however, is generally not practical when the eigenvectors are unknown.

My thesis therefore asked:

> **Can the entire spectrum be reconstructed by repeatedly applying QPE to easily preparable states instead of preparing the unknown eigenstates?**

### Randomly Sampling a Single Computational-Basis State

Consider the spectral decomposition

$$
H=\sum_{j=1}^{N}\lambda_j\lvert u_j\rangle\langle u_j\rvert.
$$

For **each QPE run**, I randomly select one index $$b$$ from the computational basis and prepare **only the corresponding basis state** $$\lvert b\rangle$$. The input is not a coherent uniform superposition of all computational-basis states.

Although a single $$\lvert b\rangle$$ is a basis state in the computational basis, it can be expanded in the unknown eigenbasis of $$H$$:

$$
\lvert b\rangle=\sum_{j=1}^{N}\alpha_{bj}\lvert u_j\rangle,
\qquad \alpha_{bj}=\langle u_j\vert b\rangle.
$$

In the ideal fully resolved, nondegenerate setting, applying Hamiltonian evolution and QPE to this **single selected input** produces the eigenvalue associated with eigenstate $$j$$ with probability

$$
P(j\mid b)=|\alpha_{bj}|^2.
$$

When $$b$$ is sampled uniformly, the orthonormality of the two bases gives

$$
P(j)=\frac{1}{N}\sum_{b=1}^{N}|\langle u_j\vert b\rangle|^2=\frac{1}{N}.
$$

Thus, averaging over **classically sampled basis-state preparations** makes each eigenvector index equally likely in the ideal case. Repeating QPE can recover the full spectrum as a sampling problem analogous to the **coupon collector problem**, provided the eigenvalues can be distinguished at the available precision. Degenerate eigenvalues are collected with probabilities proportional to their multiplicities.

The superposition used **inside the phase-estimation register** of QPE is a separate aspect of the algorithm; it does not mean that the system input is a superposition of computational-basis states.

---

## Initial Complexity Analysis

Under sparse-matrix oracle assumptions, I combined the cost of Hamiltonian simulation with the number of samples needed to observe the complete spectrum.

My initial runtime estimate was

$$
\tilde{O}\left(Ns^2(\log N)^2\right),
$$

where $$N$$ is the matrix dimension and $$s$$ denotes matrix sparsity.

This suggested a potentially favorable scaling relative to conventional full diagonalization within the assumed oracle model. However, that estimate did not fully include the cost of **resolving eigenvalues that become very close together**.

---

## After My Master's Degree: The Minimum-Gap Bottleneck

I subsequently re-examined the algorithm by asking how accurately QPE must resolve the spectrum in order to identify every eigenvalue.

Let $$\Delta_{\min}$$ denote the minimum separation between distinct eigenvalues. A sufficient resolution condition for separating adjacent eigenvalue estimates with bounded absolute error $$\epsilon$$ is

$$
2\epsilon<\Delta_{\min}.
$$

The Hamiltonian evolution time or query cost needed to resolve this energy scale grows at least inversely with the required precision:

$$
T=\Omega(1/\epsilon).
$$

A spectrum with many eigenvalues in a bounded energy interval can therefore contain gaps that are too small to resolve economically. The limiting resource is not just **how many QPE samples are collected**, but **how precisely each sample must be measured**.

This exposed an important limitation in my original complexity analysis: favorable sampling complexity alone does not establish an end-to-end quantum advantage for full-spectrum reconstruction.

### Exploring QSVT-Based Spectral Transformations

After identifying this issue, I explored whether **quantum singular value transformation (QSVT)** and related polynomial spectral transformations could help address the minimum-gap bottleneck.

The idea was to apply a function $$f$$ to the spectrum while keeping the eigenvectors unchanged:

$$
f(H)=\sum_{j=1}^{N}f(\lambda_j)\lvert u_j\rangle\langle u_j\rvert.
$$

If an implementable spectral transformation could enlarge the separation between nearby eigenvalues, the transformed spectrum might become easier to distinguish using phase-estimation or filtering procedures. Implementing such a transformation with QSVT requires a suitable block encoding and a bounded polynomial approximation (subject to the transformation's admissibility constraints).

The challenge is that resolving a narrow spectral interval requires a sufficiently sharp polynomial transformation. For a bounded polynomial approximation, producing sharper changes over a smaller gap generally requires higher polynomial degree and therefore greater implementation cost. QSVT offers a systematic framework for such transformations, but it does not by itself remove the resources needed to distinguish arbitrarily close eigenvalues.

I investigated this direction as a possible response to the bottleneck, **not as a demonstrated solution to the minimum-gap problem**.

---

## Further Investigation: Sine-Windowed Quantum Phase Estimation

I also investigated whether the finite-precision behavior of QPE itself could be improved.

Standard QPE uses a uniform superposition in its **phase-estimation register**. With sine-windowed QPE, this register is instead initialized using sinusoidally weighted amplitudes, reducing spectral leakage away from the true eigenphase.

Compared with the standard rectangular window, the off-target leakage has a faster asymptotic decay, improving from approximately

$$
O\!\left(\frac{1}{\delta^2}\right)
$$

to

$$
O\!\left(\frac{1}{\delta^4}\right),
$$

where $$\delta$$ is the offset from the target phase measured in QPE-bin units.

To avoid evaluating only favorable phase-grid alignments, I swept the target phase across an entire QPE bin and examined both average- and worst-case success probabilities.

For the full **12-qubit LiH/STO-3G Hamiltonian**, at $$N_{\mathrm{EVAL}}=13$$, sine-windowed QPE increased the grid-offset-averaged single-shot probability of estimating the ground-state energy within chemical accuracy from **88.4% to 96.6%**. The worst-case probability increased from **79.0% to 94.9%**.

{% include figure.liquid
   loading="eager"
   path="assets/img/all-eigenvalues/sine_window_lih.png"
   class="img-fluid rounded z-depth-1"
   zoomable=true %}

*Single-shot chemical-accuracy success for conventional and sine-windowed QPE on the full 12-qubit LiH/STO-3G Hamiltonian. Results are averaged over energy-grid offsets and evaluated at the worst-case offset.*

These results demonstrate an improvement in the **finite-precision QPE kernel**. They do not eliminate the minimum-gap bottleneck for reconstructing an arbitrarily dense spectrum.

---

## Real-QPU Experience and Implementation Limits

Beyond idealized analysis, I gained hands-on experience executing quantum circuits on **real QPUs**. This made the practical costs of circuit implementation, sampling, and noise an important part of how I evaluate quantum algorithms.

Separately, I examined the robustness of the sine-windowed QPE improvement using **Trotterized Hamiltonian evolution and calibration-based noise models from IBM Heron processors**. These LiH/QPE results were obtained from simulations and noise modeling; they should not be confused with direct experimental measurements of the sine-windowed LiH benchmark on a QPU.

With Trotterized evolution, the sine window retained its worst-case advantage in the tested LiH active-space instances when the systematic phase bias was approximately **0.14 QPE bin or less**. At biases of approximately **0.4 bin or greater**, that advantage could reverse.

Under IBM Heron calibration-based noise models, the idealized sine-window advantage disappeared in the tested circuits. The additional state-preparation overhead, including an extra ancilla and entangling gates, was consistent with the scale of the observed degradation.

| Setting | Observation |
| --- | --- |
| Ideal QPE kernel, $$r \in [1,2)$$ | Sine window improves average- and worst-case single-shot success |
| Full 12-qubit LiH, $$N_{\mathrm{EVAL}}=13$$ | Average: **88.4% → 96.6%**; worst case: **79.0% → 94.9%** |
| Trotter phase bias $$\lesssim 0.14$$ bin | Sine-window advantage preserved in the tested instances |
| Trotter phase bias $$\gtrsim 0.4$$ bin | Worst-case advantage can reverse |
| IBM Heron calibration-based noise model | Idealized advantage disappears in the tested circuits |

Together, the QPU experience and these separate numerical tests reinforced the distinction between an algorithm that works under ideal mathematical assumptions and one that retains its benefits after compilation and implementation costs are included.

---

## What I Learned

This research evolved in stages. My master's thesis proposed a way to recover a full eigenvalue spectrum **without explicitly preparing its eigenstates**, using individually sampled computational-basis inputs. Later, I recognized that the original sampling-based complexity estimate had not adequately captured the **minimum spectral resolution** required to distinguish closely spaced eigenvalues.

I then explored **QSVT-based spectral transformations** as a possible way to tackle that issue. Further work on **sine-windowed QPE**, implementation errors, and real-QPU experience revealed how improvements at the level of an ideal algorithmic component can be limited by other resources.

As a result, I now evaluate potential quantum advantage in terms of the combined costs of **spectral precision, Hamiltonian evolution time, oracle or block-encoding implementation, circuit depth, shot requirements, and hardware noise**, rather than relying on query-complexity expressions alone.

This progression has shaped my current interest in designing and assessing quantum algorithms under realistic computational and hardware constraints.

---

## Key Contributions and Follow-Up Investigations

* Developed an approach to full-spectrum estimation without prior eigenstate preparation.
* Used **one randomly sampled computational-basis state per QPE run**, with repeated independent preparations providing probabilistic access to the spectrum.
* Analyzed the initial sparse-oracle complexity of Hamiltonian simulation combined with spectrum collection.
* Identified the **minimum eigenvalue gap and finite spectral resolution** as limitations of the initial end-to-end resource estimate.
* After my master's degree, explored **QSVT-based polynomial spectral transformations** as a possible approach to the resolution bottleneck, while recognizing their degree-dependent costs.
* Investigated **sine-windowed QPE**, including phase-grid-offset averages and worst cases, on the full 12-qubit LiH/STO-3G Hamiltonian.
* Examined **Trotter errors and IBM Heron calibration-based noise models**, alongside separate hands-on experience running circuits on real QPUs, to assess hardware-level constraints.

---

## Research Topics

`Quantum Algorithms` · `Quantum Phase Estimation` · `Hamiltonian Simulation` · `Sparse Hermitian Matrices` · `Spectrum Estimation` · `QSVT` · `Spectral Resolution` · `Quantum Hardware`
