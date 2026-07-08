# Hyperspectral Vegetation Classification

This project analyzes hyperspectral vegetation imagery and tests whether a small set of selected wavelength bands can preserve most of the useful classification information from a full 430-band hyperspectral cube.

The main goal is to reduce hyperspectral data complexity while maintaining strong land-cover classification performance.

## Project Overview

Hyperspectral imaging captures hundreds of narrow wavelength bands across the visible, near-infrared, and short-wave infrared spectrum. This provides detailed spectral information, but it also creates large datasets with high redundancy.

This project investigates:

- how redundant the 430-band hyperspectral dataset is
- how many principal components are needed to represent most variance
- which wavelengths best separate land-cover classes
- how full bands, PCA features, and selected wavelengths compare in classification

## Citation

Original dataset/study:

Sławik, Ł., Niedzielko, J., Kania, A., Piórkowski, H., & Kopeć, D. (2019). *Multiple Flights or Single Flight Instrument Fusion of Hyperspectral and ALS Data? A Comparison of their Performance for Vegetation Mapping*. Remote Sensing, 11(8), 970. https://doi.org/10.3390/rs11080970

## Dataset

The dataset is aerial hyperspectral vegetation imagery from the SA1 region.

| Property | Value |
|---|---:|
| Image size | 6896 x 5496 pixels |
| Spectral bands | 430 |
| Wavelength range | 0.416-2.396 micrometers |
| Spatial resolution | 1 m x 1 m |
| Sensor type | HySpex VNIR + SWIR |
| Classes | Water, Soil, Canopy, Grass |


RGB reference image:

<img width="274" height="344" alt="image" src="https://github.com/user-attachments/assets/c75a3575-b11c-43b0-aca1-8b445f6d8fde" />


## Workflow

### 1. Data Preparation

The original hyperspectral cube was converted to Zarr format for chunked processing. This made it possible to work with the large image without loading the full dataset into memory.

Manual masks were created for the four land-cover classes and used to generate labeled samples. A balanced stratified sample of 200,000 pixels was used: 50,000 pixels per class.


### 2. Exploratory Data Analysis

The EDA step examined mean spectral reflectance, per-class spectral profiles, visible/NIR/SWIR behavior, and pairwise band correlation. The correlation analysis showed strong redundancy between nearby wavelength bands.


### 3. PCA and Dimensionality Reduction

Principal Component Analysis was applied using Singular Value Decomposition. The PCA results showed that most variance is captured by only a few components:

| Variance Retained | Number of PCs |
|---|---:|
| 90% | 2 |
| 95% | 3 |
| 99% | 5 |

This confirmed that the 430-band dataset has a much lower intrinsic dimensional structure.

### 4. Wavelength Selection

Class mean reflectance curves were compared using a pairwise distance score. Peaks in the distance curve were used to identify informative wavelengths.

The selected 7 bands were:

```python
SELECTED_BANDS = [31, 85, 143, 184, 226, 296, 364]
```

These bands form a compact multispectral-like feature set.

### 5. Model Comparison

Three feature sets were compared:

| Feature Set | Description | Features |
|---|---|---:|
| Full 430-Band | Original hyperspectral spectra | 430 |
| Top 5 PCs | PCA-reduced representation | 5 |
| Selected 7-Band | Selected wavelength subset | 7 |

K-Means clustering was used as an unsupervised comparison step. Random Forest was then used for supervised classification with an 80/20 train-test split from the stratified sample.

Random Forest performance was evaluated using normalized confusion matrices, per-class accuracy, F1 score, and Intersection over Union.

## Full-Image Prediction Maps

After training, each Random Forest model was applied to the entire hyperspectral image to generate full-image prediction maps. The notebook records how long each model takes to predict the full image.

The full-image maps are used for visual and runtime comparison; accuracy is measured only on the held-out stratified test sample.

<img width="742" height="328" alt="image" src="https://github.com/user-attachments/assets/3843b39b-92a9-4d26-af1b-f59ef62efe65" />

## Main Result

The selected 7-band model retained much of the classification performance of the full 430-band model while using only a small fraction of the original spectral features.

This supports the main conclusion:

> Hyperspectral vegetation data contains strong redundancy, and a carefully selected small set of wavelength bands can preserve much of the useful classification information.

## Repository Structure

```text
.
├── notebooks/
│   ├── 1_eda.ipynb
│   ├── 2_convert_zarr.ipynb
│   ├── 3_stratified_sampling.ipynb
│   ├── 4_pca.ipynb
│   ├── 5_clustering.ipynb
│   └── 6_supervised_ml.ipynb
│
├── masks/
│   ├── .rgb_reference.png
│   ├── class_mask.npy
│   ├── water_mask.png
│   ├── soil_mask.png
│   ├── grass_mask.png
│   └── canopy_mask.png
│
├── data/
│   └── (not included)
│
└── README.md
```

## Requirements

Main Python libraries used:

```text
numpy
pandas
matplotlib
scikit-learn
zarr
tqdm
rasterio
Pillow
joblib
```

Install dependencies with:

```bash
pip install numpy pandas matplotlib scikit-learn zarr tqdm rasterio pillow joblib
```

## How to Run

Run the notebooks in order:

```text
1. Exploratory Data Analysis
2. Convert hyperspectral cube to Zarr
3. Create masks and stratified samples
4. Compute PCA and global statistics
5. Run wavelength selection and K-Means comparison
6. Train Random Forest models and generate prediction maps
```

Notebook 6 assumes that `global_stats.npz` already contains the sampled pixels, full-band spectra, standardized spectra, global mean/std, and PCA components.

## Notes

The full hyperspectral dataset is large and is not included in this repository. The notebooks assume the data is available locally in the expected directory structure.


## Limitations

The labels come from manually created masks, so results depend on mask quality. The full-image prediction maps are model outputs, not independently validated ground truth maps. Canopy and grass are spectrally similar and are the most difficult classes to separate.

## Conclusion

This project shows that hyperspectral vegetation data is highly redundant. PCA confirmed that most variance can be represented with only a few components, and wavelength distance analysis identified a compact 7-band subset. Random Forest classification showed that selected wavelength bands can retain much of the useful information from the full 430-band dataset while greatly reducing feature dimensionality.
