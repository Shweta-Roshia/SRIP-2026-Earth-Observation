# Earth Observation Pipeline for Land-Use Classification

This repository implements a geospatial machine learning pipeline for land-use classification using satellite imagery and raster land-cover data.

The objective is to construct a full pipeline that:

1. Performs spatial reasoning using geospatial data  
2. Generates labels from a land-cover raster  
3. Trains a convolutional neural network to classify land-use from satellite image patches  

The workflow combines **geospatial analysis (GeoPandas, Rasterio)** with **deep learning (PyTorch)**.

---

# Repository Structure


.
├── Earth_Observation_pipeline.ipynb # Complete pipeline implementation
├── README.md # Project documentation
├── image_centers.csv # Coordinates extracted from image filenames
├── filtered_images.csv # Image centers inside Delhi NCR region
├── dataset_labels.csv # Dataset with raster-derived labels
├── train_labels.csv # Training split (60%)
├── test_labels.csv # Test split (40%)
├── resnet18_landuse.pth # Trained model weights
└── .gitignore # Ignored files and datasets


Large datasets such as satellite images and raster files are excluded from the repository.

---

# Dataset Overview

### Sentinel-2 RGB Image Patches
- Image size: **128 × 128 pixels**
- Resolution: **10 meters per pixel**
- Each image filename encodes the **center latitude and longitude**

Example:


28.2056_76.8558.png


This corresponds to:


Latitude = 28.2056
Longitude = 76.8558


---

### ESA WorldCover 2021 Raster

A global land-cover dataset at **10 m spatial resolution** used to generate labels for satellite image patches.

The raster is used to determine the **dominant land-cover class** within each 128×128 patch.

---

### Delhi NCR Boundary Shapefile

Used for spatial filtering of satellite images and grid visualization.

---

# Pipeline Overview

The notebook implements the workflow in three main stages.

---

# 1. Spatial Reasoning & Data Filtering

The Delhi NCR boundary shapefile is loaded using GeoPandas and visualized.

The coordinate system is converted from:


EPSG:4326 → EPSG:32644


This allows spatial operations in **meters**.

A **60 × 60 km grid** is generated over the region using Shapely polygons.

Satellite image centers are extracted from image filenames and converted into geospatial points.

Images are filtered so that only those whose **center coordinates lie inside the NCR boundary** are retained.

Result:


Images before filtering : 9216
Images after filtering : 8015


Filtered metadata is saved to:


filtered_images.csv


---

# 2. Label Construction & Dataset Preparation

Land-cover labels are generated using the ESA WorldCover raster.

For each satellite image:

1. The center coordinate is converted to raster pixel coordinates
2. A **128 × 128 raster window** is extracted around the center
3. The **dominant land-cover class (mode)** is computed
4. The ESA class is mapped to a simplified label

Example mapping:

| ESA Class | Label |
|-----------|------|
| 10 | Tree_cover |
| 20 | Shrubland |
| 30 | Grassland |
| 40 | Cropland |
| 50 | Built-up |
| 60 | Bare_sparse_veg |
| 80 | Perm_water |
| 90 | Herbaceous_wetland |

Each dataset sample contains:


filename
latitude
longitude
esa_class
label
class_id
class_count


The labeled dataset is saved as:


dataset_labels.csv


---

# Train-Test Split

The dataset is split using a **60/40 stratified split**.


Training samples : 4809
Testing samples : 3206


Saved as:


train_labels.csv
test_labels.csv


Class distributions are visualized using bar plots.

---

# 3. Model Training

A **ResNet-18 convolutional neural network** is trained for land-use classification.

The final fully connected layer is replaced to match the number of land-use classes.

Training configuration:

- Model: ResNet-18  
- Optimizer: Adam  
- Loss function: CrossEntropyLoss  
- Batch size: 32  
- Epochs: 10  

Data augmentation using **Albumentations**:

- Horizontal flip
- Vertical flip
- Rotation
- Random brightness/contrast
- Gaussian noise
- Normalization

Training loss is recorded and visualized.

The trained model is saved as:


resnet18_landuse.pth


---

# 4. Model Evaluation

The trained model is evaluated on the test dataset using:

- **Accuracy**
- **Weighted F1 Score**
- **Confusion Matrix**

Results:


Test Accuracy ≈ 0.66
F1 Score ≈ 0.61


The confusion matrix shows the model predicts **Cropland and Built-up classes most frequently**, which is expected because these classes dominate the dataset.

This behavior is caused by **class imbalance**, where minority classes have very few training examples.

---

# Reproducibility

To reproduce the pipeline:

### 1. Clone the repository

```bash
git clone <repository_url>
2. Install dependencies
pip install geopandas rasterio shapely matplotlib pandas numpy torch torchvision albumentations scikit-learn seaborn
3. Place datasets inside a data/ directory
data/
├── rgb/                          # Satellite image patches
├── delhi_ncr_region.geojson
└── worldcover_bbox_delhi_ncr_2021.tif
4. Run the notebook
Earth_Observation_pipeline.ipynb

Running the notebook sequentially will recreate the entire pipeline.

Acknowledgment

AI tools were used only to clarify concepts and assist with debugging errors during development.

All pipeline design, implementation, experimentation, and analysis were carried out independently.
