# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v2 | Bổ sung quy tắc tâm xe (Ego Anchor Rule); làm rõ quy tắc không vẽ ignore_region cho lề đường ngoài vạch liền; xử lý vệt phản chiếu trời mưa | Kết quả đo calibration nội bộ 4 người cho thấy có bất đồng ở BDD17, BDD22 và BDD03 | BDD17, BDD22, BDD03 trong 06_calibration_report.csv |
