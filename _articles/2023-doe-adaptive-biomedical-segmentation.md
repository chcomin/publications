---
layout: article
title: "Adaptive Segmentation of Biomedical Images Using Convolutional Neural Networks with Uncertainty Estimation"
authors:
  - name: "Jane Doe"
    orcid: "0000-0001-2345-6789"
  - name: "John Smith"
    orcid: "0000-0002-3456-7890"
  - name: "Maria Oliveira"
date: 2023-04-15
year: 2023
journal: "IEEE Transactions on Medical Imaging"
volume: "42"
issue: "4"
pages: "1120-1134"
doi: "10.1109/TMI.2023.0000001"
abstract: |
  Accurate segmentation of anatomical structures in biomedical images remains
  a challenging task due to high intra-class variability and limited annotated
  data.  We propose an adaptive convolutional neural network architecture that
  incorporates Monte Carlo dropout for uncertainty estimation during inference.
  Our method produces pixel-wise confidence maps alongside segmentation masks,
  enabling downstream algorithms to discount unreliable predictions.  We
  evaluate the approach on three public benchmarks — DRIVE (retinal vessels),
  MoNuSeg (cell nuclei), and BraTS 2021 (brain tumors) — and report Dice
  coefficients of 0.821, 0.793, and 0.886, respectively, outperforming five
  competing methods.  Ablation studies confirm that uncertainty-guided training
  consistently improves boundary delineation under domain shift.
keywords:
  - biomedical image segmentation
  - convolutional neural networks
  - uncertainty estimation
  - Monte Carlo dropout
  - retinal vessel segmentation
mathjax: true
license: "CC BY 4.0"
bibtex: |
  @article{doe2023adaptive,
    title   = {Adaptive Segmentation of Biomedical Images Using Convolutional
               Neural Networks with Uncertainty Estimation},
    author  = {Doe, Jane and Smith, John and Oliveira, Maria},
    journal = {{IEEE} Transactions on Medical Imaging},
    year    = {2023},
    volume  = {42},
    number  = {4},
    pages   = {1120--1134},
    doi     = {10.1109/TMI.2023.0000001}
  }
---

## Introduction

Semantic segmentation of biological and clinical images underpins a broad
range of computer-aided diagnosis pipelines.  Despite the impressive
performance of fully convolutional networks, production deployments are
hampered by overconfident predictions: the model yields high-probability
masks even when the input is noisy, artefacted, or outside the training
distribution.

Bayesian deep learning provides a principled solution.  Treating network
weights as random variables and approximating the posterior with Monte Carlo
(MC) dropout [CITE] yields calibrated predictive uncertainty at the cost of
$T$ stochastic forward passes, where $T$ is a small integer.  The predictive
entropy at pixel $i$ is

$$
H_i = -\sum_{c} \bar{p}_{ic} \log \bar{p}_{ic},
\qquad \bar{p}_{ic} = \frac{1}{T}\sum_{t=1}^{T} p_{ic}^{(t)},
$$

where $p_{ic}^{(t)}$ is the softmax probability for class $c$ at pixel $i$
in sample $t$.

The contributions of this paper are:

1. An end-to-end adaptive segmentation network whose encoder selectively
   suppresses features with high epistemic uncertainty.
2. A training curriculum that progressively increases the proportion of
   uncertain regions presented to the model.
3. State-of-the-art Dice scores on three public benchmarks.

## Methods

### Network Architecture

Our backbone follows a U-Net topology [CITE] with residual blocks replacing
standard convolutional stacks.  All dropout layers use $p = 0.3$ and remain
active during both training and inference.  The decoder incorporates an
attention gate [CITE] scaled by the inverse of the predictive entropy
$H_i$:

$$
\alpha_i = \sigma\!\left(\frac{W_x x_i + W_g g_i}{1 + \lambda H_i}\right),
$$

where $x_i$ is the skip connection feature, $g_i$ the gating signal, and
$\lambda$ a learned scalar initialized to 1.

### Uncertainty-Guided Training

At each training step we perform $T = 10$ stochastic forward passes and
compute $H_i$ for every pixel.  Pixels whose entropy exceeds the 80th
percentile are assigned twice the cross-entropy weight in the loss:

$$
\mathcal{L} = \frac{1}{N}\sum_{i=1}^{N} w_i \,
  \mathrm{CE}(\hat{y}_i, y_i), \qquad
w_i = \begin{cases} 2 & H_i > Q_{0.8} \\ 1 & \text{otherwise.} \end{cases}
$$

### Datasets

**DRIVE.** 40 fundus images ($565 \times 584$ px) with manual vessel
annotations split into 20/20 train/test.

**MoNuSeg.** 30 H&E stained tissue images at $1000 \times 1000$ px with
instance-level nucleus annotations.

**BraTS 2021.** 1251 multi-parametric MRI cases with expert segmentations
of glioma sub-regions.

## Results

Table 1 summarises Dice coefficients ($\pm$ standard deviation over five
runs) for all baselines and our method.

| Method | DRIVE | MoNuSeg | BraTS 2021 |
|---|---|---|---|
| U-Net | 0.798 ± 0.006 | 0.764 ± 0.009 | 0.867 ± 0.004 |
| Attention U-Net | 0.809 ± 0.005 | 0.775 ± 0.007 | 0.874 ± 0.003 |
| MC-Dropout U-Net | 0.814 ± 0.004 | 0.781 ± 0.008 | 0.879 ± 0.003 |
| **Ours** | **0.821 ± 0.003** | **0.793 ± 0.006** | **0.886 ± 0.002** |

Our uncertainty-weighted curriculum consistently improves boundary Dice by
1–2 points across all three tasks.  Qualitative inspection reveals tighter
boundaries around thin retinal vessels and cell edges, where MC entropy is
highest.

## Conclusion

We presented an adaptive segmentation framework that leverages predictive
uncertainty to focus learning on ambiguous regions.  The entropy-gated
attention and uncertainty-weighted loss jointly improve calibration and
segmentation accuracy.  Future work will extend the approach to 3-D
volumetric segmentation and explore consistency regularisation for
semi-supervised settings.

## References

[CITE] Gal, Y., & Ghahramani, Z. (2016). Dropout as a Bayesian approximation.
*ICML*.

[CITE] Ronneberger, O., Fischer, P., & Brox, T. (2015). U-Net: Convolutional
networks for biomedical image segmentation. *MICCAI*.

[CITE] Oktay, O., et al. (2018). Attention U-Net: Learning where to look for
the pancreas. *MIDL*.
