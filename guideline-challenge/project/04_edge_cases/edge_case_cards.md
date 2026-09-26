# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder). Copy khối dưới cho mỗi case.

---

CASE ID: CASE-01
Sample: BDD05
Scene: Highway (đường cao tốc thẳng ban ngày, partly cloudy)
Observation: Xe ego đang di chuyển thẳng trên cao tốc. Bên phải làn xe ego xuất hiện một vùng gore area lớn hình tam giác kẻ các vạch sơn chéo màu trắng (chevron / diagonal hatching) để phân tách làn chạy với lề đường/nhánh rẽ. Phía trước có xe tải và xe con di chuyển cùng chiều. Bên trái có vạch sơn vàng ngăn cách với hàng xe chạy chậm.
Decision: LABEL / IGNORE
Expected: 
- 1 polygon `drivable_direct` bao phủ làn xe ego đang chạy, bám sát vạch phân làn bên trái và mép vạch liền bên phải, cạnh đáy dừng sát mép nắp capo xe; attribute `boundary_visibility=clear`.
- 1 polygon `ignore_region` bao phủ chính xác toàn bộ vùng gore area có vạch kẻ chéo màu trắng ở bên phải để cảnh báo cấm xe tự hành di chuyển vào.
Rationale: Downstream Contract (mục 3, 01_problem_statement.md): False positive là lỗi nghiêm trọng nhất (critical). Gore area có bề mặt trải nhựa giống hệt mặt đường chạy, nếu planner coi đây là drivable thì xe tự hành có thể lao vào dải phân cách/rào chắn khi chuyển làn ở tốc độ cao. Do đó bắt buộc dùng `ignore_region` để loại trừ.
Common mistake: Vẽ polygon `drivable_direct` hoặc `drivable_alternative` tràn sang vùng vạch kẻ chéo xương cá; hoặc bỏ qua không khoanh `ignore_region` cho gore area.
Diversity: conflict / critical

---

CASE ID: CASE-02
Sample: BDD10
Scene: City street (đường phố đô thị ban ngày dốc phố San Francisco, partly cloudy)
Observation: Đường phố hai chiều có một làn xe ego đang chạy thẳng ở giữa. Phía trước có một xe ô tô con màu bạc đang đi cùng làn. Bên phải có vạch trắng liền ngăn cách làn xe chạy với một làn đỗ xe (parking lane) có nhiều xe đang đỗ dọc lề đường. Bên trái có vạch vàng đứt nét và hàng xe đỗ phía đối diện.
Decision: LABEL / IGNORE
Expected: 
- 1 polygon `drivable_direct` bao phủ làn xe ego đang chạy, cạnh đáy dừng ở mép nắp capo, vẽ trùm qua lốp/thân xe con màu bạc phía trước (không đục lỗ); attribute `boundary_visibility=occluded` do xe trước che khuất một phần mặt đường.
- 1 polygon `ignore_region` khoanh vùng toàn bộ làn đỗ xe dọc lề bên phải (phía ngoài vạch trắng liền) có các xe đang đỗ.
Rationale: Làn đỗ xe có kết cấu nhựa bằng phẳng liền mạch với lòng đường. Nếu module path planning nhận diện vùng này là drivable, xe tự hành có thể lập quỹ đạo lách vào các khoảng trống giữa các xe đang đỗ, tạo nguy cơ va chạm khi các xe này mở cửa hoặc bắt đầu lăn bánh. Bị xe khác che khuất một phần phải đánh dấu `boundary_visibility=occluded` theo contract.
Common mistake: Cố gắng cắt rỗng/đục lỗ (cutout) quanh chiếc xe ô tô con màu bạc phía trước thay vì vẽ trùm qua; hoặc vẽ lấn polygon `drivable_direct` sang các khoảng trống trên làn đỗ xe bên phải.
Diversity: ambiguity / occlusion

---

CASE ID: CASE-03
Sample: BDD12
Scene: City street / Intersection (giao lộ ngã tư ban ngày nhiều mây, overcast)
Observation: Xe ego dừng trước giao lộ có vạch kẻ sang đường cho người đi bộ dạng sọc lớn (crosswalk). Phía trước có xe SUV đen bên trái và xe van trắng ở làn giữa. Bên phải là lối vào trạm xăng Citgo có mép vỉa hè hạ thấp (curb cut) và người đi bộ đứng gần mép đường. Nắp capo xe ego màu trắng chiếm diện tích lớn ở đáy ảnh với hình ảnh phản chiếu mờ của tòa nhà. Vạch kẻ phân làn trên ngã tư bị mờ và đứt đoạn.
Decision: LABEL / IGNORE
Expected: 
- 1 polygon `drivable_direct` bao phủ làn xe ego đang đi qua vạch crosswalk tới đuôi xe van trắng, cạnh đáy polygon PHẢI dừng chính xác ở mép nắp capo trắng (không vẽ đè lên capo); attribute `boundary_visibility=occluded` do xe phía trước che khuất.
- 1 polygon `drivable_alternative` bao phủ làn xe cùng chiều bên trái (sau xe SUV đen), attribute `boundary_visibility=occluded`.
- 1 polygon `ignore_region` khoanh vùng khu vực lối vào trạm xăng (curb cut) bên phải mép đường để tránh nhầm thành làn rẽ.
Rationale: Vạch sang đường (crosswalk) tại giao lộ vẫn thuộc phạm vi mặt đường hợp pháp xe ego được phép lưu thông qua. Nếu annotator lầm tưởng crosswalk là chướng ngại vật/vùng cấm mà ngắt polygon thì planner sẽ phanh gấp trước giao lộ. Cạnh đáy polygon vẽ đè lên nắp capo sẽ gây lỗi ước lượng khoảng cách gầm xe.
Common mistake: Cắt ngắn polygon drivable trước vạch crosswalk vì nghĩ vạch đi bộ không phải là đường xe chạy; cạnh đáy vẽ tràn lên bề mặt nắp capo xe ego màu trắng và các vết bóng phản chiếu trên kính lái; vẽ lấn sang lối vào cây xăng (curb cut) thành đường rẽ.
Diversity: ambiguity / small_far / occlusion

---

CASE ID: CASE-04
Sample: BDD15
Scene: City street / Boulevard (đại lộ nội đô ban ngày nắng gắt, clear)
Observation: Đường phố rộng nhiều làn xe. Nắp capo xe Mercedes có logo ngôi sao nổi ở chính giữa đáy ảnh. Làn ego đang chạy có vạch đứt bên trái phân cách với làn cùng chiều. Phía bên phải có bóng râm lớn in đậm của hàng cây và tòa nhà đổ dài trên mặt đường; lề đường bên phải uốn lượn có các hốc đỗ xe và xe con màu xanh, bạc đỗ dọc mép.
Decision: LABEL / IGNORE
Expected: 
- 1 polygon `drivable_direct` bao phủ làn xe ego, mép đáy dừng sát mép nắp capo uốn tránh logo Mercedes, mép phải bám sát vạch sơn/mép đường thật (không ăn theo mép bóng râm); attribute `boundary_visibility=clear`.
- 1 polygon `drivable_alternative` bao phủ làn đường cùng chiều kế bên phía bên trái, attribute `boundary_visibility=clear`.
- 1 polygon `ignore_region` khoanh vùng hốc đỗ xe có xe màu xanh/bạc đang đỗ ở lề bên phải.
Rationale: Vệt bóng râm đổ đậm nét trên mặt đường tạo ra độ tương phản giả rất mạnh giống như vạch sơn hoặc lề đường. Annotator thiếu kinh nghiệm dễ bị ảo giác quang học dẫn đến vẽ lệch ranh giới làn đường, làm planner bị bóp hẹp quỹ đạo di chuyển không cần thiết.
Common mistake: Nhầm ranh giới vệt bóng râm của cây/tòa nhà là vạch phân làn hoặc mép đường; vẽ đè polygon lên logo nổi và mui xe Mercedes; bỏ sót làn cùng chiều bên trái (`drivable_alternative`).
Diversity: ambiguity / occlusion

---

CASE ID: CASE-05
Sample: BDD16
Scene: Highway / Underpass (đường nhánh chui gầm cầu vượt, overcast)
Observation: Xe ego đang di chuyển qua đoạn hầm chui cầu vượt với sự thay đổi độ sáng đột ngột giữa vòm hầm và bên ngoài. Phía trước có một xe ô tô thể thao Dodge Charger màu đen đang chạy. Bên trái có một dải gore area kẻ sọc chéo màu vàng (chevron vàng) áp sát dải phân cách bê tông. Bên phải có vạch sơn trắng gạch góc sát bờ tường bê tông của hầm.
Decision: LABEL / IGNORE
Expected: 
- 1 polygon `drivable_direct` bao phủ duy nhất làn xe ego đang đi qua hầm, vẽ trùm qua lốp xe Dodge Charger phía trước; attribute `boundary_visibility=occluded`.
- 1 polygon `ignore_region` bao phủ toàn bộ dải vạch sọc chéo màu vàng bên trái dẫn vào dải phân cách bê tông.
- 1 polygon `ignore_region` bao phủ góc vạch trắng hẹp sát tường bê tông bên phải.
Rationale: Critical Risk (mục 3, 01_problem_statement.md): Vùng sọc chéo màu vàng bên trái dẫn thẳng vào dải phân cách cứng bê tông. Nếu dán nhãn `drivable_alternative` hoặc vẽ lấn `drivable_direct` vào đây (False Positive), hệ thống tự hành có thể điều khiển xe lao thẳng vào khối bê tông ở tốc độ cao gây tai nạn thảm khốc.
Common mistake: Dán nhãn `drivable_alternative` lên vùng vạch chéo vàng bên trái vì thấy mặt đường nhựa còn rộng; hoặc cố gắng đục lỗ quanh chiếc xe Charger màu đen trong điều kiện thiếu sáng dưới gầm cầu.
Diversity: critical / conflict

---

CASE ID: CASE-06
Sample: BDD20
Scene: Residential (đường khu phố dân cư ban ngày nhiều mây, overcast)
Observation: Đường khu dân cư hai chiều với vạch đôi màu vàng nổi bật ở chính giữa. Làn ego nằm bên phải vạch vàng đôi. Bên phải làn ego là vạch trắng liền và làn đỗ xe có xe SUV trắng đỗ sát lề cùng một số xe khác phía xa và cọc tiêu giao thông hình nón màu cam. Bên trái vạch vàng đôi là toàn bộ làn đường ngược chiều. Trên kính chắn gió có vệt phản chiếu/sticker giá treo camera.
Decision: LABEL / IGNORE
Expected: 
- 1 polygon `drivable_direct` bao phủ duy nhất làn xe ego đang đi (nằm gọn giữa vạch vàng đôi bên trái và vạch trắng liền bên phải), đáy dừng ở mép nắp capo xe; attribute `boundary_visibility=clear`.
- 1 polygon `ignore_region` khoanh vùng toàn bộ làn đỗ xe bên phải (nơi có xe SUV trắng và cọc tiêu cam đỗ dọc lề).
- TUYỆT ĐỐI KHÔNG vẽ bất kỳ polygon drivable nào sang phía bên trái vạch đôi màu vàng (làn ngược chiều).
Rationale: Tránh lỗi Critical (Frontal Collision Risk): Đường khu dân cư thường có bề mặt nhựa trải liền khối không có dải phân cách cứng. Nếu vẽ trùm cả phần đường bên trái vạch vàng đôi thành drivable area, hệ thống planner sẽ hiểu nhầm đó là làn chuyển hướng hoặc làn chạy, dẫn đến việc lái xe sang phần đường đối đầu trực diện với luồng xe ngược chiều.
Common mistake: Vẽ một polygon lớn trùm qua cả vạch vàng đôi bao trùm cả làn đường ngược chiều; hoặc vẽ tràn sang làn đỗ xe bên phải xe SUV trắng vì thấy không có rào chắn cứng.
Diversity: critical / conflict

---

CASE ID: CASE-07
Sample: BDD24
Scene: City street / Snowy (đường phố trung tâm mùa đông tuyết phủ dày, daytime)
Observation: Đường phố trung tâm tuyết rơi tích tụ dày đặc và bị cày xới bẩn thỉu. Dọc bên trái có hàng rào sắt và tuyết vun đống cao. Ở giữa lòng đường xuất hiện một đống tuyết lớn (snow mound / snowbank) ngăn cách giữa các phương tiện. Phía trước có xe sedan màu trắng bị tuyết bao quanh. Bên phải có xe bán đồ ăn (food truck) và người đi bộ. Toàn bộ vạch sơn kẻ đường, vỉa hè và mép lề đều bị tuyết vùi lấp hoàn toàn, không thể xác định đâu là mép đường nhựa chịu lực và đâu là lòng lề đường/chướng ngại vật.
Decision: ESCALATE
Expected: 
- Gắn tag `escalate` cho toàn bộ khung hình ảnh.
- Điền attribute `reason`: "Tuyết phủ kín xóa nhòa toàn bộ vạch kẻ và mép lề đường, có đống tuyết lớn chắn lòng đường không thể xác định an toàn ranh giới drivable".
- TUYỆT ĐỐI KHÔNG vẽ bất kỳ polygon drivable nào (tuân thủ nguyên tắc fail-safe: chưa có quyết định thì không vẽ theo mục 7 guideline).
Rationale: Theo Mục 7 của Guideline và Downstream Contract (Escalation Path): Khi điều kiện thời tiết khắc nghiệt làm mất hoàn toàn căn cứ nhận diện ranh giới mặt đường, việc cố tình đoán mò (hallucinate) ranh giới drivable sẽ dẫn xe tự hành leo lên đống tuyết lún hoặc lao vào vỉa hè/người đi bộ. Hệ thống cần kích hoạt chế độ escalation / fallback an toàn cho tài xế hoặc dừng xe an toàn thay vì tự hành mù quáng.
Common mistake: Cố gắng "đoán" rồi vẽ polygon drivable bừa bãi đè lên các đống tuyết bẩn; hoặc chỉ gán attribute `boundary_visibility=unknown` mà không kích hoạt tag `escalate` theo đúng quy định cho ca tuyết vùi lấp mất hoàn toàn lề đường.
Diversity: escalation

---

CASE ID: CASE-08
Sample: BDD25
Scene: City street / Dawn-Dusk / Wet (đường phố hoàng hôn/chạng vạng, ánh sáng yếu, mặt đường ẩm ướt phản chiếu)
Observation: Đường phố nội đô vào thời điểm chạng vạng tối. Đèn đường, đèn neon tòa nhà và đèn hậu phương tiện bật sáng. Mặt đường ẩm ướt hoặc bóng dầu tạo ra vô số vệt phản quang chói lóa (specular reflections). Bên phải có xe taxi vàng đang áp sát chuyển làn. Vạch sơn kẻ làn màu trắng bị phản chiếu ánh đèn làm mờ nhạt và đứt quãng. Mép vỉa hè bên trái chìm vào vùng tối dưới chân nhà cao tầng.
Decision: LABEL / UNKNOWN
Expected: 
- 1 polygon `drivable_direct` bao phủ làn xe ego đang chạy, đáy dừng ở mép nắp capo; attribute `boundary_visibility=unknown` do mặt đường ướt phản chiếu ánh sáng và vạch kẻ bị mờ nhòe.
- 1 polygon `drivable_alternative` bao phủ làn đường cùng chiều kế bên (phía sau taxi vàng), attribute `boundary_visibility=unknown` (hoặc `occluded` tại phần bị taxi vàng che).
Rationale: Ánh sáng yếu kết hợp phản xạ gương từ mặt đường ướt là nguyên nhân chính gây ảo giác quang học (optical illusions) cho camera thị giác máy tính. Đánh dấu `boundary_visibility=unknown` là tín hiệu quan trọng để module path planning kích hoạt hệ số an toàn cao hơn (giảm tốc độ, mở rộng khoảng cách an toàn với các phương tiện xung quanh).
Common mistake: Chọn `boundary_visibility=clear` vì nhầm tưởng các vệt sáng phản chiếu đèn neon trên mặt đường là vạch sơn rõ ràng; hoặc bỏ qua xe taxi vàng đang chèn làn bên phải hoặc không gán `occluded` khi ranh giới bị xe này che khuất.
Diversity: small_far / ambiguity

---
