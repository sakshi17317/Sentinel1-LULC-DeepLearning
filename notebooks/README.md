# Sentinel-1 LULC Monthly Monitoring Prototype



## 1. Overview



This prototype demonstrates monthly Land Use and Land Cover (LULC) monitoring using Sentinel-1 Synthetic Aperture Radar (SAR) imagery and a trained deep learning semantic segmentation model.



The prototype allows users to specify:

- Latitude

- Longitude

- Start month (`YYYY-MM`)



It generates \*\*12 consecutive monthly LULC maps\*\*, along with class-area statistics and changes over time.



The prototype uses Sentinel-1 VV and VH polarization data and predicts six merged LULC classes.



## 2. Prototype Objectives



The main objectives are to:



- Generate monthly LULC predictions for a user-selected geographic location.

- Visualize 12 consecutive monthly LULC maps.

- Calculate the percentage of the area occupied by each LULC class.

- Calculate month-to-month changes in class-area percentages.

- Compare the first and last months to identify overall percentage-point changes.



## 3. Input Parameters



The prototype requires exactly three user inputs:



| Input | Description |

|---|---|

| Latitude | Geographic latitude of the selected location |

| Longitude | Geographic longitude of the selected location |

| Start month | First month of the 12-month monitoring period (`YYYY-MM`) |



The selected coordinates define the centre of the analysis region.



## 4. Data Source



The prototype retrieves Sentinel-1 GRD imagery through Google Earth Engine.



**Dataset:** `COPERNICUS/S1\_GRD`



The two input channels are:

- VV polarization

- VH polarization



Monthly median composites are generated from Sentinel-1 observations.



The model was trained using Sentinel-1 observations from 2021 and ESA WorldCover 2021 reference labels.



## 5. Temporal Data Retrieval



To generate 12 consecutive monthly outputs, the prototype retrieves a total temporal period of 23 months.



This is necessary because each output month uses a rolling 12-month observation window.



For example, if the selected start month is March 2025:



- First output: March 2025

- Last output: February 2026

- Required observation period: April 2024 – February 2026



Each monthly prediction uses the 12-month Sentinel-1 window ending in the corresponding output month.



Monthly median VV and VH composites are calculated from the available observations.



If observations are unavailable for a particular month, a nearby-month fallback may be used. The prototype records the actual source month, scene count and fallback status.



## 6. Spatial Processing



The analysis region is constructed around the input coordinates.



The prototype uses:

- Spatial resolution: 30 metres

- Image dimensions: 256 × 256 pixels

- Approximate coverage: 7.68 km × 7.68 km

- Input channels: VV and VH



The imagery is transformed into the input format expected by the trained segmentation model.



## 7. Deep Learning Model



The prototype uses the trained ResNet34 U-Net semantic segmentation model.



The model receives Sentinel-1 VV and VH inputs and predicts a six-class LULC segmentation map.



The selected model was developed and evaluated using geographically separated training, validation and test datasets.



## 8. LULC Classification Scheme



The original ESA WorldCover categories were merged into six classes.



| Class ID | LULC Class |

|---|---|

| 0 | Tree cover |

| 1 | Shrubland / Grassland / Moss |

| 2 | Cropland |

| 3 | Built-up |

| 4 | Bare land / Snow / Ice |

| 5 | Water / Wetland / Mangrove |

| 255 | Invalid pixels / No data |



Invalid pixels are excluded from area calculations.



## 9. Monthly Outputs



For each of the 12 months, the prototype generates:



1. A predicted LULC segmentation map.

2. A visualization of the predicted land-cover distribution.

3. The percentage of valid pixels belonging to each class.

4. The corresponding class-area statistics.



All 12 monthly maps are visualized to allow temporal comparison.



## 10. Area Calculation



Each pixel represents an approximate ground area of:



- 30 m × 30 m = 900 m²

- 900 m² = 0.0009 km²



For each class:



**Class area (km²):**



`Number of class pixels × 0.0009`



**Class-area percentage:**



`(Number of pixels belonging to the class / Total valid pixels) × 100`



Invalid pixels with class ID 255 are excluded from the denominator.



## 11. Temporal Change Analysis



The prototype calculates two types of changes.



### A. Month-to-Month Change



For each class, the percentage-point change between consecutive months is:



`Change (%) = Current month percentage − Previous month percentage`



### B. First-to-Last Month Change



The overall change across the monitoring period is calculated as:



`Total change (%) = Final month percentage − First month percentage`



These values represent percentage-point differences, not relative percentage growth.



## 12. Outputs and Visualizations



The prototype produces:



- Twelve consecutive monthly LULC maps.

- Monthly class-area percentage tables.

- Month-to-month percentage-point change tables.

- First-to-last month percentage-point change summaries.

- Temporal visualizations of LULC class distributions.

- Metadata describing the input location and observation periods.



The generated results are stored in the `prototype\_outputs` directory.



## 13. Execution Requirements



The prototype requires:



- Python

- PyTorch

- Google Earth Engine Python API

- NumPy

- Pandas

- Matplotlib

- Rasterio

- Geospatial processing dependencies

- The trained ResNet34 U-Net model checkpoint



Google Earth Engine authentication and an appropriate Earth Engine project must be configured before execution.



The prototype notebook is located in this `notebooks` directory.



## 14. Important Limitations



The prototype demonstrates a monthly monitoring workflow, but several limitations should be considered.



**Temporal generalization:** The model was trained using 2021 Sentinel-1 observations and annual ESA WorldCover 2021 reference labels. Its predictions for other years may be affected by changes in land-cover conditions and acquisition characteristics.



**SAR classification ambiguity:** Different land-cover types can have similar radar backscatter characteristics, which can lead to classification errors.



**Seasonality:** Sentinel-1 backscatter can vary with vegetation conditions, soil moisture, rainfall and agricultural cycles.



**Rolling-window interpretation:** Each monthly prediction uses a rolling 12-month observation window. Therefore, consecutive predictions share most of their input observations.



**Fallback observations:** When a monthly observation is unavailable, nearby-month data may be used. Such outputs should be interpreted with caution.



**Change interpretation:** Changes in predicted class-area percentages do not necessarily represent confirmed real-world land-cover transitions. They may also result from model uncertainty, seasonal effects or differences in available observations.



The monthly predictions should therefore be interpreted as model-based monitoring estimates rather than independently validated monthly ground truth.



## 15. Project Structure



```text

LULC\_Project/

│

├── notebooks/

│   ├── prototype.ipynb

│   └── README.md

│

├── prototype\_outputs/

│   └── Generated monthly maps, tables and metadata

│

├── models/

│   └── Trained model files (if distributed separately)

│

└── results/

&#x20;   └── Model evaluation results and visualizations

```



## 16. Conclusion



This prototype demonstrates how Sentinel-1 SAR imagery and deep learning-based semantic segmentation can be combined to produce monthly LULC monitoring estimates.



By accepting only latitude, longitude and a start month, it provides a straightforward workflow for generating 12 consecutive monthly maps, calculating class-area percentages and examining temporal differences in land-cover predictions.

