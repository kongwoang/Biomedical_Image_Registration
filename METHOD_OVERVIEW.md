# Tổng quan 4 phương pháp registration

File này giải thích 4 method trong benchmark OASIS-1 3D hiện tại theo cách dễ trình bày. Kết quả bên dưới lấy từ run cuối:

`outputs/benchmark/oasis1_3d_functional_clean_affine_gpu0_retry_20260601_190859/`

Lưu ý: VoxelMorph và TransMorph trong repo là bản functional/compact cho benchmark này, không phải reproduction đầy đủ theo paper gốc.

## Bài toán

Image registration cố gắng đưa ảnh moving về thẳng hàng với ảnh fixed.

Với mỗi cặp ảnh:

- Input: fixed MRI volume và moving MRI volume.
- Output: registered moving volume.
- Một số method còn xuất deformation field, tức là bản đồ cho biết từng voxel bị dịch chuyển thế nào.

Các metric chính:

- MSE: càng thấp càng tốt, đo sai khác intensity.
- NCC: càng cao càng tốt, đo mức tương đồng pattern intensity.
- Dice: càng cao càng tốt, đo overlap label giải phẫu.
- Runtime: càng thấp càng nhanh.
- Jacobian folding: càng thấp càng ổn định; folding cao nghĩa là deformation có vùng bị gập/lật.

## Kết quả hiện tại

| Method | MSE | NCC | Dice | Runtime/pair | Folding |
|---|---:|---:|---:|---:|---:|
| Classical | 0.013728 | 0.732291 | 0.164172 | 0.5117s | 0.0000 |
| PSO affine | 0.010877 | 0.793471 | 0.330794 | 9.3619s | N/A |
| VoxelMorph | 0.008916 | 0.827261 | 0.164668 | 0.3631s | 0.1585 |
| TransMorph | 0.009916 | 0.807193 | 0.218955 | 0.3852s | 0.0346 |

Kết luận nhanh:

- VoxelMorph tốt nhất về MSE/NCC, tức là align intensity tốt nhất.
- PSO affine tốt nhất về Dice, tức là overlap giải phẫu tốt nhất.
- TransMorph là learned method cân bằng hơn VoxelMorph về Dice và folding.
- Classical ổn định nhất về deformation vì folding bằng 0.

## So với kết quả trước

Thay đổi lớn nhất là PSO. Trước đó PSO dùng rigid transform; hiện tại PSO dùng affine transform và nhiều iteration hơn.

| Method | Thay đổi chính |
|---|---|
| Classical | Hầu như không đổi. |
| PSO | Dice tăng từ 0.279971 lên 0.330794; NCC tăng từ 0.755121 lên 0.793471; runtime tăng từ 1.7631s lên 9.3619s mỗi pair. |
| VoxelMorph | NCC/MSE tốt hơn nhẹ; folding giảm mạnh từ 0.3745 xuống 0.1585; Dice hơi giảm từ 0.169673 xuống 0.164668. |
| TransMorph | Dice tăng từ 0.187009 lên 0.218955; NCC gần như không đổi; folding tăng nhẹ từ 0.0288 lên 0.0346. |

Vậy câu trả lời ngắn là: có thay đổi đáng kể, nhưng chủ yếu nằm ở PSO affine và kết quả training sạch hơn. Ranking tổng quát vẫn giống: VoxelMorph thắng intensity, PSO thắng Dice, Classical thắng stability.

## So sánh nhanh 4 method

| Method | Loại | Transform | Có train không? | Ý tưởng chính | Mạnh nhất ở |
|---|---|---|---|---|---|
| Classical | Optimization truyền thống | Dense deformation | Không | Tối ưu deformation riêng cho từng pair | Stability/folding |
| PSO affine | Metaheuristic optimization | Global affine 3D | Không | Tìm rotation, translation, scale, shear tốt nhất | Dice |
| VoxelMorph | Neural network | Dense deformation | Có | CNN dự đoán deformation field một lần | MSE/NCC và tốc độ |
| TransMorph | Neural network | Dense deformation | Có | Transformer dùng context rộng hơn để dự đoán field | Cân bằng Dice/folding |

## 1. Classical

Implementation: `src/methods/classical/register.py`

Classical trong repo dùng SimpleITK Diffeomorphic Demons.

Cách hiểu đơn giản:

1. Bắt đầu từ ảnh moving.
2. So sánh moving với fixed.
3. Ước lượng một bước deformation nhỏ.
4. Warp moving image.
5. Lặp lại nhiều lần.

Điểm quan trọng:

- Không cần train.
- Có iteration thật ở test time.
- Tạo dense deformation field.
- Có thể tính Jacobian/folding.

Ưu điểm:

- Ổn định nhất trong benchmark hiện tại.
- Folding bằng 0.
- Dễ giải thích vì nó tối ưu trực tiếp trên từng cặp ảnh.

Nhược điểm:

- Không đạt MSE/NCC tốt nhất.
- Dice thấp hơn PSO affine và TransMorph.
- Chất lượng phụ thuộc vào số iteration, smoothing, và cấu hình optimizer.

Nên trình bày thế nào:

Classical là baseline ổn định. Nó không phải method mạnh nhất về alignment metric, nhưng là điểm tham chiếu tốt để xem deformation có bị gập hay không.

## 2. PSO affine

Implementation: `src/methods/metaheuristic/pso.py`

PSO là Particle Swarm Optimization. Trong benchmark mới, PSO dùng affine 3D thay vì rigid 3D.

Affine 3D gồm:

- Rotation.
- Translation.
- Scale.
- Shear.

Cách hiểu đơn giản:

1. Tạo nhiều candidate transform gọi là particles.
2. Mỗi particle thử một affine transform.
3. Chấm điểm transform bằng metric alignment.
4. Particle di chuyển dần về vùng có score tốt hơn.
5. Sau nhiều iteration, lấy transform tốt nhất.

Điểm quan trọng:

- Không cần train.
- Có iteration thật ở test time.
- Không tạo dense deformation field.
- Vì là global affine transform, nó không có folding kiểu dense nonlinear field.

Ưu điểm:

- Dice tốt nhất trong benchmark hiện tại: 0.330794.
- Dễ giải thích hơn dense deformation vì toàn bộ volume dùng một global transform.
- GIF true-iteration của PSO là có ý nghĩa thật, vì nó thể hiện quá trình optimizer tìm transform.

Nhược điểm:

- Chậm nhất: 9.3619s mỗi pair.
- Không mô hình hóa được biến dạng local.
- Kết quả phụ thuộc vào số particles, số iterations, metric, seed.

Nên trình bày thế nào:

PSO affine rất mạnh về overlap label, nhưng đổi lại runtime cao. Đây là method tốt khi muốn chứng minh global affine search có thể cải thiện Dice rõ ràng.

## 3. VoxelMorph

Implementation:

- Model: `src/methods/voxelmorph/model.py`
- Training/inference utility: `src/methods/deep_common.py`

VoxelMorph là learned dense registration method. Trong repo này, nó là bản compact 3D CNN/UNet-style.

Cách hiểu đơn giản:

1. Ghép fixed và moving làm input.
2. CNN học feature local.
3. Model predict dense flow field.
4. Spatial transformer warp moving image theo flow field.

Điểm quan trọng:

- Cần train.
- Lúc inference predict một lần, không tối ưu iterative.
- Có dense deformation field.
- Có thể tính Jacobian/folding.

Ưu điểm:

- MSE thấp nhất: 0.008916.
- NCC cao nhất: 0.827261.
- Runtime nhanh nhất trong benchmark hiện tại: 0.3631s mỗi pair.
- Folding đã giảm mạnh so với run trước.

Nhược điểm:

- Dice thấp hơn PSO affine và TransMorph.
- Có folding, dù đã giảm so với trước.
- High NCC không đồng nghĩa high Dice.

Nên trình bày thế nào:

VoxelMorph là method tốt nhất cho intensity alignment và tốc độ. Nhưng nếu mục tiêu chính là anatomical label overlap, PSO affine vẫn tốt hơn.

## 4. TransMorph

Implementation:

- Model: `src/methods/transmorph/model.py`
- Training/inference utility: `src/methods/deep_common.py`

TransMorph cũng là learned dense registration method. Khác VoxelMorph ở chỗ nó dùng Transformer-style context.

Cách hiểu đơn giản:

1. Chia input thành patch/features.
2. Transformer encode context rộng hơn CNN local.
3. Decode về full resolution.
4. Predict dense deformation field.
5. Spatial transformer warp moving image.

Điểm quan trọng:

- Cần train.
- Inference một lần, không có true iteration.
- Có dense deformation field.
- Có thể tính Jacobian/folding.

Ưu điểm:

- Dice tốt hơn VoxelMorph: 0.218955 so với 0.164668.
- Folding thấp hơn VoxelMorph: 0.0346 so với 0.1585.
- NCC cao hơn Classical và PSO.
- Runtime vẫn nhanh: 0.3852s mỗi pair.

Nhược điểm:

- Không thắng VoxelMorph về MSE/NCC.
- Không thắng PSO affine về Dice.
- Model phức tạp hơn VoxelMorph.

Nên trình bày thế nào:

TransMorph là method cân bằng trong nhóm learned methods. Nó không mạnh nhất về intensity như VoxelMorph, nhưng tốt hơn VoxelMorph ở Dice và deformation stability.

## Iteration có ý nghĩa thế nào?

| Method | Có true iteration ở test time? | GIF iteration có nên dùng không? |
|---|---|---|
| Classical | Có | Có thể dùng, nhưng là Demons iteration, không giống PSO. |
| PSO affine | Có | Có, đây là true optimizer progress. |
| VoxelMorph | Không | Không nên gọi là iteration; chỉ có thể visualize scaling final field. |
| TransMorph | Không | Không nên gọi là iteration; chỉ có thể visualize scaling final field. |

Điểm cần nói khi trình bày:

- PSO GIF trong folder `05_true_iterations` là thật sự có ý nghĩa.
- VoxelMorph và TransMorph predict ra kết quả một lần, nên không có quá trình iteration ở inference.
- Nếu có GIF cho learned method, nó chỉ là minh họa final deformation mạnh dần từ 0 đến 1, không phải quá trình model đang optimize.

## Kết luận trình bày

Không có một method thắng toàn bộ.

- Muốn MSE/NCC tốt và chạy nhanh: chọn VoxelMorph.
- Muốn Dice/anatomical overlap tốt: chọn PSO affine.
- Muốn learned method cân bằng hơn về Dice/folding: chọn TransMorph.
- Muốn deformation ổn định, không folding: chọn Classical.

Thông điệp chính: mỗi metric đang đo một mục tiêu khác nhau. NCC tốt không nhất thiết Dice tốt, và Dice tốt có thể phải đánh đổi bằng runtime.
