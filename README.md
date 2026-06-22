# GNSS Multipath Classification using Range Residuals and Random Forest

A signal-processing case study on real GPS L1 observation data that detects and classifies satellite signals into **Line-of-Sight (LOS)**, **Multipath**, and **Non-Line-of-Sight (NLOS)** conditions using range residual magnitude, then trains a **Random Forest classifier** to learn the same classification from signal-quality features alone.

## Motivation

GNSS receivers suffer significant positioning error when satellite signals reflect off buildings, terrain, or other surfaces before reaching the receiver (multipath), or are blocked entirely (NLOS). Detecting these conditions is a core problem in:

- Aircraft and UAV navigation validation
- GPS / GNSS signal quality monitoring
- Autonomous vehicle localization
- Geodetic and survey-grade receivers

This project analyzes real GPS L1 satellite observation data, derives physics-based labels from range residual error, and trains a classifier to predict signal condition from elevation angle, signal strength, and Doppler — features available in real time, without needing the residual itself.

## Method

### 1. Exploratory signal analysis
- **C/N0 vs elevation** — verifies that signal strength increases with satellite elevation, as expected from GNSS physics (lower elevation means longer atmospheric path and more reflections).
- **C/N0 over time** — tracks signal strength stability and identifies drops associated with loss of lock or obstruction.
- **Range residual vs elevation** — shows how positioning error grows at low elevation angles.
- **Satellite skyplot** — visualizes satellite azimuth/elevation geometry over the observation period.

### 2. Physics-based labeling
Each observation is labeled directly from its range residual magnitude:

| Label | Condition |
|---|---|
| **LOS** | Range Residual under 0.3 m |
| **Multipath** | Range Residual between 0.3 m and 0.8 m |
| **NLOS** | Range Residual 0.8 m or greater |

Range residual error grows with multipath reflection and signal degradation, so thresholding on its magnitude gives a physically grounded label without manual annotation.

### 3. Classification
A **Random Forest classifier (200 estimators)** is trained on:
- Elevation angle
- C/N0 (carrier-to-noise ratio)
- Doppler shift

These are features a receiver can observe directly, so the trained model can flag likely multipath/NLOS conditions in real time, without needing the range residual, which is often only available after positioning is already computed.

Train/test split: 70/30, stratified by class, random_state 42.

## Results

Running this pipeline on GPS L1 observation data (SATB_L1.xlsx) produced:

- **C/N0 vs elevation**: clear positive correlation, confirming signal strength degrades at low elevation, consistent with GNSS theory.
- **Classification performance**: see the full classification report (precision/recall/F1 per class) printed by the script, and the confusion matrix below.
- **Feature importance**: printed per feature, showing which of elevation, C/N0, and Doppler contributes most to distinguishing LOS from multipath/NLOS.

![Signal Strength vs Elevation](results/cn0_vs_elevation.png)

![Residual Error vs Elevation](results/residual_vs_elevation.png)

![Satellite Skyplot](results/skyplot.png)

![Confusion Matrix](results/confusion_matrix.png)

## Repository structure

```
gnss-multipath-classification/
├── data/                       Sample GPS L1 observation data (SATB_L1.xlsx)
├── notebooks/
│   └── gnss_multipath_analys.ipynb   Original exploratory notebook
├── src/
│   └── ccd_classifier.py             Cleaned, modular pipeline
├── results/                    Generated plots (saved automatically when src script is run)
├── requirements.txt
└── README.md
```

## Running it

```
pip install -r requirements.txt
python src/ccd_classifier.py
```

This loads the data, generates all exploratory plots, builds physics-based labels, trains the Random Forest model, prints the classification report, and saves all plots including the confusion matrix to results/.

## Tech stack

Python, NumPy, Pandas, Scikit-learn, Matplotlib, Seaborn

## Background

Built as part of independent GNSS signal processing research, applying real receiver observation data to a practical signal-quality classification problem relevant to navigation and positioning systems.
