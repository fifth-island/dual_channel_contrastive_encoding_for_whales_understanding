# Large Artifacts

This repository intentionally does not commit the largest binary artifacts directly to GitHub.
They are either public upstream data, generated outputs, or pretrained model checkpoints that exceed normal Git hosting limits.

## Omitted from Git

| Path | Approx. size | How to recover |
|---|---:|---|
| `datasets/dswp_audio/` | 567 MB | Download the DSWP audio from HuggingFace `orrp/DSWP`. |
| `datasets/synthetic_audio/` | 152 MB | Regenerate with the Phase 4 notebook or copy from the local experiment archive. |
| `datasets/wham_embeddings_all_layers.npy` | 147 MB | Regenerate with `notebooks/phase2_wham_probing.ipynb`. |
| `wham/vampnet/models/coarse.pth` | 1.3 GB | Download WhAM/VampNet checkpoints from the upstream WhAM release. |
| `wham/vampnet/models/codec.pth` | 573 MB | Download WhAM/VampNet checkpoints from the upstream WhAM release. |

## Included in Git

The repository does include labels, metadata, reports, notebooks with outputs, the final paper PDF, WhAM source code, train/test split indices, and smaller feature caches such as `datasets/wham_embeddings.npy`, `datasets/X_mel_full.npy`, and `datasets/X_mel_synth_1000.npy`.
