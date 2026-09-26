# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v2 | Làm rõ quy tắc tách biển phụ trên biển chỉ dẫn lớn; quy định biển mờ do sương mù chọn speed_value=unknown và bật needs_review; bắt buộc gán tag image_status cho 100% ảnh; nhắc nhở không bỏ sót biển tốc độ 30km/h và bảng phụ. | Khắc phục các bất đồng lớn ghi nhận trong đợt calibration nội bộ giữa 4 annotator (Trí, Minh, Vũ, Ly). | `06_calibration_report.csv` (các dòng GTS09, GTS04, GTS06, GTS01), `06_calibration_measure.csv` |
