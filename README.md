# GEOAI Challenge for Cropland Mapping in Dry Environments

**Silver Medal Solution (30th Place)**  
Public Score: 0.75556 | Private Score: 0.68571

This repository contains the solution notebook for the [Zindi GEOAI Challenge](https://zindi.africa/competitions/geoai-challenge-for-cropland-mapping-in-dry-environments) – a binary classification task to map cropland areas using Sentinel-1 and Sentinel-2 satellite time series in dry environments (Uzbekistan & Russian Federation).

## Competition Overview

- **Goal**: Predict whether a given geolocation point is cropland (1) or non-cropland (0) based on multi‑year Sentinel‑1 and Sentinel‑2 observations.
- **Training data**: Shapefiles with point labels (`Fergana_training_samples.shp`, `Orenburg_training_samples.shp`) and corresponding Sentinel‑1/‑2 CSVs.
- **Test data**: Sentinel‑1/‑2 CSVs with masked coordinates (`Test.csv` provides only IDs).
- **Evaluation metric**: Accuracy.

## Approach

The solution follows a **spatial nearest‑neighbor label assignment** combined with **aggregated spectral features** and a **small neural network classifier**.

### 1. Data Preprocessing
- Load Sentinel‑1 (VH, VV bands) and Sentinel‑2 (B2–B8A, B11, B12 bands) data.
- Load training shapefiles (Fergana + Orenburg), reproject to EPSG:4326, extract point coordinates (lon/lat).
- Clean and validate geometries.

### 2. Label Propagation from Training Points
- Because training points are sparse and test coordinates are masked, we **assign a cropland label** to each Sentinel‑2 observation ID based on the **nearest training point** (using `cKDTree` on latitude/longitude).
- Each test ID receives the label of its closest training point – a simple but effective way to transfer ground truth.

### 3. Feature Aggregation
For each unique ID (i.e., each geographic location across time), we aggregate all available Sentinel‑1 and Sentinel‑2 measurements into **statistical features**:
- **Mean**, **standard deviation**, **min**, **max** for each valid band.
- This reduces irregularly sampled time series into a fixed‑length feature vector per ID.

### 4. Model Architecture
A small **fully connected neural network** (TensorFlow/Keras) is used:
- Binary cross‑entropy loss, Adam optimizer.
- Trained for 15 epochs with batch size 4.
- Validation split: 20% stratified.

### 5. Prediction & Submission
- Features are standardized (`StandardScaler`) and aligned between train and test.
- Predictions thresholded at 0.5 to produce final binary labels.
- Output `submission.csv` with columns `ID, Target`.

## Key Decisions & Why They Worked

| Decision | Rationale |
|----------|------------|
| **Nearest‑neighbor label assignment** | Overcomes masked test coordinates; leverages spatial proximity in a dry landscape where cropland patterns are contiguous. |
| **Aggregated statistics (mean/std/min/max)** | Captures both central tendency and seasonal variability of spectral bands without complex time‑series models. |
| **Small neural network** | Prevents overfitting given limited training IDs; simpler than SVM or gradient boosting while achieving competitive accuracy. |
| **Merging Sentinel‑1 & Sentinel‑2** | Combines radar (sensitivity to structure/moisture) with optical (vegetation indices) for robust feature space. |

## Results
- **Validation accuracy**: ~66.7%
- **Public leaderboard**: 0.75556
- **Private leaderboard**: 0.68571
- **Final rank**: 30th / 618 (Silver Medal)
