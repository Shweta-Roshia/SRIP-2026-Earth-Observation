# Earth Observation Pipeline for Land-Use Classification

This repository implements a geospatial machine learning pipeline for land-use classification using satellite imagery and raster land-cover data.

The objective is to construct a full pipeline that:

1. Performs spatial reasoning using geospatial data.
2. Generates labels from a land-cover raster.
3. Trains a convolutional neural network to classify land-use from satellite image patches.

The workflow combines geospatial analysis (GeoPandas, Rasterio) with deep learning (PyTorch).

---

# Repository Structure
.
├── Earth_Observation_pipeline.ipynb # Complete pipeline implementation
├── README.md # Project documentation
├── image_centers.csv # Coordinates extracted from image filenames
├── filtered_images.csv # Image centers inside Delhi NCR region
├── dataset_labels.csv # Final dataset with raster-derived labels
├── train_labels.csv # Training split (60%)
├── test_labels.csv # Test split (40%)
├── resnet18_landuse.pth # Trained model weights
└── .gitignore # Ignored files and datasets
