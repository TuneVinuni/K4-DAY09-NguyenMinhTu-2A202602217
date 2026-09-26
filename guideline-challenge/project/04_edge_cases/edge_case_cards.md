# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder). Copy khối dưới cho mỗi case.

Mọi card dưới đây viết theo guideline v1 hiện tại: chỉ có 3 class polygon, không attribute, không tag escalate; polygon
drivable **không vẽ đè lên xe** (guideline mục 6).

**Quyết định có chủ đích:** BDD10 và BDD12 (split calibration) được giữ làm ví dụ ở guideline mục 9, vì hai ảnh này
minh họa rule làn xe đạp, bó vỉa hạ thấp và "dừng tại chân xe" rõ nhất. Hệ quả: khi calibration, bất đồng ở hai ảnh này
chủ yếu đo xem annotator có đọc guideline hay không (execution). Bất đồng do rule mơ hồ chủ yếu đọc từ BDD06, 07, 08, 09,
11, 13.

---

CASE ID: CASE-01
Sample: BDD05 (example)
Scene: Đường cao tốc/đường nhánh ban ngày, partly cloudy
Observation: Làn ego nằm giữa vạch vàng liền (trái) và vạch trắng liền (phải). Xe con màu đen chạy phía trước giữa làn ego. Bên phải vạch trắng là một vùng rộng kẻ nhiều vạch trắng song song (vùng gạch chéo), liền mặt nhựa với làn chạy. Bên trái vạch vàng là dải nhựa hẹp, tiếp theo là đất/sỏi và hàng xe đỗ ở đường bên cạnh.
Decision: LABEL / IGNORE
Expected:
- 1 polygon `drivable_direct` trong làn ego: mép trái ở mép trong vạch vàng, mép phải ở mép trong vạch trắng liền, cạnh đáy theo viền capo, cạnh trên **dừng tại chân xe đen** phía trước.
- 1 polygon `ignore_region` phủ toàn bộ vùng kẻ vạch bên phải (tính cả vạch trắng liền).
- Dải nhựa hẹp ngoài vạch vàng (hẹp hơn một xe con, không đạt làn khẩn cấp), đất/sỏi và đường bên cạnh: để trống.
Rationale: False positive là lỗi critical (01_problem_statement.md, contract câu 3). Vùng gạch chéo có mặt nhựa giống hệt làn chạy; nếu planner coi đây là drivable, xe có thể lao vào vùng cấm/nhánh tách ở tốc độ cao.
Common mistake: Vẽ `drivable_alternative` lên vùng kẻ vạch vì "trông rộng như một làn"; vẽ `drivable_direct` trùm qua xe đen tới giới hạn xa.
Diversity: conflict / critical

---

CASE ID: CASE-02
Sample: BDD10 (calibration)
Scene: Phố dốc San Francisco, ban ngày, partly cloudy
Observation: Đường hai chiều, vạch vàng đứt ở giữa. Làn ego nằm bên phải vạch vàng. Xe con màu bạc chạy phía trước giữa làn ego. Bên phải: vạch trắng liền, một làn hẹp (làn xe đạp), vạch trắng thứ hai, rồi làn đỗ xe có nhiều xe. Bên trái vạch vàng: làn ngược chiều, rồi một vạch trắng và hàng xe đỗ bên trái.
Decision: LABEL / IGNORE
Expected:
- 1 polygon `drivable_direct` từ mép phải vạch vàng đứt tới mép trong vạch trắng liền bên phải, cạnh đáy theo viền capo, cạnh trên **dừng tại chân xe bạc**. Không vẽ phần làn phía sau xe bạc.
- 1 polygon `ignore_region` cho làn ngược chiều bên trái vạch vàng (được trùm qua xe đỗ bên trái).
- 1 polygon `ignore_region` cho làn xe đạp + làn đỗ xe bên phải (được trùm qua xe đỗ).
Rationale: Làn xe đạp và làn đỗ có mặt nhựa liền với làn chạy; planner coi là drivable thì có thể lách vào khoảng trống giữa các xe đỗ. Vạch vàng đứt vẫn là ranh giới cứng với làn ngược chiều (guideline mục 5 câu 4).
Common mistake: Vẽ trùm qua xe bạc để lấy mặt đường phía sau; coi vạch vàng đứt là cho phép gộp làn ngược chiều thành `drivable_alternative`; gộp làn xe đạp vào `drivable_direct`.
Diversity: occlusion / conflict

---

CASE ID: CASE-03
Sample: BDD12 (calibration)
Scene: Giao lộ đô thị ban ngày, overcast
Observation: Xe ego đứng trước vạch qua đường dạng sọc lớn. Van trắng ở làn ego phía trước, SUV đen ở làn trái. Vạch phân làn mờ và đứt đoạn. Bên phải là sân trạm xăng Citgo nối với đường qua chỗ bó vỉa hạ thấp, có người đi bộ đứng gần mép. Capo trắng lớn ở đáy ảnh phản chiếu tòa nhà.
Decision: LABEL
Expected:
- 1 polygon `drivable_direct` đi thẳng qua vạch qua đường, cạnh trên **dừng tại chân van trắng**; cạnh đáy theo viền trên capo, không lấy phần phản chiếu.
- 1 polygon `drivable_alternative` cho làn trái, cạnh trên **dừng tại chân SUV đen**.
- Sân trạm xăng: để trống (nằm sau đường bó vỉa kéo dài, guideline mục 5 câu 5). Drivable dừng ở đường bó vỉa kéo dài.
Rationale: Vạch qua đường thuộc lòng đường ego được đi qua; nếu cắt polygon trước vạch, planner sẽ phanh gấp ở giao lộ. Sân trạm xăng không phải lòng đường nên không dùng `ignore_region`.
Common mistake: Cắt polygon trước vạch qua đường; vẽ lên capo trắng/phần phản chiếu; vẽ `ignore_region` lên sân trạm xăng; nối polygon qua thân van trắng.
Diversity: ambiguity / occlusion

---

CASE ID: CASE-04
Sample: BDD15 (blind)
Scene: Đại lộ đô thị ban ngày nắng, clear
Observation: Đường nhiều làn cùng chiều, phân làn bằng vạch trắng đứt. Xe ego ở làn ngoài cùng bên phải, sát bó vỉa sơn đỏ-trắng. Sau bó vỉa là dải đỗ xe cao hơn mặt đường, có xe bạc và xe xanh đang đỗ. Nắp capo Mercedes có logo nổi ở giữa đáy ảnh. Góc trên-trái kính chắn gió có vệt tối/chói phủ lên mặt đường. Không có xe nào gần trong làn ego.
Decision: LABEL
Expected:
- 1 polygon `drivable_direct`: mép trái ở tâm vạch trắng đứt, mép phải ở **mép bó vỉa đỏ-trắng**, cạnh đáy theo viền capo (đi vòng qua logo), cạnh trên ở giới hạn xa hoặc chân xe đầu tiên trong làn.
- 1 polygon `drivable_alternative` cho **mỗi** làn cùng chiều bên trái (ngăn nhau bởi vạch đứt).
- Dải đỗ xe sau bó vỉa và xe đỗ trên đó: **để trống**, không phải `ignore_region` (nằm ngoài lòng đường).
- Vệt chói trên kính: bỏ qua, vẽ mặt đường bên dưới như bình thường.
Rationale: Ranh giới lòng đường là bó vỉa, không phải xe đỗ. Dải đỗ nằm sau bó vỉa là ngoài lòng đường nên để trống (guideline mục 5 câu 5), khác với làn đỗ cùng mặt nhựa ở CASE-02.
Common mistake: Vẽ `ignore_region` lên dải đỗ sau bó vỉa; cắt polygon theo vệt chói trên kính; gộp các làn trái vào 1 polygon `drivable_alternative`; vẽ lên logo capo.
Diversity: ambiguity / conflict

---

CASE ID: CASE-05
Sample: BDD16 (blind)
Scene: Đường nhánh chui dưới cầu vượt, overcast, ánh sáng yếu
Observation: Xe ego đi dưới gầm cầu. Dodge Charger đen chạy ngay phía trước trong làn ego. Bên trái làn ego là vùng kẻ sọc chéo màu vàng có viền vàng liền; bên kia vùng sọc là một làn khác có SUV trắng. Bên phải: vạch trắng liền, sau đó là vùng kẻ sọc trắng sát tường bê tông của hầm.
Decision: LABEL / IGNORE
Expected:
- 1 polygon `drivable_direct` giữa mép trong viền vàng (trái) và mép trong vạch trắng liền (phải), cạnh trên **dừng tại chân xe Charger đen**.
- 1 polygon `ignore_region` phủ vùng sọc chéo vàng **và** làn có SUV trắng bên kia vùng sọc (làn nằm bên kia vùng gạch chéo, guideline mục 5 câu 4).
- 1 polygon `ignore_region` phủ vùng kẻ sọc trắng giữa vạch trắng liền và tường hầm.
- Tường bê tông: để trống.
Rationale: Vùng sọc vàng dẫn thẳng vào dải phân cách/nhập làn; vẽ drivable vào đây là false positive critical, planner có thể lái xe vào vùng tách làn ở tốc độ cao.
Common mistake: Vẽ `drivable_alternative` cho làn SUV trắng bên kia vùng sọc vàng; vẽ drivable lên vùng sọc vì thấy mặt nhựa rộng; nối polygon qua thân Charger trong điều kiện thiếu sáng.
Diversity: critical / conflict / low_visibility

---

CASE ID: CASE-06
Sample: BDD20 (blind)
Scene: Đường khu dân cư ban ngày, overcast
Observation: Đường hai chiều, vạch vàng đôi ở giữa. Làn ego bên phải vạch vàng đôi, bên phải là vạch trắng liền, một dải hẹp, vạch trắng thứ hai, rồi làn đỗ có SUV trắng đỗ gần và nhiều xe đỗ phía xa; cọc tiêu cam trên lề cỏ. Bên trái vạch vàng đôi: làn ngược chiều, vạch trắng, rồi hàng xe đỗ bên trái. Giá đỡ điện thoại trên táp-lô che một phần mặt đường ở đáy giữa ảnh; góc trên-phải kính có phản chiếu logo lớn.
Decision: LABEL / IGNORE
Expected:
- 1 polygon `drivable_direct` từ mép phải vạch vàng đôi tới mép trong vạch trắng liền bên phải, lên tới giới hạn xa (không có xe trong làn ego). Cạnh đáy theo viền táp-lô và **khoét theo viền giá đỡ điện thoại** (guideline mục 3: vật đặc trong xe ego).
- 1 polygon `ignore_region` cho làn ngược chiều + làn đỗ bên trái vạch vàng đôi.
- 1 polygon `ignore_region` cho dải hẹp + làn đỗ bên phải vạch trắng liền (trùm qua SUV trắng đỗ).
- Lề cỏ, vỉa hè, cọc tiêu trên lề: để trống. Phản chiếu logo trên kính: bỏ qua.
Rationale: Đường dân cư không có dải phân cách cứng; vẽ trùm qua vạch vàng đôi khiến planner lái sang làn ngược chiều, đối đầu trực diện (critical).
Common mistake: Một polygon lớn trùm qua vạch vàng đôi; vẽ lên giá đỡ điện thoại; vẽ `drivable_alternative` cho dải hẹp giữa hai vạch trắng.
Diversity: critical / conflict / occlusion

---

CASE ID: CASE-07
Sample: BDD24 (blind)
Scene: Phố trung tâm sau tuyết, ban ngày
Observation: Mặt đường ướt, tuyết dồn thành đống dọc rào bên trái và một đống tuyết lớn nằm ngay trong làn ego phía trước (xe Tesla trắng phía sau đống tuyết). Vạch trắng giữa hai làn vẫn thấy ở đáy ảnh. Làn trái có limousine trắng. Bên phải là bó vỉa, vỉa hè có xe bán đồ ăn, thùng rác và người đi bộ.
Decision: LABEL
Expected:
- 1 polygon `drivable_direct` từ vạch trắng tới bó vỉa phải, cạnh trên **dừng tại chân đống tuyết** (vật cản bất thường xử lý như xe, guideline mục 7 câu 3). Không vẽ phần làn sau đống tuyết.
- 1 polygon `drivable_alternative` cho làn trái, cạnh trên **dừng tại chân limousine trắng**; mép trái khoét theo các đống tuyết sát rào.
- Vỉa hè, xe bán đồ ăn, người đi bộ: để trống.
Rationale: Đống tuyết là vật cản thật trên lòng đường; vẽ drivable qua nó khiến planner lái xe vào tuyết. Làn ego vẫn xác định được nhờ vạch trắng và bó vỉa nên không thuộc trường hợp "không xác định được làn ego".
Common mistake: Không vẽ gì cả vì thấy "tuyết" (bỏ sót quá mức); vẽ trùm qua đống tuyết lên tới Tesla; vẽ drivable lên đống tuyết sát rào bên trái.
Diversity: occlusion / low_visibility / conflict

---

CASE ID: CASE-08
Sample: BDD25 (blind)
Scene: Đại lộ đô thị lúc chạng vạng, mặt đường ướt
Observation: Đường một chiều nhiều làn, vạch trắng đứt mờ. Mặt đường ướt phản chiếu đèn đường, đèn neon và đèn hậu thành nhiều vệt sáng dọc. Taxi vàng áp sát ở làn bên phải ngay cạnh xe ego; phía trước có taxi vàng khác và xe tải trắng. Vỉa hè bên trái chìm trong bóng tối.
Decision: LABEL
Expected:
- 1 polygon `drivable_direct` giữa hai vạch đứt của làn ego (nối dài đoạn vạch còn thấy), cạnh trên dừng tại chân xe đầu tiên chiếm ≥ 1/2 làn ego hoặc giới hạn xa; polygon **co vào trong** ở đoạn vạch mờ.
- 1 polygon `drivable_alternative` cho làn phải: phần taxi vàng sát bên chiếm ≥ 1/2 làn nên cạnh trên dừng tại chân taxi đó.
- 1 polygon `drivable_alternative` cho mỗi làn bên trái tới mép bó vỉa trái.
- Vệt sáng phản chiếu trên mặt đường ướt không phải vạch/ranh giới.
Rationale: Vệt phản chiếu trông giống vạch sơn; vẽ ranh giới theo vệt sáng làm lệch làn. Rule co vào trong (guideline mục 6) giữ an toàn khi mép mờ.
Common mistake: Lấy vệt sáng phản chiếu làm vạch làn; vẽ làn phải trùm qua taxi vàng; nới polygon ra vỉa hè tối bên trái.
Diversity: low_visibility / occlusion

---

CASE ID: CASE-09
Sample: BDD26 (blind)
Scene: Đường phố ban đêm, rất tối
Observation: Chỉ thấy rõ một đoạn vạch trắng đứt ở phía phải-giữa ảnh. Xe đen lớn ở làn phải ngay cạnh xe ego. Bên trái làn ego là vùng tối có vài vệt sọc mờ (có thể là vạch qua đường hoặc đường ngang của giao lộ) và cột đèn tín hiệu; không thấy vạch vàng, không thấy xe nào đi ở phía trái nên không xác định được chiều.
Decision: UNKNOWN (fail-safe: không vẽ drivable ở vùng không chắc; đây là escalation path của guideline mục 7)
Expected:
- 1 polygon `drivable_direct` cho làn ego, mép phải ở tâm vạch trắng đứt, mép trái **co vào** tới phần mặt đường chắc chắn (bề ngang khoảng một làn tính từ vạch đứt); cạnh trên dừng ở giới hạn xa hoặc chân xe đầu tiên trong làn.
- 1 polygon `drivable_alternative` cho làn phải, dừng tại **chân xe đen** bên phải.
- Vùng tối bên trái làn ego: **không vẽ drivable** (guideline mục 7 câu 2 và 5). Không đủ bằng chứng là lòng đường cấm chạy nên cũng không vẽ `ignore_region`.
Rationale: Không thấy vạch vàng hay hướng xe nên không biết vùng trái là làn cùng chiều, làn ngược chiều hay đường ngang. Vẽ drivable ở đó có thể là false positive critical; fail-safe là bỏ trống và ghi vào clarification log để Spec owner quyết định.
Common mistake: Vẽ `drivable_alternative` cho toàn bộ vùng tối bên trái; không vẽ gì cả dù làn ego vẫn xác định được; vẽ làn phải trùm qua xe đen.
Diversity: escalation / low_visibility / ambiguity

---

CASE ID: CASE-10
Sample: BDD23 (blind)
Scene: Phố dân cư một chiều, trời mưa nhẹ sau tuyết
Observation: Một làn chạy ở giữa, hai bên là hàng xe đỗ dọc lề. Vạch trắng liền bên trái và vạch trắng đứt bên phải tách làn chạy với làn đỗ. Người đi xe đạp nhỏ ở xa giữa làn ego. Kính chắn gió có nhiều giọt mưa. Tuyết còn sót trên vỉa hè trái.
Decision: LABEL / IGNORE
Expected:
- 1 polygon `drivable_direct` từ mép trong vạch trắng trái tới mép trong vạch đứt phải; **người đi xe đạp chiếm < 1/2 bề ngang làn nên khoét theo viền** người + xe đạp, polygon tiếp tục lên giới hạn xa ở phần làn còn trống.
- 1 polygon `ignore_region` cho làn đỗ bên trái, 1 polygon `ignore_region` cho làn đỗ bên phải (trùm qua xe đỗ).
- Giọt mưa trên kính: bỏ qua. Vỉa hè có tuyết: để trống.
Rationale: Kiểm rule "< 1/2 làn thì khoét viền" với vật nhỏ, xa. Dừng polygon tại chân xe đạp sẽ làm mất gần hết làn ego phía xa; vẽ trùm qua thì vi phạm rule không đè lên người/xe.
Common mistake: Dừng cả làn ego tại chân xe đạp; vẽ trùm qua người đi xe đạp; cắt polygon theo giọt mưa; gộp làn đỗ vào làn chạy vì cùng mặt nhựa.
Diversity: small_far / occlusion / conflict
