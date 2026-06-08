# BioMedReg OASIS-1 3D Results and Visualization Report

Report date: 2026-06-01

This report describes the current final benchmark run and every main PNG/GIF in `visualize/`.

## Source Artifacts

- Final benchmark summary: `outputs/benchmark/oasis1_3d_functional_clean_affine_gpu0_retry_20260601_190859/benchmark_summary.md`
- Test rows: `outputs/benchmark/oasis1_3d_functional_clean_affine_gpu0_retry_20260601_190859/benchmark_results.csv`
- Validation rows: `outputs/benchmark/oasis1_3d_functional_clean_affine_gpu0_retry_20260601_190859/validation_results.csv`
- Training logs:
  - `outputs/benchmark/oasis1_3d_functional_clean_affine_gpu0_retry_20260601_190859/training/voxelmorph/training_log.json`
  - `outputs/benchmark/oasis1_3d_functional_clean_affine_gpu0_retry_20260601_190859/training/transmorph/training_log.json`
- Static figure folder: `visualize/oasis1_3d_functional_20260527_025659/`
- GIF folder: `visualize/oasis3d_benchmark_gifs/`

Note: the PNG folder name is kept stable, but the images have been overwritten using the final retrained benchmark above.

## Protocol

- Dataset: OASIS-1 FreeSurfer-derived 3D volumes.
- Subjects: 425.
- Total adjacent pairs: 424.
- Split: 297 train pairs, 42 validation pairs, 85 test pairs.
- Resolution: `64^3`.
- Dice labels: FreeSurfer `aseg/aparc+aseg` volumes.
- PSO setting: affine 3D, 16 particles, 30 iterations.
- Learned model training: VoxelMorph and TransMorph retrained for 20 epochs on `cuda:0`.
- Training was sequential, not parallel, because available GPUs were already under load.

## Main Test Results

| Method | Test pairs | Mean sec | MSE | Delta MSE | NCC | Delta NCC | Dice | Dice delta | Folding % |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Classical | 85 | 0.511668 | 0.013728 | 0.008073 | 0.732291 | 0.130908 | 0.164172 | 0.036924 | 0.000000 |
| PSO affine | 85 | 9.361914 | 0.010877 | 0.010924 | 0.793471 | 0.192088 | 0.330794 | 0.203545 | N/A |
| VoxelMorph | 85 | 0.363106 | 0.008916 | 0.012885 | 0.827261 | 0.225878 | 0.164668 | 0.037419 | 0.158458 |
| TransMorph | 85 | 0.385216 | 0.009916 | 0.011885 | 0.807193 | 0.205810 | 0.218955 | 0.091706 | 0.034584 |

## Per-pair Winner Counts

| Criterion | Classical | PSO | VoxelMorph | TransMorph | Main read |
|---|---:|---:|---:|---:|---|
| Best NCC | 8 | 10 | 54 | 13 | VoxelMorph wins most intensity-alignment cases. |
| Best Dice | 9 | 55 | 6 | 15 | PSO affine wins most anatomical-overlap cases. |
| Fastest runtime | 0 | 0 | 66 | 19 | Learned methods are fastest at inference. |
| Lowest folding among dense fields | 85 | N/A | 0 | 0 | Classical has the most stable dense deformation. |

## Training Results

| Method | Epochs | Train pairs | Device | Best loss | Training seconds |
|---|---:|---:|---|---:|---:|
| VoxelMorph | 20 | 297 | `cuda:0` | 0.010429 | 421.9039 |
| TransMorph | 20 | 297 | `cuda:0` | 0.010865 | 639.8388 |

## Overall Interpretation

- VoxelMorph is the best method for intensity metrics: lowest MSE and highest NCC.
- PSO affine is the best method for anatomical Dice, but it is much slower than the learned models.
- TransMorph is the better learned model for Dice and folding stability, but VoxelMorph is better for MSE/NCC.
- Classical is stable and has zero folding, but it is weaker on MSE/NCC/Dice.
- The important tradeoff is clear: VoxelMorph is best for fast intensity alignment, while PSO affine is best for label overlap.

## Static PNG Figures

| File | What it shows | Main use |
|---|---|---|
| [`summary_metrics.png`](visualize/oasis1_3d_functional_20260527_025659/summary_metrics.png) | Mean MSE, NCC, Dice, and runtime across methods. | Quick final comparison. |
| [`metric_distributions.png`](visualize/oasis1_3d_functional_20260527_025659/metric_distributions.png) | Boxplots over 85 test pairs. | Shows stability and spread, not only means. |
| [`runtime_quality_tradeoff.png`](visualize/oasis1_3d_functional_20260527_025659/runtime_quality_tradeoff.png) | Runtime versus NCC/Dice. | Shows PSO accuracy-cost tradeoff and learned-model speed. |
| [`label_dice_heatmap.png`](visualize/oasis1_3d_functional_20260527_025659/label_dice_heatmap.png) | Mean Dice by anatomical label. | Shows which structures align well or poorly. |
| [`qualitative_pair_073.png`](visualize/oasis1_3d_functional_20260527_025659/qualitative_pair_073.png) | Fixed, registered, overlay, and difference panels for pair 073. | Visual inspection of registration quality. |
| [`deformation_pair_073.png`](visualize/oasis1_3d_functional_20260527_025659/deformation_pair_073.png) | Displacement, Jacobian, and folding for dense methods. | Checks deformation behavior and local folding. |
| [`training_loss_curves.png`](visualize/oasis1_3d_functional_20260527_025659/training_loss_curves.png) | Loss, image loss, and smoothness over 20 epochs. | Confirms both learned models trained normally. |
| [`training_summary.png`](visualize/oasis1_3d_functional_20260527_025659/training_summary.png) | Training time, best loss, and train pair count. | Confirms clean retrain used 297 train pairs. |
| [`validation_vs_test_learned.png`](visualize/oasis1_3d_functional_20260527_025659/validation_vs_test_learned.png) | Validation and test comparison for learned models. | Checks validation-test consistency. |
| [`method_win_counts.png`](visualize/oasis1_3d_functional_20260527_025659/method_win_counts.png) | Per-pair winner counts. | Shows which method wins each metric most often. |
| [`ncc_vs_dice_scatter.png`](visualize/oasis1_3d_functional_20260527_025659/ncc_vs_dice_scatter.png) | Pair-level NCC versus Dice. | Shows NCC and Dice are related but not identical objectives. |

## GIFs

### 01 Moving-to-registered Fade

Folder: `visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/`

These GIFs fade from moving to registered output for three cases: best NCC gain `pair_068`, median NCC `pair_033`, and worst NCC gain `pair_105`.

| GIF | What it shows |
|---|---|
| [`best_delta_ncc_pair_068_classical_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/best_delta_ncc_pair_068_classical_fade.gif) | Classical fade on best NCC-gain case. |
| [`best_delta_ncc_pair_068_pso_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/best_delta_ncc_pair_068_pso_fade.gif) | PSO affine fade on best NCC-gain case. |
| [`best_delta_ncc_pair_068_voxelmorph_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/best_delta_ncc_pair_068_voxelmorph_fade.gif) | VoxelMorph fade on best NCC-gain case. |
| [`best_delta_ncc_pair_068_transmorph_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/best_delta_ncc_pair_068_transmorph_fade.gif) | TransMorph fade on best NCC-gain case. |
| [`median_after_ncc_pair_033_classical_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/median_after_ncc_pair_033_classical_fade.gif) | Classical fade on median case. |
| [`median_after_ncc_pair_033_pso_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/median_after_ncc_pair_033_pso_fade.gif) | PSO affine fade on median case. |
| [`median_after_ncc_pair_033_voxelmorph_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/median_after_ncc_pair_033_voxelmorph_fade.gif) | VoxelMorph fade on median case. |
| [`median_after_ncc_pair_033_transmorph_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/median_after_ncc_pair_033_transmorph_fade.gif) | TransMorph fade on median case. |
| [`worst_delta_ncc_pair_105_classical_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/worst_delta_ncc_pair_105_classical_fade.gif) | Classical fade on worst NCC-gain case. |
| [`worst_delta_ncc_pair_105_pso_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/worst_delta_ncc_pair_105_pso_fade.gif) | PSO affine fade on worst NCC-gain case. |
| [`worst_delta_ncc_pair_105_voxelmorph_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/worst_delta_ncc_pair_105_voxelmorph_fade.gif) | VoxelMorph fade on worst NCC-gain case. |
| [`worst_delta_ncc_pair_105_transmorph_fade.gif`](visualize/oasis3d_benchmark_gifs/01_moving_to_registered_fade/worst_delta_ncc_pair_105_transmorph_fade.gif) | TransMorph fade on worst NCC-gain case. |

### 02 Axial Slice Sweep

Folder: `visualize/oasis3d_benchmark_gifs/02_axial_slice_sweep/`

These GIFs sweep across axial slices for the representative median case `pair_033`.

| GIF | What it shows |
|---|---|
| [`pair_033_classical_axial_sweep.gif`](visualize/oasis3d_benchmark_gifs/02_axial_slice_sweep/pair_033_classical_axial_sweep.gif) | Classical registered volume across slices. |
| [`pair_033_pso_axial_sweep.gif`](visualize/oasis3d_benchmark_gifs/02_axial_slice_sweep/pair_033_pso_axial_sweep.gif) | PSO affine registered volume across slices. |
| [`pair_033_voxelmorph_axial_sweep.gif`](visualize/oasis3d_benchmark_gifs/02_axial_slice_sweep/pair_033_voxelmorph_axial_sweep.gif) | VoxelMorph registered volume across slices. |
| [`pair_033_transmorph_axial_sweep.gif`](visualize/oasis3d_benchmark_gifs/02_axial_slice_sweep/pair_033_transmorph_axial_sweep.gif) | TransMorph registered volume across slices. |

### 03 Final Field Scale

Folder: `visualize/oasis3d_benchmark_gifs/03_final_field_scale/`

These GIFs gradually apply dense deformation strength for methods that have dense fields. PSO is absent here because affine PSO has a global transform, not a dense deformation field.

| GIF | What it shows |
|---|---|
| [`pair_033_classical_field_scale.gif`](visualize/oasis3d_benchmark_gifs/03_final_field_scale/pair_033_classical_field_scale.gif) | Classical deformation applied from 0 to full strength. |
| [`pair_033_voxelmorph_field_scale.gif`](visualize/oasis3d_benchmark_gifs/03_final_field_scale/pair_033_voxelmorph_field_scale.gif) | VoxelMorph deformation applied from 0 to full strength. |
| [`pair_033_transmorph_field_scale.gif`](visualize/oasis3d_benchmark_gifs/03_final_field_scale/pair_033_transmorph_field_scale.gif) | TransMorph deformation applied from 0 to full strength. |

### 04 Label Contours

Folder: `visualize/oasis3d_benchmark_gifs/04_label_contours/`

These GIFs overlay fixed and registered label contours across slices for `pair_033`. This is useful for anatomical alignment, but it is less visually dramatic than the fade/sweep GIFs.

| GIF | What it shows |
|---|---|
| [`pair_033_classical_label_contours.gif`](visualize/oasis3d_benchmark_gifs/04_label_contours/pair_033_classical_label_contours.gif) | Classical label contour alignment. |
| [`pair_033_pso_label_contours.gif`](visualize/oasis3d_benchmark_gifs/04_label_contours/pair_033_pso_label_contours.gif) | PSO affine label contour alignment. |
| [`pair_033_voxelmorph_label_contours.gif`](visualize/oasis3d_benchmark_gifs/04_label_contours/pair_033_voxelmorph_label_contours.gif) | VoxelMorph label contour alignment. |
| [`pair_033_transmorph_label_contours.gif`](visualize/oasis3d_benchmark_gifs/04_label_contours/pair_033_transmorph_label_contours.gif) | TransMorph label contour alignment. |

### 05 PSO True Iterations

Folder: `visualize/oasis3d_benchmark_gifs/05_true_iterations/`

These are true PSO affine optimization animations. Other methods are intentionally absent because Classical, VoxelMorph, and TransMorph do not have the same PSO-style particle iteration process.

| GIF | What it shows |
|---|---|
| [`pair_033_pso_affine_true_iterations.gif`](visualize/oasis3d_benchmark_gifs/05_true_iterations/pair_033_pso_affine_true_iterations.gif) | PSO affine search over 40 visualization frames for representative `pair_033`. |
| [`pair_255_pso_affine_true_iterations.gif`](visualize/oasis3d_benchmark_gifs/05_true_iterations/pair_255_pso_affine_true_iterations.gif) | PSO affine search over 40 visualization frames for a high Dice-gain case. |

