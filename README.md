<div align="center">

# SafeMed-SSL
### Uncertainty-Gated Semi-Supervised Learning for Safer Medical AI

[![PyTorch](https://img.shields.io/badge/PyTorch-Implementation-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white)](https://pytorch.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)
[![Preprint](https://img.shields.io/badge/Preprint-bioRxiv-blue?style=for-the-badge)](https://doi.org/10.64898/2026.10.13.758712)

**Can a medical AI model learn from unlabelled images without becoming confidently wrong?**

SafeMed-SSL explores this question through uncertainty-gated pseudo-labeling for semi-supervised malaria cell classification.

📄 [Read the paper on bioRxiv](https://doi.org/10.64898/2026.10.13.758712) · 💻 [Explore the code](https://github.com/HaloMind-Research/SafeMed-SSL)

</div>

---

## About the Project

Medical image annotation takes time, expertise, and resources. In settings where trained specialists are limited, collecting large amounts of labelled data can be particularly challenging.

Semi-supervised learning (SSL) offers one way to address this problem: train a model using a small labelled dataset alongside a larger collection of unlabelled images.

There is a catch, though. Methods such as FixMatch use confidence thresholds to decide which predictions become pseudo-labels. When a model is confidently wrong, it can teach itself from an incorrect prediction.

We study a simple but important question: **can uncertainty estimation help a model avoid learning from unreliable pseudo-labels?**

Our approach combines Monte Carlo dropout, predictive entropy and confidence-weighted pseudo-labeling. The goal is to reduce high-confidence classification errors while retaining the data-efficiency benefits of semi-supervised learning.

This repository accompanies our bioRxiv preprint on uncertainty-gated semi-supervised learning for malaria screening.

## How It Works

The proposed method uses two complementary checks before an unlabelled image contributes to the unsupervised training loss.

1. **Estimate uncertainty.** We keep dropout active and obtain 10 stochastic predictions for an unlabelled image. Normalized predictive entropy is used to estimate uncertainty in the averaged prediction.

2. **Filter unreliable predictions.** A pseudo-label is excluded when normalized predictive entropy is at least 0.2.

3. **Weight accepted pseudo-labels.** A sigmoid-based confidence weighting scheme gives different weights to accepted predictions according to model confidence.

4. **Learn from both data sources.** The model uses ordinary supervised cross-entropy on labelled images and an uncertainty-weighted consistency loss on unlabelled images.

The experiments use an ImageNet-pretrained ResNet-18 backbone and weak/strong image augmentations.

The aim is not simply to maximize classification accuracy. It is to investigate whether uncertainty-aware training can reduce errors that look trustworthy because the model assigns them high confidence.

## Framework

<div align="center">
  <img src="final_architecture.drawio.png"
       alt="SafeMed-SSL uncertainty-gated semi-supervised learning framework showing labelled and unlabelled data paths, Monte Carlo dropout, entropy filtering, and confidence-weighted pseudo-labeling"
       width="900"/>
  <p><em>Overview of the proposed training framework and its uncertainty-based pseudo-label filtering mechanism.</em></p>
</div>

## Experimental Setup

We evaluate the method on the public Malaria Cell Images dataset.

| Setting | Details |
|---|---|
| Task | Binary malaria cell classification |
| Dataset | Malaria Cell Images |
| Total images | 27,558 |
| Parasitized images | 13,779 |
| Uninfected images | 13,779 |
| Input resolution | 128 × 128 pixels |
| Data split | 70% training, 10% validation, 20% test |
| Labelled portion | 20% of the training partition |
| Unlabelled portion | Remaining 80% of the training partition |
| Backbone | ImageNet-pretrained ResNet-18 |
| Uncertainty estimation | 10 Monte Carlo dropout passes |
| Independent runs | 6 random seeds |

The comparison includes a supervised baseline trained with the labelled subset, FixMatch with confidence thresholds of 0.95 and 0.80, and the proposed uncertainty-gated method.

### Training Configuration

The reported experiments use SGD with momentum 0.9, weight decay of 0.0005, an initial learning rate of 0.03 with cosine annealing, and 50 training epochs.

For uncertainty-guided pseudo-labeling, the main settings are:

- Dropout probability: 0.3
- Monte Carlo samples: 10
- Normalized predictive entropy threshold: 0.2
- Minimum confidence parameter: 0.7
- Sigmoid weighting parameter: 5.0
- Unsupervised loss weight: 1.0

See `FINAL.ipynb` for the notebook and experiment implementation.

## Results

Our primary comparison is against FixMatch with a confidence threshold of 0.80. This relaxed threshold provides a useful comparison because accepting more pseudo-labels can also introduce more confident mistakes.

| Method | Accuracy | AUC | Silent Failure Rate |
|---|---:|---:|---:|
| Supervised baseline (20% labels) | 96.7 ± 0.5% | 0.989 ± 0.004 | 2.01 ± 0.42% |
| FixMatch (threshold = 0.95) | 96.8 ± 0.4% | 0.992 ± 0.003 | 1.97 ± 0.38% |
| FixMatch (threshold = 0.80) | 96.4 ± 0.5% | 0.991 ± 0.003 | 2.28 ± 0.47% |
| **Ours: uncertainty-gated** | **96.6 ± 0.3%** | **0.991 ± 0.002** | **1.52 ± 0.61%** |

*Values are reported as mean ± standard deviation across six independent runs.*

### The Main Finding

Compared with FixMatch at a threshold of 0.80, our method reduced the mean silent failure rate from 2.28% to 1.52%.

That is a **33.2% relative reduction**, with a paired t-test reporting *t* = 4.29 and *p* = 0.0078 across the six runs.

The proposed method also achieved a lower silent failure rate than the tuned FixMatch baseline in each of the six runs.

Importantly, overall accuracy and AUC remained broadly comparable. The main improvement was in reducing confidently incorrect predictions, rather than producing a large increase in conventional classification metrics.

### What Is a Silent Failure?

In this study, a silent failure is a misclassification made with prediction confidence greater than 0.95.

Accuracy and AUC are useful measures of overall predictive performance, but they do not by themselves show how often a model makes an incorrect prediction with high confidence. We therefore evaluate silent failure rate alongside these conventional metrics.

<div align="center">
  <img src="FINAL_PAPER_GRAPH_N6.jpeg"
       alt="Per-seed silent failure rates comparing FixMatch at training threshold 0.80 with the proposed uncertainty-gated method"
       width="750"/>
  <p><em>Silent failure rates across the six random seeds. The proposed method has a lower rate than the tuned FixMatch baseline in all six runs.</em></p>
</div>

## Dataset

We use the publicly available [Malaria Cell Images dataset on Kaggle](https://www.kaggle.com/datasets/iarunava/cell-images-for-detecting-malaria).

The dataset contains 27,558 cell images, split evenly between parasitized and uninfected cells.

Download the dataset from its source and configure the dataset paths in the notebook to match your local directory structure.

Please review the dataset's original terms and attribution requirements before reusing it.

### An Important Evaluation Limitation

The experiments split the data at the individual cell-image level, rather than at the patient level. Because multiple cell images may originate from the same patient or blood smear, images from the same source may occur in more than one partition.

This means the absolute performance estimates may be optimistic. Although all methods were compared under the same split protocol, patient-level evaluation is needed to establish how well the results generalize to unseen patients.

## Running the Notebook

The main experiment notebook in this repository is `FINAL.ipynb`.

Clone the repository:

```bash
git clone https://github.com/HaloMind-Research/SafeMed-SSL.git
cd SafeMed-SSL
