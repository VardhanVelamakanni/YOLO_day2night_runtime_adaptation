<div align="center">

# Day-to-Night Runtime Adaptation

### Self-Healing Object Detection Through Online Domain Adaptation Without Retraining

<p>
  <img src="https://img.shields.io/badge/YOLO11-Ultralytics-blue"/>
  <img src="https://img.shields.io/badge/PyTorch-2.x-red"/>
  <img src="https://img.shields.io/badge/OpenVINO-Optimized-green"/>
  <img src="https://img.shields.io/badge/Test--Time-Adaptation-purple"/>
</p>


<img width="1862" height="500" alt="image" src="https://github.com/user-attachments/assets/5d4e5920-4185-43ba-ae18-815a86ad3586" />



</div>

---

## Project Objective

Object detectors trained on daytime driving data often experience severe performance degradation when deployed under unseen conditions such as **night-time scenes**, **low illumination**, and **domain shift**.

This project introduces a **runtime self-healing object detection framework** that continuously monitors model reliability and performs lightweight online recovery **without retraining the detector or requiring labeled target data**.

Unlike conventional domain adaptation methods that require additional optimization on new datasets, this framework adapts **during deployment**.

---

## Key Contribution

Instead of modifying YOLO through offline retraining, this framework performs **runtime adaptation** using:

- BatchNorm Re-estimation
- Test-Time Entropy Minimization (TENT)
- Runtime Failure Prediction
- Automatic Adaptation Policy
- Collapse Guard for safe rollback

The original source model remains intact while the deployed detector learns from incoming night-time data.

<p align="center">
  <img src="assets/pipeline.png" width="900"/>
</p>

---

## Methodology

The complete deployment pipeline consists of five stages.

<p align="center">
  <img src="assets/architecture.png" width="900"/>
</p>

### Stage 1 — Source Training

- YOLO11s
- BDD100K
- Daytime-only training
- Six driving classes

### Stage 2 — Runtime Monitoring

The detector continuously extracts deployment reliability signals:

- Detection confidence
- Prediction entropy
- Brightness statistics
- Flip consistency
- BatchNorm distribution drift

### Stage 3 — Failure Prediction

A calibrated logistic regression model estimates whether the detector is likely failing under the current domain.

### Stage 4 — Self-Healing

Depending on predicted failure probability, the policy controller automatically selects:

- Continue inference
- BatchNorm Re-estimation
- TENT adaptation

### Stage 5 — Safety Rollback

A collapse guard monitors adaptation quality and restores the original BatchNorm statistics whenever degradation is detected.

---

## Runtime Monitoring

Rather than adapting every frame blindly, the framework first evaluates deployment health using multiple reliability signals.

<p align="center">
  <img src="assets/monitoring_signals.png" width="850"/>
</p>

These signals allow the detector to distinguish between stable operation and domain-shift-induced degradation before adaptation begins.

---

## Self-Healing Results

The qualitative examples below compare the baseline daytime-trained detector against the runtime-adapted detector on unseen night-time scenes.

<p align="center">
  <img src="assets/before_after_grid.png" width="950"/>
</p>

The adapted detector consistently recovers additional vehicles and traffic infrastructure while preserving most baseline detections.

---

## Adaptation Policy

Instead of applying expensive adaptation continuously, the framework makes runtime decisions based on predicted failure probability.

<p align="center">
  <img src="assets/policy_controller.png" width="850"/>
</p>

| Failure Probability | Runtime Action |
|----------------------|----------------|
| Low | Continue inference |
| Medium | BatchNorm Re-estimation |
| High | TENT adaptation |
| Unsafe | Rollback |

---

## Project Workflow

The entire pipeline can be reproduced using two notebooks.

<p align="center">
  <img src="assets/notebooks.png" width="850"/>
</p>

| Notebook | Purpose |
|----------|---------|
| `01_YOLO11_BDD100K_Day_Training.ipynb` | Train YOLO11 on daytime BDD100K |
| `02_Self_Healing_Runtime_Adaptation.ipynb` | Runtime adaptation, monitoring, evaluation, and deployment |

---

## Repository Structure

```text
day2night_runtime_adaptation/
├── notebooks/
│   ├── 01_YOLO11_BDD100K_Day_Training.ipynb
│   └── 02_Self_Healing_Runtime_Adaptation.ipynb
│
├── src/
├── scripts/
├── configs/
├── assets/
├── results/
├── weights/
├── README.md
└── WRITEUP.md
```

---

## Getting Started

### Clone

```bash
git clone https://github.com/yourusername/day2night_runtime_adaptation.git
cd day2night_runtime_adaptation
```

### Install

```bash
pip install -r requirements.txt
```

### Train the Detector

```bash
jupyter notebook notebooks/01_YOLO11_BDD100K_Day_Training.ipynb
```

### Run Runtime Adaptation

```bash
jupyter notebook notebooks/02_Self_Healing_Runtime_Adaptation.ipynb
```

---

## Deployment

The trained detector supports multiple deployment formats.

<p align="center">
  <img src="assets/deployment.png" width="850"/>
</p>

| Format | Purpose |
|---------|---------|
| PyTorch | Training |
| ONNX | Cross-platform inference |
| OpenVINO | CPU-optimized deployment |

---

## Tech Stack

`Python` • `PyTorch` • `Ultralytics YOLO11` • `OpenCV` • `Scikit-learn` • `OpenVINO` • `ONNX`

---

## Applications

This framework can be extended beyond night-time driving to any deployment scenario involving domain shift, including:

- Autonomous Driving
- Traffic Monitoring
- Smart City Surveillance
- Edge AI Vision Systems
- Industrial Inspection
- Robotics

---

## Limitations

- Extreme glare can still produce false positives.
- BN Re-estimation improves robustness but cannot fully replace large-scale night training.
- TENT assumes BatchNorm-based architectures.

---

## Documentation

Detailed implementation notes are available in **WRITEUP.md**, including:

- System architecture
- Runtime monitoring
- BatchNorm Re-estimation
- TENT implementation
- Policy controller
- Collapse guard
- Design decisions
- Future improvements

---

## Citation

If this repository contributes to your work, please consider citing it.

```bibtex
@misc{day2nightruntimeadaptation2026,
  title={Day-to-Night Runtime Adaptation: Self-Healing Object Detection Through Online Domain Adaptation},
  author={Hemavardhan Velamakanni},
  year={2026},
  note={GitHub Repository}
}
```
