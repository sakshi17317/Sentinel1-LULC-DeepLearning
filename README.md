# Sentinel-1-Based Land Use and Land Cover (LULC) Mapping Using Deep Learning

**A multi-regional semantic segmentation study and 12-month LULC monitoring prototype**

## Project Overview

This project develops a deep learning-based approach for Land Use and Land Cover (LULC) semantic segmentation using Sentinel-1 Synthetic Aperture Radar (SAR) imagery.

The project investigates whether Sentinel-1 VV and VH backscatter information can be used to classify land cover across geographically diverse regions. It also develops a monthly monitoring prototype that generates 12 consecutive monthly LULC maps from a user-provided geographic location and start month.

The project consists of two main components:

1. **LULC semantic segmentation:** Dataset preparation, preprocessing, model development, experimental comparisons and independent test evaluation.
2. **Monthly monitoring prototype:** Retrieval of Sentinel-1 imagery, rolling-window monthly predictions, class-area statistics and month-to-month change analysis.

## Objectives

- Develop a Sentinel-1-only LULC semantic segmentation model.
- Investigate geographic generalization using a multi-regional dataset.
- Prepare and validate a consistent dataset using Sentinel-1 imagery and ESA WorldCover labels.
- Evaluate different model configurations and training strategies.
- Assess model performance using Overall Accuracy, mean Intersection over Union, per-class IoU and F1-score.
- Develop a location-based prototype that generates 12 consecutive monthly LULC maps.
- Calculate monthly class-area percentages and percentage-point changes.
- Visualize monthly predictions and first-to-last changes.

## Data Sources

| Dataset | Source | Purpose |
|---|---|---|
| Sentinel-1 GRD | Google Earth Engine: `COPERNICUS/S1_GRD` | VV and VH SAR input features |
| ESA WorldCover 2021 | Google Earth Engine: `ESA/WorldCover/v200` | LULC reference labels |

**Important:** The segmentation model is trained using Sentinel-1 imagery and annual ESA WorldCover 2021 labels. No Sentinel-2 optical imagery is used as a model input.

## Dataset Design

The finalized combined dataset contains **602 spatial patches**.

| Split | Number of patches |
|---|---:|
| Training | 422 |
| Validation | 80 |
| Testing | 100 |
| **Total** | **602** |

The dataset uses a geographic split across 50 cities:

- Training: 32 cities
- Validation: 8 cities
- Testing: 10 cities

This separation is intended to evaluate geographic generalization rather than relying only on randomly separated patches from the same locations.

### Patch specifications

- Spatial dimensions: 256 × 256 pixels
- Approximate spatial resolution: 30 m
- Approximate patch width: 7.68 km
- SAR channels: VV and VH
- Temporal input: 12 monthly observations
- Original image tensor: `(12, 2, 256, 256)`
- Label mask: `(256, 256)`

### LULC classes

The original ESA WorldCover categories are merged into six project classes:

| Class ID | Class name |
|---:|---|
| 0 | Tree cover |
| 1 | Shrubland / Grassland / Moss |
| 2 | Cropland |
| 3 | Built-up |
| 4 | Bare land / Snow / Ice |
| 5 | Water / Wetland / Mangrove |

## Preprocessing

The preprocessing workflow includes:

1. Retrieval of Sentinel-1 GRD imagery through Google Earth Engine.
2. Selection of VV and VH polarizations.
3. Temporal compositing of the 2021 observations.
4. Spatial alignment of SAR imagery and WorldCover labels.
5. Resampling and preparation of 256 × 256 patches.
6. Mapping the original land-cover categories into six merged classes.
7. Data quality and validity checks.
8. Robust normalization using training-set statistics.

### Training-set normalization statistics

| Channel | Median | IQR |
|---|---:|---:|
| VV | -10.2969 | 5.75 |
| VH | -16.9375 | 6.7656 |

The robust normalization uses training-only statistics to avoid information leakage from validation and test data.

## Model Development

The project investigates multiple segmentation experiments, including:

- U-Net baseline.
- ResNet34-based U-Net with Cross-Entropy and Dice loss.
- Rare-class sampling experiments.
- Pretrained ResNet50-based U-Net.
- Regularization experiments.
- Targeted fine-tuning experiments.

The best recorded experiment is **Experiment 23 (E23)**, using a targeted fine-tuning strategy based on the ResNet34 U-Net approach.

The model predicts one of the six LULC classes for each valid pixel.

## Final Model Evaluation

### Experiment 23 — Test-set results

| Metric | Score |
|---|---:|
| Overall Accuracy (OA) | 79.042% |
| Mean IoU (mIoU) | 57.411% |
| Macro F1-score | 70.111% |

### Per-class test IoU

| LULC class | IoU |
|---|---:|
| Tree cover | 66.031% |
| Shrubland / Grassland / Moss | 34.861% |
| Cropland | 71.098% |
| Built-up | 57.396% |
| Bare land / Snow / Ice | 23.144% |
| Water / Wetland / Mangrove | 91.935% |

Water/wetland/mangrove achieves the strongest IoU, while bare land/snow/ice and shrubland/grassland/moss remain challenging classes.

The repository contains evaluation summaries, per-class metrics, confusion matrices, training histories and visualizations.

## Monthly LULC Monitoring Prototype

A second component of the project implements a location-based monthly monitoring prototype.

The user supplies exactly three inputs:

1. Latitude
2. Longitude
3. Start month (`YYYY-MM`)

The prototype produces 12 consecutive monthly LULC predictions.

### Prototype outputs

- Twelve monthly LULC maps.
- Monthly class-area statistics.
- Monthly class-area percentages.
- Month-to-month percentage-point changes.
- First-to-last percentage-point changes.
- Visualizations of monthly trends and total changes.
- Metadata describing the retrieval and prediction process.

Each monthly prediction uses a rolling 12-month Sentinel-1 observation window ending in the corresponding output month. This requires 23 months of imagery to generate the full sequence of 12 monthly predictions.

For additional implementation details, see [`prototype_outputs/README.md`](prototype_outputs/README.md) and [`notebooks/prototype.ipynb`](notebooks/prototype.ipynb).

## Repository Structure

```text
LULC_Project/
│
├── README.md
├── .gitignore
│
├── notebooks/
│   ├── data collection and preprocessing.ipynb
│   ├── dataset final expansion.ipynb
│   ├── training.ipynb
│   ├── testing.ipynb
│   └── prototype.ipynb
│
├── prototype_outputs/
│   └── [location and start-month run]
│       ├── maps/
│       ├── monthly_class_area.csv
│       ├── monthly_percentage_point_changes.csv
│       ├── first_to_last_class_change.csv
│       └── metadata files
│
└── results/
    ├── E23_monthly_prototype/
    ├── exp19_geographic_generalization/
    ├── exp22_focal_finetune/
    ├── exp23_e21_targeted_finetune/
    ├── exp25_targeted_bare_finetune/
    ├── final_hidden_test_evaluation/
    ├── final_model_evaluation_standalone/
    ├── metrics/
    └── plots/
```

Large datasets and model checkpoint files are intentionally maintained separately from the GitHub source repository.

## Technologies Used

- Python
- PyTorch
- Torchvision
- NumPy
- Pandas
- Matplotlib
- Rasterio
- Google Earth Engine
- Sentinel-1 GRD
- ESA WorldCover
- Jupyter Notebook

## Reproducibility

To reproduce the project:

1. Clone this repository.
2. Install the required Python dependencies.
3. Obtain access to the finalized dataset using the public dataset link below.
4. Configure Google Earth Engine authentication and the appropriate Earth Engine project for the prototype.
5. Run the notebooks in the intended workflow.

### Dataset download

**Final combined dataset:** [ADD PUBLIC DATASET LINK / ZENODO DOI HERE]

The dataset contains 602 patches divided into training, validation and testing splits.

The dataset is hosted separately because of its size. The GitHub repository contains the project code, notebooks, evaluation outputs and visualizations, but does not contain the complete training dataset or model checkpoints.

### Model checkpoints

Model checkpoint files (`.pth`) are not included in this repository because of their size.

The final recorded E23 checkpoint was used for the best reported test evaluation. A reproducible release should provide the corresponding checkpoint separately if redistribution is permitted.

## Important Scientific Considerations and Limitations

- Sentinel-1 SAR backscatter can be affected by surface moisture, vegetation structure, acquisition geometry and seasonal conditions.
- Different land-cover classes can have similar radar signatures, creating classification ambiguity.
- Rare classes, particularly bare land/snow/ice, remain difficult to classify.
- The reference labels are derived from ESA WorldCover 2021 and do not represent independently verified monthly ground truth.
- Monthly prototype predictions are rolling-window model estimates, not direct observations of confirmed monthly land-cover transitions.
- Month-to-month percentage-point differences should therefore be interpreted as changes in model-estimated class composition, not automatically as verified land-cover change.
- Any missing-month fallback or imputed imagery must be interpreted using the prototype metadata.

## Future Work

- Improve rare-class representation and segmentation performance.
- Investigate temporal deep learning architectures.
- Evaluate the model on additional geographically independent regions.
- Incorporate independently validated temporal reference data.
- Improve monthly monitoring through more robust temporal compositing and missing-data handling.
- Develop a user-friendly deployment interface.

## Author

**Sakshi Manjrekar**  
M.Sc. Applied Data Science and Artificial Intelligence  
SRH University Heidelberg  
Germany

## Acknowledgements

- European Space Agency (ESA) for Sentinel-1 and WorldCover data products.
- Google Earth Engine for satellite data access and processing infrastructure.
- The open-source Python and deep learning communities.


---

**Project:** Sentinel-1-Based LULC Semantic Segmentation and Monthly Monitoring Prototype.
