# BioMedReg

Clean, runnable Python baselines for biomedical image registration. The repository is synthetic-data-first, and also supports compact 3D OASIS-1 FreeSurfer preprocessing for smoke tests and later benchmarking.

1. Classical optimization-based registration with SimpleITK Demons or Diffeomorphic Demons, plus an optional ANTsPyX SyN wrapper when `antspyx` is installed.
2. Metaheuristic registration with a from-scratch PSO optimizer for 2D rigid/affine and 3D rigid transforms.
3. A minimal VoxelMorph-style PyTorch model with 2D and 3D UNet encoder-decoder variants, dense displacement output, `grid_sample` spatial transformer, image similarity loss, and smoothness regularization.
4. A minimal TransMorph-style PyTorch model with 2D and 3D patch embedding, Transformer encoder, convolutional decoder, dense displacement output, and the same spatial transformer/loss setup.

The included learning models are intentionally small. They are runnable baselines, not exact reproductions of the original VoxelMorph or TransMorph training regimes.

## Setup

Create a local virtual environment inside the repository:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

If your machine already provides PyTorch and you only need missing packages in the local environment, `python -m venv --system-site-packages .venv` is also usable.

## Synthetic Data

Generate medical-like 2D fixed/moving pairs:

```bash
python scripts/make_synthetic_data.py --out data/synthetic --num_pairs 20 --size 128
```

Each pair includes:

- `fixed_000.nii.gz`
- `moving_000.nii.gz`
- `gt_affine_000.json`
- `gt_displacement_000.npy`
- `preview_000.png`
- `dataset.json`

The fixed images are generated from ellipses, blobs, rings, and low-frequency texture. The moving image is produced by applying a known affine transform, a smooth deformation field, and mild intensity/noise changes.

## Run Methods

Classical Diffeomorphic Demons:

```bash
python -m src.run_method --method classical --fixed data/synthetic/fixed_000.nii.gz --moving data/synthetic/moving_000.nii.gz --out outputs/classical
```

PSO affine registration:

```bash
python -m src.run_method --method pso --fixed data/synthetic/fixed_000.nii.gz --moving data/synthetic/moving_000.nii.gz --out outputs/pso
```

Train and run VoxelMorph:

```bash
python -m src.methods.voxelmorph.train --config configs/voxelmorph.yaml
python -m src.run_method --method voxelmorph --fixed data/synthetic/fixed_000.nii.gz --moving data/synthetic/moving_000.nii.gz --checkpoint outputs/voxelmorph/best.pt --out outputs/voxelmorph
```

Train and run TransMorph:

```bash
python -m src.methods.transmorph.train --config configs/transmorph.yaml
python -m src.run_method --method transmorph --fixed data/synthetic/fixed_000.nii.gz --moving data/synthetic/moving_000.nii.gz --checkpoint outputs/transmorph/best.pt --out outputs/transmorph
```

All methods save:

- `registered.nii.gz`
- `deformation_field.npy` and `deformation_field.nii.gz`, or `transform_params.json` for PSO
- `overlay.png`
- `metrics.json`
- `log.json`

Metrics include before/after MSE and NCC between the fixed image and moving or registered image.

## Smoke Test

Run all four methods on a tiny synthetic dataset:

```bash
bash scripts/run_all_smoke_tests.sh
```

The expected final line is:

```text
ALL 4 METHODS SMOKE TEST PASSED
```

## OASIS-1 FreeSurfer Data

Use OASIS-1 FreeSurfer, not OASIS raw, for this branch of work. The FreeSurfer archives contain 3D MGZ MRI volumes and segmentation labels such as `aseg.mgz` and `aparc+aseg.mgz`.

Download, extract the useful MRI/label files, and delete each `.tar.gz` after successful extraction:

```bash
python scripts/download_freesurfer_discs.py --disc 1-11
```

By default this extracts only:

- `mri/T1.mgz`
- `mri/norm.mgz`
- `mri/brain.mgz`
- `mri/brainmask.mgz`
- `mri/aseg.mgz`
- `mri/aparc+aseg.mgz`
- `label/*.label`

Extracted files are placed under `data/oasis/freesurfer/subjects/`. Archives are kept under `data/oasis/freesurfer/raw/` only while they are being downloaded or extracted.

Prepare small 3D NIfTI pairs from extracted FreeSurfer subjects:

```bash
python scripts/prepare_freesurfer_3d.py --subjects-root data/oasis/freesurfer/subjects --out data/oasis/freesurfer_3d_smoke --num_pairs 3 --size 32
```

Run the four-method FreeSurfer smoke test:

```bash
bash scripts/run_freesurfer_smoke_tests.sh
```

The expected final line is:

```text
ALL 4 FREESURFER METHODS SMOKE TEST PASSED
```

Prepared FreeSurfer pairs include:

- `fixed_000.nii.gz`
- `moving_000.nii.gz`
- `fixed_label_000.nii.gz`
- `moving_label_000.nii.gz`
- `preview_000.png`
- `pair_000.json`
- `dataset.json`

Real OASIS-1 FreeSurfer data provide anatomical labels, but not dense ground-truth deformation fields. The prepared dataset records source subject metadata instead of synthetic ground truth transforms.

## Dataset Helpers

External datasets are not required for the synthetic milestone. The older helper remains only to report manual access requirements:

```bash
python scripts/download_datasets.py --dataset oasis --out data
python scripts/download_datasets.py --dataset dirlab --out data
python scripts/download_datasets.py --dataset all --out data
```

DIR-Lab 4DCT case packets are password protected. Complete the Emory DIR-Lab access request form, download the provided case packets, extract them, and place files under `data/dirlab/`. The helper does not create placeholder datasets or claim that data are available when manual access is required.

## Tests

```bash
python -m pytest -q
```

## Benchmark

Run a small functional benchmark over prepared NIfTI pairs. This trains the learned methods first, then runs all four methods and writes per-pair CSV/JSON plus a Markdown summary:

```bash
python scripts/run_benchmark.py --dataset-root data/oasis/freesurfer_3d_smoke --out outputs/benchmark/freesurfer_smoke
```

Use `--device cuda` or `--device cuda:0` for VoxelMorph/TransMorph training and inference when a CUDA GPU is available:

```bash
python scripts/run_benchmark.py --dataset-root data/oasis/freesurfer_3d_smoke --out outputs/benchmark/freesurfer_smoke_gpu --device cuda:0
```

Useful outputs:

- `outputs/benchmark/oasis/benchmark_results.csv`
- `outputs/benchmark/oasis/benchmark_results.json`
- `outputs/benchmark/oasis/benchmark_summary.json`
- `outputs/benchmark/oasis/benchmark_summary.md`

This benchmark is intended to verify runnable behavior and provide initial timing/similarity numbers. It is not a final scientific benchmark protocol.

## Repository Layout

```text
src/
  data/
  utils/
  methods/
    classical/
    metaheuristic/
    voxelmorph/
    transmorph/
scripts/
configs/
tests/
outputs/
README.md
requirements.txt
```
