# cephalometric-landmark-detection
An automated 2D landmark tracking pipeline for lateral cephalograms using PyTorch.
# Cephalometric Landmark Detection

A deep learning pipeline for automated detection of cephalometric landmarks (A-point, Nasion, B-point) on lateral cephalograms, built as a self-directed project combining clinical background (DDS, Oral & Maxillofacial Surgery) with applied machine learning.

**Full technical report:** [`report/cephalometric_landmark_detection_report.md`](./report/cephalometric_landmark_detection_report.md)

## Summary

- Trained a CNN-based regression model to predict 3 foundational cephalometric landmarks on the public [Aariz (CEPHA29)](https://www.nature.com/articles/s41597-025-05542-3) dataset (Khalid et al., *Scientific Data*, 2025).
- Identified and fixed an aspect-ratio distortion bug in image preprocessing (naive resize-to-square vs. correct pad-then-resize) — verified visually before and after.
- Achieved a mean radial error of **4.1–4.9mm** at baseline (256×256 input).
- Formed and tested a hypothesis that image resolution contributes to error: found a moderate correlation (**r = 0.50**) between per-image `pixel_size` and landmark error; a controlled 512×512 resolution experiment produced a modest, consistent improvement (largest effect on the worst-case outlier: −4.1mm).
- Compared results honestly against the dataset's own published benchmark (1.79mm, heatmap-based architecture) — the ~2–3x gap is attributed to architecture choice (direct coordinate regression vs. heatmap regression), not left unexplained.
- Conducted failure-case analysis on the worst-performing validation samples, reporting both an explained pattern (resolution-correlated) and an honestly unexplained residual case.

## Repository Structure

```
├── notebooks/
│   └── cephalometric_landmark_detection.ipynb   # Full pipeline: data loading, model, training, evaluation
├── report/
│   └── cephalometric_landmark_detection_report.md  # Full technical report
├── figures/
│   └── ...                                       # Landmark overlay visualizations, error plots
└── README.md
```

## Dataset

This project uses the public **Aariz (CEPHA29)** dataset (Khalid et al., 2025), 1,000 lateral cephalograms from 7 imaging devices, annotated by junior and senior orthodontists. The dataset is not included in this repository — download it separately from its [official source](https://www.nature.com/articles/s41597-025-05542-3) and place it according to the path structure expected in the notebook (see setup below).

**Landmarks used in this project:** A-point (A), Nasion (N), B-point (B) — sufficient to compute the ANB angle, a foundational cephalometric measurement.

## Method (Brief)

1. **Preprocessing:** Each image is padded to a square canvas (preserving aspect ratio) before resizing to a fixed input size, with landmark coordinates adjusted accordingly.
2. **Model:** A CNN (3 convolutional layers + 2 fully connected layers) trained via direct coordinate regression (MSE loss) to predict 3 landmarks (6 output values).
3. **Evaluation:** Mean Radial Error (MRE) computed per-image in millimeters, using each image's individual `pixel_size` (mm/pixel) value from the dataset's machine mapping file — not a single global conversion factor.

Full methodological detail, including the specific bugs found and fixed along the way, is in the [full report](./report/cephalometric_landmark_detection_report.md).

## Results

| Landmark | Mean Error 256px (mm) | Mean Error 512px (mm) |
|----------|------------------------|-------------------------|
| A-point  | 4.078                  | 3.757                   |
| Nasion   | 4.229                  | 4.177                   |
| B-point  | 4.885                  | 4.775                   |

**Comparison to published Aariz benchmarks:**

| Model | MRE (mm) | Architecture |
|-------|----------|--------------|
| This project (baseline) | 4.1–4.9 | Direct coordinate regression, CNN from scratch |
| Khalid et al., 2025 (dataset baseline) | 1.79 | Two-stage cascaded CNN, heatmap regression |
| Khan et al., 2024 | 1.94 | Multi-scale CNN, heatmap regression |
| Best published (CephRes-MHNet) | 1.23 | Multi-head residual CNN, heatmap regression |
| Human inter-observer variability (reference) | 0.33 | — |

## Setup & Reproduction

```bash
# Clone the repo
git clone https://github.com/[your-username]/[repo-name].git

# Open notebooks/cephalometric_landmark_detection.ipynb in Google Colab or Jupyter
# Requires: torch, torchvision, PIL, pandas, matplotlib
```

1. Download the Aariz dataset and upload/mount it in your environment.
2. Update dataset paths at the top of the notebook.
3. Run cells in order — the notebook is structured to cache preprocessed sample lists and trained model weights, so subsequent runs after the first don't need to repeat data loading or training from scratch.

## Limitations & Future Work

- Trained from scratch on 700 images — no pretrained backbone or transfer learning used.
- Only 3 of the dataset's 29 available landmarks used.
- No data augmentation (scale/rotation/translation) applied — likely contributes to the observed resolution sensitivity.
- Next planned step: re-implement using **heatmap-based regression**, the architecture used by all better-performing published results on this dataset, rather than direct coordinate regression.

Full discussion in the [technical report](./report/cephalometric_landmark_detection_report.md#5-limitations-and-future-work).

## References

Khalid, M.A., Zulfiqar, K., Bashir, U., Shaheen, A., Iqbal, R., Rizwan, Z., Rizwan, G., & Fraz, M.M. (2025). A Benchmark Dataset for Automatic Cephalometric Landmark Detection and CVM Stage Classification. *Scientific Data*, 12(1), 1336.

Khalid, M.A., et al. (2022). CEPHA29: Automatic Cephalometric Landmark Detection Challenge 2023. *arXiv preprint arXiv:2212.04808*.

## Author

Samer Moshref, DDS, Oral & Maxillofacial Surgery — self-directed project undertaken as part of preparation for orthodontic residency, combining clinical background with applied machine learning (Python, PyTorch).
