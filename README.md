# A Machine Learning Approach for Identifying Gene Markers in Colon Cancer

Bachelor thesis, German University in Cairo (GUC), Media Engineering and Technology. Grade: **A+**.

An end-to-end pipeline that takes raw TCGA-COAD RNA-seq counts for **517 samples**, finds the genes that are differentially expressed between colon tumor and normal tissue, checks which biological processes they belong to, and trains classifiers that tell tumor from normal tissue.

▶ **[Open the notebook](colon_mRNA-seq.ipynb)**

## Pipeline

| Step | What happens | Tools |
|---|---|---|
| 1. Data loading | 517 STAR gene-count files from the GDC portal are merged into one protein-coding expression matrix, duplicate gene symbols are averaged, and each sample is labelled tumor or normal from the GDC sample sheet. | R, readr |
| 2. Exploration and filtering | Box plots, histograms and PCA of log-transformed counts; the 10% least variable genes are removed. | base R |
| 3. Differential expression | Tumor vs. normal with DESeq2. DEGs: adjusted p < 0.05 and \|log2 fold change\| > 1. | DESeq2 |
| 4. Visualisation | Heatmap of the DEGs, 2D PCA, volcano plot. | ComplexHeatmap, EnhancedVolcano |
| 5. Enrichment | The DEG list is run through DAVID; significant GO terms (Benjamini < 0.05) are plotted. | DAVID, GOplot |
| 6. Machine learning | LASSO, SVM (radial kernel) and Random Forest (500 trees), each trained with and without ROSE class balancing. | caret, glmnet, e1071, randomForest, pROC, ROSE |

## Fixing a data leakage problem

The first version of the classifier used genes selected with **all** samples and only then split the data into train and test sets, so the test samples had already influenced which genes were chosen. The final pipeline:

1. splits the 507 labelled samples 80/20 (stratified, seed 42) **before** any feature selection,
2. picks the top 200 genes using the training set only,
3. balances the training set with ROSE (the data is 466 tumor vs. 41 normal),
4. evaluates on the untouched test set (94 tumor, 9 normal).

## Results (held-out test set, 103 samples)

| Model | Accuracy | AUC | Sensitivity | F1 |
|---|---|---|---|---|
| LASSO | 0.990 | 1.00 | 1.000 | 0.995 |
| SVM | 1.000 | 1.00 | 1.000 | 1.000 |
| Random Forest | 0.990 | 1.00 | 1.000 | 0.995 |
| LASSO + ROSE | 1.000 | 1.00 | 1.000 | 1.000 |
| SVM + ROSE | 1.000 | 1.00 | 1.000 | 1.000 |
| Random Forest + ROSE | 0.990 | 1.00 | 0.989 | 0.995 |

Tumor and normal colon tissue have very different expression profiles, so all three models separate them almost perfectly. The test set contains only 9 normal samples, so one misclassification moves accuracy by about one percentage point.

**Top genes across all three models** (normalised LASSO, SVM and Random Forest importance, averaged): CA2, CA1, ITM2C, GSN, TXNIP, LGALS4, JCHAIN, CEACAM7, KRT20, SLC26A3.

## Running it

1. Download the TCGA-COAD gene-expression files and the sample sheet from the [GDC Data Portal](https://portal.gdc.cancer.gov/).
2. In your Google Drive, create a `colon/` folder containing the zipped data (`colon data set.zip`), the sample sheet and the DAVID export (`DAVIDChart.csv`).
3. Open `colon_mRNA-seq.ipynb` in Google Colab and run all cells. The R packages install on the first run, which takes 20–30 minutes.

The raw TCGA data is not included in this repository.
