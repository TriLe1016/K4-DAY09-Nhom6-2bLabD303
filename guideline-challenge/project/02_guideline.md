Version: v1

# Guideline gán nhãn biển báo GTSDB

Bản đầu tiên, dùng để nhóm thử gán nhãn và ghi lại những chỗ chưa rõ. Các ngưỡng dưới đây là quy ước của nhóm, chưa được kiểm chứng qua calibration. Tài liệu này dùng với `03_cvat_labels.json` hiện tại.

## 1. Objective & Scope — Mục tiêu và phạm vi

Gán nhãn các biển giao thông trong ảnh GTSDB để phục vụ phát hiện và phân loại biển báo.

- Gán mặt trước của biển cấm, hiệu lệnh, cảnh báo, ưu tiên, chỉ dẫn và bảng phụ. Gán cả biển ở bên trái hoặc khác làn xe camera.
- Chỉ dùng ảnh GTS được giao; không cần đọc tên địa danh hay toàn bộ chữ trên bảng phụ.
- Không gán đèn giao thông, cột, quảng cáo hoặc hình phản chiếu.
- Các nhóm nhãn là cách chia của bài lab, không phải bản sao đầy đủ nhãn gốc GTSDB.

## 2. Annotation Unit — Đơn vị gán nhãn

Một tấm biển là một đối tượng. Hai biển chung một cột phải vẽ hai box. Bảng phụ có biên riêng cũng vẽ riêng. Một tấm có nhiều dòng chữ vẫn là một đối tượng.

Dùng rectangle ở chế độ **Shape**, không dùng Track.

## 3. Geometry Rule — Cách vẽ box

- Vẽ box ôm sát toàn bộ mặt biển nhìn thấy, gồm cả viền biển.
- Không bao cột, giá đỡ, bóng hoặc quầng sáng. Không chỉ vẽ quanh chữ số.
- Biển bị che: bao phần còn nhìn thấy, không đoán phần bị che. Biển bị cắt mép: dừng box tại mép ảnh.
- Đo kích thước ở ảnh gốc, không theo mức zoom. Cạnh ngắn dưới 10 pixel: dùng `ignore_region`, chọn `reason=too_small`; đúng 10 pixel được giữ.
- Khi kiểm box, cho phép mỗi cạnh lệch tối đa 1 pixel nếu cạnh ngắn dưới 30 pixel, hoặc 2 pixel nếu từ 30 pixel trở lên. Không tăng box để cố đạt ngưỡng 10 pixel.

## 4. Taxonomy — Nhãn và thuộc tính

Các label trong CVAT:

| Label | Cách dùng |
|---|---|
| `traffic_sign` | Box cho biển đủ điều kiện |
| `ignore_region` | Box cho biển bị loại theo mục 5–6 |
| `image_escalate` | Tag ảnh khi có vật nghi là biển hoặc không thể vẽ box chắc chắn |
| `image_status` | Tag ghi trạng thái ảnh sau khi làm xong |

Với `traffic_sign`, chọn các thuộc tính sau:

| Thuộc tính | Giá trị và ý nghĩa |
|---|---|
| `family` | `prohibitory`: cấm/hạn chế; `mandatory`: hiệu lệnh; `warning`: cảnh báo; `priority`: ưu tiên; `information`: chỉ dẫn/thông tin; `supplementary`: bảng phụ; `unknown`: chưa xác định |
| `sign_kind` | `speed_limit`: biển tròn giới hạn tốc độ tối đa; `stop`: STOP; `yield`: nhường đường; `other`: biết không thuộc ba loại đó; `unknown`: chưa phân biệt được |
| `speed_value` | Chọn số đọc được: 5, 10, 15, 20, 25, 30, 40, 50, 60, 70, 80, 90, 100, 110, 120, 130. Không áp dụng: `not_applicable`; không đọc được: `unknown`; số ngoài danh sách: `other` và ghi số vào note |
| `occlusion` | `none`: không bị vật khác che; `partial`: che một phần không quá 60%; `unknown`: không xác định được mức che |
| `truncated` | Bật khi biển bị cắt bởi mép ảnh |
| `degraded` | Bật khi mờ, tối hoặc lóa làm khó đọc biển |
| `needs_review` | Bật khi cần người kiểm tra giải quyết |
| `review_note` | Ghi vị trí và điều chưa rõ; không để trống nếu cần review |

Một số quy tắc phân loại:

- STOP và tam giác ngược nhường đường thuộc `priority`, không phải `warning`; `speed_value=not_applicable`.
- Biển giới hạn tốc độ đang áp dụng: `prohibitory` + `speed_limit`. Biển hết giới hạn tốc độ: `information` + `other` + `not_applicable`.
- Biển tròn xanh hiệu lệnh thuộc `mandatory`; không coi mọi biển xanh là hiệu lệnh. Biển vuông chỉ dẫn người đi bộ thuộc `information`.
- `sign_kind=other` thì tốc độ là `not_applicable`; `sign_kind=unknown` thì tốc độ là `unknown`. Nếu `family=unknown`, chọn cả `sign_kind=unknown`.
- Có giá trị `unknown` hoặc tốc độ `other`: bật `needs_review` và điền note. Không dùng `other` để thay cho việc chưa nhận ra biển.
- Dropdown có mặc định `__undefined__`: phải chọn lại trước khi nộp. Checkbox mặc định tắt nhưng vẫn phải kiểm từng ô.

## 5. Inclusion / Exclusion — Gán gì, bỏ gì

| Gán nhãn | Bỏ qua hoặc ghi ignore |
|---|---|
| Mặt biển rõ, đủ kích thước; kể cả biển tạm thời, chỉ đường, người đi bộ/xe đạp | Đèn, cột, quảng cáo, hình phản chiếu: không vẽ |
| Mỗi bảng phụ là một box riêng | Không gộp bảng phụ vào biển chính |
| Biển nhỏ nhưng cạnh ngắn từ 10 pixel trở lên | Nhỏ hơn 10 pixel: `ignore_region`, reason=`too_small` |
| Biển còn đủ phần nhìn thấy theo mục 6 | Che quá 60%: reason=`heavy_occlusion`; còn dưới 40% trong khung: reason=`heavy_truncation` |
| Biển nghiêng nhưng vẫn thấy được mặt và đủ điều kiện | Chỉ thấy mặt sau: reason=`backside`; chỉ thấy cạnh tấm: reason=`edge_on` |

Nếu có nhiều lý do ignore, chọn theo thứ tự: backside → edge_on → heavy_occlusion → heavy_truncation → too_small. Không tạo cả traffic_sign và ignore_region cho cùng một biển. Ignore region không được coi là biển positive khi dùng dữ liệu huấn luyện.

## 6. Visibility & Occlusion — Che khuất và chất lượng ảnh

- Ước lượng mức che trên diện tích mặt biển, không phải diện tích box. Che từ trên 0% đến 60% vẫn label và chọn `partial`; che hơn 60% thì ignore.
- Bật cờ **Occluded** của CVAT nếu có vật khác che biển. Tối hoặc lóa dùng `degraded`, không tự coi là che khuất.
- Biển cắt mép còn ít nhất 40% mặt biển trong ảnh thì giữ, bật `truncated`; ít hơn thì ignore. Xét điều kiện che và cắt mép riêng.
- Không chắc mức che/cắt có vượt ngưỡng: bật review và ghi lý do. Mức che chưa xác định thì chọn `occlusion=unknown`; nếu chỉ chưa rõ phần cắt mép thì vẫn điền occlusion theo vật che thực tế.
- Chắc là biển nhưng không đọc được nội dung: giữ box nếu đủ điều kiện, phần chưa biết chọn unknown. Không đoán số tốc độ từ loại đường.

## 7. Ambiguity & Escalation — Khi không chắc chắn

- **LABEL:** Draw new rectangle → traffic_sign → Shape, rồi điền thuộc tính.
- **IGNORE:** biển bị loại dùng rectangle ignore_region và chọn reason; vật ngoài scope rõ ràng như quảng cáo thì không vẽ.
- **UNKNOWN:** biết là biển nhưng chưa biết nội dung thì chọn unknown ở thuộc tính tương ứng, bật needs_review, ghi note.
- **ESCALATE:** nếu đã có box, dùng needs_review. Nếu chưa chắc là biển hoặc không thể đặt box, dùng Setup tag → image_escalate, ghi review_note gồm vị trí và câu hỏi. Một tag có thể ghi nhiều ứng viên, phân cách bằng dấu chấm phẩy.

Annotator chuyển vấn đề cho reviewer được nhóm phân công. Reviewer chưa giải quyết được thì giữ trạng thái chờ, không ép chọn một nhãn. Trong blind test, ghi câu hỏi vào log, không nhờ owner giải thích miệng.

Sau khi quét hết ảnh, tạo đúng một tag `image_status` và chọn `status`:

- `unresolved`: còn needs_review hoặc image_escalate, kể cả ảnh có biển rõ khác.
- `positive`: có traffic_sign và không còn review.
- `negative`: không có traffic_sign và không còn review; vẫn có thể có ignore_region.

Chỉ bỏ cờ/tag review sau khi reviewer giải quyết hết vấn đề; cập nhật thuộc tính, ghi kết luận và sửa image_status. Trước export, kiểm không còn __undefined__, review có note, ảnh có status. Lưu **Ctrl+S**, xuất **CVAT for images 1.1**.

## 8. Temporal Rule — Quy tắc thời gian

Không áp dụng vì đây là ảnh tĩnh. Không nối track giữa các ảnh, không nội suy hoặc dùng Outside. Các thuộc tính đều `mutable=false`.

## 9. Examples — Ví dụ

Các ảnh này thuộc split example, không dùng lại cho blind. Bảng chỉ minh họa đối tượng được nêu, không liệt kê toàn bộ annotation của ảnh.

| sample_id | Ví dụ và cách làm |
|---|---|
| GTS18 | Tam giác cảnh báo nằm trên biển tốc độ 30: hai box; trên warning/other, dưới prohibitory/speed_limit với speed_value=30 |
| GTS02 | Tam giác ngược gần camera: priority/yield; biển tròn xanh bên dưới: mandatory/other; không vẽ đèn tín hiệu |
| GTS21 | Biển tròn xanh mũi tên: mandatory; biển vuông người đi bộ: information; các tấm chỉ hướng riêng phải tách box |
| GTS27 | Tam giác ngược sáng bên phải vẫn nhận được nhóm warning; bật degraded, không coi bóng tối là occlusion |
| GTS28 | Bảng treo cửa hàng không phải biển giao thông; quét toàn ảnh rồi ghi negative nếu không có target đủ điều kiện |

## 10. Common Mistakes — Lỗi thường gặp

- Gộp nhiều biển trên một cột: kiểm lại số tấm trước khi vẽ.
- Vẽ box gồm cột hoặc chỉ quanh chữ số: box phải ôm mặt biển.
- Nhầm nhường đường thành cảnh báo: kiểm chiều của tam giác.
- Bỏ biển nhỏ theo cảm giác: đo pixel gốc trước khi quyết định.
- Đoán nội dung biển mờ: dùng unknown và review.
- Quên chọn dropdown hoặc cập nhật status: kiểm toàn bộ ảnh trước khi xuất.
