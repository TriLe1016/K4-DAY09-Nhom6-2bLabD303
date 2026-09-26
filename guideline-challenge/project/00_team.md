# Team

Điền trước phút 15. Thay mọi placeholder; còn sót thì `make status` báo ở gate G1.

- **Team:** _(điền tên nhóm Lab Coach công bố)_
- **Nhóm peer test bài của mình:** _(chờ Lab Coach công bố)_
- **Nhóm mình test bài của:** _(chờ Lab Coach công bố)_
- **Problem family:** Traffic sign taxonomy — phân loại biển báo theo nhóm (family/kind/speed) cho biển nhỏ, xa, bị che
  hoặc cắt mép
- **Nguồn ảnh:** `gtsdb` (28 ảnh biển báo Đức, gồm cả ảnh không có biển để làm negative)

| Thành viên | GitHub | Vai trò chính | File phụ trách |
|---|---|---|---|
| Trí | TriLe1016 | Lead + gold owner (người duy nhất chạy `make freeze`) | `00_team.md`, `04_edge_cases/edge_case_cards.md`, `04_edge_cases/gold_decisions.csv` |
| Vũ | cnt-vu | Spec owner — guideline | `01_problem_statement.md`, `02_guideline.md`, `08_revision_log.md` |
| Minh | _(bổ sung)_ | CVAT owner | `03_ontology_and_cvat_setup.md`, `03_cvat_labels.json`, `sample_pack.csv`, `09_cvat_export_or_task_reference.txt` |
| Ly | _(bổ sung)_ | QA owner | `05_qa_plan.md`, `06_calibration_report.csv`, `07_blind_handoff/` |

Calibration: cả 4 người label độc lập. Setup test CVAT: Vũ (chưa tham gia setup). Blind handoff: Vũ + Minh đi làm
peer tester cho nhóm kia; Trí + Ly ở lại làm owner (gửi gói blind, ghi clarification log, chấm điểm).
