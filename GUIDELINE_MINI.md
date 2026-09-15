# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Xuân Quang`
Clip: `clip_01`, `clip_02`

---


## 1. Phạm vi

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |


## 2. Luật ID

| Tình huống | Luật | Vì sao |
| --- | --- | --- |
| Che một phần rồi hiện lại | Giữ ID nếu che dưới 25 frame (2 giây @ 12.5 fps), bật Occluded và chỉ ôm phần thấy được. | Cùng một xe không nên bị fragmentation. |
| Che lâu hơn 25 frame | Kiểm frame-by-frame khi xuất hiện lại; nếu quỹ đạo/vẻ ngoài không chắc thì track mới. | Không suy đoán identity. |
| Rời khung rồi quay lại | Outside tại frame đầu vắng mặt; khi quay lại tạo track mới. | Track chỉ tồn tại khi có bbox. |
| Crossing/chồng lên nhau | Theo vị trí, hướng chuyển động, frame trước/sau crossing; không đổi ID vì bbox chồng nhau. | Giữ identity xuyên thời gian. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: bắt đầu tại frame đầu tiên có thể xác định đáng tin là xe bốn bánh. |
| Xe đang đỗ, không di chuyển |  giữ ID đến frame đầu xe vắng mặt |
| Keyframe đặt dày ở đâu |ở entry/exit, đổi hướng, đổi scale, occlusion, crossing và midpoint của đoạn interpolation dài.|


## 4. Ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.


### Ca 1
- Clip / frame / ID: `clip_01`, MOT 82–96 (CVAT 81–95), ID 5.
- Tình huống: bbox drift giữa interpolation; IoU thấp nhất 0.508 tại MOT 82.
- Quyết định: giữ ID 5; chính sách là thêm keyframe dày quanh đoạn đổi hình học.
- Lý do: đây là lỗi geometry/interpolation, không phải đổi danh tính.

### Ca 2
- Clip / frame / ID: `clip_01`, MOT 79–100 (CVAT 78–99), ID 6.
- Tình huống: bbox xuất hiện trước track tham chiếu, tạo 22 bbox thừa.
- Quyết định: kiểm frame đầu tiên xe xác định được; dùng Outside đúng endpoint.
- Lý do: không vẽ bbox khi xe chưa có mặt/xác định được.

### Ca 3
- Clip / frame / ID: `clip_01`, MOT 133–135 và 169–171 (CVAT 132–134 và 168–170), ID 8.
- Tình huống: bbox sớm trước entry và còn sau exit.
- Quyết định: kiểm frame-by-frame quanh transition; bật Outside ở frame đầu xe vắng mặt.
- Lý do: không kéo dài track sang frame không có vật thể.

## 5. Cập nhật sau evaluation

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

Bản pre-gold đạt IDF1 0.956, MOTA 0.908, MOTP 0.858 nhưng có 6 đoạn bbox thừa/treo và 8 frame drift. Annotation không rework sau evaluation; lần sau bắt buộc kiểm endpoint và midpoint của các đoạn interpolation dài, nhất là ID 5/6/8.
