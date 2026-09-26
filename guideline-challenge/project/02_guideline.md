# Annotation guideline — Phân vùng Drivable Area

**Version:** v1 (Cập nhật bổ sung quy tắc làn khẩn cấp và loại bỏ thuộc tính/quy trình escalate)

## 1\. Objective \+ scope

* **Mục tiêu:** Phân vùng "drivable area" (vùng xe ego được phép và có thể chạy) trên ảnh dashcam. Dữ liệu này dùng để huấn luyện model segmentation cho module **path planning** của xe tự hành. Planner sẽ chỉ sinh quỹ đạo di chuyển an toàn bên trong vùng được vẽ.  
* **Scope (Phạm vi):**  
  * **Trong scope (Được phép đi):** Mặt đường trải nhựa hoặc bê tông mà xe ego được đi hợp pháp theo luật giao thông (bao gồm làn xe đang đi và các làn cùng chiều có thể chuyển sang, bao gồm cả làn khẩn cấp trong những điều kiện nhất định).  
  * **Ngoài scope (Không label hoặc Ignore):** Bất kỳ khu vực nào không thuộc phần đường ego được phép đi.  
    * **Tuyệt đối không label (để trống hoàn toàn):** Các khu vực ngoài lề như vỉa hè, lề đường/shoulder (trường hợp không dùng làm làn khẩn cấp lưu thông), bãi cỏ.  
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

Hệ thống phân loại chỉ gồm Class như sau:

**Classes (Nhãn lớp):**

* `drivable_direct`: Làn đường hiện tại mà xe ego đang trực tiếp chạy trên đó.  
* `drivable_alternative`: Các làn đường cùng chiều kế bên mà xe có thể chuyển sang hợp pháp (bao gồm cả làn khẩn cấp/hard shoulder trong các trường hợp được phép di chuyển hoặc mở làn theo quy định giao thông).  
* `ignore_region`: Vùng nằm *trên mặt đường* nhưng **KHÔNG ĐƯỢC CHẠY** (VD: làn ngược chiều, gore area, làn đỗ xe). *Lưu ý: Không dùng class này để vẽ lề đường thông thường, vỉa hè hay bãi cỏ (những phần ngoài đường này phải để trống hoàn toàn).*

## 5\. Inclusion / exclusion

* **Bắt buộc label (Drivable):**  
  * Mặt đường trải nhựa/bê tông của làn xe ego đang đi.  
  * Các làn xe cùng chiều kế bên (nếu không có vạch liền cấm chuyển làn).  
  * Làn khẩn cấp (Hard shoulder) được đánh giá là phần mặt đường liền mạch cùng chiều, trong các kịch bản mô hình cần dự phòng không gian lưu thông hoặc mở làn (gắn nhãn `drivable_alternative`).  
  * Phần mặt đường thuộc giao lộ phía trước (ngã tư, ngã ba).  
* **Bắt buộc Ignore (Dùng class `ignore_region` cho các phần TRÊN mặt đường):**  
  * Làn ngược chiều (nằm bên kia vạch vàng đôi hoặc dải phân cách).  
  * Làn đỗ xe (kể cả khi đang có xe đỗ hay trống).  
  * Vùng gạch chéo (Gore area) ở các điểm tách/nhập làn cao tốc.  
  * Phản chiếu của nội thất xe trên kính chắn gió.  
* **Không label (Tuyệt đối để trống, KHÔNG dùng `ignore_region`):**  
  * Vỉa hè, bãi cỏ.  
  * Lề đường (Shoulder) phân cách rõ bằng vật lý mà xe hoàn toàn không được phép tiếp cận.

## 6\. Visibility / occlusion

* **Bị che khuất (Occlusion):** **Tuyệt đối không vẽ đè lên các phương tiện hoặc vật thể khác.** Drivable Area phải dừng lại ở ranh giới tiếp xúc của các phương tiện/chướng ngại vật (như xe phía trước, xe kế bên) với mặt đường. Cần vẽ polygon khoét theo viền của xe/vật thể che khuất.  
* **Bị mờ do thời tiết/ánh sáng:** Nếu mép đường bị tuyết che lấp, hoặc trời quá tối, mưa làm mờ nhưng vẫn áng chừng được vùng an toàn, vẽ polygon bám mép phần đường thấy được.

## 7\. Ambiguity Handling

Khi gặp tình huống mơ hồ, không đủ bằng chứng bằng mắt thường để xác định ranh giới đường, áp dụng nguyên tắc an toàn: **Nghiêng về hướng không vẽ vùng đó là drivable** và không vẽ phần đường đang bị nghi ngờ.

## 8\. Temporal rule

* Không áp dụng — task ảnh tĩnh (Image task).

## 9\. Examples

| sample\_id | Thấy gì | Expected output | Rule áp dụng |
| :---- | :---- | :---- | :---- |
| (Edge Case) | Đường cao tốc có làn khẩn cấp bên phải phân cách bằng vạch trắng liền/nét đứt. | Vẽ `drivable_alternative` trùm lên làn khẩn cấp nếu nằm trong tiêu chí cho phép xe mở rộng hướng di chuyển hoặc dự phòng không gian. | Làn khẩn cấp được gán nhãn `drivable_alternative` để hệ thống định vị xem xét như một phương án chuyển hướng khẩn cấp khi cần thiết. |
| (Edge Case) | Đường cao tốc có vùng gạch chéo (Gore area) chia tách nhánh rẽ. | Polygon `drivable_direct` chạy theo làn ego. Vẽ `ignore_region` bao trùm kín vùng gore area gạch chéo. | Vẽ nhầm xe vào gore area là `critical`. Bắt buộc dùng `ignore_region` cho vùng giống đường, nằm trên mặt đường nhưng cấm chạy. |
| (Edge Case) | Xe tải to cồng kềnh chạy ngay phía trước, che khuất một phần lớn làn đường. | Cạnh của polygon phải dừng lại và bám dọc theo viền tiếp xúc của xe tải với mặt đường (vẽ khoét viền xe). Tuyệt đối không vẽ xuyên qua xe tải. | Không vẽ đè lên các vật thể/phương tiện khác đang chiếm dụng mặt đường. |
| (Edge Case) | Nắp capo xe ego xuất hiện lớn ở mép dưới khung hình. | Cạnh đáy của polygon mặt đường phải dừng lại và bám chính xác theo viền trên của nắp capo xe. | Nắp capo ngoài scope. Lỗi vẽ đè lên capo làm nhiễu model. |
| (Edge Case) | Đường phủ đầy tuyết trắng, hoàn toàn không thấy vạch kẻ đường hay ranh giới mép cỏ. | Đoán vùng an toàn dựa theo vết bánh xe đi trước. Nếu quá mơ hồ không thấy rõ, bỏ qua vùng đó. | Khi không xác định được mép do thời tiết, ưu tiên thu hẹp vùng an toàn. |

## 10\. Common mistakes

1. **\[CRITICAL\] Vẽ lấn sang vùng cấm:** Vẽ nhầm lề đường đất, làn đỗ xe, hoặc vỉa hè hạ thấp thành vùng Drivable. Hậu quả là xe tự hành lao ra khỏi làn. **Cách tránh:** Luôn tự hỏi "Xe ego có được đánh lái vào vùng này theo luật không?". Nếu không, hãy dùng `ignore_region` (đối với phần trên đường như làn đỗ) hoặc không vẽ gì (đối với phần ngoài đường như vỉa hè/lề đất).  
2. **Vẽ gộp làn ngược chiều:** Vẽ polygon lấn sang bên kia vạch vàng kép hoặc dải phân cách. **Cách tránh:** Xác định rõ chiều di chuyển dựa vào màu vạch kẻ (vàng/trắng) hoặc hướng xe chạy và dùng `ignore_region` cho đường ngược chiều.  
3. **Quên xử lý mép nắp capo:** Vẽ đè polygon lên nắp capo xe hoặc hình phản chiếu trên kính chắn gió. **Cách tránh:** Luôn zoom kỹ ở cạnh đáy ảnh và cắt polygon bám dọc theo đường viền capo.  
4. **Vẽ mặt đường quá xa (Out of bounds):** Kéo polygon tít lên tận đường chân trời hoặc vượt qua điểm tụ (vanishing point). **Cách tránh:** Chỉ vẽ mặt đường thực sự cần thiết cho path planning (khoảng 50px từ đáy polygon hoặc tối đa đến điểm tụ).