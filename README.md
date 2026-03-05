# SRIP 2026 – Earth Observation Land Use Classification

## Overview

This project implements a geospatial machine learning pipeline for analyzing satellite imagery over the **Delhi-NCR region**. The objective is to build a complete workflow that performs spatial filtering, label construction from land-cover data, and dataset preparation for land-use classification.

The pipeline integrates **vector geospatial data, satellite imagery metadata, and raster land-cover data** to generate labeled training samples suitable for deep learning models.

---

# Problem Statement

The task is part of the **SRIP 2026 Earth Observation assignment**.

Given:

* Delhi-NCR region shapefile
* Sentinel-2 RGB satellite image patches (128×128 pixels) with center coordinates
* ESA WorldCover 2021 land-cover raster
* Delhi Airshed boundary

The goal is to construct a complete pipeline that:

1. Creates a spatial grid over the Delhi-NCR region
2. Filters satellite images located inside the study region
3. Extracts land-cover patches from the raster dataset
4. Assigns labels based on dominant land-cover class
5. Prepares a dataset suitable for training a land-use classifier

---

# Project Structure

```
srip-earth-observation
│
├── Earth_Observation_pipeline.ipynb     # Main notebook implementing the pipeline
├── dataset_labels.csv                   # Final labeled dataset
├── train_split.csv                      # Training split (60%)
├── test_split.csv                       # Test split (40%)
├── dataset_labels.csv                   
├── filtered_images.csv                  # images lying inside study region
│
├── data
│   ├── delhi_ncr_region.geojson         # Delhi NCR boundary
│   └── delhi_airshed.geojson            # Delhi Airshed boundary
│
└── README.md
```

Large raster datasets and image patches are **not included** in the repository.

---

# Methodology

## Q1 – Spatial Reasoning and Data Filtering

Steps performed:

1. Load the **Delhi-NCR boundary shapefile** using GeoPandas.
2. Convert the coordinate system to **EPSG:32644** for metric distance calculations.
3. Generate a **60 × 60 km uniform spatial grid** over the region.
4. Overlay the grid with the NCR boundary.
5. Extract satellite image center coordinates from filenames.
6. Convert coordinates into geospatial points.
7. Filter images whose center lies inside the NCR region.

Outputs:

* Spatial grid visualization
* Filtered image metadata (filtered_images.csv)
* CSV containing valid image centers (image_centers.csv)

---

## Q2 – Label Construction and Dataset Preparation

For each filtered satellite image:

1. Identify the image center coordinate.
2. Extract a **128×128 land-cover patch** from the ESA WorldCover raster.
3. Compute the **dominant land-cover class (mode)** in the patch.
4. Map ESA class codes to simplified land-use categories.

Example mapping:

| ESA Code | Land Cover Type          |
| -------- | ------------------------ |
| 10       | Tree Cover               |
| 20       | Shrubland                |
| 30       | Grassland                |
| 40       | Cropland                 |
| 50       | Built-up                 |
| 60       | Bare / Sparse Vegetation |
| 70       | Snow and ice             |
| 80       | Permanent water          |
| 90       | Herbaceous Wetland       |
| 95       | Mangroves                |
| 100      | Moss and Lichen          |

The dominant class becomes the **image label**.

---

## Dataset Preparation

After label assignment:

1. Convert land-cover labels into integer class IDs
2. Shuffle the dataset randomly
3. Perform **60/40 train-test split**
4. Visualize **class distribution**

Outputs generated:

```
dataset_labels.csv
train_split.csv
test_split.csv
```

---

# Example Visualization

The pipeline generates visualizations including:

* Delhi NCR boundary
* 60 × 60 km spatial grid
* Filtered image center locations
* Land-use class distribution

These visualizations help verify spatial correctness and dataset balance.

---

# Technologies Used

Python libraries used in the project:

* **GeoPandas** – vector geospatial operations
* **Shapely** – geometric computations
* **Rasterio** – raster data processing
* **NumPy / Pandas** – data processing
* **Matplotlib** – visualization
* **Scikit-learn** – dataset splitting

---

# Dataset Information

| Dataset                 | Description                               |
| ----------------------- | ----------------------------------------- |
| Sentinel-2 RGB patches  | Satellite imagery used for classification |
| ESA WorldCover 2021     | Land-cover raster used to generate labels |
| Delhi NCR shapefile     | Region boundary                           |
| Delhi Airshed shapefile | Environmental monitoring boundary         |

Note: Large datasets are excluded from this repository due to size constraints.

---

# Outputs

The pipeline produces:

* Filtered satellite image metadata
* Land-cover labels for each image
* Train/test dataset split
* Visualizations for spatial verification

These outputs form the dataset for the next stage: **training a CNN classifier for land-use classification.**

---

# Reproducibility

To run the project:

1. Install required libraries:

```bash
pip install geopandas rasterio shapely matplotlib pandas numpy scikit-learn
```

2. Place the required datasets inside the `data/` directory.

3. Run the notebook:

```
Earth_Observation_pipeline.ipynb
```

---

# Acknowledgment

Some guidance and explanations for geospatial processing and code structuring were obtained using AI-based tools for learning and debugging assistance. All implementation decisions and final code integration were performed by the author.

---

# Author

**Shweta Roshia**
B.Tech – Artificial Intelligence
SRIP 2026 Applicant

---
