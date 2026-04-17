# Dataset: BraTS 2021 Task 1 — Brain Tumor Segmentation

## Overview

This project uses the **BraTS 2021 (Brain Tumor Segmentation) Task 1 Dataset**, which contains multi-parametric MRI (mpMRI) scans of 1,251 subjects with annotated glioblastoma tumor regions. The dataset is too large to include in this repository and must be downloaded separately.

## Where to Get the Data

The dataset is hosted on the **Synapse platform** by the CBICA (Center for Biomedical Image Computing and Analytics):

**Download Link:** [https://www.synapse.org/#!Synapse:syn25829067](https://www.synapse.org/#!Synapse:syn25829067)

### Steps to Download

1. Create a free account at [synapse.org](https://www.synapse.org/).
2. Navigate to the BraTS 2021 challenge page linked above.
3. Accept the data use agreement / terms of use.
4. Download the **Training Data** archive (`BraTS2021_Training_Data.tar`).
5. Extract the archive into the `data/` directory of this project.

## Dataset Structure

After extraction, the data should be organized as follows:

```
data/
└── BraTS2021_Training_Data/
    ├── BraTS2021_00000/
    │   ├── BraTS2021_00000_t1.nii.gz        # T1-weighted MRI
    │   ├── BraTS2021_00000_t1ce.nii.gz      # T1 contrast-enhanced MRI
    │   ├── BraTS2021_00000_t2.nii.gz        # T2-weighted MRI
    │   ├── BraTS2021_00000_flair.nii.gz     # FLAIR MRI
    │   └── BraTS2021_00000_seg.nii.gz       # Ground truth segmentation mask
    ├── BraTS2021_00001/
    │   └── ...
    └── ...
```

Each subject folder contains **five NIfTI files** (`.nii.gz`):

| File Suffix | Modality | Purpose |
|---|---|---|
| `_t1.nii.gz` | T1-weighted | Native structural scan |
| `_t1ce.nii.gz` | T1 contrast-enhanced | Highlights enhancing tumor regions |
| `_t2.nii.gz` | T2-weighted | Highlights peritumoral edema |
| `_flair.nii.gz` | FLAIR | Optimized for edema detection |
| `_seg.nii.gz` | Segmentation mask | Ground truth labels |

## Segmentation Labels

The ground truth masks (`_seg.nii.gz`) contain the following voxel-level labels:

| Label | Value | Description |
|---|---|---|
| Background / Healthy Tissue | 0 | Normal brain tissue |
| Necrotic Core | 1 | Dead tissue within the tumor |
| Peritumoral Edema | 2 | Swelling around the tumor |
| Enhancing Tumor | 4 | Actively growing tumor region |

> **Note:** Label 3 is not used in BraTS 2021. The labels jump from 2 to 4.


## Preprocessing Notes

Before training, apply the following preprocessing steps (as described in our project methodology):

1. **Z-score normalization** — Normalize each patient's volume independently per modality to zero mean and unit variance.
2. **Patching** — Extract 128 × 128 × 128 patches from the full 240 × 240 × 155 volumes to fit GPU memory.
3. **Augmentation** — Apply random 3D rotations and flips during training.

## Citation

If you use this dataset, please cite the BraTS challenge:

```
@article{baid2021rsna,
  title={The RSNA-ASNR-MICCAI BraTS 2021 Benchmark on Brain Tumor Segmentation and Radiogenomic Classification},
  author={Baid, Ujjwal and others},
  journal={arXiv preprint arXiv:2107.02314},
  year={2021}
}
```

## License

The BraTS 2021 dataset is provided under the terms specified on the Synapse platform. You must accept the data use agreement before downloading. The data is intended for research purposes only.
