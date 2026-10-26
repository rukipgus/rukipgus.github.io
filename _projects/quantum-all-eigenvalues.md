---
layout: page
title: Quantum Algorithm for Finding All Eigenvalues of a Sparse Hermitian Matrix
description: During my master's degree, developed a full-spectrum quantum estimation method using randomly sampled computational-basis inputs and QPE, and studied sine-windowed QPE to reduce finite-precision leakage. After graduation, revisited the minimum-gap bottleneck, explored QSVT-based approaches, and performed numerical simulations and real-QPU experiments to assess practical constraints.
importance: 2
category: research
related_publications: false
---

**M.S. Thesis Research and Post-Graduation Investigations**  
[Thesis PDF](/assets/pdf/Seongmuk_khang_master_thesis.pdf)

## Overview

My master's research investigated whether a quantum computer could recover **all eigenvalues of a large sparse Hermitian matrix without first preparing its eigenstates**.

During my master's degree, I developed an approach in which **one computational-basis state is randomly selected and prepared for each quantum phase estimation (QPE) run**. Repeated runs probabilistically collect different eigenvalues. I also studied **sine-windowed QPE** to reduce leakage in finite-precision phase estimation, and derived an initial sparse-oracle complexity estimate.

**After graduation**, I re-examined the **minimum spectral gap** as a resolution bottleneck, explored **QSVT-based spectral transformations** as a possible way to address it, and assessed QPE performance through **numerical simulations, hardware-noise modeling, and separate experiments on real QPUs**. These subsequent investigations shifted my attention from idealized query complexity toward quantum algorithms that remain useful under realistic precision and hardware constraints.

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

## Sine-Windowed QPE During My Master's Degree

During my master's research, I also investigated **sine-windowed quantum phase estimation** as a way to improve the finite-precision behavior of QPE.

Standard QPE uses a uniform superposition in its **phase-estimation register**. In sine-windowed QPE, this register is initialized with sinusoidally weighted amplitudes to reduce spectral leakage away from the true eigenphase. This is a modification of the phase-estimation register, **not** a preparation of a coherent superposition over all computational-basis inputs to the system register.

Compared with the standard rectangular window, the off-target probability decreases more rapidly in the asymptotic regime, from approximately

$$
O\left(\frac{1}{\delta^2}\right)
$$

to

$$
O\left(\frac{1}{\delta^4}\right),
$$

where $$\delta$$ denotes the phase offset in QPE-bin units. This theoretical improvement was part of my **master's-stage investigation**; the numerical benchmarks and hardware-related tests described below were carried out **after graduation**.

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

## After Graduation: Simulations and Real-QPU Experiments

Following my master's degree, I investigated how the sampling-based approach and the sine-windowed QPE kernel behave under finite precision and implementation constraints. This work included **numerical simulations**, **calibration-based hardware-noise modeling**, and **separate experiments using real QPUs**.

### Numerical Evaluation of Sine-Windowed QPE

To avoid evaluating only favorable alignments between an eigenphase and the discrete QPE grid, I swept the phase across a full QPE bin and examined both grid-offset-averaged and worst-case single-shot success probabilities.

For the full **12-qubit LiH/STO-3G Hamiltonian**, at $$N_{\mathrm{EVAL}}=13$$, sine-windowed QPE increased the grid-offset-averaged probability of estimating the ground-state energy within chemical accuracy from **88.4% to 96.6%**. The corresponding worst-case probability increased from **79.0% to 94.9%**.

{% include figure.liquid
   loading="eager"
   path="assets/img/all-eigenvalues/sine_window_lih.png"
   class="img-fluid rounded z-depth-1"
   zoomable=true %}

*Single-shot chemical-accuracy success for conventional and sine-windowed QPE on the full 12-qubit LiH/STO-3G Hamiltonian. Results are averaged over energy-grid offsets and evaluated at the worst-case offset.*

These numerical results support an improvement in the **finite-precision QPE kernel**. They do not imply that sine-windowing removes the minimum-gap bottleneck for reconstructing an arbitrarily dense spectrum.

### Robustness Under Simulation and Noise Models

I also evaluated the sine-windowed QPE improvement with **Trotterized Hamiltonian evolution** and **calibration-based noise models from IBM Heron processors**.

In the tested LiH active-space instances, the sine window retained its worst-case advantage when systematic phase bias was approximately **0.14 QPE bin or less**. At biases of approximately **0.4 bin or greater**, the advantage could reverse.

Under IBM Heron calibration-based noise models, the idealized advantage disappeared in the tested circuits. The additional state-preparation overhead, including an extra ancilla and entangling gates, was consistent with the scale of the observed degradation.

| Setting | Observation |
| --- | --- |
| Ideal QPE kernel, $$r \in [1,2)$$ | Sine window improves average- and worst-case single-shot success |
| Full 12-qubit LiH, $$N_{\mathrm{EVAL}}=13$$ | Average: **88.4% → 96.6%**; worst case: **79.0% → 94.9%** |
| Trotter phase bias $$\lesssim 0.14$$ bin | Sine-window advantage preserved in the tested instances |
| Trotter phase bias $$\gtrsim 0.4$$ bin | Worst-case advantage can reverse |
| IBM Heron calibration-based noise model | Idealized advantage disappears in the tested circuits |

### Hands-On Real-QPU Experience

After graduating, I also conducted experiments with **real quantum processing units (QPUs)**. These experiences made circuit implementation, sampling overhead, device noise, and hardware limitations concrete considerations in my evaluation of quantum algorithms.

The **LiH sine-window results and IBM Heron noise-model results above are from simulations and noise modeling**, not direct measurements of that specific benchmark on a real QPU. I therefore distinguish them from my separate hands-on QPU experiments.

---

## What I Learned

This research developed across two stages. **During my master's degree**, I proposed full-spectrum reconstruction using one randomly sampled computational-basis state per QPE run, studied sine-windowed QPE to mitigate finite-precision leakage, and derived an initial sparse-oracle complexity estimate.

**After graduation**, I revisited the **minimum eigenvalue-gap bottleneck**, explored QSVT-based spectral transformations as a possible response, and carried out numerical simulations and experiments with real QPUs. Subsequent noise and Trotter-error analyses showed that improvements to an ideal QPE component do not necessarily survive implementation overhead.

The combined experience made clear that plausible quantum advantage must be assessed using **spectral precision, Hamiltonian evolution time, oracle or block-encoding implementation, circuit depth, shot requirements, and hardware noise**, rather than query-complexity estimates alone.

This perspective shapes my current interest in developing quantum algorithms under realistic computational and hardware constraints.

---

## Key Contributions and Follow-Up Investigations

**Master's thesis research**

* Developed a full-spectrum estimation approach without prior eigenstate preparation, using **one randomly sampled computational-basis state per QPE run**.
* Analyzed spectrum collection and Hamiltonian-simulation costs under sparse-matrix oracle assumptions.
* Investigated **sine-windowed QPE** as a finite-precision enhancement and its suppression of spectral leakage.

**Post-graduation investigations**

* Identified the **minimum eigenvalue gap and finite spectral resolution** as limitations omitted from the initial resource estimate.
* Explored **QSVT-based polynomial spectral transformations** as a potential approach to this bottleneck, while recognizing degree-dependent implementation costs.
* Performed numerical **LiH/STO-3G simulations** quantifying sine-windowed QPE's grid-averaged and worst-case success probabilities.
* Examined **Trotter errors and IBM Heron calibration-based noise models**, and separately conducted experiments with **real QPUs**, to assess practical constraints.

---

## Research Topics

`Quantum Algorithms` · `Quantum Phase Estimation` · `Hamiltonian Simulation` · `Sparse Hermitian Matrices` · `Spectrum Estimation` · `QSVT` · `Spectral Resolution` · `Quantum Hardware`
