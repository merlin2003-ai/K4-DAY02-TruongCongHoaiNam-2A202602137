# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Trương Công Hoài Nam<br>
**MSSV:** 2A202602137<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` (xe buýt) | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` (xe van) | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt; khoang hàng tách biệt như xe tải |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `visibility` (mức nhìn thấy) | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy |
| `boundary` (quan hệ mép ảnh) | `inside` (trong ảnh), `truncated` (bị cắt) | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại) | đánh dấu quyết định cần quay lại |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: drive_008, van 23
- Dấu hiệu nhìn thấy: Xe có hình dáng và các đặc điểm nhận dạng giống xe bus nhưng kích thước bé hơn
- Quy tắc áp dụng: Xe bus là xe chở khách cỡ lớn, 1 hoặc 2 tầng, thường có 6 bánh. Xe van là xe chở hàng/khách có kích thước bé hơn, thường 4 bánh
- Quyết định: gán nhãn xe van
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Giữ nguyên phân loại tạm thời là van theo cấu trúc thân xe thực tế, gán thuộc tính needs_review nếu quá mơ hồ hoặc chưa đủ đặc điểm nhận dạng

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: drive_033, truck 64
- Dấu hiệu nhìn thấy: xe có hình dáng vuông, gọn do góc nhìn nên thấy nhỏ
- Quy tắc áp dụng: có thùng chở hàng tách biệt với khoang lái
- Quyết định: gán nhãn xe tải
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng to để quan sát khe hở phân cách giữa cabin người lái và thùng hàng phía sau

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: drive_008, car 21
- Dấu hiệu nhìn thấy khi phóng 100%: vẫn thấy phần kính và nóc của xe
- Giá trị `visibility`: unclear
- Giá trị `boundary`: truncated
- Trạng thái `review_state`: needs_review
- Lý do: Đối tượng đồng thời vừa bị cắt cụt bởi mép ảnh, vừa bị che khuất một phần bởi vật thể khác. Hình dạng nhìn thấy không còn đầy đủ cấu trúc đặc trưng của một chiếc ô tô hoàn chỉnh.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [x] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [x] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 120 — 40–60 là mục tiêu khối lượng, không phải điểm cắt.
