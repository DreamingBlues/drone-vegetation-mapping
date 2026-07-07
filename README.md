# Drone Vegetation Mapping

Hyperspectral remote-sensing workflow for land-cover classification using airborne HySpex imagery and hand-labeled masks. The project compares full-spectrum hyperspectral features, PCA-reduced features, selected wavelength bands, K-Means clustering, and Random Forest classification.

## Repository Structure

```text
notebooks/
  01_eda.ipynb
  02_convert_zarr.ipynb
  03_stratified_sampling.ipynb
  04_pca.ipynb
  05_clustering.ipynb
  06_supervised_ml.ipynb
labels/
  masks/
reports/
  figures/
tools/
  mask_labeler.html
docs/
  DATA.md
  project_proposal.pdf
```

## Data

Raw hyperspectral and RGB data are not included in this repository because of size and redistribution constraints. Hand-labeled PNG masks are included under `labels/masks` as project annotations. See `docs/DATA.md` for dataset citation and access notes.

## Workflow

1. Explore ENVI hyperspectral data and metadata.
2. Convert the SA1 hyperspectral mosaic to Zarr for chunked processing.
3. Build ground-truth labels from hand-labeled water, soil, canopy, and grass masks.
4. Analyze class-level spectral signatures and PCA/SVD components.
5. Compare K-Means clustering across full bands, selected bands, and PCA features.
6. Train and evaluate Random Forest classifiers.

## Citation

Jarocinska, A., Kopec, D., Niedzielko, J. et al. The utility of airborne hyperspectral and satellite multispectral images in identifying Natura 2000 non-forest habitats for conservation purposes. Scientific Reports 13, 4549 (2023). https://doi.org/10.1038/s41598-023-31705-6
