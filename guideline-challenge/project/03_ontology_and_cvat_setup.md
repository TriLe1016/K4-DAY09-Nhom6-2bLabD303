# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `traffic_sign` | `rectangle` | class | - | - | false | Đối tượng biển báo giao thông mặt trước hợp lệ cần phát hiện và phân loại cho hệ thống ADAS |
| `family` | - | attribute (select) | `__undefined__`, `prohibitory`, `mandatory`, `warning`, `priority`, `information`, `supplementary`, `unknown` | `__undefined__` | false | Nhóm chức năng chính theo chuẩn Vienna Convention / StVO Đức (Cấm, Hiệu lệnh, Cảnh báo, Ưu tiên, Chỉ dẫn, Biển phụ) |
| `sign_kind` | - | attribute (select) | `__undefined__`, `speed_limit`, `stop`, `yield`, `other`, `unknown` | `__undefined__` | false | Phân loại loại biển trọng yếu tác động an toàn trực tiếp đến xe tự hành (Tốc độ, Dừng, Nhường đường, loại khác) |
| `speed_value` | - | attribute (select) | `__undefined__`, `not_applicable`, `unknown`, `5`, `10`, `15`, `20`, `25`, `30`, `40`, `50`, `60`, `70`, `80`, `90`, `100`, `110`, `120`, `130`, `other` | `__undefined__` | false | Giá trị tốc độ cụ thể cho module Speed Assist; nếu không phải biển giới hạn tốc độ thì chọn `not_applicable` |
| `occlusion` | - | attribute (select) | `__undefined__`, `none`, `partial`, `unknown` | `__undefined__` | false | Đánh giá mức độ che khuất của vật thể bởi cành cây, cột điện, xe khác |
| `truncated` | - | attribute (checkbox) | `false`, `true` | `false` | false | Biển báo nằm ở rìa ảnh bị khung hình cắt mất một phần |
| `degraded` | - | attribute (checkbox) | `false`, `true` | `false` | false | Biển bị suy giảm chất lượng hiển thị (bạc màu, rỉ sét, lóa sáng/ngược sáng mạnh, biến dạng vật lý) |
| `needs_review` | - | attribute (checkbox) | `false`, `true` | `false` | false | Đánh dấu escalate ở cấp vật thể khi annotator không đủ bằng chứng khẳng định, yêu cầu QA rà soát lại |
| `review_note` | - | attribute (text) | Free text | `""` | false | Ghi chú lý do nghi vấn cho QA khi bật `needs_review` |
| `ignore_region` | `rectangle` | class | - | - | false | Vùng chứa biển báo không đạt tiêu chuẩn sử dụng (loại khỏi tập train/eval) |
| `reason` | - | attribute (select) | `__undefined__`, `too_small`, `heavy_occlusion`, `heavy_truncation`, `backside`, `edge_on` | `__undefined__` | false | Lý do loại trừ: nhỏ (<10px), che nặng (>60%), cắt mép nặng (>60%), mặt sau biển, góc nhìn nghiêng mép cạnh |
| `image_escalate` | `tag` | class (image tag) | - | - | false | Escalate cấp toàn ảnh khi ảnh bị lỗi nặng, hỏng file hoặc không thể gán nhãn tin cậy |
| `review_note` | - | attribute (text) | Free text | `""` | false | Ghi chú lý do escalate toàn ảnh cho QA |
| `image_status` | `tag` | class (image tag) | - | - | false | Nhãn tag bắt buộc cho toàn bộ ảnh để xử lý đặc thù tập GTSDB có ảnh negative |
| `status` | - | attribute (select) | `__undefined__`, `positive`, `negative`, `unresolved` | `__undefined__` | false | Trạng thái ảnh: `positive` (có biển hợp lệ), `negative` (xác nhận không có biển), `unresolved` (chưa giải quyết) |

## Class hay attribute

- **Vì sao `traffic_sign` và `ignore_region` là class riêng biệt:**
  - Downstream model detector cần tách biệt hoàn toàn giữa vật thể cần học nhận diện (`traffic_sign`) và các vùng nhiễu cần loại trừ (`ignore_region`). Nếu gộp chung và dùng attribute, model sẽ dễ học nhầm các vùng mặt sau hoặc biển hỏng.
  - Quy tắc hình học và QA khác nhau: `traffic_sign` yêu cầu độ chính xác bounding box rất cao (dung sai ≤ 1–2 px) và kiểm tra 8 thuộc tính ngữ nghĩa; trong khi `ignore_region` chỉ cần khoanh vùng và gán 1 lý do loại trừ.
- **Vì sao `family`, `sign_kind`, `speed_value` là attribute thay vì tách thành hàng chục class:**
  - Nếu tách thành class (như `prohibitory_speed_limit_50`, `priority_stop`,...), số lượng class sẽ bùng nổ tổ hợp (> 50 class), khiến annotator hoa mắt, tăng mạnh tỷ lệ gán nhầm và không phù hợp với kiến trúc phân loại multi-task của mô hình ADAS.
  - Chia theo phân tầng `family → sign_kind → speed_value` phản ánh chính xác cấu trúc ngữ nghĩa phân cấp, dễ kiểm soát và mở rộng.
- **Default nào có thể gây bias khi annotator quên đổi:**
  - Nếu đặt `speed_value` mặc định là `"50"`, annotator quên chọn sẽ tạo ra hàng loạt biển 50 km/h giả mạo ("silent speed bias"), gây thảm họa nếu áp vào module điều khiển xe tự hành.
  - Nếu đặt `status` của `image_status` mặc định là `"positive"`, các ảnh không có biển (`negative`) sẽ bị ghi nhận sai lệch.
  - **Giải pháp:** Toàn bộ các dropdown `select` đều đặt default là `__undefined__`. Bất kỳ export nào còn sót giá trị `__undefined__` đều bị script kiểm tra tự động phát hiện và từ chối.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): `v2.74.1` (theo chuẩn cài đặt local Day 2 tại `http://localhost:8080`)
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): `nhom-calib-v1-minh`
- **Guide của task đã dán `02_guideline.md`?** Có (đã dán toàn bộ markdown của guideline vào phần Description của task)
- **Nhóm dùng Track hay Shape, vì sao:** Dùng **Shape**, vì toàn bộ 28 ảnh trong tập `gtsdb` là ảnh chụp tĩnh độc lập (still frames), không phải chuỗi video liên tục như clip LISA. Dùng Shape tránh phát sinh keyframe nội suy sai và khớp chuẩn với định dạng xuất `CVAT for images 1.1`.

## Setup test

Một thành viên **chưa tham gia setup** (Vũ — spec owner, cnt-vu) mở task trên máy mình và thực hiện kiểm tra:
- **Người test:** Vũ (`cnt-vu`).
- **Khả năng nắm bắt workflow:**
  - *Label gì:* Hiểu rõ việc vẽ `traffic_sign` cho biển hợp lệ, vẽ `ignore_region` cho biển nhỏ/mặt sau/quá che, và gán tag `image_status` cho ảnh.
  - *Tool nào:* Dùng công cụ **Draw new rectangle** (phím tắt `N` để vẽ liên tiếp các box) và **Setup tag** cho nhãn toàn ảnh.
  - *Attribute nào:* Biết chọn phân cấp `family`, `sign_kind`, chọn tốc độ ở `speed_value` (chọn `not_applicable` nếu không phải biển tốc độ), kiểm tra `occlusion` / `truncated` / `degraded`.
  - *Khi nào escalate:* Khi biển bị lóa/mờ không đọc được số thì chọn `speed_value = unknown`, tích `needs_review = true` và nhập ghi chú; nếu ảnh bị hỏng/quá tối toàn phần thì gán tag `image_escalate`.
- **Điểm vấp phát hiện và hành động khắc phục:**
  1. *Điểm vấp:* Khi vẽ biển STOP, người test lúng túng ở dropdown `speed_value` vì thấy danh sách tốc độ dài và giá trị đang là `__undefined__`.
     ➔ *Khắc phục:* Bổ sung quy tắc in đậm trong Guideline: *"Nếu `sign_kind != speed_limit`, luôn luôn chọn `speed_value = not_applicable`"*.
  2. *Điểm vấp:* Ở ảnh hoàn toàn không có biển báo nào (ảnh negative như `GTS28`), người test không biết làm sao để lưu job vì không có box nào để vẽ.
     ➔ *Khắc phục:* Hướng dẫn người test dùng công cụ **Setup tag** ➔ chọn `image_status` ➔ gán giá trị `negative`.
