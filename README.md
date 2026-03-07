# Earth Observation Pipeline for Land-Use Classification

An end-to-end geospatial machine learning pipeline that performs **spatial filtering, land-cover label generation, dataset preparation, and CNN-based land-use classification** for the Delhi-NCR region using satellite imagery and ESA WorldCover data.

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

# Project Overview

This project implements a **geospatial machine learning pipeline** for classifying land-use categories using satellite imagery.

The workflow integrates **geospatial analysis and deep learning** to automatically generate labels from land-cover raster data and train a convolutional neural network for land-use classification.

The pipeline includes:

- Spatial filtering of satellite images within a geographic region
- Grid-based spatial reasoning
- Land-cover patch extraction from raster data
- Automatic dataset labeling
- Train–test dataset creation
- CNN model training and evaluation

The study region used in this project is **Delhi-NCR**.

---

# Repository Structure
