# AROMA: Adaptive ROI Modeling via Complexity-Aware Morphology and Context Analysis

## 1. Introduction

Synthetic defect generation has become an important strategy for industrial anomaly detection, where defect samples are often scarce and highly imbalanced.

Recent approaches such as CASDA have demonstrated that selecting appropriate Regions of Interest (ROIs) significantly influences the quality of synthetic defect generation. By considering defect morphology and background characteristics, CASDA improves the realism of generated anomalies.

However, CASDA relies on manually engineered ROI design:

* Handcrafted morphology categories
* Handcrafted background categories
* Fixed threshold rules
* Expert-defined compatibility matrices
* Dataset-specific parameter tuning

As a result, the framework must be reconfigured whenever a new industrial dataset is introduced.

This limitation becomes increasingly problematic when moving across diverse domains:

* Steel surface inspection
* Semiconductor wafer inspection
* PCB inspection
* Manufacturing quality inspection
* Industrial texture inspection

To overcome these limitations, we propose AROMA (Adaptive ROI Modeling via Complexity-Aware Morphology and Context Analysis), a framework that automatically learns ROI modeling policies from dataset statistics rather than relying on manually designed rules.

---

# 2. Core Research Question

Traditional ROI selection approaches ask:

> Which ROI should be selected?

AROMA instead asks:

> How should ROI modeling strategies be configured according to dataset complexity?

The goal is not simply selecting ROI candidates but learning the most appropriate ROI modeling strategy for a given dataset.

---

# 3. Conceptual Shift

## CASDA

```text
Rule-Based ROI Engineering

Dataset
 ↓
Handcrafted Morphology Categories
 ↓
Handcrafted Context Categories
 ↓
Manual Compatibility Matrix
 ↓
ROI Selection
```

---

## AROMA

```text
Data-Driven ROI Policy Learning

Dataset
 ↓
Complexity Analysis
 ↓
ROI Modeling Policy Generator
 ↓
Morphology Modeling
 +
Context Modeling
 +
Morphology-Context Prior Learning
 ↓
ROI Selection
 ↓
Prompt Generation
 ↓
Synthetic Defect Generation
```

---

# 4. Research Hypothesis

We hypothesize that:

1. Different datasets exhibit significantly different morphology and context distributions.

2. A fixed ROI modeling strategy is suboptimal across domains.

3. Dataset complexity can be quantified and used to automatically determine ROI modeling strategies.

4. Adaptive ROI modeling improves ROI quality, synthetic defect quality, and downstream anomaly detection performance.

---

# 5. Overall Framework

```text
Dataset
    ↓
Complexity Analysis
    ↓
ROI Modeling Policy Generator
    ↓
Morphology Modeling
    ↓
Context Modeling
    ↓
Morphology-Context Prior Learning
    ↓
ROI Selection
    ↓
Semantic Prompt Generation
    ↓
Synthetic Defect Generation
```

The central contribution is the ROI Modeling Policy Generator.

---

# 6. Complexity Analysis

The first stage characterizes the statistical complexity of the dataset.

---

## 6.1 Morphology Complexity Analysis

### Input Features

* Aspect Ratio
* Log Aspect Ratio
* Circularity
* Solidity
* Extent
* Eccentricity
* Fill Ratio

### Distribution Analysis

For each feature:

* Unimodal Detection
* Bimodal Detection
* Multimodal Detection
* Valley Count
* Entropy
* Skewness
* Kurtosis

### Morphology Complexity Index (MCI)

```text
MCI =
α·Entropy
+
β·Valley Count
+
γ·Distribution Complexity
+
δ·Cluster Separability
```

### Interpretation

Low MCI:

* Few morphology types
* Well-separated clusters

High MCI:

* Diverse defect structures
* Complex morphology distributions

---

## 6.2 Context Complexity Analysis

### Input Features

* Local Variance
* Texture Entropy
* Edge Density
* Frequency Energy
* Orientation Consistency

### Distribution Analysis

* Cluster Diversity
* Entropy
* Spatial Complexity
* Frequency Complexity

### Context Complexity Index (CCI)

```text
CCI =
α·Texture Entropy
+
β·Cluster Diversity
+
γ·Frequency Complexity
+
δ·Orientation Variance
```

### Interpretation

Low CCI:

* Homogeneous backgrounds

High CCI:

* Diverse texture environments

---

# 7. ROI Modeling Policy Generator

This stage represents the primary novelty of AROMA.

The framework determines how ROI modeling should be performed according to dataset complexity.

---

## Example Policies

### Unimodal Distribution

```text
Percentile Partition
```

### Bimodal Distribution

```text
Otsu Thresholding
```

### Multimodal Distribution

```text
Gaussian Mixture Modeling
```

### High Complexity Dataset

```text
Hierarchical Clustering
```

### Heavy-Tailed Distribution

```text
Log Transform
+
GMM
```

---

## Policy Outputs

The policy generator determines:

* Clustering algorithm
* Number of clusters
* Thresholding strategy
* Sampling strategy
* Prior learning strategy

AROMA therefore learns ROI modeling policies rather than fixed ROI rules.

---

# 8. Morphology Modeling

Morphology clusters are automatically learned.

### Inputs

* Log Aspect Ratio
* Circularity
* Solidity
* Eccentricity
* Extent

### Candidate Algorithms

* GMM
* K-Means
* Hierarchical Clustering

### Outputs

Example:

```text
M1: Compact Defect

M2: Elongated Defect

M3: Irregular Defect

M4: Diffuse Contamination

M5: Structural Defect
```

Unlike CASDA, cluster count is automatically determined.

---

# 9. Context Modeling

Background environments are modeled using domain-independent descriptors.

### Inputs

* Local Variance
* Texture Entropy
* Frequency Energy
* Orientation Consistency

### Outputs

```text
C1: Smooth Surface

C2: Directional Texture

C3: Repetitive Pattern

C4: High Frequency Texture

C5: Complex Background
```

Context groups emerge automatically from data.

---

# 10. Morphology-Context Prior Learning

Instead of manually constructing compatibility matrices, AROMA learns realistic morphology-context relationships.

---

## Objective

Learn:

```text
P(Morphology, Context)
```

---

## Example

| Morphology | Context | Probability |
| ---------- | ------- | ----------- |
| M1         | C1      | 0.31        |
| M1         | C2      | 0.04        |
| M2         | C1      | 0.08        |
| M2         | C3      | 0.22        |

---

## Output

Morphology-Context Prior

This prior represents realistic morphology-background combinations observed in real data.

---

# 11. ROI Selection

ROI candidates are scored using:

```text
Morphology Cluster
+
Context Cluster
+
Morphology-Context Prior
```

---

## ROI Score

```text
ROI_score =
f(Morphology, Context, Prior)
```

---

## Sampling Strategies

### Top-K

Highest scoring ROIs

### Weighted Sampling

Probability-based sampling

### Deficit-Aware Sampling

Oversampling rare morphology-context combinations

---

# 12. Semantic Prompt Generation

AROMA extends ROI modeling into prompt generation.

---

## Morphology Descriptor

Example:

```text
Highly elongated metallic scratch
```

---

## Context Descriptor

Example:

```text
Directional brushed metal texture
```

---

## Prior-Aware Modifier

Example:

```text
Aligned with dominant texture direction
```

---

## Generated Prompt

```text
An elongated metallic scratch aligned with the dominant brushed surface texture.
```

This allows domain-independent ControlNet prompt generation.

---

# 13. Synthetic Defect Generation

Generated ROIs and prompts can be used in:

* Copy-Paste Synthesis
* Diffusion-Based Synthesis
* Inpainting
* ControlNet Training
* Latent Diffusion Models

AROMA is independent of any specific synthesis framework.

---

# 14. Experimental Design

A key design goal is minimizing computational cost while directly validating the proposed ROI modeling framework.

---

## Experiment 1: ROI Distribution Modeling

### Objective

Evaluate how accurately ROI distributions are modeled.

### Comparison

* CASDA
* AROMA

### Metrics

* KL Divergence
* Wasserstein Distance
* Earth Mover Distance

### Purpose

Validate Morphology-Context Prior Learning.

---

## Experiment 2: ROI Quality Analysis

### Objective

Evaluate ROI candidate quality.

### Metrics

* Morphology Coverage
* Context Coverage
* Rare Defect Coverage
* Compatibility Consistency

### Purpose

Validate ROI Modeling Policy Learning.

---

## Experiment 3: Synthetic Quality Evaluation

### Dataset

Single representative dataset (e.g., Severstal)

### Methods

* Random ROI
* CASDA ROI
* AROMA ROI

### Generator

Fixed ControlNet Pipeline

### Metrics

* FID
* KID
* LPIPS

### Purpose

Validate synthesis improvement while minimizing GPU cost.

---

## Experiment 4: Downstream Anomaly Detection

### Training

Real Data + Synthetic Data

### Models

* PatchCore
* SimpleNet
* EfficientAD
* RD++

### Metrics

* Image AUROC
* Pixel AUROC
* PRO

### Purpose

Evaluate practical utility of generated defects.

---

## Experiment 5: Cross-Domain Generalization

### Datasets

* Severstal
* MVTec AD
* VisA
* PCB Inspection

### Evaluation

Policy adaptation behavior

Example:

| Dataset   | MCI    | CCI    | Selected Policy         |
| --------- | ------ | ------ | ----------------------- |
| Severstal | Low    | Low    | Otsu                    |
| MVTec     | Medium | Medium | GMM                     |
| PCB       | High   | High   | Hierarchical            |
| VisA      | High   | Medium | GMM + Adaptive Sampling |

### Purpose

Demonstrate adaptive ROI policy generation.

No additional ControlNet retraining required.

---

# 15. Expected Contributions

## Contribution 1

Complexity-aware dataset characterization through MCI and CCI.

## Contribution 2

Adaptive ROI Modeling Policy Learning.

## Contribution 3

Automatic morphology representation learning.

## Contribution 4

Automatic context representation learning.

## Contribution 5

Data-driven Morphology-Context Prior Learning.

## Contribution 6

Semantic Prompt Generation.

## Contribution 7

Cross-domain ROI generalization.

## Contribution 8

Computationally efficient evaluation framework.

---

# Final Definition

AROMA is a Complexity-Aware ROI Modeling Framework that automatically learns how morphology modeling, context modeling, prior learning, and ROI selection should be configured according to the statistical characteristics of a target dataset.

Unlike CASDA, which performs rule-based ROI engineering, AROMA performs data-driven ROI policy learning, enabling scalable and domain-adaptive synthetic defect generation across diverse industrial inspection domains.
