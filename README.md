# gpu-train-images

Container images for training models on rented GPUs. Each folder is one image, built by GitHub Actions and published
to `ghcr.io/nhz-io/gpu-train-images/<folder>:lock-<12 hex>`; users pin it by digest.

Everything in an image is public and hash-checked: Ubuntu, PyPI and PyTorch wheels (hash-locked), Hugging Face
models at a pinned revision. No data, no keys.

| Image | For |
|---|---|
| `modernbert-cu128` | ModernBERT-base fine-tunes: torch 2.8.0+cu128, the training deps, ModernBERT-base. The host brings the NVIDIA driver (CUDA 12.8 or newer) |
