# Earth Observation Pipeline for Land-Use Classification

An end-to-end geospatial machine learning pipeline that performs **spatial filtering, land-cover label generation, dataset preparation, and CNN-based land-use classification** for the Delhi‑NCR region using satellite imagery and ESA WorldCover data.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Repository Structure](#repository-structure)
- [Pipeline Overview](#pipeline-overview)
- [1. Spatial Reasoning & Data Filtering](#1-spatial-reasoning--data-filtering)
- [2. Label Construction & Dataset Preparation](#2-label-construction--dataset-preparation)
- [3. Model Training & Evaluation](#3-model-training--evaluation)
- [Results](#results)
- [Technologies Used](#technologies-used)
- [Installation](#installation)
- [Usage](#usage)
- [Reproducibility](#reproducibility)
- [Notes on AI Usage](#notes-on-ai-usage)
- [Contributing](#contributing)
- [License](#license)

---

## Project Overview

This project implements a **geospatial machine learning pipeline** for classifying land-use categories using satellite imagery.

The workflow integrates **geospatial analysis and deep learning** to automatically generate labels from land-cover raster data and train a convolutional neural network for land-use classification.

The pipeline includes:

1. Spatial filtering of satellite images within a geographic region  
2. Grid-based spatial reasoning  
3. Land-cover patch extraction from raster data  
4. Automatic dataset labeling  
5. Train–test dataset creation  
6. CNN model training and evaluation  

The study region used in this project is **Delhi‑NCR**.

---

## Repository Structure

```text
AI for Sustainability - Earth Observation/
│
├── data/
│   ├── rgb/                                     # Satellite RGB image patches
│   ├── delhi_airshed.geojson                    # Delhi airshed boundary
│   ├── delhi_ncr_region.geojson                 # Delhi NCR region shapefile
│   └── worldcover_bbox_delhi_ncr_2021.tif       # ESA WorldCover land-cover raster
│
├── Earth_Observation_pipeline.ipynb  # Main notebook implementing the full pipeline
│
├── image_centers.csv          # Extracted coordinates of satellite image centers
├── filtered_images.csv        # Images whose centers fall inside the NCR region
│
├── dataset_labels.csv         # Generated labeled dataset
├── train_labels.csv           # Training split
├── test_labels.csv            # Test split
│
├── resnet18_landuse.pth       # Trained CNN model weights
│
├── README.md                  # Project documentation
└── .gitignore                 # Ignored files (large data, checkpoints)
```

---

## Pipeline Overview

The entire pipeline is implemented in:

```text
Earth_Observation_pipeline.ipynb
```

The workflow consists of three main stages:

1. **Spatial reasoning and filtering**  
2. **Land-cover label construction**  
3. **CNN training and evaluation**

---

## 1. Spatial Reasoning & Data Filtering

### Region Loading  
The Delhi‑NCR region boundary is loaded using **GeoPandas** (CRS: `EPSG:4326`).  
For spatial operations, the shapefile is projected to `EPSG:32644` (UTM projection).

### Grid Generation  
A **60 × 60 km uniform grid** is generated over the NCR region.

Steps:

1. Compute bounding box of the region  
2. Generate grid cells using `shapely.box`  
3. Retain cells intersecting the NCR boundary

Total grid cells created: 29  

The grid and NCR boundary are visualized using **Matplotlib**.

### Extracting Image Coordinates  
Satellite image filenames encode the center coordinates.  
Example:

```text
28.2056_76.8558.png
```

Coordinates are parsed to create metadata:

| filename | latitude | longitude |
|----------|----------|-----------|

Total images before filtering: 9216

### Spatial Filtering  
Image centers are converted into a **GeoDataFrame** and tested against the NCR polygon.

Filtering condition:

```python
point.within(NCR_polygon)
```

Result:

| Stage            | Number of Images |
|------------------|------------------|
| Before filtering |  9216            |
| After filtering  |  8015            |

Filtered metadata is saved as: `filtered_images.csv`.

---

## 2. Label Construction & Dataset Preparation

Each image is assigned a **land-use label** based on the dominant land-cover class inside a corresponding raster patch.

### Patch Extraction  
For each image center:

1. Convert coordinates to raster indices  
2. Extract a **128 × 128 window** from the land-cover raster  

*Raster resolution:* 10 m/pixel  
Patch coverage: 128 × 128 pixels ≈ 1.28 km × 1.28 km

### Dominant Class Assignment  
The label is determined by the **mode** of land-cover classes inside the patch. No-data pixels (`0`) are ignored.

### ESA Class Mapping  
ESA WorldCover classes are mapped to simplified labels:

| ESA Code | Category            |
|----------|---------------------|
| 10       | Tree_cover          |
| 20       | Shrubland           |
| 30       | Grassland           |
| 40       | Cropland            |
| 50       | Built-up            |
| 60       | Bare_sparse_veg     |
| 80       | Perm_water          |
| 90       | Herbaceous_wetland  |

If no valid class exists, the label is set to `Other`.  
The labeled dataset is stored in `dataset_labels.csv`.  
Total labeled samples: 8015

### Train–Test Split  
The dataset is split using **stratified sampling** to preserve class distribution.  
Split ratio: Train 60 % / Test 40 %

Result:

| Split | Samples |
|-------|---------|
| Train |  4809   |
| Test  |  3206   |

Files generated: `train_labels.csv`, `test_labels.csv`.  
Class distributions are visualized using bar plots.

---

## 3. Model Training & Evaluation

A **ResNet18 convolutional neural network** is used for land-use classification.

### Data Loading  
A custom PyTorch `Dataset` loads:

- RGB satellite images  
- Corresponding labels from CSV files

Data augmentation uses **Albumentations**:

- Resize  
- Horizontal & vertical flip  
- Rotation  
- Random brightness/contrast  
- Gaussian noise  
- ImageNet normalization

### Model Architecture  
ResNet18 (ImageNet-pretrained) with the final fully connected layer modified to match the number of classes.

### Training Configuration  
| Parameter   | Value              |
|-------------|--------------------|
| Batch size  | 32                 |
| Epochs      | 10                 |
| Optimizer   | Adam               |
| Loss        | CrossEntropyLoss   |

Training loss is tracked across epochs.  
Final model saved as:

```text
resnet18_landuse.pth
```

---

## Results

Evaluation metrics:

| Metric            | Score   |
|-------------------|---------|
| Test Accuracy     | 0.6621  |
| Weighted F1 Score | 0.6154  |

A **confusion matrix** is generated to analyze prediction performance.

### Interpretation  
The model mainly predicts **Cropland** and **Built-up** classes.  
This occurs due to **class imbalance**: cropland dominates the dataset, while several classes have few examples, causing bias toward majority classes.

---

## Technologies Used

**Geospatial Processing**

- GeoPandas  
- Shapely  
- Rasterio

**Data Processing**

- NumPy  
- Pandas

**Visualization**

- Matplotlib  
- Seaborn

**Deep Learning**

- PyTorch  
- Torchvision  
- Albumentations  
- OpenCV

**Machine Learning Utilities**

- Scikit-learn

---

## Installation

```bash
git clone https://github.com/Shweta-Roshia/SRIP-2026-AI-for-Sustainability
cd "AI for Sustainaibility - Earth Observation"

pip install -r requirements.txt
```

---

## Usage

Run the main notebook:

```bash
jupyter notebook Earth_Observation_pipeline.ipynb
```

The notebook sequentially performs:

1. Spatial filtering of satellite images  
2. Raster-based label construction  
3. Dataset splitting  
4. CNN training  
5. Model evaluation

---

## Reproducibility

To reproduce the results:

1. Clone the repository  
2. Install the required dependencies  
3. Ensure the dataset files are present inside the `data/` directory  
4. Run all cells in `Earth_Observation_pipeline.ipynb`

The pipeline will generate:

- `image_centers.csv`  
- `filtered_images.csv`  
- `dataset_labels.csv`  
- `train_labels.csv`  
- `test_labels.csv`  
- `resnet18_landuse.pth`

---

## Notes on AI Usage

AI tools were used for conceptual understanding and debugging assistance during development. All pipeline design, algorithmic logic, and code implementation were written manually as part of the project.

---

## Contributing

Contributions are welcome if they improve the project implementation, efficiency, or reproducibility. Contributions that enhance the project are appreciated.

---

## License

This project is licensed under the MIT License. See the LICENSE file for details.
