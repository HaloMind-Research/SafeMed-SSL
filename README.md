<div align="center">

# SafeMed-SSL: Uncertainty-Guided Semi-Supervised Learning for Safe Medical Image Classification

[![Framework](https://img.shields.io/badge/PyTorch-2.0-orange?style=for-the-badge&logo=pytorch)](https://pytorch.org/)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

*An official PyTorch implementation for robust and safe medical image classification in low-resource settings.*

</div>

---

## Problem Statement
In resource-constrained healthcare environments, expert medical image annotation is expensive and time-consuming. While Semi-Supervised Learning (SSL) reduces this burden, standard SSL methods (like FixMatch) suffer from **Silent Failures**—classifying ambiguous images incorrectly but with high confidence.

**Our Solution:** We introduce an **Uncertainty-Guided Safety Filter** that leverages Monte Carlo Dropout. By rejecting ambiguous pseudo-labels based on both confidence and uncertainty, we significantly reduce overconfident clinical errors.

---

##  Key Contributions
- ** Dual-Path Architecture:** Seamlessly integrates standard Supervised Learning (20% data) with an Uncertainty-Guided SSL teacher-student model (80% unlabeled data).
- ** Two-Layer Safety Mechanism:** Filters unreliable predictions using a strict criteria.
- ** Low-Resource Optimized:** Built on a lightweight ResNet-18 backbone, making it highly suitable for edge deployment in rural clinics.

<div align="center">
  <img src="final_architecture.drawio.png" alt="SafeMed-SSL Architecture Diagram" width="800"/>
  <br>
  <em>Figure 1: The SafeMed-SSL Framework. Unlabeled data passes through a safety filter before pseudo-labels are generated.</em>
</div>

---

## Results & Clinical Impact

We evaluated our framework on the task of Malaria Cell Classification. Our method outperforms standard SSL baselines by specifically targeting dangerous "silent failures."

| Method | Accuracy | AUC | Silent Failure Rate (SFR) |
| :--- | :---: | :---: | :---: |
| Supervised Baseline (20% Labels) | 96.7% | 0.989 | 2.01% |
| FixMatch (Standard SSL, τ=0.80) | 96.4% | 0.991 | 2.50% |
| **Ours (SafeMed-SSL)** | **96.6%** | **0.991** | **1.77%** |

> **Impact:** We achieved a statistically significant **29.2% reduction in silent failures** ($p=0.0137$), directly preventing dangerous misdiagnoses.

<div align="center">
  <img src="FINAL_PAPER_GRAPH_N6.jpeg" alt="Clinical Robustness Graph" width="600"/>
  <br>
  <em>Figure 2: Clinical Robustness. SafeMed-SSL (blue) maintains a consistently low failure rate across varying thresholds compared to standard methods (red).</em>
</div>

---

## Dataset
This project uses the **NIH Malaria Cell Images Dataset**.
* **Total Images:** 27,558 (13,779 Parasitized / 13,779 Uninfected)
* **Source:** [Download from Kaggle](https://www.kaggle.com/datasets/iarunava/cell-images-for-detecting-malaria)

**Data Structure:**
```
data/
└── malaria/
    ├── Parasitized/
    └── Uninfected/
```

**Quick Start:**
1. Installation
Clone the repository and install the required dependencies:
```
git clone [https://github.com/Latchan-Ch/SafeMed-SSL.git](https://github.com/Latchan-Ch/SafeMed-SSL.git)
cd SafeMed-SSL
```
2. Training the Model
3. Evaluate and Use

 **Authors:**
 
Latchan Chhetri - Dept. of AI & DS, Sikkim Manipal Institute of Technology (SMIT)

Aman Kumar - Dept. of AI & DS, SMIT

 **Citation:**
 
If you find this code or our methodology useful in your research, please consider citing our paper:

```
Code snippet
@inproceedings{chhetri2026safemed,
  title={Uncertainty-Guided Semi-Supervised Learning for Safe Medical Image Classification in Low-Resource Settings},
  author={Chhetri, Latchan and Kumar, Aman and Das, Hrishikesh},
  booktitle={2026 International Conference on Sustainable AI for Social Impact and Global Development (SASIGD)},
  year={2026},
  organization={IEEE}
}
```
