Earth Observation Pipeline for Land-Use Classification

This repository implements a geospatial machine learning pipeline for land-use classification using satellite imagery and raster land-cover data.

The objective is to build an end-to-end workflow that combines geospatial analysis with deep learning to classify land-use categories in the Delhi-NCR region.

The pipeline performs the following tasks:

Spatial reasoning with geospatial data

Filtering satellite images based on geographic boundaries

Generating labels from a land-cover raster

Preparing a dataset for supervised learning

Training and evaluating a CNN model for land-use classification

The implementation integrates tools from GeoPandas, Rasterio, PyTorch, and computer vision libraries.

Repository Structure
AI for Sustainability - Earth Observation/
│
├── data/
│   ├── rgb/                                  # Satellite RGB image patches
│   ├── archive.zip                           # Original archive of RGB patches
│   ├── delhi_airshed.geojson                 # Delhi airshed boundary
│   ├── delhi_ncr_region.geojson              # Delhi-NCR region shapefile
│   └── worldcover_bbox_delhi_ncr_2021.tif    # ESA WorldCover land-cover raster
│
├── Earth_Observation_pipeline.ipynb          # Main notebook containing the full pipeline
│
├── image_centers.csv                         # Extracted image center coordinates
├── filtered_images.csv                       # Images whose centers lie inside Delhi-NCR
│
├── dataset_labels.csv                        # Dataset with generated land-use labels
├── train_labels.csv                          # Training split (60%)
├── test_labels.csv                           # Test split (40%)
│
├── resnet18_landuse.pth                      # Final trained CNN model
│
├── README.md                                 # Project documentation
└── .gitignore                                # Ignored files (data artifacts, checkpoints)
Dataset Overview
Satellite Images

RGB satellite image patches

Resolution: 128 × 128 pixels

Extracted from Sentinel-2 imagery

File naming format:

latitude_longitude.png

Example:

28.2056_76.8558.png

The filename directly encodes the geographic coordinates of the image center.

Land-Cover Raster

The project uses ESA WorldCover 2021 raster data.

Properties:

Spatial resolution: 10 meters

CRS: EPSG:4326

Classes include vegetation, cropland, built-up areas, water, etc.

Pipeline Overview

The entire workflow is implemented in the notebook:

Earth_Observation_pipeline.ipynb

The pipeline consists of three main stages.

1. Spatial Reasoning & Data Filtering
Region of Interest

The Delhi-NCR region shapefile is loaded and visualized using GeoPandas.

CRS: EPSG:4326

For spatial calculations, the shapefile is projected to:

EPSG:32644 (UTM projection)
Grid Generation

A uniform grid of 60 × 60 km cells is generated over the region.

Steps:

Compute bounding box of Delhi-NCR

Generate square grid cells using shapely.box

Keep only cells that intersect the NCR boundary

Total grid cells created:

29

The grid is visualized together with the NCR boundary.

Extracting Image Coordinates

Satellite image filenames contain the coordinates of their center.

Example:

28.2056_76.8558.png

These coordinates are parsed to construct a metadata table:

filename	latitude	longitude

Total images before filtering:

9216
Spatial Filtering

Image centers are converted to a GeoDataFrame and projected to the same CRS as the grid.

Images are filtered using:

point.within(NCR_polygon)

Result:

Stage	Image Count
Before filtering	9216
After filtering	8015

The filtered dataset is saved as:

filtered_images.csv
2. Label Construction & Dataset Preparation

Each image is assigned a land-use label based on the dominant land-cover class inside a corresponding raster patch.

Patch Extraction

For every image center:

Convert geographic coordinates to raster indices

Extract a 128 × 128 pixel window from the land-cover raster

Patch size corresponds to:

128 pixels × 10 m resolution = 1280 m
Dominant Class Assignment

The label for each image is determined by:

mode (most frequent value) of raster pixels in the patch

Pixels with value 0 (no data) are ignored.

ESA Class Mapping

ESA WorldCover classes are mapped to simplified labels:

ESA Code	Category
10	Tree_cover
20	Shrubland
30	Grassland
40	Cropland
50	Built-up
60	Bare_sparse_veg
80	Perm_water
90	Herbaceous_wetland

If no valid class is found, the label is assigned:

Other

The labeled dataset is stored in:

dataset_labels.csv

Total labeled samples:

8015
Train/Test Split

The dataset is split using stratified sampling to preserve class distribution.

Train: 60%
Test: 40%

Result:

Split	Samples
Train	4809
Test	3206

Files generated:

train_labels.csv
test_labels.csv

Class distributions are visualized using bar charts.

3. Model Training & Evaluation

A ResNet18 CNN is used for land-use classification.

Data Loading

A custom PyTorch dataset loads images and labels from CSV files.

Transformations applied using Albumentations:

Resize to 128×128

Horizontal / Vertical flips

Rotation

Random brightness and contrast

Gaussian noise

ImageNet normalization

Model

Architecture:

ResNet18 (pretrained on ImageNet)

Final classification layer modified to match the number of classes.

Training

Parameters:

Batch size: 32
Epochs: 10
Optimizer: Adam
Loss function: CrossEntropyLoss

Training loss is recorded across epochs.

The final trained model is saved as:

resnet18_landuse.pth
Evaluation Metrics

Model performance is evaluated using:

Accuracy

Weighted F1 Score

Confusion Matrix

Results on the test set:

Accuracy: 0.66
F1 Score: 0.61
Confusion Matrix Interpretation

The model primarily predicts Cropland and Built-up classes.

This behavior occurs due to strong class imbalance in the dataset:

Cropland contains the majority of samples

Several classes have very few examples

As a result, the model tends to favor majority classes, which limits performance on minority categories.

Reproducibility

To reproduce the pipeline:

1. Clone the repository
git clone <repo_url>
cd Earth-Observation-Pipeline
2. Install dependencies
pip install geopandas rasterio shapely matplotlib numpy pandas
pip install torch torchvision albumentations opencv-python
pip install scikit-learn seaborn
3. Run the notebook

Open and execute:

Earth_Observation_pipeline.ipynb

The notebook sequentially performs:

Spatial filtering

Label generation

Dataset splitting

CNN training

Model evaluation

Notes on AI Usage

Artificial intelligence tools were used only as a support tool during development for:

Understanding certain concepts

Debugging implementation errors

All core logic, pipeline design, and code implementation were written manually as part of the project.

Output Artifacts

Running the full pipeline generates the following outputs:

image_centers.csv
filtered_images.csv
dataset_labels.csv
train_labels.csv
test_labels.csv
resnet18_landuse.pth

These files represent the processed dataset and the trained model.
