# Attention-Enhanced DeepSurv for Interpretable Gene-Expression Survival Modeling

This repository contains the code, models, and experimental configurations for
an interpretable survival-modeling framework built on **DeepSurv** and
feature-level attention mechanisms. The project aims to bridge predictive
performance and biological interpretability in high-dimensional
gene-expression survival modeling for precision oncology.

---

## My Contributions

- Led METABRIC data preprocessing and survival-outcome construction, including
  patient matching, feature selection, standardization, and stratified data splitting.
- Implemented and evaluated the L2-regularized Cox proportional hazards baseline,
  including C-index and Kaplan–Meier risk-stratification analysis.
- Conducted interpretability analysis through attention-ranking comparisons,
  Cox-based gene ranking, and perturbation validation.
- Contributed to model evaluation, result interpretation, final report writing,
  and presentation development.

> This repository contains the complete team project. See the original repository
> and final report for the full team contribution breakdown.

---

## Abstract

Survival analysis is fundamental in precision oncology for modeling time-to-event
outcomes using clinical and molecular data. While deep learning extensions of
the Cox model, such as **DeepSurv**, improve predictive flexibility by capturing
nonlinear interactions in high-dimensional gene-expression data, they provide
limited gene-level interpretability.

In this project, we developed an **Attention-Enhanced DeepSurv framework** to
bridge the gap between predictive performance and biological interpretability.
We explored multiple attention mechanisms that provide adaptive,
patient-specific gene-importance signals while predicting survival risk.

The proposed models were evaluated on the **METABRIC breast cancer cohort**
against L2-regularized Cox PH and baseline DeepSurv models. Interpretability was
further evaluated using attention-ranking comparison, Cox-based gene ranking,
perturbation validation, and pathway enrichment analysis.

---

## Dataset: METABRIC

The models are trained and evaluated using the **Molecular Taxonomy of Breast
Cancer International Consortium (METABRIC)** dataset.

- **Patients:** 1,980 after clinical and molecular record matching
- **Original features:** 20,385 unique gene-expression features
- **Selected features:** Top 3,000 most variable genes, selected using the training set only
- **Outcomes:** Overall survival time and event indicator
- **Splitting:** Stratified 70% / 15% / 15% Train / Validation / Test
- **Preprocessing:** Training-set-based feature selection, standardization,
  median imputation, and survival-outcome construction

> **Note:** Due to dataset size and access constraints, the raw METABRIC
> gene-expression data are not directly included in this repository.
> Data-access resources are provided below.

---

## Methodology & Architectures

<p align="center">
  <img src="Figures/GatedAttention_ResidualGatedAttention.png" width="700" height="350">
</p>

The project evaluates standard survival-analysis baselines and progressively
develops attention-enhanced DeepSurv architectures.

### 1. Cox PH Baseline

A traditional Cox proportional hazards model with **L2 regularization** was used
as an interpretable statistical baseline for high-dimensional
gene-expression data.

### 2. DeepSurv Baseline

A PyTorch implementation of **DeepSurv** replaces the linear Cox predictor with
a neural network while retaining the Cox partial log-likelihood objective.

### 3. Feature-Wise Self-Attention DeepSurv

Feature-wise self-attention was initially explored to model gene-to-gene
relationships. However, a full attention matrix scales quadratically with the
number of genes.

For 3,000 genes, this requires a `3000 × 3000` attention matrix and caused
out-of-memory (OOM) errors in our computational environment. The model was
therefore evaluated only as a reduced-scale feasibility experiment.

### 4. Gated Attention DeepSurv

To improve scalability, the full pairwise attention mechanism was replaced by
a **patient-specific continuous gate for each gene**:

\[
g_i = \sigma(f_\phi(x_i)), \qquad
\tilde{x}_i = x_i \odot g_i
\]

The gated representation is then passed to the DeepSurv risk-prediction
network.

This design enables all 3,000 selected genes to be used while providing
gene-level feature-weighting signals.

### 5. Residual Gated Attention DeepSurv

To address validation instability and potential over-suppression of useful
features in the standard gated model, a residual branch was added.

The architecture combines:

- a gated feature branch, and
- a residual branch that preserves the original normalized gene-expression signal.

The two representations are fused before survival-risk prediction, improving
validation stability while retaining adaptive feature weighting.

---

## Evaluation & Interpretability

### Predictive Performance

Model performance was evaluated using the **concordance index (C-index)**,
which measures the ability of a survival model to correctly rank patient risk.

### Risk Stratification

For the Cox PH baseline, patients were divided into high- and low-risk groups
and evaluated using:

- Kaplan–Meier survival curves
- Log-rank tests

### Interpretability Evaluation

Attention-derived gene rankings were not treated as direct biological truth.
Instead, interpretability was evaluated through multiple complementary analyses:

- **Attention-ranking comparison:** Compared gene rankings across Gated and
  Residual Gated Attention models.
- **Cox-based comparison:** Compared attention-derived rankings with
  Cox PH gene rankings.
- **Perturbation validation:** Shuffled candidate genes and measured the
  resulting decrease in model C-index.
- **Pathway enrichment:** Applied over-representation analysis using Enrichr
  against KEGG, GO Biological Process, Reactome, and MSigDB Hallmark databases.

---

## Results

### Predictive Performance

| Model | Train C-Index | Val C-Index | Test C-Index |
| :--- | :---: | :---: | :---: |
| **Cox PH (L2 Regularization)** | 0.8721 | 0.6333 | 0.6500 |
| **DeepSurv Baseline** | 0.9082 | 0.6303 | 0.6543 |
| **Feature-Wise Self-Attention DeepSurv** *(500 genes)* | 0.7806 | 0.6075 | 0.6107 |
| **Gated Attention DeepSurv** | 0.8203 | 0.6296 | **0.6660** |
| **Residual Gated Attention DeepSurv** | 0.8175 | **0.6507** | 0.6615 |

**Key findings:**

- **Gated Attention DeepSurv** achieved the highest held-out test C-index
  of **0.6660**.
- **Residual Gated Attention DeepSurv** achieved the highest validation
  C-index of **0.6507** and showed smoother validation behavior.
- Baseline DeepSurv achieved the highest training performance, but this did
  not translate into a substantial validation/test advantage.

---

## Interpretability Analysis: Gene-Ranking Stability

Because attention weights can be model-dependent, we compared the gene rankings
produced by the Gated Attention and Residual Gated Attention models.

| Top-k | Shared genes | Overlap rate | Jaccard index |
| :--- | :---: | :---: | :---: |
| **10** | 0 | 0.000 | 0.000 |
| **20** | 0 | 0.000 | 0.000 |
| **50** | 3 | 0.060 | 0.031 |
| **100** | 9 | 0.090 | 0.047 |
| **200** | 19 | 0.095 | 0.050 |
| **500** | 105 | 0.210 | 0.117 |

The low agreement between attention rankings indicates that attention scores
should be interpreted as **model-dependent feature-importance signals**, rather
than definitive biological evidence.

This motivated additional validation through Cox-based comparison,
perturbation testing, and pathway-level interpretation.

---

## Biological Interpretability

### Attention vs. Cox Ranking

Attention-derived gene rankings were compared with Cox PH rankings based on
absolute log hazard ratios and Cox p-values.

### Perturbation Validation

Candidate genes were perturbed by shuffling their expression values across
patients while preserving all other features.

The resulting decrease in C-index was used to estimate each gene's predictive
influence.

Representative perturbation-supported genes included:

- `CARD18`
- `IL12A`
- `GPR6`
- `ADIRF`
- `ASB4`
- `UGT2B17`
- `DBH`

### Pathway Enrichment

Perturbation-supported gene sets were evaluated using over-representation
analysis against:

- KEGG
- GO Biological Process
- Reactome
- MSigDB Hallmark

The resulting biological themes included immune regulation, vascular signaling,
metabolism, apoptosis, and protein regulation.

These findings are interpreted as **candidate survival-risk-related biological
signals**, not as confirmed causal biomarkers.

---

## Repository Contents

This repository includes:

- METABRIC data preprocessing and survival-outcome construction scripts
- L2-regularized Cox PH baseline
- PyTorch DeepSurv baseline
- Feature-Wise Self-Attention DeepSurv
- Gated Attention DeepSurv
- Residual Gated Attention DeepSurv
- Training and evaluation scripts
- Attention-ranking and Cox-ranking analyses
- Perturbation-validation analysis
- Pathway-enrichment analysis
- Figures and experimental configurations
- Documentation for reproducibility

---

## Team Contributions

All team members contributed to project discussion, result interpretation,
presentation preparation, and final report writing.

- **Irene Tsai — Data & Statistical Modeling**
  - METABRIC preprocessing and survival-outcome construction
  - L2-regularized Cox PH baseline
  - Attention-ranking and Cox-ranking comparison
  - Perturbation validation

- **Ananya Patel — Deep Survival Modeling**
  - PyTorch DeepSurv baseline
  - Residual Gated Attention DeepSurv
  - Neural-model training and validation analysis

- **Sang Hoon Chung — Attention Modeling & Biological Interpretation**
  - Gated Attention DeepSurv
  - Model comparison and interpretation
  - Pathway enrichment analysis
  - GitHub repository organization and documentation

*(Texas A&M University — ECEN 766 Algorithms in Structural Bioinformatics Final Project)*

---

## Data Access

Large METABRIC data files are hosted on Zenodo:

- **HiSeqV2 file:** https://doi.org/10.5281/zenodo.19359791
- **METABRIC processed pickle file:** https://doi.org/10.5281/zenodo.19866910
