# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Ai review, review bao nhiêu:**
  - Reviewer chính: Lê Hùng Cường.
  - Khối lượng review: Review **100%** toàn bộ các ảnh trong đợt Blind Handoff và Calibration; đối với các ảnh sản xuất đại trà review **50%** (trong đó ưu tiên kiểm tra 100% ảnh rủi ro cao).
- **Chọn sample theo rule nào** (Risk-based sampling):
  - Kiểm tra 100% các ảnh mang tag nguy cơ cao: `critical` (tuyết phủ BDD24), `low_visibility` (mưa phản chiếu BDD17, ban đêm BDD18), `edge` (chạng vạng BDD22, BDD25) và `ambiguity` (đường hẹp khu dân cư BDD20).
  - Chọn ngẫu nhiên (random) 20% các ảnh điều kiện chuẩn (`normal` như BDD01, BDD15, BDD16).
- **Issue được ghi ở đâu, đóng thế nào:**
  - Ghi nhận trực tiếp qua tính năng Issue/Comment trên từng polygon của CVAT hoặc ghi vào bảng QA Log chung của nhóm.
  - Issue chỉ được đóng (`Closed`) sau khi annotator đã sửa lại polygon đạt chuẩn và reviewer bấm duyệt (`Accepted`).
- **Khi phát hiện guideline gap thì update và version ra sao:**
  - Tạm dừng việc review cá nhân, chuyển trường hợp nghi vấn lên cho Spec Lead.
  - Họp nhóm thống nhất bổ sung quy tắc mới vào `02_guideline.md`, tăng version (`v1` → `v2` → `v3`), và ghi rõ lý do kèm bằng chứng vào `08_revision_log.md`.

## Defect severity

Nhóm được đổi mapping nếu downstream contract khác, nhưng phải giải thích và chốt trước khi QA.

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Lỗi sai lệch ngữ nghĩa trực tiếp đe dọa an toàn xe tự hành: vẽ nhầm vùng cấm thành drivable (False Positive trèo lên vỉa hè, lấn sang làn ngược chiều qua vạch liền), hoặc tráo đổi nhầm giữa direct và alternative khiến xe bẻ lái đột ngột. | Vẽ `drivable_direct` lấn qua vạch vàng kép sang làn xe ngược chiều ở ảnh BDD24 tuyết phủ; hoặc tráo ngược direct thành alternative ở BDD03. | Bắt buộc **REWORK** ngay lập tức; annotator phải sửa lại trước khi merge. Nếu lặp lại 2 lần thì REJECT task. |
| Major | Bỏ sót hoàn toàn một làn xe cùng chiều hợp lệ (`drivable_alternative`), hoặc không vẽ `ignore_region` cho các vùng cấm trên đường như vạch gạch chéo gore area. | Bỏ sót làn cùng chiều kế bên khi có bóng râm cây che ở BDD03; hoặc không vẽ `ignore_region` cho ô đỗ xe ven đường ở BDD04. | Yêu cầu **REWORK**; annotator phải bổ sung polygon còn thiếu trước khi được tính hoàn thành. |
| Minor | Sai số hình học nhỏ (mép polygon lệch 3–5px so với mép vạch sơn, hoặc cạnh đáy chưa cắt sát viền nắp ca-pô) nhưng không làm sai lệch bản chất làn đường. | Cạnh đáy polygon vẽ đè lên nắp ca-pô xe ego 3px; hoặc viền polygon bị răng cưa nhẹ ở phía xa. | Reviewer tự nắn chỉnh nhanh trên CVAT hoặc ghi chú nhắc nhở annotator cải thiện ở batch sau mà không cần trả bài. |
| Question | Tình huống điều kiện môi trường quá khắc nghiệt (tuyết phủ kín hoàn toàn, ngã tư không còn vết tích vạch sơn) khiến annotator không đủ bằng chứng suy đoán. | Mặt đường phủ tuyết trắng xóa ở BDD24 không rõ mép rãnh thoát nước ở đâu. | **ESCALATE** lên Spec Lead để họp nhóm chốt ranh giới vùng an toàn tối thiểu theo vết bánh xe trước khi duyệt. |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| **mIoU (Mean Intersection over Union)** | Diện tích phần giao nhau / Diện tích phần hợp nhau giữa polygon annotator vẽ và polygon chuẩn đối chiếu: $IoU = \frac{|A \cap B|}{|A \cup B|}$. | Bài toán Drivable Area là segmentation vùng không gian liên tục; mIoU phản ánh chính xác mức độ trùng khớp hình học của quỹ đạo đường đi cho module path planning. |
| **Critical Defect Rate (CDR)** | $\frac{\text{Tổng số lỗi Critical phát hiện}}{\text{Tổng số ảnh được review}}$. | Đo lường mức độ an toàn nghiêm ngặt; xe tự hành đòi hỏi Zero-Tolerance với lỗi lấn làn ngược chiều và trèo lên vỉa hè. |
| **Boundary Precision Error** | Khoảng cách pixel trung bình từ các đỉnh polygon đến ranh giới mép đường thực tế. | Đảm bảo mép ngoài của làn đường không bị lấn ra bãi cỏ hoặc rãnh thoát nước hai bên đường. |

Metric high-risk tách riêng (ví dụ critical defect escape rate):
- **Critical Defect Escape Rate = 0%**: Tuyệt đối không cho phép bất kỳ lỗi Critical nào lọt qua khâu QA vào tập dữ liệu cuối cùng bàn giao cho downstream model.

## Quality gate

Threshold là đề xuất của nhóm, không phải chuẩn ngành. Giải thích trade-off cost/risk.

```text
PASS if:
  - Critical defects = 0 (bắt buộc tuyệt đối trên toàn bộ ảnh review).
  - mIoU >= 85% đối với cả drivable_direct và drivable_alternative.
  - Major defects = 0.
  - Minor defects <= 2 lỗi / ảnh.

REWORK if:
  - Có đúng 1 lỗi Critical hoặc >= 1 lỗi Major.
  - Hoặc mIoU nằm trong khoảng 70% đến 84%.
  -> Trả về cho annotator sửa lại và nộp lại trong vòng 10 phút.

REJECT / ESCALATE if:
  - Xuất hiện từ 2 lỗi Critical trở lên, hoặc mIoU < 70%.
  - Hoặc annotator sau khi rework vẫn tái diễn lỗi cũ.
  - Hoặc phát sinh tranh chấp do guideline thiếu quy tắc -> Chuyển lên Spec Lead để cập nhật Guideline.
```

Trade-off: Nhóm chấp nhận tốn thêm thời gian và công sức để review 100% các ca rủi ro cao (tuyết, mưa, ban đêm) và áp mức chặn lỗi nghiêm ngặt `Critical Defect = 0` (Safety-first Downstream Contract), bởi vì xe tự hành chỉ cần 1 lần nhận diện nhầm làn ngược chiều là có thể gây tai nạn chết người. Đổi lại, nhóm nới lỏng dung sai hình học (Minor $\le$ 5px) ở các khu vực phía xa gần đường chân trời để giữ tốc độ gán nhãn của annotator kịp tiến độ 15 phút.
