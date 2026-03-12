# Interpot-Sustainability-Audit
## Overview

This repository contains a sustainability audit of the biological interpretability project **InterProt**, which investigates how protein language models (pLMs) represent biological features.

The goal of this audit is to **measure and analyze the environmental impact of a key computational stage in the InterProt workflow**, and to propose improvements that could reduce its carbon footprint.

The audit focuses on the stage where protein sequences are processed using a large protein language model to generate internal features, which are then used to train a simple classifier to evaluate whether these features capture meaningful biological information.

Energy consumption was measured using **CodeCarbon**, and a structured sustainability report was generated using the **GreenMetaData** tool.

---

## Repository Contents

| File | Description |
|-----|-------------|
| `Copy_of_subcellular_localization_linear_probe` | Jupyter notebook containing the workflow execution and carbon footprint tracking using CodeCarbon |
| `interpot_sustainability_audit (1).yml` | Sustainability report generated using the GreenMetaData tool |
| `Logbook.pdf` |A compilation of step-by-step snapshots of this project, results and recommendations  |

---

## Background

**InterProt** (Adams et al., 2025) is a biological interpretability project that aims to understand how protein language models represent biological features.

The project trains **Sparse Autoencoders (SAEs)** on the internal representations of protein language models to identify interpretable features.

Protein language models are capable of learning patterns from large collections of protein sequences. Research has shown that these models encode both:

- **Generic protein features**
- **Family-specific biological patterns**

Biologically meaningful properties  like  **subcellular localization** (i.e where a protein resides in a cell) can be identified by training simple classifiers on the features extracted from the model.

However, interpretability workflows of this type often require **large numbers of repeated forward passes through large neural models**, making them computationally convoluted and potentially energy inefficient.

This audit anlalyzes the **carbon footprint of primary energy hopsots of this workflow**, followed by proposals of optimizations that could improve its environmental sustainability.

---

## Methodology

### 1. Identifying an Energy Hotspot

The InterProt repository contains several notebooks:

| Notebook | Purpose |
|--------|--------|
| `sae_inference` | Runs sparse autoencoder inference on protein sequences using a pretrained model |
| `sae_test_steering` | Tests interventions based on sparse autoencoder features |
| `subcellular_localization_linear_probe` | Trains a classifier to predict protein subcellular localization |

The **activation and latent feature collection stage** was identified as the main energy hotspot.

This stage:

1. Processes many protein sequences through the **ESM-2 protein language model**
2. Extracts internal model activations using the sparse autoencoder
3. Uses these activations to train a **linear classifier** that predicts biological properties

This stage was selected because it involves **repeated GPU-heavy forward passes**, making it the most computationally intensive part of the workflow.

---

### 2. Running the Workflow with Carbon Tracking

The notebook was executed using **Google Colab with a T4 GPU runtime**.

Before running the activation generation step, **CodeCarbon** tracking was added to measure energy consumption.

The tracker was started immediately before the computational stage and stopped after the process completed.

To ensure accurate measurement:

- The relevant notebook cells were executed sequentially
- Idle runtime was avoided using the **Run All** option on the relevant code blocks
- This ensured that CodeCarbon only measured **active computation**

---

### 3. Generating a Sustainability Report

After the experiment completed:

1. CodeCarbon produced a detailed record of the energy usage
2. The **GreenMetaData** tool was used to generate a structured sustainability report
3. The resulting **YAML report** was added to this repository

---

## Results

### Overall Energy Consumption

| Metric | Value |
|------|------|
| Electricity used | **0.096957 kWh** |
| CO₂ emissions | **0.013453 kg CO₂eq** |
| Water usage | **0.000 L** |

Breakdown by hardware component:

| Component | Energy Consumption |
|----------|-------------------|
| GPU | 0.051571 kWh |
| CPU | 0.036742 kWh |
| RAM | 0.008644 kWh |

GPU computation represented the **largest contributor to total energy consumption**, confirming that model inference is the primary energy hotspot.

---

### Process-Level Energy Consumption

| Process | CPU Energy | GPU Energy | GPU Power | Electricity Used |
|-------|------------|------------|-----------|----------------|
| ESM & SAE Inference | 0.029545 kWh | 0.045735 kWh | 65.46 W | 0.082231 kWh |
| Linear Classifiers | 0.036622 kWh | 0.036622 kWh | 34.47 W | 0.096710 kWh |

The **ESM-2 + SAE inference stage** consumed the most GPU energy due to repeated model forward passes across many protein sequences.

---

## Sustainability Improvements

Several redesign strategies could significantly reduce the environmental footprint of this workflow.

### 1. Batch the Inference Loop

Currently, protein sequences are processed **one at a time** in a loop.

Batching multiple sequences together (e.g., batches of 8–32) would:

- Improve GPU utilization
- Reduce repeated kernel overhead
- Decrease total runtime and energy consumption

---

### 2. Cache Intermediate Activations

After computing the activation tensors (`seq_esm_acts` and `seq_sae_acts`), they could be saved to disk using:

- `torch.save()`
- `safetensors`

This would allow later probing experiments to **reuse previously computed features**, avoiding repeated forward passes.

---

### 3. Model Compression

The **ESM-2 model** could be optimized using techniques such as:

- **Quantization**
- **Knowledge distillation**

These methods reduce model size and computational cost while preserving most predictive performance.

This aligns with the green software pattern of **using energy-efficient AI/ML models**.

---

## Limitations

This audit evaluated **the primary energy hotspots** within the InterProt workflow.

Other components i.e development and testing were not measured.

---

## Future Work

In the future, this audit will be improved by:

- Measuring energy consumption across the **entire pipeline**
- Implementing the proposed optimizations
- Comparing the **carbon footprint before and after optimization**
- Evaluating sustainability across multiple interpretability experiments
---

## Tools Used

- CodeCarbon — energy and carbon footprint tracking  
- GreenMetaData — sustainability reporting framework  
- Google Colab — computational environment  
- PyTorch — machine learning framework

---

## References

Adams et al. (2025). *InterProt: Interpreting Protein Language Models using Sparse Autoencoders.*

Lannelongue, L., et al. (2021).  
*Green Algorithms: Quantifying the Carbon Footprint of Computation.* Advanced Science.

Green Software Foundation.  
Software sustainability principles.

CodeCarbon Documentation  
https://codecarbon.io
