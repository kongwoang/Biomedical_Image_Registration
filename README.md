# BioMedReg

BioMedReg is a compact biomedical image registration repository for a Computer Vision project. It provides reproducible baselines for registering fixed and moving medical images, with synthetic data generation for fast local runs and optional OASIS-1 FreeSurfer preprocessing for 3D experiments.

The repository includes:

- classical registration with SimpleITK Demons and Diffeomorphic Demons, plus optional ANTsPyX SyN support
- a from-scratch particle swarm optimization baseline for rigid/affine registration
- lightweight VoxelMorph-style and TransMorph-style PyTorch baselines
- synthetic 2D registration data generation with known transforms
- smoke tests, benchmark scripts, visualizations, and metric reports

The learned models are intentionally small runnable baselines. They are not exact reproductions of the original VoxelMorph or TransMorph training protocols.

## Repository Layout

```text
BioMedReg/
  configs/                 Training configs for synthetic, OASIS, and smoke runs
  scripts/                 Data preparation, smoke test, and benchmark scripts
  src/
    data/                  Dataset loading and synthetic data helpers
    methods/
      classical/           SimpleITK/ANTs registration wrappers
      metaheuristic/       PSO registration baseline
      voxelmorph/          VoxelMorph-style model, training, and inference
      transmorph/          TransMorph-style model, training, and inference
    utils/                 I/O, metrics, seeding, warping, visualization
    run_method.py          Main CLI for running one registration method
  tests/                   Unit tests
  requirements.txt
  README.md
```

Run commands from the `BioMedReg/` directory unless noted otherwise:

```bash
cd BioMedReg
```

## Environment Setup

Create and activate a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

If your machine already provides PyTorch, you can reuse system site packages:

```bash
python -m venv --system-site-packages .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Optional: install `antspyx` if you want to use the `ants_syn` classical backend. It is not required for the default smoke tests.

## Quick Reproduction

This path reproduces the project on synthetic data and does not require external datasets.

### 1. Generate Synthetic Data

```bash
python scripts/make_synthetic_data.py \
  --out data/synthetic \
  --num_pairs 20 \
  --size 128 \
  --seed 7
```

Generated files include:

- `fixed_000.nii.gz`
- `moving_000.nii.gz`
- `gt_affine_000.json`
- `gt_displacement_000.npy`
- `preview_000.png`
- `dataset.json`

### 2. Run Classical Registration

```bash
python -m src.run_method \
  --method classical \
  --fixed data/synthetic/fixed_000.nii.gz \
  --moving data/synthetic/moving_000.nii.gz \
  --out outputs/classical
```

Alternative classical backends:

```bash
python -m src.run_method \
  --method classical \
  --backend demons \
  --fixed data/synthetic/fixed_000.nii.gz \
  --moving data/synthetic/moving_000.nii.gz \
  --out outputs/classical_demons
```

```bash
python -m src.run_method \
  --method classical \
  --backend ants_syn \
  --fixed data/synthetic/fixed_000.nii.gz \
  --moving data/synthetic/moving_000.nii.gz \
  --out outputs/classical_ants
```

### 3. Run PSO Registration

```bash
python -m src.run_method \
  --method pso \
  --fixed data/synthetic/fixed_000.nii.gz \
  --moving data/synthetic/moving_000.nii.gz \
  --out outputs/pso \
  --particles 24 \
  --iterations 40 \
  --metric ncc \
  --transform affine
```

### 4. Train and Run VoxelMorph

```bash
python -m src.methods.voxelmorph.train --config configs/voxelmorph.yaml
```

```bash
python -m src.run_method \
  --method voxelmorph \
  --fixed data/synthetic/fixed_000.nii.gz \
  --moving data/synthetic/moving_000.nii.gz \
  --checkpoint outputs/voxelmorph/best.pt \
  --out outputs/voxelmorph_infer
```

### 5. Train and Run TransMorph

```bash
python -m src.methods.transmorph.train --config configs/transmorph.yaml
```

```bash
python -m src.run_method \
  --method transmorph \
  --fixed data/synthetic/fixed_000.nii.gz \
  --moving data/synthetic/moving_000.nii.gz \
  --checkpoint outputs/transmorph/best.pt \
  --out outputs/transmorph_infer
```

## Smoke Tests

Run the complete synthetic smoke test:

```bash
bash scripts/run_all_smoke_tests.sh
```

Expected final line:

```text
ALL 4 METHODS SMOKE TEST PASSED
```

This script generates a small synthetic dataset, runs all four methods, and checks that each method writes the expected outputs.

## Outputs

Most method runs write the following files under the selected output directory:

- `registered.nii.gz`: warped moving image
- `overlay.png`: fixed/moving/registered visual comparison
- `metrics.json`: before/after similarity metrics
- `log.json`: run metadata and output paths
- `deformation_field.npy` and `deformation_field.nii.gz`: dense field for classical and learned methods
- `transform_params.json`: transform parameters for PSO

The main reported metrics are mean squared error (MSE) and normalized cross-correlation (NCC), measured before and after registration.

## OASIS-1 FreeSurfer Reproduction

The synthetic path above is enough to verify the repository. For 3D experiments, this repo also supports OASIS-1 FreeSurfer subject archives. These archives contain MRI volumes and labels such as `T1.mgz`, `aseg.mgz`, and `aparc+aseg.mgz`.

### 1. Download and Extract FreeSurfer Data

```bash
python scripts/download_freesurfer_discs.py --disc 1-11
```

By default, the extraction keeps only the files needed by this project:

- `mri/T1.mgz`
- `mri/norm.mgz`
- `mri/brain.mgz`
- `mri/brainmask.mgz`
- `mri/aseg.mgz`
- `mri/aparc+aseg.mgz`
- `label/*.label`

Extracted subjects are written to:

```text
data/oasis/freesurfer/subjects/
```

### 2. Prepare Small 3D Registration Pairs

```bash
python scripts/prepare_freesurfer_3d.py \
  --subjects-root data/oasis/freesurfer/subjects \
  --out data/oasis/freesurfer_3d_smoke \
  --num_pairs 3 \
  --size 32
```

Prepared pairs include:

- `fixed_000.nii.gz`
- `moving_000.nii.gz`
- `fixed_label_000.nii.gz`
- `moving_label_000.nii.gz`
- `preview_000.png`
- `pair_000.json`
- `dataset.json`

Real OASIS-1 FreeSurfer data provide anatomical labels but not dense ground-truth deformation fields. The prepared dataset stores subject metadata instead of synthetic ground-truth transforms.

### 3. Run the FreeSurfer Smoke Test

```bash
bash scripts/run_freesurfer_smoke_tests.sh
```

Expected final line:

```text
ALL 4 FREESURFER METHODS SMOKE TEST PASSED
```

The script expects either extracted Disc 1 subjects at `data/oasis/freesurfer/subjects/disc1` or an archive at `data/oasis/freesurfer/raw/oasis_cs_freesurfer_disc1.tar.gz`.

## Benchmarking

Run a small functional benchmark over prepared NIfTI pairs:

```bash
python scripts/run_benchmark.py \
  --dataset-root data/oasis/freesurfer_3d_smoke \
  --out outputs/benchmark/freesurfer_smoke
```

Limit the number of pairs:

```bash
python scripts/run_benchmark.py \
  --dataset-root data/oasis/freesurfer_3d_smoke \
  --out outputs/benchmark/freesurfer_smoke \
  --num-pairs 2
```

Use existing checkpoints instead of retraining learned methods:

```bash
python scripts/run_benchmark.py \
  --dataset-root data/oasis/freesurfer_3d_smoke \
  --out outputs/benchmark/freesurfer_smoke \
  --skip-training \
  --voxelmorph-checkpoint outputs/freesurfer_smoke/voxelmorph_train/best.pt \
  --transmorph-checkpoint outputs/freesurfer_smoke/transmorph_train/best.pt
```

Benchmark outputs:

- `benchmark_results.csv`
- `benchmark_results.json`
- `benchmark_summary.json`
- `benchmark_summary.md`
- per-method run folders under `runs/`

The benchmark is intended to check pipeline functionality and produce initial timing/similarity numbers. It is not a final scientific evaluation protocol.

## Testing

Run unit tests:

```bash
python -m pytest -q
```

Run smoke tests:

```bash
bash scripts/run_all_smoke_tests.sh
```

## Configuration

Training configs live in `configs/`.

Useful starting points:

- `configs/voxelmorph.yaml`: synthetic VoxelMorph training
- `configs/transmorph.yaml`: synthetic TransMorph training
- `configs/voxelmorph_smoke.yaml`: tiny synthetic smoke config
- `configs/transmorph_smoke.yaml`: tiny synthetic smoke config
- `configs/voxelmorph_freesurfer_smoke.yaml`: tiny 3D FreeSurfer smoke config
- `configs/transmorph_freesurfer_smoke.yaml`: tiny 3D FreeSurfer smoke config

Edit `data.root`, `output_dir`, model size, epoch count, batch size, and device settings as needed.

## Notes

- External datasets are not required for the synthetic reproduction path.
- Outputs are written under `outputs/` by default.
- Dataset files are expected under `data/` by default.
- Commands using `python -m src...` should be run from the repository root directory `BioMedReg/`.
- The optional `ants_syn` backend requires `antspyx`; default classical runs use SimpleITK.
