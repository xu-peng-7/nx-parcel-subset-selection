# n–x: Training Subset Selection for Parcel Instance Segmentation Using Instance-Level Mask Quality Gaps

Official project page for the manuscript:

> **Training Subset Selection for Parcel Instance Segmentation Using Instance-Level Mask Quality Gaps**  
> *Submitted to MDPI AI (Under Review)*

## Release Status

> **Note:** The complete implementation—including the offline *n–x* instance-matching and capped top-3/3 selection pipeline, fixed dataset split manifests (base, candidate pool, anchor, and held-out test splits), cached instance-score records, evaluation scripts, and exact reproduction instructions—will be released publicly in this repository **upon acceptance** of the paper.
>
> During peer review, reviewers and editors may request access to the selection scripts and split manifests through the editorial office or by contacting the corresponding author.

## Overview

We present **n–x**, an offline whole-image subset selection framework for parcel instance segmentation under fixed retraining budgets. By reusing frozen lightweight (`YOLO26n-seg`, 3.05M parameters) and high-capacity (`YOLO26x-seg`, 70.48M parameters) segmenters from a prior update cycle, **n–x**:

1. Matches both frozen scorers one-to-one against every ground-truth parcel mask via lexicographic two-stage same-class IoU assignment ($\tau_{\text{IoU}} = 0.10$).
2. Computes signed instance-level mask quality gaps $g(i) = q_x(i) - q_n(i)$ and converts them into pool-wide midrank percentiles $r(i) \in [0, 1]$.
3. Aggregates frame priority via a capped top-3/3 image score to construct a static, reusable training subset with $O(N_{\text{new}})$ incremental GPU scoring and zero edge deployment overhead.

## Summary of Main Results (640 px, 839 Images, 100 Epochs, 3 Paired Seeds)

| Method | LSCD AP50 | OSCD AP50 | STRIPES v1.1 AP50 | Macro AP50 | Macro COCO Mask AP |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Random | 93.10 ± 0.67 | 89.59 ± 0.26 | 69.93 ± 1.04 | 84.21 ± 0.54 | 65.56 ± 0.67 |
| TFDP | 93.28 ± 0.10 | 89.21 ± 0.30 | 72.87 ± 0.26 | 85.12 ± 0.21 | 65.21 ± 0.46 |
| CCS+AUM | 93.44 ± 0.44 | 90.11 ± 0.07 | 72.79 ± 0.46 | 85.45 ± 0.11 | 66.63 ± 0.14 |
| RHO-LOSS (Online) | 93.93 ± 0.18 | 90.25 ± 0.43 | 70.15 ± 0.66 | 84.78 ± 0.16 | 66.13 ± 0.30 |
| **n–x (Ours)** | **94.32 ± 0.37** | **90.55 ± 0.32** | **74.64 ± 0.57** | **86.50 ± 0.19** | **67.64 ± 0.19** |

- **Incremental Scoring Cost**: Scoring 14,753 newly added parcel images across all three benchmarks completes in **15.2 min** wall-clock time on two NVIDIA RTX 3090 GPUs (23.66 GPU-min + 40.1–101.2 ms CPU re-ranking).
- **Cross-Domain Transfer (PASCAL VOC 2012, Mask R-CNN, all 1,444 val images)**: **27.87 ± 0.36** Mask AP50 (vs. 27.49 TFDP, 27.04 Entropy, 26.90 CCS+AUM, 22.10 Random).

## Public Benchmark Datasets

All raw datasets used in our experiments are publicly available from their original sources:
- **SCD (LSCD & OSCD)**: Yang et al., *Sensors* 2022 ([DOI: 10.3390/s22103617](https://doi.org/10.3390/s22103617))
- **STRIPES v1.1**: Ani et al., Zenodo 2026 ([DOI: 10.5281/zenodo.22799027](https://doi.org/10.5281/zenodo.22799027))
- **PASCAL VOC 2012**: Everingham et al., *IJCV* 2010 ([DOI: 10.1007/s11263-009-0275-4](https://doi.org/10.1007/s11263-009-0275-4))

## Planned Repository Structure (Upon Acceptance)

```text
.
├── configs/           # Training and 640 px FP32 scoring configurations
├── manifests/         # Deduplicated base, pool, anchor, test, and selected 839-image IDs
├── scoring/           # Two-stage Hungarian IoU matching, midrank percentiles, and top-3/3 ranking
├── eval/              # Unified 640 px Mask AP50 / COCO Mask AP and VOC 1,444-image evaluation scripts
└── README.md
```

## License

This repository will be released under the MIT License upon publication.
