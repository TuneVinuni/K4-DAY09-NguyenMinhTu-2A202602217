# Kế hoạch nhóm — Day 9 Guideline Design Challenge

Đọc hết file này trước phút 0, **tìm số thứ tự (#) của mình trong bảng vai trò** rồi đọc cột "Việc chính" và lịch bên
dưới. Chi tiết đầy đủ của đề: [README.md](README.md), cách chấm: [RUBRIC.md](RUBRIC.md), thao tác CVAT: [GUIDE.md](GUIDE.md).

## 1. Bài này là gì

Cả nhóm dùng **chung một repo** để thiết kế một annotation project **bàn giao được**: một nhóm khác chỉ cầm guideline
và task CVAT của mình là label đúng được, không cần mình giải thích.

**Điều kiện hoàn thành:** nhóm peer label **blind** (không được xem đáp án) và ra kết quả khớp với **gold** (đáp án)
mà nhóm mình đã **freeze** (khoá lại) từ trước.

Bài không chấm ai vẽ đẹp. Bài chấm guideline của mình có rõ đến mức người lạ làm theo được hay không.

## 2. Luồng 240 phút

| Phút | Pha | Việc chính | File trong `project/` | Gate |
|---|---|---|---|---|
| 0–35 | Chọn bài toán | Thu hẹp chủ đề (ví dụ "trạng thái đèn và đèn nào áp dụng cho xe mình ở ngã tư nhiều đầu đèn"), trả lời 4 câu downstream contract | `00_team.md`, `01_problem_statement.md` | G1 |
| 35–80 | Guideline v1 | Đủ 10 mục bắt buộc, lập ontology table (class hay attribute). Mọi rule phải được viết ra | `02_guideline.md`, `03_ontology_and_cvat_setup.md` | |
| 80–110 | Setup CVAT | Viết labels JSON, chia ảnh: example (3–5), calibration (5–8), blind (4–5), chạy `pack` | `03_cvat_labels.json`, `sample_pack.csv` | G2 |
| 110–120 | Nghỉ | | | |
| 120–140 | Calibration | Mỗi người label **độc lập**, `calib` đo bất đồng, chẩn đoán ≥ 3 chỗ lệch | `06_calibration_report.csv`, guideline v2 | G3 |
| 140–160 | QA plan + freeze gold | QA plan có severity + threshold. Gold ≥ 10 decision (≥ 2 critical, ≥ 1 geometry). **Một người duy nhất** chạy `freeze` | `05_qa_plan.md`, `gold_decisions.csv` | G4 |
| 160–185 | Blind handoff | Gửi gói blind cho peer, đồng thời label gói của peer. Không giải thích bằng miệng, mọi câu hỏi ghi vào log | `clarification_log.csv` | G5 |
| 185–205 | Chấm điểm | `score`, điền cột `correct` từng dòng, rồi `gts` | `transfer_score.csv`, `peer_feedback.md` | |
| 205–240 | Sửa và nộp | Guideline v3, ≥ 8 edge case card, `check`, push kèm tag | `08_revision_log.md`, `edge_case_cards.md` | G6 |

## 3. Chia vai — 9 người

**Quy tắc vàng: mỗi file chỉ một người sửa chính.** Muốn góp nội dung vào file của người khác thì gửi qua chat cho
người giữ file, không tự sửa thẳng — 9 người cùng sửa một repo rất dễ conflict git.

| # | Tên | Vai | File phụ trách | Việc chính |
|---|---|---|---|---|
| 1 | | **Trưởng nhóm / Spec owner** | `00_team.md`, `01_problem_statement.md` | Chốt topic, viết downstream contract, giữ timeline, gỡ conflict git, chạy `check` và push cuối |
| 2 | | **Guideline editor** | `02_guideline.md` (duy nhất người sửa) | Viết đủ 10 mục, gộp nội dung mọi người gửi, đổi version v1 → v2 → v3 |
| 3 | | **Guideline researcher** | (gửi nháp cho #2) | Tra BDD100K/LISA/GTSDB định nghĩa label thế nào; nháp mục Examples + Common mistakes; làm **setup test** phút 80–110 |
| 4 | | **Ontology owner** | `03_ontology_and_cvat_setup.md`, `03_cvat_labels.json` | Ontology table (class/attribute, giá trị, default); đảm bảo JSON khớp bảng |
| 5 | | **CVAT / data owner** | `sample_pack.csv`, `09_cvat_export_or_task_reference.txt` | Chọn ảnh vào 3 split, chạy `pack`, tạo task mẫu, gỡ lỗi CVAT cho cả nhóm |
| 6 | | **Gold owner** | `04_edge_cases/gold_decisions.csv` | Viết gold, quét kỹ ảnh blind ở kích thước gốc. **Người duy nhất chạy `freeze`** |
| 7 | | **Edge-case owner** (dự phòng của #6) | `04_edge_cases/edge_case_cards.md` | ≥ 8 card, có case critical + escalation; soát chéo gold của #6 |
| 8 | | **QA + calibration owner** | `05_qa_plan.md`, `06_calibration_report.csv`, `08_revision_log.md` | Chạy `calib`, chẩn đoán bất đồng, gửi rule mới cho #2; viết QA plan |
| 9 | | **Handoff owner** | `project/07_blind_handoff/*` | Chạy `handoff`, gửi gói cho peer, ghi clarification log, chạy `score` / `gts`, viết `peer_feedback.md` |

## 4. Ai làm gì theo từng mốc

| Phút | Việc |
|---|---|
| **0–15** | #1 tạo repo từ template, thêm 8 người vào **Settings → Collaborators**. Mọi người clone, bật CVAT, chạy `cvat-status`. #1 điền `00_team.md` |
| **15–35** | #1 + #3 chốt topic, #1 viết `01`. #5 chạy `samples` kiểm topic có đủ ảnh. Những người khác đọc README + RUBRIC |
| **35–80** | #2 viết guideline v1 (#3 gửi nháp examples/mistakes). #4 làm ontology. #7 bắt đầu edge case card. #8 soạn khung QA plan. #6, #9 cùng #7 săn edge case trong ảnh |
| **80–110** | #4 viết labels JSON. #5 làm `sample_pack.csv`, chạy `pack calibration`. #3 làm setup test: mở task như người mới, ghi chỗ khó hiểu vào `03_ontology_and_cvat_setup.md` |
| **110–120** | Nghỉ |
| **120–140** | **Cả 9 người** tạo task riêng từ `build/calibration/`, label **độc lập** (không nhìn màn hình nhau), export đặt tên `<tên>.zip`, push vào `project/06_calibration_exports/`. #8 chạy `calib` |
| **140–160** | #8 chẩn đoán bất đồng, gửi rule cho #2 → guideline v2. #8 xong QA plan. #6 viết gold theo v2, #7 soát, #6 chạy `freeze` + `git push --follow-tags`. Sau đó **mọi người** chạy `git pull && git fetch --tags --force` |
| **160–185** | Chia 2 đội chạy song song:<br>**Đội owner** (#1, #2, #9): #9 gửi gói blind, ghi mọi câu hỏi của peer vào log. Không ai giải thích bằng miệng.<br>**Đội peer tester** (#3, #4, #5, #7, #8): label gói blind của nhóm peer đúng theo guideline của họ, ghi chỗ khó hiểu, export gửi lại + trả lời 5 câu feedback.<br>#6 đứng ngoài để không lỡ lộ gold. |
| **185–205** | #9 chạy `score`. #6 + #7 điền `correct` / `note` (geometry thì mở export của peer trong CVAT xem). #9 chạy `gts`, viết `peer_feedback.md`. #8 phân loại nguyên nhân sai: guideline gap / data ambiguity / execution error |
| **205–225** | #2 viết guideline v3 từ bằng chứng blind test. #8 ghi dòng v3 vào `08_revision_log.md`. #7 đủ ≥ 8 card. #5 điền file `09` |
| **225–240** | #1 chạy `check`, commit, `git push --follow-tags`. #1 hoặc #9 debrief 2 phút với nhóm peer |

## 5. Lệnh hay dùng

Chạy **trong thư mục `guideline-challenge/`**. Không có `make` (Windows) thì dùng cột phải; máy không nhận `python`
thì thử `py` hoặc `python3`.

| Việc | Có `make` | Không có `make` |
|---|---|---|
| Kiểm CVAT | `make cvat-status` | `python lab9.py cvat` |
| Xem ảnh | `make samples SOURCE=bdd100k` | `python lab9.py samples --source bdd100k` |
| Gom ảnh calibration | `make pack SPLIT=calibration` | `python lab9.py pack calibration` |
| Đo bất đồng | `make calib FILES="project/06_calibration_exports/an.zip project/06_calibration_exports/binh.zip"` | `python lab9.py calib project/06_calibration_exports/an.zip project/06_calibration_exports/binh.zip` |
| Freeze gold (**chỉ #6**) | `make freeze` | `python lab9.py freeze` |
| Tạo gói blind (#9) | `make handoff` | `python lab9.py handoff` |
| Chấm bài peer (#9) | `make score FILE=peer.zip` | `python lab9.py score peer.zip` |
| Tính GTS (#9) | `make gts` | `python lab9.py gts` |
| Xem gate và việc còn thiếu | `make status` | `python lab9.py status` |
| Kiểm gói nộp (#1) | `make check` | `python lab9.py check` |

## 6. Điểm số

| Tiêu chí | Điểm |
|---|---|
| Guideline rõ và đủ | 20 |
| Blind handoff (GTS + feedback + revision) | 20 |
| Ontology + CVAT setup | 15 |
| Edge case library | 15 |
| QA design + metrics | 15 |
| Problem + downstream framing | 10 |
| Calibration evidence | 5 |

**GTS = 0.60·D + 0.20·C + 0.10·G + 0.10·I** — D: decision đúng, C: decision critical đúng, G: geometry đúng,
I: độc lập (0 câu hỏi = 100, 1–2 = 70, 3–4 = 40, ≥ 5 = 0).

⚠ **Critical cap:** peer mắc lỗi critical vì guideline thiếu/mơ hồ mà mình không có rule hay escalation để ngăn →
phần Blind handoff tối đa **10/20**. Vì vậy #6 và #7 phải đảm bảo mọi decision critical đều có rule rõ trong guideline.

## 7. Luật không được phá

- **Sau freeze không sửa gold và sample pack.** Chỉ #6 chạy `freeze`; hai người cùng freeze sẽ ra hai tag khác nhau.
- **Không cho nhóm peer xem** gold, tag hay edge-case card trước khi test xong.
- **Không giải thích rule bằng miệng** trong blind window. Câu hỏi của peer ghi trung thực vào clarification log.
- Rule chỉ nói miệng thì coi như không tồn tại — mọi thứ phải nằm trong `02_guideline.md`.
- Kẹt domain ("cái này có phải lane không?") là việc của nhóm: ghi thành rule, UNKNOWN hoặc ESCALATE. Lab Coach không
  trả lời thay. Kẹt kỹ thuật (CVAT, Docker, git) quá 3 phút thì gọi Lab Coach.
- Git: `git pull` trước khi sửa, commit nhỏ, push thường xuyên, chỉ commit file mình giữ.
- Chỉ dùng ảnh trong `data/`, giữ nguyên `ATTRIBUTION.txt`.

## 8. Việc cần làm ngay

1. Điền tên vào cột "Tên" ở bảng mục 3.
2. #1 hỏi Lab Coach: đề quy định nhóm 3–5 người — nhóm 9 người có được giữ nguyên hay phải tách 4 + 5 (khi đó hai
   nhóm test chéo bài nhau).
3. Mọi người bật CVAT và chạy `python lab9.py cvat` trước phút 0.
