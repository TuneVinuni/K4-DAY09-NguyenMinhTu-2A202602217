# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây.

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `drivable_direct` | polygon | Class | - | - | - | Phần còn trống của làn ego, nối dài qua giao lộ, dừng tại chân xe phía trước. Planner cần tách làn hiện tại khỏi làn có thể chuyển sang. 1 polygon/ảnh |
| `drivable_alternative` | polygon | Class | - | - | - | Phần còn trống của mỗi làn cùng chiều liền kề (kể cả làn khẩn cấp đủ 3 điều kiện ở guideline mục 5), 1 polygon/làn. Planner dùng để sinh quỹ đạo chuyển làn / tránh khẩn cấp |
| `ignore_region` | polygon | Class | - | - | - | Vùng **trong lòng đường**, liền mặt nhựa nhưng cấm chạy: ngược chiều, làn đỗ, làn xe đạp, gạch chéo/gore. Là hard negative cho model; vùng ngoài lòng đường (vỉa hè, cỏ, lề hẹp) để trống |

Không có attribute. `03_cvat_labels.json` có đúng 3 label trên, `attributes: []`.

## Class hay attribute

- `drivable_direct` / `drivable_alternative` là **class**: downstream cần phân loại trực tiếp, QA rule khác nhau
  (direct đúng 1 polygon/ảnh, alternative 1 polygon/làn).
- `ignore_region` là **class**, không phải attribute của drivable: vùng này mang nghĩa ngược lại (cấm chạy). Nếu gộp vào
  drivable dưới dạng attribute thì chỉ cần quên chọn attribute là tạo ra false positive critical.
- **Bỏ attribute `boundary_visibility`** (có ở bản nháp trước): vì polygon drivable không vẽ đè lên xe (guideline mục
  6), không còn mép nào phải suy luận qua xe, nên giá trị `occluded` không còn nghĩa. Mép mờ do thời tiết thì xử lý
  bằng rule "co polygon vào trong" thay cho `unknown`. Bớt attribute cũng bớt một nguồn bất đồng khi calibration.
- **Bỏ tag `escalate`:** vùng không chắc chắn thể hiện bằng việc **không có polygon drivable** (fail-safe, guideline mục
  7). Đánh đổi: trong export không phân biệt được "annotator cố ý bỏ trống vì không chắc" với "annotator quên vẽ".

## Quyết định → CVAT

| Quyết định | Trong export |
|---|---|
| LABEL | polygon `drivable_direct` / `drivable_alternative` |
| IGNORE | polygon `ignore_region` (trong lòng đường); ngoài lòng đường thì không có polygon |
| UNKNOWN / không chắc | không có polygon drivable ở vùng đó |
| Không xác định được làn ego | ảnh không có polygon drivable nào |

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): Đang chạy local CVAT 2.4+
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): `nhom4-calib-v1`
- **Guide của task đã dán `02_guideline.md`?** Có
- **Nhóm dùng Track hay Shape, vì sao:** Dùng Shape, vì bộ dữ liệu là các ảnh tĩnh độc lập (image).

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào escalate. Ghi lại ai test và chỗ họ vấp:

- Test bởi: Member 2
- Tool: Draw Polygon
- Escalation: Khi gặp tuyết mù không đoán được làn ego, không vẽ polygon drivable nào (guideline mục 7) thay vì tự vẽ bừa.
