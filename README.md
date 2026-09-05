# Familial Alzheimer's Disease iPSC Differential Expression Analysis

Re-analyzing gene expression in Presenilin-2 (PSEN2) N141-mutant iPSCs to look for early molecular changes linked to familial Alzheimer's disease, and to figure out how much of that signal a 4-sample public dataset can actually support.

## Background
Familial Alzheimer's Disease (FAD) is a rare, early-onset, inherited form of Alzheimer's, often caused by mutations in the *PSEN2* gene. Because you can't ethically biopsy a living patient's brain, researchers reprogram skin cells from FAD
patients into induced pluripotent stem cells (iPSCs) cells that behave like an early, moldable version of any cell type in the body, as a stand-in model. This project re-analyzes a public dataset of iPSCs carrying the PSEN2 N141 mutation 
(Yagi et al., 2011) to see which genes differ from a healthy control line, and just as importantly, to be honest about what a dataset this small can and can't prove.

## Data
- Source: [GDS4141 / GSE28379](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE28379) (NCBI Gene Expression Omnibus), Affymetrix GPL570 array
- Size: 4 arrays total, ~54,675 probes each; `201B7 iPSC` (control), `PD01-25 iPSC` (Sporadic Parkinson's disease patient, included in the original series for comparison), `PS2-1 iPSC` and `PS2-2 iPSC` (the two PSEN2 N141-mutant clones)
- Known issue this repo corrects: an earlier version of this analysis renamed `GSM701543` (`PD01-25 iPSC`, a Parkinson's disease sample) to `control2` and averaged it into the control group. The dataset's own sample metadata clearly labels
it `"disease state: Sporadic Parkinson's disease"` it isn't a control. That mistake made the comparison silently confounded by an unrelated disease. This repo keeps that sample under its own label (`sporadic_pd`) instead of folding it into
the control group.

## Method
1. Load the GEO series matrix and confirm sample identities directly against the file's own `!Sample_title` / `!Sample_characteristics_ch1` metadata (not by assumed column order)
2. Log2-transform expression values to stabilize variance
3. Correlation heatmap and PCA across all four samples, correctly labeled as three groups (control, sporadic PD, AD-mutant)
4. Log2 fold-change between the control sample and the mean of the two AD-mutant samples, ranked to surface candidate genes
5. No t-test / FDR correction is run on the control-vs-mutant comparison with only one true control array, there's no way to estimate control-group variance, so a "statistically significant" result would be manufactured, not real. This is
flagged directly in the notebook instead of glossed over.

## Key Result
With the mislabeling fixed, this dataset supports a **descriptive candidate-gene ranking**, not a statistically validated DEG list, 30% of all probes clear a loose |log2FC| > 1 cutoff with no statistical filter, which is far too large a 
fraction to trust as a real hit list from a single control array. Reassuringly, this doesn't overturn the original headline finding: reproducing the earlier (mislabeled) pipeline still gives zero genes significant after Benjamini-Hochberg 
correction (minimum adjusted p = 0.916). The original conclusion survives, but the write-up's stated "n=2 vs n=2" comparison was never actually the one that ran, and this version fixes that.

## How to Run
```bash
git clone [https://github.com/cod3astro/Alzheimers-IPSC-DEG-Analysis]
pip install -r requirements.txt
jupyter notebook notebooks/GSE28379_corrected.ipynb
```

## Repo Structure
```
README
requirements
data/       raw GEO series matrix file
notebooks/  the corrected, executed analysis notebook
results/    figures and output tables
```

Related Writing

Original write-up on Medium: [Computational Analysis of Differentially Expressed Genes in Familial Alzheimer's Disease iPSCs with Presenilin 2 Mutation](https://medium.com/@cod3astro/computational-analysis-of-differentially-expressed-genes-in-familial-alzheimers-disease-ipscs-be7c94a80f5a?sharedUserId=cod3astro)

Notes

This is a personal/independent re-analysis project, not a lab-supervised study. The published Medium article's control-group labeling has since been corrected here, GSM701543 (Sporadic Parkinson's disease iPSC) is no longer treated as a second control replicate. With more time and access to additional public control iPSC lines, the natural next step is a properly powered t-test with real biological replicates on both sides, plus full probe-to-gene-symbol mapping and a GSEA/WGCNA pass, both of which the original write-up's Discussion section already flagged as better fits for small datasets like this one.
