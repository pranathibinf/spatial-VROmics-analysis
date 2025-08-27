# VR-Omics Demonstration: Human Lymph Node (Visium V1)
## 1. Data sources
This workflow is based on the spatial transcriptomics software VR‑Omics, which offers automated integration of multi-slice spatial transcriptomics in 2D and 3D using a user-friendly GUI.
I used the publicly available Visium Human Lymph Node V1 data, the filtered feature-barcode matrix (HDF5) to recreate this workflow.

**Paper:** Automated integration of multi-slice spatial transcriptomics data in 2D and 3D using VR‑Omics.
Genome Biology, 2025, 26(1):182. 
**DOI:** 10.1186/s13059-025-03630-6 

**Download link:**
https://cf.10xgenomics.com/samples/spatial-exp/1.1.0/V1_Human_Lymph_Node/V1_Human_Lymph_Node_filtered_feature_bc_matrix.h5

### Repository Layout
```bash
spatial-VROmics-analysis/
├── README.md
└── spatial-VROmics-analysis/
    ├── data/
    │   └── V1_Human_Lymph_Node_filtered_feature_bc_matrix.h5
    ├── metadata.yaml
    ├── questions.yaml
    ├── answers.yaml
    └── workflow/
        ├── 01_preprocess.ipynb   # Scanpy-based analysis workflow
        └── outputs/              # Generated results and plots
            ├── lymph_node_preprocessed.h5ad
            ├── lymph_node_preprocessed.h5Seurat  
            ├── clusters.csv
            ├── cluster_counts.png
            └── figures/
                └── umap_leiden.png
```
## 2. Installation & Execution
**Step 1:** Open the notebook
Open workflow/workflow.ipynb in Google Colab.

**Step 2:** Mount Google Drive
```python
from google.colab import drive
drive.mount('/content/drive')
```
**Step 3: ** Install Required Python Packages
```python
!pip install --upgrade pip
!pip install "scanpy[leiden]" seaborn matplotlib anndata h5py louvain
```
**Step 4:** Download the Dataset
```python
!wget -O /content/drive/MyDrive/spatial-VROmics-analysis/sample_submission/data/V1_Human_Lymph_Node_filtered_feature_bc_matrix.h5 \
  https://cf.10xgenomics.com/samples/spatial-exp/1.1.0/V1_Human_Lymph_Node/V1_Human_Lymph_Node_filtered_feature_bc_matrix.h5
```
### You can execute the workflow notebook in three ways:
**Google Colab (recommended):** Open workflow.ipynb in Google Colab. Click Runtime → Run all to execute all cells
**Local Jupyter Notebook:** 
```bash
git clone https://github.com/pranathibinf/spatial-VROmics-analysis.git
cd spatial-VROmics-analysis/workflow
jupyter notebook workflow.ipynb
```
Then open the notebook in your browser and select Run All

**Command-line (non-interactive):** 
```bash
jupyter nbconvert --to notebook --execute workflow.ipynb --output=workflow_executed.ipynb
```
**Step 5:** Run Preprocessing workflow
The notebook performs the following:
- Loads the .h5 file using Scanpy.
- Conducts QC → normalization → HVG selection.
- Performs PCA → UMAP → Leiden clustering.
Saves outputs to the outputs/ folder (counts, plots, AnnData).

**Step 6:** Convert .h5ad to Seurat via R (requires only if working on colab)
Enable R support and install SeuratDisk:
```python
%load_ext rpy2.ipython
```
```R
%%R
if (!require("remotes")) install.packages("remotes")
remotes::install_github("mojaveazure/seurat-disk")
library(SeuratDisk)
Convert("workflow/outputs/lymph_node_preprocessed.h5ad",
        dest = "h5seurat", overwrite = TRUE)
```
## 3. Primary Outputs

After running, you should have:

- lymph_node_preprocessed.h5ad

- lymph_node_preprocessed.h5Seurat (if working on colab)

- clusters.csv

- cluster_counts.png

- figures/umap_leiden.png

## Notes:
- The input .h5ad dataset from the 10x Genomics Human Lymph Node V1 demo is already <200 MB (wrt Taiga submiddiond) , so no extra subsampling is needed. 
- All analysis is contained in workflow/workflow.ipynb (Colab-ready). 

You can either run it interactively (Runtime → Run all) or execute headlessly with:
```bash
jupyter nbconvert --to notebook --execute workflow.ipynb --output=workflow_executed.ipynb
```

Results (.h5ad, .csv, .png, optional .h5Seurat) are written to:
```bash
spatial-VROmics-analysis/workflow/outputs/
```

- Seurat conversion: The SeuratDisk tool is only available in R. In Colab, enable the R kernel/magic and install SeuratDisk before running the conversion cell. This step is optional if you only want to use the .h5ad file.
- Total file sizes remain under 1 GB (27 MB input .h5 + processed outputs), which fits Taiga’s upload limits.
