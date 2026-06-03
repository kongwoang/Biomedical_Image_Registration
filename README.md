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
# BioMedReg

BioMedReg is a reproducible biomedical image registration repository that provides:

- classical registration baselines with SimpleITK and optional ANTsPyX support
- a from-scratch PSO-based registration baseline
- lightweight VoxelMorph-style and TransMorph-style PyTorch baselines
- synthetic-data generators and small smoke-test pipelines for repeatable runs

The code is designed to be runnable end to end on small synthetic examples, and to support compact OASIS-1 FreeSurfer preprocessing for 3D smoke tests and benchmarking.

## Reproducibility Goals

This repository is intended to make it easy to:

1. install dependencies
2. generate synthetic datasets
3. train or run each method
4. validate the full pipeline with tests and smoke tests

The learning-based models are intentionally small baselines. They are not exact reproductions of the original VoxelMorph or TransMorph training recipes.

## Environment Setup

Create and activate a local virtual environment, then install dependencies:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

If your system already provides PyTorch, you may also create the environment with system packages enabled:

```bash
python -m venv --system-site-packages .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Quick Reproduction Path

### 1. Generate synthetic data

```bash
python scripts/make_synthetic_data.py --out data/synthetic --num_pairs 20 --size 128
```

Each generated pair typically includes:

- fixed and moving NIfTI volumes
- affine ground-truth metadata
- displacement-field ground truth for synthetic data
- a preview image
- a dataset manifest

### 2. Run a method

Classical registration:

```bash
python -m src.run_method --method classical --fixed data/synthetic/fixed_000.nii.gz --moving data/synthetic/moving_000.nii.gz --out outputs/classical
```

PSO registration:

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

### 3. Validate the pipeline

Run the full smoke test suite:

```bash
bash scripts/run_all_smoke_tests.sh
```

Expected success message:

```text
ALL 4 METHODS SMOKE TEST PASSED
```

## OASIS-1 FreeSurfer Reproduction Path

For 3D experiments, this repo supports OASIS-1 FreeSurfer subject data. These archives contain MRI volumes and segmentation labels such as `aseg.mgz` and `aparc+aseg.mgz`.

### 1. Download and extract FreeSurfer subjects

```bash
python scripts/download_freesurfer_discs.py --disc 1-11
```

By default, this extracts only the files needed for the smoke-test workflow:

- `mri/T1.mgz`
- `mri/norm.mgz`
- `mri/brain.mgz`
- `mri/brainmask.mgz`
- `mri/aseg.mgz`
- `mri/aparc+aseg.mgz`
- `label/*.label`

Extracted subject files are written to `data/oasis/freesurfer/subjects/`.

### 2. Prepare small 3D pairs

```bash
python scripts/prepare_freesurfer_3d.py --subjects-root data/oasis/freesurfer/subjects --out data/oasis/freesurfer_3d_smoke --num_pairs 3 --size 32
```

### 3. Run the FreeSurfer smoke test

```bash
bash scripts/run_freesurfer_smoke_tests.sh
```

Expected success message:

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

Note: real OASIS-1 FreeSurfer data provide anatomical labels, but not dense ground-truth deformation fields. The prepared dataset stores source subject metadata instead of synthetic ground-truth transforms.

## Testing

Run unit tests with:

```bash
python -m pytest -q
```

## Benchmarking

Run the small functional benchmark on prepared NIfTI pairs:

```bash
python scripts/run_benchmark.py --dataset-root data/oasis/freesurfer_3d_smoke --out outputs/benchmark/freesurfer_smoke
```

If CUDA is available, you can specify a GPU device for the learned methods:

```bash
python scripts/run_benchmark.py --dataset-root data/oasis/freesurfer_3d_smoke --out outputs/benchmark/freesurfer_smoke_gpu --device cuda:0
```

Useful outputs:

- `outputs/benchmark/.../benchmark_results.csv`
- `outputs/benchmark/.../benchmark_results.json`
- `outputs/benchmark/.../benchmark_summary.json`
- `outputs/benchmark/.../benchmark_summary.md`

This benchmark is meant to confirm that the pipeline runs and to provide initial similarity and timing numbers. It is not a final scientific evaluation protocol.

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
README.md
requirements.txt
```

## Notes

- External datasets are not required to reproduce the synthetic smoke tests.
- The dataset helper scripts document manual download steps when a dataset cannot be fetched automatically.
- Outputs are written under `outputs/` by default.
