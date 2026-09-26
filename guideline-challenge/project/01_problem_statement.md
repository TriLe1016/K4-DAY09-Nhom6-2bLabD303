# Problem statement + downstream contract

## Bài toán

Phát hiện và phân loại biển báo giao thông Đức (GTSDB) theo taxonomy phân cấp `family → sign_kind → speed_value`, khó
ở biển nhỏ/xa, bị che, bị cắt mép, ngược sáng và nhiều biển/bảng phụ chung một cột.

## Downstream contract

1. **Downstream task / model / user là ai?** Bộ dữ liệu huấn luyện và đánh giá cho model detect + classify biển báo
   của hệ thống hỗ trợ lái (ADAS), cụ thể là module cảnh báo tốc độ (speed assist) và cảnh báo biển nhường đường/STOP.
2. **Output annotation nào thực sự cần?** Rectangle ôm mặt biển nhìn thấy cho từng tấm biển; attribute `family`,
   `sign_kind`, `speed_value`; cờ chất lượng `occlusion`, `truncated`, `degraded`; vùng `ignore_region` để loại biển
   không dùng được khỏi training/đánh giá; tag `image_status` cho từng ảnh.
3. **Failure nào gây hậu quả lớn nhất?** (a) Ghi sai `speed_value` hoặc nhầm biển hết giới hạn thành biển giới hạn tốc
   độ — xe áp sai tốc độ; (b) bỏ sót hoặc phân loại sai biển STOP / nhường đường (`priority`) — xe không dừng/nhường;
   (c) bỏ sót biển đủ điều kiện, khiến ảnh bị ghi `negative`. Đây là các decision `critical` trong gold.
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?** Annotator bật `needs_review` + ghi
   `review_note` trên box, hoặc gắn tag `image_escalate` cho cả ảnh, và đặt `image_status=unresolved`. QA owner (Ly)
   review; chưa giải quyết được thì ảnh giữ `unresolved` và bị loại khỏi tập training, không ép chọn nhãn.

## Scope

- **Trong scope (bắt buộc label):** mặt trước biển cấm, hiệu lệnh, cảnh báo, ưu tiên, chỉ dẫn và bảng phụ (mỗi tấm một
  box), kể cả biển ở làn khác hoặc bên trái đường, có cạnh ngắn ≥ 10 px ở ảnh gốc.
- **Ngoài scope (ignore):** đèn giao thông, cột, quảng cáo, biển cửa hàng, hình phản chiếu — không vẽ. Biển nhỏ < 10 px,
  che > 60%, còn < 40% trong khung, mặt sau hoặc nhìn cạnh — vẽ `ignore_region` kèm `reason`.
- **Geometry tolerance:** box ôm sát mặt biển nhìn thấy gồm cả viền, không gồm cột; mỗi cạnh lệch ≤ 1 px nếu cạnh ngắn
  của biển < 30 px, ≤ 2 px nếu ≥ 30 px.

## Output chấm được

- **LABEL:** box `traffic_sign` + đủ attribute (không còn `__undefined__`).
- **IGNORE:** box `ignore_region` + `reason`.
- **UNKNOWN:** giá trị `unknown` trong `family` / `sign_kind` / `speed_value` / `occlusion`, kèm `needs_review`.
- **ESCALATE:** `needs_review` trên box, hoặc tag `image_escalate` cho cả ảnh.
- **Trạng thái ảnh:** tag `image_status` = `positive` / `negative` / `unresolved`.

Tất cả đều nằm trong export **CVAT for images 1.1**.

## Dữ liệu và giới hạn

Chỉ dùng 28 ảnh `gtsdb` (1360×800, biển Đức): 5 ảnh example, 5–8 ảnh calibration, 5 ảnh blind. Giới hạn: biển theo quy
chuẩn Đức nên taxonomy không áp thẳng được cho biển Việt Nam; ít ảnh nên mỗi edge case chỉ có 1–2 mẫu; không có ảnh
đêm/mưa nên `degraded` chủ yếu đến từ ngược sáng hoặc biển nhỏ.
