# ExDD: Explicit Dual Distribution Learning for Surface Defect Detection via Diffusion Synthesis

<p align="center">
  <a href="https://link.springer.com/chapter/10.1007/978-3-032-10192-1_12"><img src="https://img.shields.io/badge/Paper-Springer-blue" alt="Paper"></a>
  <a href="https://github.com/aqeeelmirza/ExDD-Defect-Detection"><img src="https://img.shields.io/badge/GitHub-ExDD-black?logo=github" alt="GitHub"></a>
  <img src="https://img.shields.io/badge/Python-3.8+-green" alt="Python">
  <img src="https://img.shields.io/badge/PyTorch-1.10+-orange" alt="PyTorch">
</p>

> **ExDD: Explicit Dual Distribution Learning for Surface Defect Detection via Diffusion Synthesis**  
> *International Conference on Image Analysis and Processing (ICIAP)*  
> [[Paper]](https://link.springer.com/chapter/10.1007/978-3-032-10192-1_12) · [[Code]](https://github.com/aqeeelmirza/ExDD-Defect-Detection)

---

## Overview

**ExDD** is a surface defect detection framework that leverages a **dual memory bank architecture** combined with **diffusion-based synthetic data generation**. Unlike standard anomaly detection methods that model only the normal distribution, ExDD explicitly models *both* normal and anomalous distributions, enabling more discriminative anomaly scoring.

Key contributions:
- **Dual Distribution Learning**: Separate positive (anomalous) and negative (normal) memory banks using PatchCore-style feature matching.
- **Diffusion Synthesis**: Synthetic defect images generated via diffusion models augment the positive memory bank, improving coverage of rare or unseen defect types.
- **Explicit Scoring**: Anomaly scores are derived by comparing against both distributions jointly.

---

## Method

### Core Components

#### 1. PatchCore Backbone (`patchcore.py`)

- **`PatchCoreSingle`**: Standard PatchCore with a single normal memory bank.
- **`PatchCoreDual`**: Extended PatchCore with two memory banks:
  - **Negative bank** — built from normal (defect-free) samples.
  - **Positive bank** — built from real and synthetically generated defect crops.
- **Feature Extraction**: Pre-trained ResNet50 or WideResNet50_2 backbones extract patch-level features.
- **Coreset Subsampling**: Reduces memory bank size while retaining representative features.
- **Anomaly Scoring**: Scores from both banks are combined for final prediction.

#### 2. Experiment Runner (`run_patchcore.py`)

- Loads the **KSDD2 dataset** via custom data classes (`KolektorSDD2`, `KolektorSDD2Crops`).
- Supports flexible configuration: backbone selection, subsampling rates, augmented data toggle.
- Evaluates image-level and pixel-level **AUROC** scores.
- Saves results to `.npy` files.

#### 3. Synthetic Data Generation (`generate_augmented_images.py`)

- Generates synthetic defect images using **diffusion models**.
- Augmented images are incorporated into the positive memory bank to improve detection of diverse defect types.

---

## Dataset

This project uses the **Kolektor Surface Defect Dataset 2 (KSDD2)**, containing industrial surface images with and without defects.

- **Normal samples**: Defect-free surface images used to build the negative memory bank.
- **Anomalous samples**: Images with surface defects; pre-cropped defect regions are used for the positive memory bank.

---

## Installation

```bash
git clone https://github.com/aqeeelmirza/ExDD-Defect-Detection.git
cd ExDD-Defect-Detection
pip install -r requirements.txt
```

---

## Usage

### 1. Download Dataset

```bash
python src/utils/ksdd2_downloader.py --out_path ./data/KSDD2_original
```

### 2. Preprocess Data

```bash
python src/utils/ksdd2_preprocess.py \
    --src_dir ./data/KSDD2_original \
    --dst_dir ./data/ksdd2_preprocessed
```

### 3. (Optional) Generate Synthetic Defects

```bash
python src/utils/generate_augmented_images.py \
    --src_dir ./data/ksdd2_preprocessed \
    --imgs_per_prompt 50
```

### 4. (Optional) Extract Anomaly Crops

```bash
python src/utils/extract_anomalous_crops.py \
    --train_dir ./data/ksdd2_preprocessed/train \
    --output_dir ./data/defect_crops \
    --csv_file ./data/ksdd2_preprocessed/train.csv
```

### 5. Run Experiments

```bash
python src/run_patchcore.py \
    --dataset_path ./data/ksdd2_preprocessed \
    --crops_path ./data/defect_crops \
    --output_dir ./results \
    --backbone resnet50 \
    --neg_subsampling 0.01 \
    --pos_subsampling 0.10 \
    --seed 42
```

> Use `--add_augmented` to include diffusion-generated synthetic defects in the positive memory bank.  
> Run `python src/run_patchcore.py --help` for the full list of arguments.

---

## Results

Evaluated on KSDD2 dataset:

| Method | Image AUROC | Pixel AUROC |
|--------|-------------|-------------|
| PatchCore (single) | 91.2 | 95.8 |
| **ExDD (ours)** | **94.2** | **97.7** |

*(Fill in your numbers here)*

---

## Citation

If you find this work useful, please cite:

```bibtex
@inproceedings{aqeel2025exdd,
  title={ExDD: Explicit Dual Distribution Learning for Surface Defect Detection via Diffusion Synthesis},
  author={Aqeel, Muhammad and Leonardi, Federico and Setti, Francesco},
  booktitle={International Conference on Image Analysis and Processing},
  pages={140--151},
  year={2025},
  organization={Springer}
}
```

---

## License

This project is released under the [MIT License](LICENSE).

