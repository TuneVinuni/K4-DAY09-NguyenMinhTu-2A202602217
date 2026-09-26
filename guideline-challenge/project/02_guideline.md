# Annotation guideline — Drivable Area Segmentation

**Version:** v1

## 1. Objective + scope

Phân vùng **drivable area** (vùng xe ego được phép và có thể chạy) bằng polygon trên ảnh dashcam BDD100K. Giúp hệ thống tự hành ADAS lập kế hoạch đường đi (path planning) an toàn.
- **Trong scope (bắt buộc label):** Mặt đường nhựa/bê tông mà ego được đi. Bao gồm làn ego, các làn cùng chiều, và giao lộ phía trước.
- **Ngoài scope (ignore):** Làn ngược chiều bên kia dải phân cách/vạch đôi; vỉa hè, bãi cỏ; làn đỗ xe đang có xe đỗ; shoulder ngoài vạch trắng; gore area (vạch gạch chéo); nóc/capo xe camera. Mặt đường cách chân polygon > 50px hoặc trên điểm tụ chân trời.

## 2. Annotation unit

- **Image** độc lập tĩnh. Vẽ **Polygon** cho mỗi vùng mặt đường (region). 

## 3. Geometry rule

- **Drivable Area**: Dùng Polygon, vẽ bám theo mép vạch liền, mép vỉa hè hoặc mép tuyết.
  - **Tolerance**: Ở nửa dưới ảnh, mỗi cạnh lệch **≤ 5 px** là đạt. Ở vùng xa gần điểm tụ chân trời, lệch ≤ 10 px.
  - Cạnh đáy của polygon phải dừng mép ở mui xe/capo (không vẽ đè lên capo).
- **Phần bị xe khác che khuất**: Phần mặt đường nằm DƯỚI gầm xe / LỐP xe của xe khác vẫn tính là drivable. Khuyến khích **Vẽ trùm qua luôn xe đang đè lên mặt đường** (không đục lỗ) để tiết kiệm thời gian.

## 4. Taxonomy

- **Class `drivable_direct` (Polygon)**: Làn đường mà xe ego đang chạy trực tiếp.
- **Class `drivable_alternative` (Polygon)**: Các làn đường cùng chiều kế bên mà xe ego có thể chuyển làn sang.
  - **Attribute `boundary_visibility`**: `clear` (thấy rõ mép) / `occluded` (bị xe khác che 1 phần) / `unknown` (tối/tuyết không rõ).
- **Class `ignore_region` (Polygon)**: Dùng để khoanh các vùng cực kỳ dễ nhầm lẫn (như lề đất, làn đỗ xe, gore area gạch chéo). Việc khoanh `ignore_region` chứng minh annotator đã nhìn thấy và cố tình loại trừ nó.
- **Class `escalate` (Tag)**: Gắn nhãn toàn khung hình khi gặp ca khó. Điền lý do vào attribute `reason`.

Bảng đầy đủ ở `03_ontology_and_cvat_setup.md` — hai nơi phải khớp nhau.

## 5. Inclusion / exclusion

- **Bắt buộc vẽ**: Tất cả mặt đường hợp lệ trong ảnh.
- **Bắt buộc Ignore**: Không vẽ Polygon Drivable lên vỉa hè, lề cỏ, hoặc làn ngược chiều. Thay vào đó có thể đè `ignore_region` lên các phần nhạy cảm này.

## 6. Visibility / occlusion

- **Bị che một phần (Occluded)**: Vẽ trùm qua bánh xe/thân xe, và đánh dấu `boundary_visibility=occluded`.
- **Trời tối / mưa lóa / tuyết phủ mép (Low confidence)**: Vẽ theo suy đoán tốt nhất của bạn nhưng PHẢI chọn `boundary_visibility=unknown`. 

## 7. Ambiguity / escalation

- Gắn tag **`escalate`** cho ảnh nếu đường phủ tuyết trắng xóa hoặc tối đen đến mức không thể phân biệt được đâu là đường, đâu là lề.
- Kèm theo lý do vào ô `reason` (Ví dụ: "Tuyết mù không thấy lề").
- Khi đã dùng tag `escalate`, **KHÔNG** cần vẽ vùng Drivable cho khu vực tranh cãi đó nữa (Nghiêng về an toàn).

## 8. Temporal rule

Không áp dụng — task ảnh tĩnh.

## 9. Examples

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| BDD01 | Đường cao tốc, làn ego đang chạy | Vẽ `drivable_direct` với `boundary_visibility=clear`. | Bám sát mép vạch liền. |
| BDD04 | Làn đỗ xe có xe đỗ lề đường | Khoanh Polygon `ignore_region` bao lên vùng đỗ xe này. | Làn đỗ xe không được chạy vào (Tránh false positive). |
| BDD24 | Tuyết phủ kín không rõ mép vỉa hè | Gắn tag `escalate` và ghi `reason`="Tuyết che lề". | Nếu không chắc chắn, không vẽ drivable area. |

## 10. Common mistakes

- **Vẽ lấn lên lề/vỉa hè hoặc gore area (Vạch gạch chéo)**: Lỗi **CRITICAL**. Xe sẽ đâm lên lề. Bắt buộc dùng `ignore_region` để che đi.
- **Cố gắng đục lỗ (né) các xe trên đường**: Tốn rất nhiều thời gian vô ích. Hãy vẽ thẳng polygon xuyên qua bánh xe/dưới gầm xe.
