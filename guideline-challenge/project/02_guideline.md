# Annotation guideline — Phân vùng Drivable Area

**Version:** v1

## 1\. Objective \+ scope

* **Mục tiêu:** Phân vùng "drivable area" (vùng xe ego được phép và có thể chạy) trên ảnh dashcam. Dữ liệu này dùng để huấn luyện model segmentation cho module **path planning** của xe tự hành. Planner sẽ chỉ sinh quỹ đạo di chuyển an toàn bên trong vùng được vẽ.  
* **Scope (Phạm vi):**  
  * **Trong scope (Được phép đi):** Mặt đường trải nhựa hoặc bê tông mà xe ego được đi hợp pháp theo luật giao thông (bao gồm làn xe đang đi và các làn cùng chiều có thể chuyển sang).  
  * **Ngoài scope (Không label hoặc Ignore):** Bất kỳ khu vực nào không thuộc phần đường ego được phép đi.  
    * **Tuyệt đối không label (để trống hoàn toàn):** Các khu vực ngoài lề như vỉa hè, lề đường/shoulder, bãi cỏ.  
    * **Bắt buộc dùng `ignore_region`:** Những vùng nằm *trên mặt đường* nhưng xe không được phép đi vào (ví dụ: làn ngược chiều, vùng gạch chéo gore area, làn đỗ xe dọc phố).  
* **Nguyên tắc cốt lõi (Safety First):** Lỗi vẽ nhầm vùng ngoài scope thành vùng trong scope (False Positive) là lỗi **Critical** (rất nghiêm trọng) vì có thể khiến xe tự hành lao ra khỏi đường hoặc gây tai nạn. Thà bỏ sót còn hơn vẽ nhầm.

## 2\. Annotation unit

* **Đơn vị gán nhãn:** Khung ảnh tĩnh (Image frame).  
* **Loại nhãn:** Vùng (Region / Polygon).  
* Mỗi vùng không gian thoả mãn định nghĩa class (VD: làn xe đang đi, làn xe bên cạnh) được vẽ thành một polygon khép kín (instance) riêng biệt.

## 3\. Geometry rule

* **Công cụ:** Vẽ bằng **Polygon**.  
* **Quy tắc đặt điểm (Tolerance):** Polygon phải bám sát mép vạch liền, mép vỉa hè, hoặc ranh giới mép tuyết.  
  * Ở nửa dưới ảnh (gần xe ego): Độ lệch tối đa cho phép là **≤ 5 px**.  
  * Ở vùng xa (gần điểm tụ/vanishing point): Độ lệch tối đa cho phép là **≤ 10 px**.  
* **Cạnh đáy (Bottom edge):** Cạnh dưới cùng của polygon phải **dừng chính xác ở mép nắp capo** của xe ego. Tuyệt đối không vẽ trùm lên nắp capo.  
* **Giới hạn xa:** Không vẽ mặt đường ở quá xa. Polygon phải dừng lại ở phía dưới điểm tụ (vanishing point), hoặc cách cạnh đáy của polygon tối đa khoảng 50 px chiều cao ảnh.

## 4\. Taxonomy

Hệ thống phân loại gồm Class và Attribute như sau:

**A. Classes (Nhãn lớp):**

* `drivable_direct`: Làn đường hiện tại mà xe ego đang trực tiếp chạy trên đó.  
* `drivable_alternative`: Các làn đường cùng chiều kế bên mà xe có thể chuyển sang hợp pháp.  
* `ignore_region`: Vùng nằm *trên mặt đường* nhưng **KHÔNG ĐƯỢC CHẠY** (VD: làn ngược chiều, gore area, làn đỗ xe). *Lưu ý: Không dùng class này để vẽ lề đường, vỉa hè hay bãi cỏ (những phần ngoài đường này phải để trống hoàn toàn).*

**B. Attributes (Thuộc tính) \- Áp dụng cho class drivable:**

* `boundary_visibility` (Mức độ nhìn rõ ranh giới đường):  
  * `clear` (Mặc định): Ranh giới nhìn thấy rõ ràng.  
  * `occluded`: Ranh giới bị che khuất bởi xe khác nhưng vẫn có thể suy luận được mép đường bên dưới/phía sau.  
  * `unknown`: Không thể xác định được mép đường do điều kiện thời tiết (tuyết phủ, đêm tối, mưa mờ).

## 5\. Inclusion / exclusion

* **Bắt buộc label (Drivable):**  
  * Mặt đường trải nhựa/bê tông của làn xe ego đang đi.  
  * Các làn xe cùng chiều kế bên (nếu không có vạch liền cấm chuyển làn).  
  * Phần mặt đường thuộc giao lộ phía trước (ngã tư, ngã ba).  
* **Bắt buộc Ignore (Dùng class `ignore_region` cho các phần TRÊN mặt đường):**  
  * Làn ngược chiều (nằm bên kia vạch vàng đôi hoặc dải phân cách).  
  * Làn đỗ xe (kể cả khi đang có xe đỗ hay trống).  
  * Vùng gạch chéo (Gore area) ở các điểm tách/nhập làn cao tốc.  
  * Phản chiếu của nội thất xe trên kính chắn gió.  
* **Không label (Tuyệt đối để trống, KHÔNG dùng `ignore_region`):**  
  * Vỉa hè, bãi cỏ.  
  * Lề đường (Shoulder) nằm ngoài vạch trắng nét liền của đường cao tốc.

## 6\. Visibility / occlusion

* **Bị che khuất (Occlusion):** Phần mặt đường bị xe phía trước hoặc chướng ngại vật che khuất **vẫn được tính là drivable**. Annotator cần suy đoán và vẽ polygon xuyên qua gầm xe/đuôi xe để nối liền mặt đường, đồng thời set thuộc tính `boundary_visibility` \= `occluded`.  
* **Bị mờ do thời tiết/ánh sáng:** Nếu mép đường bị tuyết che lấp, hoặc trời quá tối, mưa làm mờ nhưng vẫn áng chừng được vùng an toàn, vẽ polygon bám mép phần đường thấy được và set `boundary_visibility` \= `unknown`.

## 7\. Ambiguity / escalation

Khi gặp tình huống mơ hồ, không đủ bằng chứng bằng mắt thường để xác định ranh giới đường, áp dụng nguyên tắc an toàn: **Nghiêng về hướng không vẽ vùng đó là drivable**.

**Quy trình Escalation (Báo cáo ca khó trên CVAT):**

1. Không vẽ phần đường đang bị nghi ngờ.  
2. Gắn tag `escalate` cho frame đó trong giao diện CVAT.  
3. Ghi rõ lý do thắc mắc vào attribute text `reason`.  
4. Spec owner sẽ quyết định và cập nhật vào file `edge_case_cards.md`. Khi chưa có quyết định, mặc định coi vùng đó không được phép chạy.

## 8\. Temporal rule

* Không áp dụng — task ảnh tĩnh (Image task).

## 9\. Examples

| sample\_id | Thấy gì | Expected output | Rule áp dụng |
| :---- | :---- | :---- | :---- |
| (Edge Case) | Đường cao tốc có vùng gạch chéo (Gore area) chia tách nhánh rẽ. | Polygon `drivable_direct` chạy theo làn ego. Vẽ `ignore_region` bao trùm kín vùng gore area gạch chéo. | Vẽ nhầm xe vào gore area là `critical`. Bắt buộc dùng `ignore_region` cho vùng giống đường, nằm trên mặt đường nhưng cấm chạy. |
| (Edge Case) | Lề đường (Shoulder) trải nhựa phẳng, không có rào chắn, nằm ngoài vạch trắng nét liền. | Dừng ranh giới `drivable` ở mép trong của vạch trắng liền. **Không label phần lề đường (để trống hoàn toàn, không vẽ `ignore_region`)**. | Vùng ngoài mép đường như Shoulder tuyệt đối không được đi, và không thuộc diện tính vào ignore. |
| (Edge Case) | Xe tải to cồng kềnh chạy ngay phía trước, che khuất một phần lớn làn đường. | Vẽ polygon băng qua gầm xe/sau lưng xe tải để hoàn thiện mặt đường. Set `boundary_visibility` \= `occluded`. | Mặt đường bị xe khác che khuất vẫn tính là drivable. |
| (Edge Case) | Nắp capo xe ego xuất hiện lớn ở mép dưới khung hình. | Cạnh đáy của polygon mặt đường phải dừng lại và bám chính xác theo viền trên của nắp capo xe. | Nắp capo ngoài scope. Lỗi vẽ đè lên capo làm nhiễu model. |
| (Edge Case) | Đường phủ đầy tuyết trắng, hoàn toàn không thấy vạch kẻ đường hay ranh giới mép cỏ. | Đoán vùng an toàn dựa theo vết bánh xe đi trước. Set `boundary_visibility` \= `unknown`. Nếu quá mơ hồ không đoán được: gắn tag `escalate` \+ ghi `reason`. | Khi không xác định được mép do thời tiết, ưu tiên thu hẹp vùng an toàn, set `unknown` hoặc escalate. |

## 10\. Common mistakes

1. **\[CRITICAL\] Vẽ lấn sang vùng cấm:** Vẽ nhầm lề đường, làn đỗ xe, hoặc vỉa hè hạ thấp thành vùng Drivable. Hậu quả là xe tự hành lao ra khỏi làn. **Cách tránh:** Luôn tự hỏi "Xe ego có được đánh lái vào vùng này theo luật không?". Nếu không, hãy dùng `ignore_region` (đối với phần trên đường như làn đỗ) hoặc không vẽ gì (đối với phần ngoài đường như vỉa hè/lề đường).  
2. **Vẽ gộp làn ngược chiều:** Vẽ polygon lấn sang bên kia vạch vàng kép hoặc dải phân cách. **Cách tránh:** Xác định rõ chiều di chuyển dựa vào màu vạch kẻ (vàng/trắng) hoặc hướng xe chạy và dùng `ignore_region` cho đường ngược chiều.  
3. **Quên xử lý mép nắp capo:** Vẽ đè polygon lên nắp capo xe hoặc hình phản chiếu trên kính chắn gió. **Cách tránh:** Luôn zoom kỹ ở cạnh đáy ảnh và cắt polygon bám dọc theo đường viền capo.  
4. **Vẽ mặt đường quá xa (Out of bounds):** Kéo polygon tít lên tận đường chân trời hoặc vượt qua điểm tụ (vanishing point). **Cách tránh:** Chỉ vẽ mặt đường thực sự cần thiết cho path planning (khoảng 50px từ đáy polygon hoặc tối đa đến điểm tụ).