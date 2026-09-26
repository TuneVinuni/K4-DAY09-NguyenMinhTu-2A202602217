# Problem statement + downstream contract

## Bài toán

Phân vùng **drivable area** (vùng xe ego được phép và có thể chạy) bằng polygon trên ảnh dashcam BDD100K. Phần khó
là **ranh giới với vùng trông giống mặt đường nhưng không được chạy**: lề/shoulder, làn đỗ xe dọc phố, vùng gạch chéo
(gore area) ở nhánh tách cao tốc, vỉa hè hạ thấp. Thêm vào đó là mép đường bị tuyết che, bị tối hoặc mờ vì mưa.

## Downstream contract

1. **Downstream task / model / user là ai?** Model segmentation drivable area cho module **path planning** của xe tự
   hành. Planner chỉ sinh quỹ đạo bên trong vùng này, gồm làn hiện tại và làn có thể chuyển sang.
2. **Output annotation nào thực sự cần?** Geometry là **polygon** mặt đường. Class gồm `drivable_direct` (làn ego đang
   đi) và `drivable_alternative` (làn cùng chiều có thể chuyển sang). Vùng giống mặt đường nhưng không được chạy vẽ
   bằng `ignore_region`. Attribute `boundary_visibility` = `clear` / `occluded` / `unknown`.
3. **Failure nào gây hậu quả lớn nhất?** **False positive**: vẽ lề, làn đỗ xe, gore area, vỉa hè hoặc làn ngược chiều
   thành drivable. Planner có thể lái xe vào đó. Đây là decision `critical`. Lỗi bỏ sót một phần mặt đường xa chỉ là
   `major` hoặc `minor`.
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?** Annotator gắn tag `escalate` cho frame trong
   CVAT, kèm lý do ở attribute text `reason`. Spec owner quyết định và ghi kết quả thành edge case card trong
   `04_edge_cases/`. Chưa có quyết định thì **không** vẽ vùng đó là drivable (nghiêng về an toàn).

## Scope

- **Trong scope (bắt buộc label):** mặt đường trải nhựa hoặc bê tông mà ego được đi theo luật. Gồm làn ego, các làn
  cùng chiều và phần giao lộ phía trước. Phần mặt đường bị xe khác che vẫn tính là drivable.
- **Ngoài scope (ignore):**
  - Làn ngược chiều bên kia vạch vàng đôi hoặc dải phân cách.
  - Vỉa hè, bãi cỏ.
  - Làn đỗ xe đang có xe đỗ.
  - Shoulder ngoài vạch trắng liền của cao tốc.
  - Gore area gạch chéo.
  - Nắp capo xe ego và phản chiếu trên kính.
  - Mặt đường ở quá xa: phía trên điểm tụ, hoặc cách chân polygon quá khoảng 50 px chiều cao ảnh.
- **Geometry tolerance:** polygon bám theo mép vạch liền, mép vỉa hè hoặc mép tuyết. Ở nửa dưới ảnh, mỗi cạnh lệch
  **≤ 5 px** là đạt. Ở vùng xa gần điểm tụ, lệch ≤ 10 px là đạt. Cạnh đáy polygon dừng ở mép capo.

## Output chấm được

Blind test chấm 5 loại decision, và mọi loại đều nằm trong file export CVAT (CVAT for images XML):

- **LABEL:** có polygon `drivable_direct` hoặc `drivable_alternative`.
- **IGNORE:** có polygon `ignore_region` phủ lên vùng dễ nhầm. Vùng hoàn toàn không phải đường thì không vẽ.
- **UNKNOWN:** polygon drivable có `boundary_visibility = unknown`, dùng khi không xác định được mép đường (tuyết, đêm).
- **ESCALATE:** frame có tag `escalate` cùng `reason`.
- **Class, attribute và geometry:** so từng vùng với gold theo class, `boundary_visibility` và IoU của polygon.

## Dữ liệu và giới hạn

- **Nguồn:** `data/bdd100k`, dùng cả 26 ảnh BDD01–BDD26, kích thước 1280×720.
- **Giới hạn:** phần lớn ảnh là ban ngày, highway hoặc city street. Chỉ có ít ảnh khó:
  - 2 ảnh đêm (BDD18, BDD26)
  - 2 ảnh chạng vạng (BDD22, BDD25)
  - 2 ảnh tuyết (BDD23, BDD24)
  - 1 ảnh mưa (BDD17)
  - 2 ảnh residential (BDD20, BDD23)

  Vì vậy case "mép đường không nhìn thấy" rất ít, và cần giữ vài ảnh trong số này cho blind test.
- **Nhiễu cố định:** ảnh có capo và phản chiếu trên kính. Không có ảnh bãi đỗ xe hay đường đất nên hai trường hợp này
  không được kiểm. BDD100K có sẵn nhãn drivable gốc nhưng nhóm không dùng làm gold; gold do nhóm tự quyết theo guideline.
