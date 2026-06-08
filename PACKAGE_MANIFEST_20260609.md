# BioMedReg Package Manifest - 2026-06-09

This package is a code + results + visualization archive. It intentionally excludes
downloaded/prepared datasets and large generated registration volumes.

Included:
- Source code: `src/`
- Scripts: `scripts/`
- Configs: `configs/`
- Tests: `tests/`
- Project docs: `README.md`, `METHOD_OVERVIEW.md`, `VISUALIZE_REPORT.md`
- Requirements: `requirements.txt`
- Proposal PDF: `CV_Proposal.pdf`
- Benchmark result tables/summaries/logs from `outputs/benchmark/`
- Model checkpoints from `outputs/`: `*.pt`
- Runtime logs from `outputs/logs/`
- Visualization notebooks, PNGs, and GIFs from `visualize/`

Excluded:
- `data/`
- `.venv/`
- `.git/`, `.agents/`, `.codex/`
- Python/Jupyter/cache folders
- Existing zip files
- Large generated registration artifacts under `outputs/`: `*.nii.gz`, `*.npy`
- Visualization cache folder: `visualize/oasis3d_benchmark_gifs/cache_classical_increasing_iters/`

The full OASIS-1 FreeSurfer dataset and prepared 64^3 NIfTI pairs are not included.
