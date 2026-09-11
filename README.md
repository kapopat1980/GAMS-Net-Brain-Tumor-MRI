# GAMS-Net: Leakage-Aware Benchmarking of a Lightweight Brain Tumor MRI Classifier

This repository accompanies the manuscript **"GAMS-Net: Design and Leakage-Aware
Benchmarking of a Lightweight Ghost-Attention Multi-Scale CNN for Brain Tumor MRI
Classification"** and contains the full experimental pipeline, raw result artifacts, and
the manuscript draft, released for reproducibility and editorial/reviewer access.

## Summary

GAMS-Net is a 2.82-million-parameter CNN combining Ghost modules, Efficient Channel
Attention (ECA), and a novel Multi-Scale Dilated Fusion (MSDF) block, designed for
four-class brain tumor MRI classification (glioma, meningioma, pituitary, no tumor). It
was benchmarked — under a leakage-audited, self-constructed, three-seed protocol — against
four ImageNet-pretrained lightweight/mid-size CNNs (MobileNetV2, EfficientNet-B0, ResNet18,
ShuffleNetV2) and a from-scratch Plain-CNN architectural control.

**Headline, honestly reported finding:** GAMS-Net (89.39% ± 2.00% mean test accuracy
across 3 seeds) did **not** outperform the pretrained lightweight baselines
(EfficientNet-B0: 95.84% ± 0.14%; MobileNetV2: 94.89% ± 0.40%) on this corpus. The
component ablation shows Ghost modules cut parameters by ~65% at a negligible accuracy
cost, the MSDF block is the dominant accuracy-contributing component (+3.25 points on
removal), and ECA's contribution was not distinguishable from the model's own run-to-run
noise. Full detail, including the statistical testing protocol and its limitations, is in
the manuscript.

This repository intentionally reports a result in which the proposed architecture does
**not** achieve a new state of the art. The contribution offered here is the leakage
audit, the identical-budget multi-model benchmark, the component ablation, and the
reproducible pipeline — not a superiority claim.

## Repository contents

```
.
├── notebooks/
│   └── GAMS-Net_Brain_Tumor_MRI_Classification.ipynb   # Full experimental pipeline
├── results/
│   ├── benchmark_results.csv          # Single-run (seed 42) comparison, all 6 models
│   ├── ablation_results.csv           # Ghost / ECA / MSDF component ablation
│   ├── seed_robustness.csv            # 3-seed (42, 123, 2024) mean ± std
│   ├── leakage_audit_after_fix.csv    # Perceptual-hash leakage audit output (0 pairs found)
│   └── figures/
│       ├── benchmark_plots.png        # Accuracy comparison + accuracy-vs-params trade-off
│       ├── confusion_matrices.png     # Confusion matrices, all 6 models
│       └── class_distribution.png     # Train/val/test class balance after the leak-audited split
├── manuscript/
│   └── GAMS-Net_Manuscript.docx       # Full manuscript draft
├── requirements.txt
├── LICENSE
├── CITATION.cff
└── README.md
```

## Dataset

**Brain Tumor MRI Dataset (Merged)**, Kaggle: [`sabersakin/brainmri`](https://www.kaggle.com/datasets/sabersakin/brainmri).
Four classes: glioma, meningioma, no-tumor, pituitary. This dataset follows the same
merge lineage as the widely used Nickparvar Kaggle corpus (SARTAJ + Cheng's Figshare
dataset + Br35H). The dataset is **not redistributed in this repository**; download it
directly from Kaggle and point the notebook at it (see "Reproducing the experiment"
below).

The archive's own shipped "test" folder was found to contain only four demonstration
images (one per class) rather than a usable evaluation split. The notebook detects this
automatically and falls back to a stratified 70/15/15 split constructed from the full
13,351-image labeled pool, followed by a perceptual-hash leakage audit of the resulting
test set against the training/validation pool (0 near-duplicate pairs found — see
`results/leakage_audit_after_fix.csv`).

## Reproducing the experiment

1. Open `notebooks/GAMS-Net_Brain_Tumor_MRI_Classification.ipynb` on
   [Kaggle](https://www.kaggle.com/) (Settings → Accelerator → GPU) or another CUDA-capable
   environment.
2. Attach the `sabersakin/brainmri` dataset as a notebook input (on Kaggle: Add Input →
   search `sabersakin/brainmri`).
3. Run all cells top to bottom. The notebook auto-detects the dataset's folder layout;
   Section 2's diagnostic output should be checked against the Kaggle "Data" panel before
   proceeding, as noted inline.
4. Expected runtime: several hours on a Kaggle P100/T4 GPU (six models × three seeds plus a
   four-variant single-seed ablation).

Package versions actually used are not pinned to exact versions in this repository since
the experiment was run on Kaggle's managed environment; `requirements.txt` lists the
direct dependencies. For exact reproducibility, record `torch.__version__` and
`torchvision.__version__` from your run (the notebook prints these) and report them
alongside any numbers you cite from this work.

## Key results

| Model | Params (M) | Accuracy (mean ± SD, 3 seeds) | F1 macro (mean ± SD) |
|---|---|---|---|
| GAMS-Net (proposed) | 2.824 | 0.8939 ± 0.0200 | 0.8964 ± 0.0204 |
| MobileNetV2 (pretrained) | 2.229 | 0.9489 ± 0.0040 | 0.9504 ± 0.0035 |
| EfficientNet-B0 (pretrained) | 4.013 | 0.9584 ± 0.0014 | 0.9600 ± 0.0014 |
| ResNet18 (pretrained) | 11.179 | 0.9372 ± 0.0073 | 0.9387 ± 0.0061 |
| ShuffleNetV2 (pretrained) | 1.258 | 0.9419 ± 0.0034 | 0.9430 ± 0.0032 |
| Plain-CNN (from scratch, control) | 0.980 | 0.8570 ± 0.0080 | 0.8600 ± 0.0075 |

See `results/ablation_results.csv` for the component-level ablation and the manuscript
for full statistical testing (Welch's t-test across seeds, with an explicit low-power
caveat at n = 3).

## What is *not* included in this repository

- **Trained model checkpoints** (`.pt` weight files) — not included due to size; retrain
  from the notebook to reproduce them exactly (fixed seeds are used throughout).
- **The full stratified split manifest** (row-level file paths for every image in the
  train/val/test split) — omitted because it references local Kaggle input paths that are
  not portable across environments; the split is fully reproducible from the notebook's
  fixed random seed (42) and the stated 70/15/15 stratified split logic.
- **Grad-CAM qualitative panels** — the notebook includes a self-installing Grad-CAM
  implementation (Section 16), but rendered example figures are not included in this
  results snapshot. Regenerate from the notebook if needed for the camera-ready
  submission.
- **The raw dataset** — redistribute-by-reference only; see "Dataset" above.

## Status and how to cite

This manuscript is a draft prepared for journal submission and has **not yet been peer
reviewed**. If this repository is made public before acceptance, check your target
journal's preprint/embargo policy first — policies vary by venue. See `CITATION.cff` for
citation metadata (update the `doi`/`url`/publication fields once assigned by the
journal).

## License

The code in this repository (Jupyter notebook) is released under the MIT License — see
`LICENSE`. The manuscript text and figures in `manuscript/` and `results/figures/` are
**not** covered by that license; copyright in the manuscript typically transfers to the
publisher upon acceptance, and it is included here for editorial/reviewer reference only
unless your target journal's policy states otherwise.

## Contact

Divyakant, Faculty of Computer Applications (FOCA), Marwadi University, Rajkot, Gujarat,
India.
