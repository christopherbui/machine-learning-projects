# Spatial Transcriptomics for Microenvironment Analysis

This research area is quite new, and it aims to track gene expression of tumor cells while preserving the spatial organization of cells in a tissue sample. Spatial interactions in the tumor microenvironment (TME) is analyzed using graph-based deep learning and spatial omics data.

The main goal is to identify cancer-immune cell interactions that drive tumor progression, allowing healthcare providers to better predict patient outcomes.

## Datasets

Spatial Transcriptomics (ST) datasets include the following type of data:

- **gene expression matrices** (cell-by-gene expression)
- **spatial coordinates** (x, y positions of each cell)
- **cell-type annotations** (i.e. immune cells, fibroblasts, tumor cells)
- **histopathology images** (for image analysis)

**Publicly Available Datasets**

1. **10X Genomics Visium Data**
   - contains spatially resolved gene expression from human cancer samples
   - example dataset: **breast cancer spatial transcriptomics**
   - https://www.10xgenomics.com/datasets/20k-mixture-of-nsclc-dtcs-from-7-donors-3-v3-1-with-intronic-reads-3-1-standard
   - **This dataset has spatial coordinates data:** https://www.10xgenomics.com/datasets/human-lung-cancer-11-mm-capture-area-ffpe-2-standard
2. **Spatial DB**
   - collection of spatial transcriptomics datasets
   - contains pancreatic cancer, melanoma, and lung cancer samples
3. **Human Tumor Atlas Network (HTAN)**
   - spatial transcriptomics for various tumors
   - includes metadata about tumor stages and patient survival
   - https://humantumoratlas.org/

**10X Dataset Explained:**

| Output files                             | File type | Size    | Description                                                 | How to Use                                        |
| ---------------------------------------- | --------- | ------- | ----------------------------------------------------------- | ------------------------------------------------- |
| Genome-aligned BAM                       | BAM       | 44.3 GB | Raw read alignments mapped to the genome                    | Use for custom RNA-seq processing                 |
| Genome-aligned BAM index                 | BAI       | 3.37 MB | Index file for BAM                                          | Needed for BAM processing                         |
| Per-molecule read information            | H5        | 2 GB    | Raw molecule-level read information                         | Rarely used directly                              |
| Feature / barcode matrix HDF5 (filtered) | H5        | 63.8 MB | Filtered feature and barcode expression matrix              | Load into Python for analysis                     |
| Feature / barcode matrix (filtered)      | GZ        | 215 MB  | Filtered feature and barcode expression matrix (compressed) | Use scanpy or other tools to load in Python       |
| Feature / barcode matrix HDF5 (raw)      | H5        | 92.6 MB | Raw feature and barcode expression matrix                   | **Main file for python analysis**                 |
| Feature / barcode matrix (raw)           | GZ        | 261 MB  | Raw feature and barcode expression matrix (compressed)      | Use scanpy or other tools to load in Python       |
| Clustering analysis                      | GZ        | 32.3 MB | Precomputed clustering results                              | Can be used for visualization                     |
| Spatial imaging data                     | GZ        | 34.4 MB | Data related to spatial imaging of the sample               | Open in appropriate software for spatial analysis |
| Summary CSV                              | CSV       | 835 B   | Summary of metadata and quality metrics                     | Load into Pandas for QC checks                    |
| Summary HTML                             | HTML      | 7.12 MB | Interactive HTML summary of the dataset                     | Open in a browser for metadata                    |
| Loupe Browser file                       | CLOUPE    | 2.14 GB | Interactive data file for 10x Loupe Browser                 | Open in Loupe Browser                             |

**10X Raw vs Processed Dataset:**

| Dataset                                                      | Pros                                                         | Cons                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Processed Dataset (Feature / Cell Matrix HDF5 per-sample)** | ✅ Already normalized and filtered. <br> ✅ Easier to use for single-cell analysis. <br> ✅ Faster to load in Python (H5 format). | ❌ Does not include spatial coordinates. <br> ❌ Precomputed clustering may bias further analysis. <br> ❌ May have pre-filtered genes/cells, losing important information. |
| **Raw Dataset (Feature / Cell Matrix HDF5 raw)**             | ✅ Contains all genes and cells (before filtering). <br> ✅ Ensures full control over normalization and QC. <br> ✅ Best for custom spatial graph construction. <br> ✅ Usually contains spatial coordinates. | ❌ Larger file sizes. <br> ❌ Requires more preprocessing (normalization, filtering, etc.). <br> ❌ Slightly more complex to work with. |

## Methods

### Objectives

1. **Spatial Gene Expression Mapping:** Preprocess ST data to visualize spatial distribution of gene expression in tumors.
   - normalize & log-transform gene expressin matrices
   - dimensionality reduction (PCA, t-SNE, or UMAP) to visualize cell clusters
   - spatial heatmaps of marker genes for key cell types (CD8+ T cells, cancer cells)
2. **Model Tumor-Immune Cell Interaction Network:** Construct graph representation of spatial interactions between tumor and immune cells.
   - define graph nodes: each cell is a node
   - define edges: connect nodes based on spatial proximity (distance-based threshold)
   - compute cell-cell interactions using gene co-expression and receptor-ligand signaling
3. **Predict Tumor Progression using GNNs:** Use graph neural networks to predict tumor aggression based on cell-cell interactions.
   - train **graph convolutional network (GCN)** or **graph attention network (GAT)**
   - use node features from gene expression profiles
   - perform node classification: tumor vs non-tumor regions
   - predict patient survival or tumor malignancy score

### Data Preprocessing

1. **Data Loading & Cleaning**
   - get data from 10X Genomics
   - convert raw gene expression counts into log-normalized values
   - filter out low quality spots (cells with few genes detected)
2. **Cell Type Annotation**
   - use **marker genes** to identify major cell types (tumor, immune, fibroblasts)
   - run cluster algorithms (i.e. Leiden clustering in Scanpy)
3. **Spatial Graph Construction**
   - create graph (i.e. KNN) where nodes are individual cells, edges based on proximity
   - compute edge weights using correlation of gene expression between cells
4. **Feature Engineering**
   - extract gene expression profiles per cell
   - compute **spatial autocorrelation** (Moran's I) to identify tumor hotspots
   - encode cell-cell interaction features

### Model Selection

**Graph Convolutional Networks (GCN)**

- Takes spatially structured gene expression data and propagates information across connected nodes.
- **Architecture:**
  - Input: Graph representation of single-cell spatial data.
  - Hidden Layers: **Graph convolution layers** (GCNConv) that learn spatial gene relationships.
  - Output: Classification (i.e. predicting tumor regions).
- **Implementation:** PyTorch Geometric (`torch_geometric`).

**Graph Attention Networks (GAT)**

- Uses **attention mechanisms** to weigh important cell-cell interactions.
- **Advantage:** Captures complex spatial relationships better than GCNs
- **Implementation:** PyTorch Geometric

### Evaluation Metrics

- **Clustering Accuracy:** ARI (adjusted rand index) for spatial clustering.
- **Graph Metrics:** Edge importance (graph centrality, betweeness).
- **Classification Metrics:** ROC-AUC, F1-score for predicting tumor severity.
- **Survival Analysis:** Kaplan-Meir curves if clinical metadata is available.

### Expected Outcomes

- spatial maps of tumor microenvironments, highlighting key gene expression hotspots.
- graph-based models that predict tumor aggressiveness based on cell-cell interactions.
- potential biomarkers for cancer progression identified from GNN analysis



## Notes

**Unique Molecular Identifiers (UMI Count):** Represents the absolute number of observed transcripts captured and counted from an individual cell. The count is an indicator of RNA expression levels. Higher UMI count means stronger gene expression.

In gene expression data file (h5), rows are individual cells are uniquely identified by a distinct DNA sequence.

- Each cell is captured in a separate microchamber and tagged with a distinct DNA sequence called a **cell barcode**.
- DNA fragment is added to each cell's RNA via reverse transcription. Allows for sequencing platform to track which cell an RNA transcript came from.
- The goal of sc-RNAseq is to analyzes gene expression on single cell level. Millions of cells are sequenced and tagged with DNA barcode during library preparation.
- h5 files contain large-scale biological data, including gene expression matrix (X) and cell barcodes (obs).

Tissue spots indicate regions within the tissue, and usually is colored to indicate gene expression levels. Many cells are located in a tissue spot.

**Violin Plots** depict the distribution of certain values per cell via **kernel density estimation (KDE)**. A histogram depicts the discrete distribution of value counts, but violin plots transform the discrete counts into a continuous density distribution. A box and whisker plot might be overlayed on top of the violin plot for additional information of the data.







