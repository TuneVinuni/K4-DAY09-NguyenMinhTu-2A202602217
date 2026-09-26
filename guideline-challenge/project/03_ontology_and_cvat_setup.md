# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây.

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `drivable_direct` | polygon | Class | - | - | - | Vùng mặt đường của làn mà ego đang đi. |
| `drivable_alternative` | polygon | Class | - | - | - | Vùng mặt đường của các làn cùng chiều kế bên. |
| `boundary_visibility` | | Attribute | `__undefined__`, `clear`, `occluded`, `unknown` | `__undefined__` | false | Đánh giá độ rõ ràng của mép polygon. Bắt buộc user phải chọn thay vì để mặc định. |
| `ignore_region` | polygon | Class | - | - | - | Phủ lên các vùng dễ nhầm lẫn như lề đường, chỗ đỗ xe dọc phố, gore area. |
| `escalate` | tag | Class | - | - | - | Đánh dấu nguyên khung hình để gọi hỗ trợ từ Spec owner. |
| `reason` | | Attribute | text | `""` | false | Ghi chú lý do escalate để Reviewer đọc. |

## Class hay attribute

- `drivable_direct` và `drivable_alternative` tách thành Class riêng để dễ vẽ (mỗi polygon có một loại màu khác nhau).
- `boundary_visibility` là attribute dùng chung cho drivable area, thiết lập `__undefined__` làm mặc định để chống thiên kiến (annotator buộc phải tự chủ động suy nghĩ và chọn).

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): Đang chạy local CVAT 2.4+
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): `nhom4-calib-v1`
- **Guide của task đã dán `02_guideline.md`?** Có
- **Nhóm dùng Track hay Shape, vì sao:** Dùng Shape, vì bộ dữ liệu là các ảnh tĩnh độc lập (image).

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào escalate. Ghi lại ai test và chỗ họ vấp:

- Test bởi: Member 2
- Tool: Draw Polygon
- Escalation: Khi gặp tuyết mù không đoán được mép đường, sẽ tag escalate thay vì tự vẽ bừa.
