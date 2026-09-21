# Runtime Self-Healing Object Detection

## Technical Implementation Notes

This document describes the implementation decisions behind the runtime adaptation framework built on top of a daytime-trained YOLO11 detector.

Unlike the README, which provides a high-level overview, this document focuses on the system architecture, implemented components, experimental setup, and current limitations.

---

# Motivation

Object detectors trained on one domain often experience performance degradation when deployed under different conditions.

In driving scenarios, nighttime scenes introduce reduced illumination, lower contrast, and distribution shifts that can affect detector confidence and object localization.

Instead of retraining the detector on nighttime data, this project explores whether lightweight runtime adaptation techniques can improve deployment robustness while preserving the original model.

---

# Project Scope

The project consists of two major stages.

1. Training a daytime-only YOLO11 detector on BDD100K.
2. Building a runtime adaptation pipeline that monitors deployment quality and performs online recovery when appropriate.

The runtime system is designed as a prototype framework rather than a fully validated deployment solution.

---

# Source Detector

The source detector was trained using:

| Setting         | Value                 |
| --------------- | --------------------- |
| Model           | YOLO11s               |
| Dataset         | BDD100K               |
| Training Images | 18,000 daytime images |
| Classes         | 6                     |
| Resolution      | 768×768               |

The detector remains unchanged throughout the adaptation experiments.

---

# Runtime Pipeline

The deployment pipeline consists of five components.

1. Runtime Monitoring
2. Failure Prediction
3. BatchNorm Re-estimation
4. TENT Adaptation
5. Collapse Guard

Each component operates independently, allowing adaptation decisions without modifying the source checkpoint.

---

# Runtime Monitoring

Several deployment signals are extracted from incoming frames.

| Signal                | Purpose                                            |
| --------------------- | -------------------------------------------------- |
| Detection confidence  | Measures prediction certainty                      |
| Brightness statistics | Estimates illumination conditions                  |
| Prediction entropy    | Intended confidence dispersion signal              |
| Flip consistency      | Checks prediction stability under horizontal flips |
| BatchNorm statistics  | Intended distribution drift indicator              |

### Current Status

* Confidence and brightness measurements are implemented.
* Flip consistency is implemented but requires further validation.
* Entropy and BatchNorm drift remain prototype measurements and are not yet used as validated deployment indicators.

---

# BatchNorm Re-estimation

BatchNorm Re-estimation updates only the running statistics of BatchNorm layers during inference.

### Procedure

* Preserve source BatchNorm statistics.
* Enable BatchNorm statistic updates.
* Pass a buffer of target-domain images through the network.
* Restore evaluation mode afterward.

The objective is to adapt normalization statistics without changing learned convolution weights.

---

# TENT Adaptation

The repository includes an implementation of Test-Time Entropy Minimization (TENT).

The intended workflow is:

1. Freeze most network parameters.
2. Optimize BatchNorm affine parameters.
3. Minimize prediction entropy on unlabeled target images.

### Current Status

The implementation exists as a prototype and has not yet been quantitatively validated within the full deployment pipeline.

---

# Failure Prediction

A lightweight Logistic Regression model was added as a proof-of-concept policy component.

It receives runtime features such as confidence and brightness statistics to estimate whether adaptation should be triggered.

### Current Status

The predictor currently uses heuristic labels generated from runtime signals and should be viewed as a pipeline verification component rather than a validated deployment classifier.

---

# Policy Controller

The policy controller chooses an adaptation strategy based on estimated failure probability.

| Estimated Condition  | Action                  |
| -------------------- | ----------------------- |
| Stable               | Continue inference      |
| Moderate degradation | BatchNorm Re-estimation |
| Higher degradation   | TENT                    |
| Unsafe adaptation    | Rollback                |

This decision logic demonstrates how multiple adaptation strategies can be coordinated during deployment.

---

# Collapse Guard

A safety mechanism restores previously saved BatchNorm statistics when adaptation becomes unstable.

The current implementation focuses on preserving recoverability rather than optimizing adaptation quality.

---

# Experimental Analysis

Several exploratory experiments were performed to inspect detector behavior.

## Corruption Experiments

The detector was evaluated under synthetic corruptions including:

* Gaussian noise
* Brightness reduction
* Fog
* Rain
* Motion blur

These experiments measure changes in detection count and average confidence.

They should not be interpreted as mAP benchmarks.

---

## Preprocessing Experiments

Three preprocessing techniques were compared.

* CLAHE
* Gamma correction
* Denoising

The observed changes were generally small and primarily served as exploratory comparisons rather than evidence that preprocessing solves nighttime domain shift.

---

## Qualitative Adaptation Examples

The repository includes selected nighttime examples showing changes before and after BatchNorm Re-estimation.

These examples demonstrate visible behavioral differences but are not a substitute for ground-truth nighttime evaluation.

---

# Repository Components

## Notebook 1

`01_YOLO11_BDD100K_Day_Training.ipynb`

Contains:

* Dataset preparation
* Training configuration
* Validation
* Model export

## Notebook 2

`02_Self_Healing_Runtime_Adaptation.ipynb`

Contains:

* Runtime monitoring
* BatchNorm Re-estimation
* TENT prototype
* Failure prediction
* Policy controller
* Qualitative evaluation

---

# Current Status

| Component                             | Status      |
| ------------------------------------- | ----------- |
| Daytime YOLO11 training               | Complete    |
| BatchNorm Re-estimation               | Implemented |
| Runtime monitoring                    | Implemented |
| Qualitative nighttime evaluation      | Complete    |
| Failure predictor                     | Prototype   |
| Policy controller                     | Prototype   |
| TENT integration                      | Prototype   |
| Quantitative nighttime mAP evaluation | Pending     |

---

# Limitations

Several components remain exploratory.

* Nighttime improvements have not yet been validated using ground-truth nighttime mAP.
* The failure predictor currently relies on heuristic labels.
* Flip consistency and BatchNorm drift require additional validation.
* TENT integration requires further evaluation under real deployment scenarios.
* Severe glare and extremely dark scenes can still produce false positives.

---

# Future Work

Possible extensions include:

* Quantitative night/rain mAP evaluation
* Real online streaming adaptation
* Stronger drift estimation methods
* Temporal consistency monitoring
* Adaptive confidence thresholding
* Multi-domain deployment across additional weather conditions

---

# Key Takeaway

This project demonstrates a practical runtime adaptation framework built around a daytime-trained YOLO11 detector.

The repository emphasizes modular deployment components—including BatchNorm Re-estimation, runtime monitoring, policy control, and safe rollback—while clearly distinguishing implemented features from components that remain prototypes or require further quantitative validation.
