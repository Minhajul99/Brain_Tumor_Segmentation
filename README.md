# Brain Tumor Segmentation using Deep Learning

Automated multiclass brain tumor segmentation from 3D MRI scans using three deep learning architectures: **3D U-Net**, **Attention U-Net**, and **Swin UNETR**.

## Authors

| Name | Email |
|------|-------|
| Md Minhajul Abedin | mminhajula@mun.ca |
| Nabil Hasan | nabilh@mun.ca |
| Md Abdul Hadi Chowdhury | mabdulc@mun.ca |

**Group 10** — Memorial University of Newfoundland

---

## Overview

Brain tumors, particularly glioblastomas, have a complex heterogeneous structure with distinct sub-regions that need precise identification for treatment planning. Manual segmentation by radiologists is time-consuming and prone to inter-observer variability. This project develops an automated pipeline that segments tumor sub-regions from multi-parametric MRI scans using deep learning.

The system classifies each voxel into one of four categories:
- **Label 0** — Healthy tissue (background)
- **Label 1** — Necrotic tumor core (NCR)
- **Label 2** — Peritumoral edema (ED)
- **Label 3** — Enhancing tumor (ET)

## Dataset

We use the **BraTS 2021 Task 1 Dataset**, which contains 3D MRI scans from **1,251 subjects**. Each subject includes four MRI modalities:

- **T1** — Native pre-contrast scan
- **T1ce** — Gadolinium-enhanced scan (highlights tumor core)
- **T2** — Highlights peritumoral edema
- **FLAIR** — Fluid-attenuated inversion recovery (edema visibility)

All volumes are provided as NIfTI files (`.nii.gz`) with dimensions of 240×240×155 voxels.

### Data Split
| Set | Subjects | Percentage |
|-----|----------|------------|
| Training | ~1,000 | 80% |
| Validation | ~125 | 10% |
| Test | ~126 | 10% |

## Models

### 3D U-Net
The baseline encoder-decoder architecture for volumetric medical image segmentation. Uses residual units with 5 encoding levels (32, 64, 128, 256, 512 channels) and skip connections.

### Attention U-Net
Extends the U-Net with attention gates that learn to suppress irrelevant regions and focus on smaller tumor components. Same channel configuration as 3D U-Net.

### Swin UNETR
A transformer-based architecture that uses a Swin Transformer encoder to capture long-range spatial dependencies. Uses shifted window self-attention with a feature size of 24.

## Preprocessing Pipeline

1. **NIfTI Loading** — Volumes loaded using nibabel/MONAI
2. **Orientation** — Standardized to RAS (Right-Anterior-Superior)
3. **Resampling** — Resampled to 1mm isotropic spacing
4. **Label Remapping** — BraTS labels {0, 1, 2, 4} remapped to contiguous {0, 1, 2, 3}
5. **Normalization** — Z-score normalization per patient, per modality (nonzero voxels only)
6. **Foreground Cropping** — Empty background margins removed
7. **Patching** — Random 3D crops (128³ for U-Net/Attention U-Net, 96³ for Swin UNETR)

### Data Augmentation (Training Only)
- Random 3D flips (all three axes)
- Random 90° rotations
- Random intensity scaling (±10%)
- Random intensity shifting (±10%)

## Training Configuration

| Parameter | Value |
|-----------|-------|
| Optimizer | AdamW |
| Learning Rate | 1e-4 |
| Weight Decay | 1e-5 |
| LR Scheduler | Cosine Annealing |
| Loss Function | Dice + Cross-Entropy |
| Batch Size | 2 |
| Max Epochs | 100 |
| Early Stopping Patience | 15 validation intervals |
| Mixed Precision | Yes (torch.amp) |
| Gradient Clipping | Max norm = 1.0 |

## Results

### Dice Similarity Coefficient (↑ higher is better)

| Model | Whole Tumor | Tumor Core | Enhancing Tumor | Mean Dice |
|-------|-------------|------------|-----------------|-----------|
| 3D U-Net | 0.9330 | 0.8980 | 0.8537 | 0.8949 |
| **Attention U-Net** | **0.9390** | **0.9060** | **0.8624** | **0.9028** |
| Swin UNETR | 0.9260 | 0.8810 | 0.8436 | 0.8902 |

### Hausdorff Distance 95% in mm (↓ lower is better)

| Model | Whole Tumor | Tumor Core | Enhancing Tumor |
|-------|-------------|------------|-----------------|
| 3D U-Net | 2.50 | 2.50 | 1.78 |
| **Attention U-Net** | **1.78** | **2.35** | **2.30** |
| Swin UNETR | 2.00 | 2.90 | 2.90 |

**Best model: Attention U-Net** with a mean Dice of 0.9028.

## Model Interpretability

We implemented **Grad-CAM (Gradient-weighted Class Activation Mapping)** to visualize which regions of the MRI each model focuses on when predicting each tumor sub-region. This provides transparency into the model's decision-making process without any retraining.

Additionally, we generated **3D volumetric surface renderings** of the predicted segmentation masks using the marching cubes algorithm, allowing visual inspection of tumor shape and structure from any angle.

## Project Structure

```
.
├── Brain_Tumor_Segmentation.ipynb    # Main notebook with all code
├── README.md                         # This file
├── data/
│   └── brats2021/                    # BraTS 2021 dataset (not included)
│       └── BraTS2021_XXXXX/
│           ├── BraTS2021_XXXXX_t1.nii.gz
│           ├── BraTS2021_XXXXX_t1ce.nii.gz
│           ├── BraTS2021_XXXXX_t2.nii.gz
│           ├── BraTS2021_XXXXX_flair.nii.gz
│           └── BraTS2021_XXXXX_seg.nii.gz
└── outputs/
    ├── unet/
    │   ├── best_model.pth
    │   ├── unet_results.json
    │   ├── gradcam_unet_subject_0.png
    │   ├── 3d_tumor_unet_subject_0.html
    │   └── 3d_comparison_unet_subject_0.html
    ├── attention_unet/
    │   ├── best_model.pth
    │   ├── attention_unet_results.json
    │   ├── gradcam_attention_unet_subject_0.png
    │   ├── 3d_tumor_attention_unet_subject_0.html
    │   └── 3d_comparison_attention_unet_subject_0.html
    └── swin_unetr/
        ├── best_model.pth
        ├── swin_unetr_results.json
        ├── gradcam_swin_unetr_subject_0.png
        ├── 3d_tumor_swin_unetr_subject_0.html
        └── 3d_comparison_swin_unetr_subject_0.html
```

## Requirements

### Environment
- Python 3.12.10
- CUDA-capable GPU (12 GB+ VRAM recommended)

### Dependencies
```
torch==2.10.0
monai
nibabel
numpy
matplotlib
scikit-learn
scikit-image
scipy
plotly
pandas
tqdm
tensorboard
opencv-python==4.13
```

### Installation

```bash
# Create conda environment
conda create -n bts python=3.12
conda activate bts

# Install PyTorch (adjust CUDA version as needed)
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121

# Install remaining dependencies
pip install monai nibabel numpy matplotlib scikit-learn scikit-image scipy plotly pandas tqdm tensorboard opencv-python
```

## Usage

### 1. Download the Dataset
Download the BraTS 2021 dataset and place it in `data/brats2021/`.

### 2. Train a Model
Open `Brain_Tumor_Segmentation.ipynb` and set the desired model in the CONFIG cell:
```python
CONFIG["model_name"] = "unet"  # options: "unet", "attention_unet", "swin_unetr"
```
Run all cells to train and evaluate. Repeat for each model.

### 3. Compare Models
After training all three models, the comparison section at the end of the notebook will automatically load results from each model's output directory and generate comparison charts.

### 4. Generate Visualizations
The Grad-CAM and 3D visualization cells can be run after training. They load saved checkpoints and require no retraining.

## Acknowledgements

- [BraTS 2021 Challenge](http://braintumorsegmentation.org/) for the dataset
- [MONAI](https://monai.io/) framework for medical image deep learning
- [Plotly](https://plotly.com/) for interactive 3D visualizations
