# Bearing Health Monitoring and Early Degradation Detection using Vibration Data

## Overview
This project builds an interpretable condition-monitoring pipeline on the IMS / NASA bearing run-to-failure dataset.

The goal is to detect when a bearing begins to show **sustained abnormal behavior** relative to an early healthy baseline. The project does **not** attempt to predict exact failure time or remaining useful life. Instead, it focuses on **early degradation detection** using vibration-based features and unsupervised anomaly detection.

## Why this project matters
In industrial systems, bearings rarely fail without warning. Their vibration behavior often changes before full failure occurs. Detecting that change early is valuable for:
- predictive maintenance
- condition monitoring
- reducing downtime
- preventing unexpected equipment failure

This project shows how vibration snapshots can be converted into interpretable numerical features and monitored over time to identify the onset of degradation.

## Dataset
This project uses the **IMS / NASA bearing run-to-failure dataset**.

### Data representation
A bearing lifetime is **not stored in one file**.  
Instead, the lifetime is represented by **many files in chronological order**.

Each file is one short vibration measurement snapshot with shape:

`(20480, 8)`

That means:
- **20480 rows** = time samples within that snapshot
- **8 columns** = simultaneous sensor channels

In this project:
- each file is treated as **one time point**
- one sensor channel is selected and analyzed consistently across all files
- the final channel used is **channel 1**

## Project pipeline
The final notebook follows this logic:

1. Load vibration files in chronological order
2. Select one sensor channel
3. Preprocess the signal by:
   - removing linear trend
   - centering around zero
4. Extract scalar time-domain and frequency-domain features from each file
5. Compare early-life and late-life feature behavior
6. Select the strongest degradation-sensitive features
7. Define the first 10% of files as an approximate healthy baseline
8. Fit a feature scaler on the healthy baseline only
9. Train an Isolation Forest on healthy baseline feature vectors
10. Score all files by deviation from the healthy baseline
11. Detect the onset of sustained abnormal behavior using a healthy-based threshold

## Preprocessing
The raw vibration signal is:
- detrended
- centered around zero

Importantly, the raw signal is **not normalized to unit variance**. This preserves amplitude-related information, which can be useful for tracking degradation.

## Feature extraction
Each file is converted into one scalar feature vector.

The final feature set used for the anomaly model is:

- **RMS**
- **Peak-to-peak amplitude**
- **3–5 kHz spectral band energy**
- **5–10 kHz spectral band energy**

These were chosen because they showed the clearest early-to-late change over the bearing lifetime.

## Modeling approach
The first 10% of files are treated as an approximate **healthy baseline**.

A `StandardScaler` is fit on the healthy baseline only, then applied to all files. This avoids contaminating the definition of healthy behavior with degraded measurements.

An **Isolation Forest** is then trained on the healthy baseline feature vectors.

### What the model learns
The model does **not** learn bearing physics or remaining useful life.

It learns what the **healthy feature-space distribution** looks like, then measures how much later files deviate from that healthy region.

## Deviation score
The model output is converted into a **deviation score** such that:

- **lower score** = more similar to healthy baseline
- **higher score** = more abnormal relative to healthy baseline

A rolling mean is used only for visualization and onset interpretation.

## Final result
Using early-life measurements as a healthy reference, the model detects a **sustained increase in deviation around file 243**, which is interpreted as the **onset of measurable bearing degradation**.

This means the bearing begins to behave persistently differently from its healthy state at that point.

## Interpretation
This result should be interpreted as:

- **degradation onset**
- **sustained abnormal deviation from healthy behavior**

It should **not** be interpreted as:

- exact failure time
- exact fault diagnosis
- remaining useful life prediction
- a supervised classification result

## Main outputs
The most important outputs of the notebook are:

1. **Raw vs preprocessed signal comparison**
2. **Feature trends over file index**
3. **Isolation Forest deviation trend**
4. **Healthy-based threshold and detected degradation onset**

## Tech stack
- Python
- NumPy
- Pandas
- SciPy
- scikit-learn
- Matplotlib
- Jupyter Notebook

## Repository structure
Example structure:

```text
project/
│
├── data/
│   └── raw/
│       └── 1st_test/
│
├── notebooks/
│   └── bearing_health_monitoring_early_degradation_detection.ipynb
│
└── README.md
