# Dual-Channel Contrastive Encoding for Sperm Whale Coda Understanding

**CS 297 Final Project — Tufts University, Spring 2026**
**Authors:** João Quintanilha & Emma Virnelli

---

## Overview

This repository contains the reproducible code, labels, notebooks, reports, and paper for our CS 297 final paper: **"Beyond WhAM: Dual-Channel Contrastive Encoding for Sperm Whale Coda Understanding."** Large raw audio files and pretrained model checkpoints are documented in `ARTIFACTS.md` rather than committed directly to GitHub.

We introduce **DCCE** (Dual-Channel Contrastive Encoder), a self-supervised model that respects the known biological structure of sperm whale codas — a GRU rhythm encoder processes inter-click interval (ICI) timing sequences, and a CNN spectral encoder processes mel-spectrograms. The two channels are trained with cross-channel NT-Xent contrastive positive pairs, inspired by the biological finding that rhythm encodes coda type while spectral texture encodes individual/social identity.

**Key result:** DCCE achieves individual-identity macro-F1 = **0.834** vs WhAM's best **0.454** (+83% relative), while matching WhAM on social-unit classification (0.878 vs 0.895).

---

## Repository Structure

```
dual_channel_contrastive_encoding_for_whales_understanding/
│
├── README.md                        ← you are here
│
├── paper/
│   └── DCCE_Beyond_WhAM.pdf         ← compiled final paper (LaTeX)
│
├── notebooks/                       ← Jupyter notebooks with all outputs preserved
│   ├── phase0_eda.ipynb             ← Phase 0: exploratory data analysis
│   ├── phase1_baselines.ipynb       ← Phase 1: raw ICI / mel baselines + WhAM L19 probe
│   ├── phase2_wham_probing.ipynb    ← Phase 2: layer-wise WhAM probing + year confound
│   ├── phase3_dcce.ipynb            ← Phase 3: DCCE training, ablations, UMAP
│   ├── phase3_dcce_youtube.ipynb    ← Phase 3 extension: YouTube sperm whale audio
│   └── phase4_synthetic_aug.ipynb   ← Phase 4: WhAM synthetic augmentation sweep
│
├── reports/                         ← Markdown write-ups for each phase (with figures)
│   ├── phase0_eda.md
│   ├── phase1_baselines.md
│   ├── phase2_wham_probing.md
│   ├── phase3_dcce.md
│   ├── phase4_synthetic_aug.md
│   └── *_files/                     ← Auto-generated figure folders for each report
│
├── datasets/                        ← Labels, metadata, splits, and compact feature caches
│   │
│   ├── — RAW AUDIO —
│   ├── dswp_audio/                  ← omitted from Git; see ARTIFACTS.md
│   ├── synthetic_audio/             ← omitted from Git; see ARTIFACTS.md
│   │
│   ├── — LABELS & METADATA —
│   ├── dswp_labels.csv              ← our assembled labels (unit, coda type, individual ID, ICI)
│   ├── DominicaCodas.csv            ← Sharma et al. 2024 source (8,719 annotated codas)
│   ├── codamd.csv                   ← Beguš et al. 2024 vowel labels (codaNUM 4933–8860)
│   ├── gero2016.xlsx                ← Gero et al. 2016 coda taxonomy reference
│   ├── focal-coarticulation-metadata.csv
│   ├── sperm-whale-dialogues.csv
│   ├── synthetic_meta.csv           ← metadata for WhAM-generated codas
│   ├── phase1_results.csv           ← Phase 1 linear probe results
│   └── phase4_results.csv           ← Phase 4 augmentation sweep results
│   │
│   ├── — PRE-COMPUTED FEATURES —
│   ├── wham_embeddings.npy          ← WhAM L19 embeddings for all 1,501 codas (7.3 MB)
│   ├── wham_embeddings_all_layers.npy ← omitted from Git; regenerate from Phase 2
│   ├── X_mel_full.npy               ← mel spectrograms for all clean codas (43 MB)
│   ├── X_mel_all.npy                ← mel spectrograms, alternative preprocessing (348 KB)
│   ├── X_mel_synth_1000.npy         ← mel spectrograms for synthetic codas (31 MB)
│   ├── X_ici_synth_1000.npy         ← ICI vectors for synthetic codas (36 KB)
│   ├── y_type_synth_1000.npy        ← coda type labels for synthetic codas
│   └── y_unit_synth_1000.npy        ← unit labels for synthetic codas
│   │
│   └── — TRAIN/TEST SPLIT INDICES —
│       ├── train_idx.npy            ← 80% split indices (stratified by unit, seed=42)
│       ├── test_idx.npy             ← 20% split indices
│       ├── train_id_idx.npy         ← train indices for IDN-labeled subset (762 codas)
│       └── test_id_idx.npy          ← test indices for IDN-labeled subset
│
├── ARTIFACTS.md                     ← how to recover omitted large files
│
└── wham/                            ← WhAM model source code; weights omitted from Git
    ├── README.md                    ← WhAM original README
    ├── setup.py
    ├── wham/                        ← Python package (inference, generation utilities)
    ├── vampnet/                     ← VampNet backbone (transformer architecture)
    │   └── models/
    │       ├── coarse.pth           ← omitted from Git; see ARTIFACTS.md
    │       └── codec.pth            ← omitted from Git; see ARTIFACTS.md
    └── assets/                      ← example audio assets
```

---

## Phases Summary

| Phase | Notebook | Key Question | Key Result |
|-------|----------|-------------|------------|
| 0 — EDA | `phase0_eda.ipynb` | What does the DSWP dataset look like? | Unit F = 59.4%; 22 coda types; ICI and spectral channels are near-zero correlated |
| 1 — Baselines | `phase1_baselines.ipynb` | What do raw features achieve? | Raw ICI → coda type F1 = 0.931; Raw mel → unit F1 = 0.740 |
| 2 — WhAM Probing | `phase2_wham_probing.ipynb` | What is WhAM actually encoding? | Unit peaks L19 (F1=0.895); recording-year confound V=0.51; individual ID peaks L10 (F1=0.454) |
| 3 — DCCE | `phase3_dcce.ipynb` | Does biological inductive bias beat scale? | DCCE individual ID F1=0.834 vs WhAM 0.454 (+83%) |
| 4 — Augmentation | `phase4_synthetic_aug.ipynb` | Does WhAM-generated data help DCCE? | No — ID F1 degrades monotonically with N_synth |

---

## Dataset Notes

**dswp_labels.csv** is our primary contribution on the data side. The DSWP HuggingFace release ships 1,501 WAV files with no labels. We discovered that Sharma et al. 2024's `DominicaCodas.csv` contains exactly rows 1–1,501 with `codaNUM2018` matching DSWP filenames directly (N.wav ↔ codaNUM2018=N). This 1:1 mapping was verified by matching ICI sequences and coda durations and is not documented in either dataset's release.

**No vowel labels exist for DSWP**: `codamd.csv` (Beguš et al. 2024 vowel annotations) covers codaNUM 4,933–8,860, entirely outside the DSWP range. This absence directly motivated the self-supervised CNN spectral encoder in DCCE rather than explicit vowel classification.

**IDN = 0**: 621 codas lack individual photo-identification (almost entirely Unit F). Individual-ID experiments use the 762 IDN-labeled codas from 12 individuals.

---

## How to Re-run Notebooks

All notebooks are self-contained with outputs already saved — you do not need to re-run them to see results. To re-run WhAM inference or synthetic generation, first restore the omitted artifacts listed in `ARTIFACTS.md`.

If you do want to re-run:

```bash
# 1. Create environment
python -m venv .venv && source .venv/bin/activate
pip install torch torchaudio numpy pandas scikit-learn matplotlib seaborn umap-learn librosa jupyter

# 2. Install WhAM
cd wham && pip install -e . && cd ..

# 3. Launch Jupyter
jupyter notebook notebooks/
```

> **Note:** WhAM inference requires ~6 GB RAM and runs on CPU or Apple MPS. Pre-computed embeddings in `datasets/` let you skip WhAM inference for Phases 1–2.

---

## Paper

The final paper (`paper/DCCE_Beyond_WhAM.pdf`) was compiled from LaTeX source using the `rho-class` two-column journal template. To recompile from source, see the LaTeX source in the original project repository.

---

## References

- Paradise et al. 2025 — WhAM: Whale Acoustic Model
- Sharma et al. 2024 — Combinatorial structure of sperm whale codas (DominicaCodas.csv)
- Beguš et al. 2024 — Vowel-like categories in sperm whale codas
- Leitão et al. 2025 — Social learning and rhythmic micro-variations
- Gero et al. 2016 — Individual, unit and vocal clan level identity cues in codas
- Chen et al. 2020 — SimCLR / NT-Xent loss
- Radford et al. 2021 — CLIP cross-modal contrastive learning
