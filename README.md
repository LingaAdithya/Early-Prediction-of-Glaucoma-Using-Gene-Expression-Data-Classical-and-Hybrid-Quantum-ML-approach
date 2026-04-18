# Early Prediction of Glaucoma Using Gene Expression Data
### A Comparative Study of Classical and Hybrid Quantum Machine Learning Models

---

## Overview

This project presents an end-to-end computational pipeline for the early prediction of Primary Open-Angle Glaucoma (POAG) from gene expression microarray data. It benchmarks three classical supervised classifiers — Logistic Regression, Random Forest, and Support Vector Machine — against a Hybrid Variational Quantum Classifier (VQC) implemented using PennyLane. The pipeline integrates multi-dataset preprocessing, differential gene expression (DEG) analysis, leakage-free within-fold feature selection, and comparative model evaluation with a focus on minority-class sensitivity under class-imbalanced conditions.

---

## Table of Contents

- [Background](#background)
- [Datasets](#datasets)
- [Pipeline Overview](#pipeline-overview)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Results](#results)
- [Project Structure](#project-structure)
- [Citation](#citation)

---

## Background

Glaucoma is a leading cause of irreversible blindness worldwide. Its most prevalent form, POAG, progresses silently — by the time clinical symptoms are detectable, significant and permanent neurodegeneration has already occurred. Conventional diagnostic tools (tonometry, OCT, perimetry) identify disease only after structural damage has reached a detectable threshold, making truly early intervention difficult.

Gene expression profiling captures transcriptional dysregulation in disease-relevant tissues (trabecular meshwork, optic nerve head, retina) at a molecular level that precedes clinical detectability. This project explores whether machine learning models trained on microarray-derived gene expression signatures can enable early, pre-symptomatic POAG prediction — and whether a Hybrid Quantum Classifier offers advantages over classical baselines in this small, high-dimensional data regime.

---

## Datasets

Both datasets are publicly available through the [NCBI Gene Expression Omnibus (GEO)](https://www.ncbi.nlm.nih.gov/geo/).

| Dataset | GEO Accession | Platform | Samples | POAG | Control |
|---|---|---|---|---|---|
| GSE138125 | [GSE138125](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE138125) | GPL21827 (Agilent) | 8 | 4 | 4 |
| GSE27276 | [GSE27276](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE27276) | GPL2507 (Sentrix) | 36 | ~8 | ~28 |

> **Note:** The two datasets originate from different microarray platforms with no overlapping probe identifiers. Cross-platform integration was performed by positional concatenation after variance-based probe sorting. This is a known limitation — see [Limitations](#limitations).

To download the data:
1. Visit the GEO links above
2. Download the `*_series_matrix.txt.gz` file for each dataset
3. Decompress and place in your working directory (e.g., `/content/` if using Google Colab)

---

## Pipeline Overview

```
Raw GEO Data (GSE138125 + GSE27276)
        │
        ▼
  Data Loading & Cleaning
  (tab-sep parsing, NaN removal)
        │
        ▼
  Variance-based Probe Sorting
  + Positional Merge (44 samples × 61,024 probes)
        │
        ▼
  Normalization
  (log2(x+1) → Gene-wise Z-score)
        │
        ▼
  Exploratory DEG Analysis
  (Welch's t-test + BH FDR + Volcano Plot)
        │
        ▼
  5-Fold Stratified Cross-Validation
        │
        ├──▶  Within-Fold DEG Feature Selection
        │     (adj_p < 0.05, |logFC| > 1, fallback top-50)
        │
        ├──▶  Classical Models
        │     (Logistic Regression, Random Forest, SVM)
        │
        └──▶  Hybrid VQC (PennyLane)
              (PCA → 4 qubits → StronglyEntanglingLayers
               → PauliZ expectation → Adam optimizer)
                    │
                    ▼
             Comparative Evaluation
             (Accuracy + F1-Score)
```

---

## Requirements

- Python 3.8+
- numpy
- pandas
- matplotlib
- scikit-learn
- scipy
- statsmodels
- pennylane >= 0.39

---

## Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/glaucoma-qml-classification.git
cd glaucoma-qml-classification

# Install dependencies
pip install numpy pandas matplotlib scikit-learn scipy statsmodels pennylane
```

Or if using Google Colab (recommended for PennyLane compatibility):

```python
!pip install pennylane statsmodels
```

---

## Usage

1. Download the GEO series matrix files for GSE138125 and GSE27276 and place them in your working directory.

2. Update the file paths in the notebook:
```python
df1 = pd.read_csv("/your/path/GSE138125_series_matrix.txt", sep="\t", comment="!", header=0)
df2 = pd.read_csv("/your/path/GSE27276_series_matrix.txt", sep="\t", comment="!", header=0)
```

3. Run the notebook `last_final_code.ipynb` sequentially from top to bottom. Each section is modular:
   - **Cells 1–9**: Data loading and metadata construction
   - **Cells 10–14**: Normalization and exploratory DEG + Volcano Plot
   - **Cells 15–18**: Train/test split with leakage-free DEG (exploratory)
   - **Cells 19–23**: 5-fold CV with classical models
   - **Cell 24**: Hybrid VQC training and evaluation
   - **Cell 25**: Comparative bar chart visualization

> The notebook was developed and tested on **Google Colab**. Running locally may require adjusting file paths and ensuring PennyLane is correctly installed.

---

## Results

### Classical Models — 5-Fold Stratified Cross-Validation

| Model | Mean Accuracy | Mean F1-Score |
|---|---|---|
| Logistic Regression | 1.0000 | 0.8000 |
| SVM (Linear) | 1.0000 | 0.8000 |
| Random Forest | 0.9778 | 0.7333 |

### Hybrid VQC — 3-Fold Stratified Cross-Validation

| Model | Mean Accuracy | Mean F1-Score |
|---|---|---|
| Hybrid VQC | 0.9111 | **0.8333** |

### Key Observation

The Hybrid VQC achieved a lower overall accuracy than classical models but a **higher F1-score**, indicating superior sensitivity to the minority POAG class. In a clinical context where false negatives (missed disease cases) carry greater consequence than false positives, F1-score is the more meaningful metric. The VQC's performance advantage on this metric — despite a simpler feature representation (4 PCA components, 4 qubits) — suggests potential value in quantum approaches for imbalanced biomedical classification tasks.


## Acknowledgements

- Gene expression data sourced from the [NCBI Gene Expression Omnibus (GEO)](https://www.ncbi.nlm.nih.gov/geo/)
- Quantum circuit implementation powered by [PennyLane](https://pennylane.ai/) (Xanadu)
- Classical ML components built with [scikit-learn](https://scikit-learn.org/)
- Statistical testing via [scipy](https://scipy.org/) and [statsmodels](https://www.statsmodels.org/)

---

*This project was developed as an academic research prototype. Results are not intended for clinical use.*
