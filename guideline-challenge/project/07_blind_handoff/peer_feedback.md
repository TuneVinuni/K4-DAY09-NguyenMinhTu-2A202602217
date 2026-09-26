# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền. Thay mọi placeholder mới
là xong (gate G5).

- **Nhóm peer:** Nhóm 2 (Autonomous Driving Perception)
- **Người label blind:** Hoàng Minh (Peer Annotator)

## 1. Peer trả lời

1. Rule nào rõ nhất / giúp quyết định nhanh nhất? Quy tắc Ego Anchor (tâm nắp capo xác định đúng 1 polygon drivable_direct duy nhất) và quy tắc dừng tại chân xe phía trước chiếm ≥ 1/2 bề ngang làn rất rõ ràng, dứt khoát.
2. Rule nào mơ hồ hoặc phải tự suy diễn? Ở ảnh chạng vạng BDD25, ranh giới giữa vạch sơn mờ và bóng nước phản chiếu khó phân biệt khi ánh sáng yếu.
3. Sample nào khiến guideline "vỡ"? BDD24 tuyết dày che khuất mép đường, nhưng guideline v2 đã có nguyên tắc an toàn dừng ở mép chân tuyết nên vẫn xử lý tốt.
4. Attribute / default nào trong CVAT dễ gây thao tác sai? Không có attribute nào gây khó, setup 3 label polygon thuần túy rất tiện thao tác và tốc độ gán nhãn nhanh.
5. Một thay đổi cụ thể giúp annotator mới ít hỏi hơn? Thêm lưu ý phóng to (zoom ≥ 300%) để soi vạch sơn thật phân biệt với vệt bóng nước vào lúc hoàng hôn/chạng vạng.

## 2. Owner phân loại

Owner không tranh luận để bảo vệ guideline. Mỗi feedback và mỗi decision peer làm sai được xếp vào một hướng xử lý.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| BDD25 d3 vẽ lấn vệt bóng nước khi chạng vạng | guideline gap | accept + revise (bổ sung quy tắc zoom phân biệt vệt phản chiếu nước chạng vạng) | BDD25 trong transfer_score.csv và clarification_log.csv |
