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
2. **Spatial DB**
   - collection of spatial transcriptomics datasets
   - contains pancreatic cancer, melanoma, and lung cancer samples
3. **Human Tumor Atlas Network (HTAN)**
   - spatial transcriptomics for various tumors
   - includes metadata about tumor stages and patient survival
   - https://humantumoratlas.org/

**10X Processed Dataset Explained:**

| File Name                | Type    | Description                                     | How to Use                        |
| ------------------------ | ------- | ----------------------------------------------- | --------------------------------- |
| Summary HTML             | .html   | Interactive summary of dataset                  | Open in a browser for metadata    |
| Summary CSV              | .csv    | Metadata & quality metrics                      | Load into Pandas for QC checks    |
| Loupe Browser File       | .cloupe | Interactive data file for 10x Loupe Browser     | Open in Loupe Browser             |
| Genome-Aligned BAM       | .bam    | Raw read alignments (mapped to genome)          | Use for custom RNA-seq processing |
| BAM Index                | .bai    | Index file for BAM                              | Needed for BAM processing         |
| Sample Barcodes          | .csv    | Cell barcodes identifying unique cells          | Used to map cells                 |
| Feature/Cell Matrix HDF5 | .h5     | Gene expression matrix                          | Main file for analysis in Python! |
| Feature/Cell Matrix (GZ) | .gz     | Alternative gene expression format (compressed) | Use scanpy to read                |
| Per-Molecule Read Info   | .h5     | Raw molecule-level counts                       | Rarely used directly              |
| Feature Reference        | .csv    | List of genes measured                          | Links gene IDs to names           |
| Clustering Analysis      | .gz     | Precomputed clusters                            | Can be used for visualization     |

**10X Raw vs Processed Dataset**

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









