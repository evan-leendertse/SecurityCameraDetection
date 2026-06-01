# Robust PCA with Nyström Random Sampling

Background/foreground separation in surveillance video using **Robust PCA**, with an accelerated variant based on the **Nyström approximation**.

---

## Overview

Given a matrix of video frames $X \in \mathbb{R}^{21600 \times 600}$, Robust PCA decomposes it into:

- **L** — a *low-rank* matrix capturing the static background
- **S** — a *sparse* matrix capturing moving foreground objects

$$\min_{L,\, S} \; \|L\|_* + \lambda \|S\|_1 \quad \text{subject to} \quad L + S = X$$

The notebook then benchmarks this against an accelerated version that runs RPCA on two small random submatrices and reconstructs the full decomposition via the Nyström pseudo-inverse formula:

$$\tilde{X} = C \, U^\dagger R$$

---

## Results

| Method | Speedup | Final MSE |
|---|---|---|
| RPCA | 1× | $5.91 \times 10^{-5}$ |
| Nyström RPCA | ~4.8× | $3.93 \times 10^{-6}$ |

> The Nyström approach achieves significant runtime savings with a modest increase in reconstruction quality.

## Method

### Block Coordinate Descent (BCD)

The standard RPCA solver alternates between:

1. **Updating L** — singular value soft-thresholding on $X - S$
2. **Updating S** — element-wise soft-thresholding on $X - L$

using the soft-threshold operator $\mathcal{S}_\tau(x) = \text{sgn}(x) \cdot \max(|x| - \tau,\, 0)$.

### Nyström Acceleration

Rather than running BCD on the full $21600 \times 600$ matrix:

1. Sample **10% of columns** → $C \in \mathbb{R}^{21600 \times 60}$
2. Sample **10% of rows** → $R \in \mathbb{R}^{2160 \times 600}$
3. Run RPCA independently on $C$ and $R$
4. Reconstruct: $L = L_C \, U^\dagger \, L_R$ where $U = L_R[:, J]$

---
