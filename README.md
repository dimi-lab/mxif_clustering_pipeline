# Nextflow Pipeline for QC and Clustering of MxIF Datasets

Pipeline visualized below

<img src="https://github.com/dimi-lab/mxif_clustering_pipeline/blob/main/images/mxif-pipeline.png" width="1000"/>

## Requirements/Dependencies

-   Nextflow 23.04.2
-   pandoc 3.1.2
-   R 4.2.2
-   R Packages
    -   knitr 1.44
    -   ggplot2 3.4.4
    -   data.table 1.14.8
    -   dplyr 1.1.3
    -   Seurat \>5.0
    -   progressr 0.14.0
    -   kableExtra 1.3.4
    -   ComplexHeatmap 2.15.4
    -   ggridges 0.5.4
    -   clustree 0.5.0
    -   pheatmap 1.0.12
    -   plyr 1.8.9
    -   pander 0.6.5

------------------------------------------------------------------------

## Instructions

Note: This pipeline requires exported QuPath (0.4.3) measurement tables (quantification files) generated from segmented single cell MxIF images.

1.  Clone repository to your machine
2.  Place quantification files in `data` directory
    i.  Files should be in the format `<fov_name>.tsv` or `<fov_name>.csv` (e.g. `region_001.tsv`)
3.  Adjust configuration values in `configs.csv`
4.  Add desired markers to `marker_confings.csv` - all markers provided must be present in the dataset
    i.  If markers are not provided or if all markers provided are not present in the dataset, the quantification file will search for a default list of markers; if all default markers are not present, all markers in the dataset will be used, excluding DAPI.
5.  Call main pipeline script: `nextflow run main.nf`
    i.  By default, all three steps will run (QC, clustering, metaclustering). It is a good idea to first run the QC by itself by using the `--qc_only` flag. If you want to run both the QC and clustering steps, use `--qc_and_cluster`.

------------------------------------------------------------------------

### Configurable parameters

| object               | value                                                                                                                                                     |
|-----------------|-------------------------------------------------------|
| sigsum_quantile_high | Upper quantile cutoff for sigsum filtering (default 0.99)                                                                                                 |
| sigsum_quantile_low  | Lower quantile cutoff for sigsum filtering (default 0.05)                                                                                                 |
| bin_size             | Size of bounding box for low-density cell search (default 50)                                                                                             |
| density_cutoff       | Cutoff number of cells defined as low-density (default 5)                                                                                                 |
| cluster_metric       | Metric to use for Seurat clustering (default Median)                                                                                                      |
| clustering_res       | If specified, this clustering resolution will be used for all ROIs and will override the clustree method. Set to NA or remove row to use clustree method. |
| min_res              | Minimum clustering resolution to search with clustree (default 0.1)                                                                                       |
| max_res              | Maximum clustering resolution to search with clustree (default 1.9)                                                                                       |
| res_step             | Increment for searching clustering resolutions; functions as `by` argument in `seq()` (default 0.2)                                                       |
| min_clusters         | Minimum number of clusters for per-ROI clustering in Seurat (default 6)                                                                                   |
| min_metaclusters     | Starting number of metaclusters to create (default 5)                                                                                                     |
| max_metaclusters     | Ending number of metaclusters to create (default 10)                                                                                                      |

------------------------------------------------------------------------

## Analysis steps and outputs

**QC.Rmd**

-   `output_reports/bin_density_report.html` and `output_reports/sigsum_report.html`
    -   Reports contain one QC image for each ROI
    -   Plots for sigsum cutoffs and bin density flags
-   `output_tables/all_markers_clean_<roi_name>.csv`
    -   Quantification file in the same format as input files but with additional columns for QC metrics and QC flags

**clustering.Rmd**

-   `output_reports/clustering_report_<roi_name>.html`
    -   Clustree plot, selected resolution, marker vs cluster heatmaps and ridgeplots
-   `output_tables/clusters_<roi_name>.html`
    -   Clusters mapped to cell coordinates - includes artifacts

**metaclustering.Rmd**

-   `output_reports/metaclustering_report.html`
    -   Marker vs metacluster heatmaps; barplots for proportion of ROI per metacluster
-   `output_tables/<roi_name>_mapped_metaclusters_n_metaclusters.csv`
    -   Clusters and metaclusters mapped to cell coordinates - includes artifacts

------------------------------------------------------------------------

## Example Data

Within the data directory of this repository, there is a sample dataset with four quantification files generated using images from the [Multiplexed Imaging Mass Cytometry of Chemokine Milieus in Metastatic Melanoma](https://zenodo.org/records/6004986) dataset (Hoch et al. 2022). The templates for configs.csv and marker_configs.csv have been set up for this dataset, therefore you can run a test of the pipeline by cloning the repo, unzipping the files into the data directory, and running `nextflow run main.nf` from the top-level directory.

Hoch, T., Schulz, D., Eling, N., Martínez-Gómez, J., Levesque, M., & Bodenmiller, B. (2022). Multiplexed Imaging Mass Cytometry of Chemokine Milieus in Metastatic Melanoma - Raw Data. <https://doi.org/10.5281/zenodo.6004986>
