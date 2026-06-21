# GNSS Multipath Classification using Code-Carrier Divergence and Machine Learning

A signal-processing case study on real GPS L1 observation data that detects and classifies satellite signals into **Line-of-Sight (LOS)**, **Multipath**, and **Non-Line-of-Sight (NLOS)** conditions using the **Code-Carrier Divergence (CCD)** method, followed by a physics-informed Random Forest classifier.

## Motivation

GNSS receivers suffer significant positioning error when satellite signals reflect off buildings, terrain, or other surfaces before reaching the receiver (multipath), or are blocked entirely (NLOS). Detecting these conditions in real time is a core problem in:

- Aircraft and UAV navigation validation
- ISRO / GPS / Galileo signal quality monitoring
- Autonomous vehicle localization
- Geodetic and survey-grade receivers

This project implements the **Code-Carrier Divergence** technique — the standard approach used in GNSS research — to detect multipath from raw pseudorange and carrier phase measurements, then uses those physics-derived labels to train a supervised classifier.

## Method

GNSS receivers report two measurements of distance to a satellite:

| Measurement | Behaviour under multipath |
|---|---|
| **Pseudorange (code)** | Noisy, directly corrupted by reflected signal paths |
| **Carrier phase** | Precise, almost unaffected by reflections |

When a signal is clean (LOS), both measurements agree. When a signal reflects or is obstructed, they diverge. This divergence is the multipath observable:

```
MP = Pseudorange − (λ × Carrier Phase)
```

where λ is the GPS L1 wavelength (≈ 0.1903 m, derived from f_L1 = 1575.42 MHz).

### Pipeline

1. **Compute the multipath observable** (`MP`) for every epoch from raw pseudorange and carrier delay.
2. **Compute rolling RMS** of `MP` (`MP_RMS`) over a 50-sample window to quantify multipath strength rather than relying on a single noisy sample.
3. **Derive physics-based labels** directly from signal behaviour, rather than guessing:
   - `LOS`: `MP_RMS < 0.5 m` and elevation `> 35°`
   - `Multipath`: `MP_RMS < 2 m`
   - `NLOS`: `MP_RMS ≥ 2 m`
4. **Train a Random Forest classifier** (200 estimators) on elevation angle, C/N₀, `MP_RMS`, multipath observable, and range residual.
5. **Visualize** the classification against elevation and C/N₀ to verify the model recovers known GNSS signal behaviour.

## Results

Running this pipeline on satellite observation data (`SATB_1.xlsx`, GPS L1) produced:

- **C/N₀ stability**: ~43–46 dB-Hz under clean LOS tracking, with sharp drops to near-zero corresponding to loss of satellite lock (NLOS entry).
- **Residual error by condition**:

  | Signal condition | Residual range |
  |---|---|
  | LOS | −0.5 m to +0.5 m |
  | Multipath | +0.5 m to +3 m |
  | Severe multipath / NLOS | +3 m to +9 m |

- **Elevation dependence**: signal quality and residual error both degrade sharply at low elevation angles, consistent with increased ground/building reflections and atmospheric path length near the horizon — matching expected GNSS physics.

See [`results/`](results/) for the generated plots.

## Repository structure

```
gnss-multipath-classification/
├── data/                 Sample GPS L1 observation data
├── notebooks/            Exploratory analysis
├── src/                  Core CCD + classifier pipeline
├── results/              Output plots (CCD scatter, time series, classification map)
├── requirements.txt
└── README.md
```

## Running it

```bash
pip install -r requirements.txt
python src/ccd_classifier.py
```

## Tech stack

Python · NumPy · Pandas · Scikit-learn · Matplotlib

## Background

Built as part of independent GNSS signal processing research, with the goal of applying physics-based methods used in operational GNSS research rather than relying on naive elevation-only thresholds.
