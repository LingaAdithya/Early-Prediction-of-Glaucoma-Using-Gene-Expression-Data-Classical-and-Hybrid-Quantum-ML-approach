Overview
This project presents an end-to-end hybrid quantum-classical machine learning pipeline for the early prediction of Primary Open-Angle Glaucoma (POAG) from microarray gene expression data. The pipeline integrates classical statistical genomics methods — differential gene expression analysis and supervised classification — with a Variational Quantum Circuit (VQC) implemented using PennyLane, benchmarking quantum against classical approaches on publicly available GEO transcriptomic datasets.
The work addresses a clinically significant diagnostic gap: conventional glaucoma detection relies on structural and functional changes that are observable only after irreversible neurodegeneration has occurred. By operating at the transcriptomic level, this pipeline aims to identify molecular signatures that precede clinical symptom onset, contributing toward the broader goal of early, molecularly-informed glaucoma prediction.

Table of Contents

Project Structure
Datasets
Pipeline Overview
Methods Summary
Results
Dependencies
How to Run
Limitations
Future Work
Citation


Project Structure
├── last_final_code.ipynb        # Main analysis notebook
├── README.md                    # Project documentation

Note: The dataset files (GSE138125_series_matrix.txt and GSE27276_series_matrix.txt) are not included in this repository due to size constraints. Download instructions are provided in the Datasets section below.


Datasets
Two publicly available microarray gene expression datasets were sourced from the NCBI Gene Expression Omnibus (GEO):
DatasetGEO AccessionPlatformSamplesPOAGControlGSE138125GSE138125GPL21827 (Agilent)844GSE27276GSE27276GPL2507 (Sentrix)36~8~28
Download Instructions

Visit each GEO accession link above
Under the Download section, download the Series Matrix File (_series_matrix.txt.gz)
Extract the .gz file
Place both .txt files in the /content/ directory if running on Google Colab, or update the file paths in the notebook accordingly


Pipeline Overview
Raw GEO Data (GSE138125 + GSE27276)
            │
            ▼
    Data Loading & Label Assignment
            │
            ▼
    Numeric Cleaning & NaN Removal
            │
            ▼
    Variance Sorting & Positional Merge
            │
            ▼
    Log2(x+1) + Z-score Normalization
            │
            ▼
    Differential Gene Expression (DEG)
    Welch's t-test + BH FDR Correction
            │
            ▼
    Volcano Plot Visualization
            │
            ├─────────────────────────────────┐
            ▼                                 ▼
  Classical ML (5-fold CV)         Hybrid VQC (3-fold CV)
  ┌─────────────────────┐         ┌──────────────────────┐
  │ Logistic Regression │         │ PCA → 4 components   │
  │ Random Forest       │         │ AngleEmbedding (RX)  │
  │ SVM (Linear)        │         │ StronglyEntangling   │
  └─────────────────────┘         │ Layers (3 layers)    │
            │                     │ Adam Optimizer       │
            │                     └──────────────────────┘
            │                                 │
            └─────────────┬───────────────────┘
                          ▼
              Comparative Performance Report
              (Accuracy + F1-Score)

Methods Summary
Preprocessing

Tab-separated loading with GEO metadata comment lines skipped
Disease labels assigned manually for GSE138125 and programmatically inferred from column names for GSE27276
Non-numeric values coerced to NaN; all-NaN rows dropped
Zero common probe IDs found across platforms; datasets merged positionally after variance sorting and index reset
Combined matrix: 61,024 probes × 44 samples

Normalization

Log2(x + 1) transformation applied to stabilize variance
Gene-wise Z-score normalization (mean-centering and standard deviation scaling per probe row)

Differential Expression Analysis

Welch's two-sample t-test (equal_var=False) per probe
logFC = mean(POAG) − mean(Control)
Benjamini-Hochberg FDR correction for multiple comparisons
Volcano plot generated with adj_p = 0.05 significance threshold

Feature Selection

Performed strictly within each cross-validation fold on training data only (leakage-free)
Selection criteria: adj_p < 0.05 AND |logFC| > 1
Fallback: top 50 probes by adj_p when fewer than 10 pass the threshold

Classical Models
ModelConfigurationLogistic Regressionmax_iter=1000, sklearn defaultsRandom Forest100 estimators, random_state=42SVMLinear kernel, sklearn defaults
Evaluated under 5-fold Stratified K-Fold Cross-Validation. Metrics: Accuracy and F1-score.
Hybrid Variational Quantum Classifier

Framework: PennyLane (default.qubit simulator)
Dimensionality reduction: PCA to 4 components
Feature scaling: Min-max normalization to [0, π]
Circuit: 4 qubits, RX AngleEmbedding + StronglyEntanglingLayers (3 layers)
Trainable parameters: 36 (weight shape: 3 × 4 × 3) + 1 bias
Optimizer: Adam (stepsize=0.1, 30 iterations)
Cost function: Mean squared error against {−1, +1} mapped labels
Evaluated under 3-fold Stratified K-Fold Cross-Validation


Results
Classical Classifiers (5-fold CV)
ModelMean AccuracyMean F1-ScoreLogistic Regression1.00000.8000SVM (Linear)1.00000.8000Random Forest0.97780.7333
Hybrid VQC vs. Best Classical Baseline (Random Forest, 3-fold CV)
ModelMean AccuracyMean F1-ScoreRandom Forest0.97780.7333Hybrid VQC0.91110.8333
The Hybrid VQC achieved a higher F1-score than all classical baselines, indicating superior sensitivity to the minority POAG class — the clinically critical outcome in disease prediction tasks. Classical models achieved higher raw accuracy, consistent with majority-class prediction bias in imbalanced datasets.

Important: These results should be interpreted with caution. The cross-platform positional merge and small sample size introduce significant methodological constraints that qualify the biological interpretability of reported performance. See Limitations.


Dependencies
Core Libraries
numpy
pandas
matplotlib
scipy
statsmodels
scikit-learn
pennylane
Install via pip
bashpip install numpy pandas matplotlib scipy statsmodels scikit-learn pennylane
Environment
This notebook was developed and executed on Google Colab. It is recommended to run it there to avoid path and dependency configuration issues. If running locally, update all file paths from /content/ to your local directory.

How to Run

Clone this repository

bashgit clone https://github.com/your-username/your-repo-name.git
cd your-repo-name

Download the datasets from GEO as described in the Datasets section and place them in the appropriate directory
Open the notebook in Google Colab or Jupyter
Update file paths in the data loading cells if running locally:

python# Change this:
df1 = pd.read_csv("/content/GSE138125_series_matrix.txt", ...)
df2 = pd.read_csv("/content/GSE27276_series_matrix.txt", ...)

# To this (example for local):
df1 = pd.read_csv("./data/GSE138125_series_matrix.txt", ...)
df2 = pd.read_csv("./data/GSE27276_series_matrix.txt", ...)

Run all cells sequentially. The notebook is organized in the following order:

Data loading and label assignment
Cleaning and merging
Normalization
DEG analysis and volcano plot
Classical ML cross-validation
Hybrid VQC training and evaluation
Comparative visualization

