# Problem statement + downstream contract

## Bài toán

Phân vùng **drivable area** (vùng xe ego được phép và có thể chạy) bằng polygon trên ảnh dashcam BDD100K. Phần khó
là **ranh giới với vùng trông giống mặt đường nhưng không được chạy**: lề hẹp (không đạt làn khẩn cấp), làn đỗ xe dọc phố, vùng gạch chéo
(gore area) ở nhánh tách cao tốc, vỉa hè hạ thấp. Thêm vào đó là mép đường bị tuyết che, bị tối hoặc mờ vì mưa.

## Downstream contract

1. **Downstream task / model / user là ai?** Model segmentation drivable area cho module **path planning** của xe tự
   hành. Planner chỉ sinh quỹ đạo bên trong vùng này, gồm làn hiện tại và làn có thể chuyển sang.
2. **Output annotation nào thực sự cần?** Geometry là **polygon** mặt đường. Class gồm `drivable_direct` (làn ego đang
   đi) và `drivable_alternative` (làn cùng chiều có thể chuyển sang). Vùng giống mặt đường nhưng không được chạy vẽ
   bằng `ignore_region`. Không có attribute. Polygon drivable **không vẽ đè lên xe**: planner cần vùng còn trống,
   không cần mặt đường bị xe chiếm.
3. **Failure nào gây hậu quả lớn nhất?** **False positive**: vẽ lề hẹp, làn đỗ xe, gore area, vỉa hè, làn ngược chiều
   hoặc thân xe khác thành drivable. **Ngoại lệ có chủ đích:** làn khẩn cấp đạt đủ 3 điều kiện ở guideline mục 5
   (liền mặt nhựa cùng chiều, rộng ≥ một xe con, không vật cản) được tính là `drivable_alternative`, vì planner cần
   không gian thoát hiểm khi làn chính bị chặn. Không chắc lề có đạt đủ 3 điều kiện hay không thì để trống, giữ nguyên
   tắc an toàn. Planner có thể lái xe vào đó. Đây là decision `critical`. Lỗi bỏ sót một phần mặt đường xa chỉ là
   `major` hoặc `minor`.
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?** Trong CVAT: **không vẽ** vùng đó là drivable
   (fail-safe). Annotator ghi sample_id và vùng không chắc vào clarification log. Spec owner quyết định và ghi kết quả
   thành edge case card trong `04_edge_cases/`.

## Scope

- **Trong scope (bắt buộc label):** mặt đường trải nhựa hoặc bê tông mà ego được đi theo luật. Gồm làn ego, các làn
  cùng chiều, làn khẩn cấp đủ điều kiện (guideline mục 5) và phần giao lộ phía trước. Polygon dừng tại chân xe phía
  trước; phần mặt đường bị xe che **không** vẽ.
- **Ngoài scope, vẽ `ignore_region`** (nằm trong lòng đường, trông như đường chạy nhưng cấm chạy):
  - Làn ngược chiều bên kia vạch vàng giữa đường.
  - Làn đỗ xe (có xe hay trống), làn xe đạp.
  - Gore area / vùng gạch chéo, và làn nằm bên kia vùng đó.
- **Ngoài scope, để trống:**
  - Lề hẹp hơn một xe con, vỉa hè, bãi cỏ, sỏi, đất.
  - Bãi xe hoặc trạm xăng sau đường bó vỉa.
  - Đường bên kia dải phân cách cứng.
  - Đường ngang trong giao lộ.
  - Nắp capo / táp-lô xe ego; xe, người, vật cản trên làn.
  - Mặt đường ở quá xa: làn hẹp dưới khoảng 20 px, bị vật cố định chắn, hoặc vượt đường chân trời.
- **Geometry tolerance:** polygon bám theo mép vạch liền, mép vỉa hè hoặc mép tuyết. Ở nửa dưới ảnh, mỗi cạnh lệch
  **≤ 5 px** là đạt. Ở vùng xa gần điểm tụ, lệch ≤ 10 px là đạt. Cạnh đáy polygon dừng ở mép capo.

## Output chấm được

Blind test chấm các loại decision sau, và mọi loại đều nằm trong file export CVAT (CVAT for images XML):

- **LABEL:** có polygon `drivable_direct` hoặc `drivable_alternative`.
- **IGNORE:** có polygon `ignore_region` phủ lên vùng dễ nhầm. Vùng hoàn toàn không phải đường thì không vẽ.
- **UNKNOWN / không chắc:** vùng đó không có polygon drivable; không xác định được làn ego thì ảnh không có polygon
  drivable nào.
- **Class và geometry:** so từng vùng với gold theo class, vị trí cạnh trên (chân xe phía trước) và IoU của polygon.

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
