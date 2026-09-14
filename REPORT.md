# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Trương Công Hoài Nam<br>
**MSSV:** 2A202602137<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: "f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33"
- Bốn mã ảnh: drive_008, drive_022, drive_033, drive_038 
- Số vật thể thực tế: ~120
- Mã SHA-256 của gói YOLO của bạn: "85627848d89f3eb50f27df23c70619710289fb8278bdf8cf41f50eba08122eba"
- Mã SHA-256 của gói CVAT gốc của bạn: "1eac984a3566796933c12df419d046d7e77390e29de9b47ca1b0cb28830339ab"
- Nguồn đối chiếu: bạn cùng cặp hoặc bộ tham chiếu do người hướng dẫn thực hành cấp: bộ tham chiếu do người hướng dẫn thực hành cấp
- Mã SHA-256 của gói đối chiếu: "c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b"
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: 1 lần lúc 3:03 PM

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

Việc tôi dán nhãn và dự đoán hoàn toàn có thể chạy được không cần đối chiếu, việc đối chiếu chỉ được thực hiện sau khi đã hoàn thành bài nên không ảnh hưởng.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| ô tô con | car | Kích thước nhỏ, vừa, thân xe con | Xe con cá nhân |
| xe tải | truck | Thân xe có phần cabin và thùng chở hàng phía sau | Phương tiện chuyên chở hàng hóa/vật liệu có cấu trúc thùng hàng chuyên dụng |
| xe buýt | bus | Thân xe dài, lớn, nhiều cửa kính và khoang hành khách lớn | Đối tượng vận tải hành khách công cộng cỡ lớn, đóng một bounding box bao trùm toàn bộ cả hai toa nối |
| xe van | van | thân xe dạng khối kín, kích thước lớn hơn ô tô con nhưng nhỏ hơn xe tải, khoang sau liền thân | Phương tiện chở người/hàng cỡ nhỏ hình hộp, đuôi phẳng |


Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Một chiếc xe có thể được gán lớp car vì nó là ô tô con, thuộc tính  là occluded nếu một phần thân xe bị phương tiện khác che khuất, hoặc clear nếu không bị che khuất
## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| Tạo bounding box cho xe mờ ở xa ngã tư (đối tượng đã cắm cờ needs_review) | phạm vi | Đối chiếu với bộ tham chiếu và kết quả YOLO: cả hai nguồn chuẩn đều không phát hiện, không gán nhãn đối tượng này. | Xóa bỏ hộp nhãn này. Quy tắc: Chỉ gán nhãn các đối tượng có kích thước và độ rõ nét |

- Số hộp `needs_review` trước và sau khi kiểm: trước: 15, sau: 15 (các cờ needs_review vẫn chưa được nhận diện)
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: Các đối tượng ở rìa xa của ảnh bị mờ nhòe nghiêm trọng và che khuất một phần (kích thước quá nhỏ, mất nét chi tiết để phân biệt rõ giữa car, van hoặc vật thể nền), do đó tôi đã chủ động gắn cờ needs_review. Đối chiếu trực tiếp với kết quả dự đoán của mô hình YOLO và bộ nhãn tham chiếu do Lab Coach cấp. Kết quả cho thấy: Cả mô hình YOLO lẫn bộ nhãn tham chiếu đều hoàn toàn không nhận diện / bỏ sót các đối tượng này.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: tâm=(0.3941, 0.7310) | kích thước=(0.4628, 0.3600)
- Tên lớp và tọa độ điểm ảnh `xyxy`: lớp=2 (bus),  [104.1, 352.7, 400.3, 583.0]
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

Một dòng nhãn YOLO với các số thực nằm hợp lệ trong khoảng [0, 1] chỉ đảm bảo tính đúng cú pháp (syntax validity) để máy đọc không bị lỗi parse dữ liệu, hoàn toàn không đảm bảo tính chính xác ngữ nghĩa và không gian thực tế:
Sai lớp: ID lớp là số nguyên hợp lệ (2) nhưng có thể bị gán nhầm đối tượng (ví dụ: xe khách cỡ lớn gán thành truck hoặc car).

Sai phạm vi: Cú pháp không thể xác định đối tượng bên trong hộp có thực sự tồn tại hay không; người gán có thể đóng khung nhầm dải phân cách, bóng râm (False Positive) hoặc vật thể nằm dưới ngưỡng kích thước cho phép.

Sai hình học: Tọa độ hợp lệ không chứng minh được độ khít (tightness). Bounding box có thể bị lệch tâm, ôm thừa bóng đổ xe dưới mặt đường, hoặc cắt xén mất một phần đuôi/đầu xe mà không hề vi phạm bất kỳ ràng buộc cú pháp nào.
## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: drive_022, drive_033, drive_038 
- Mã ảnh thẩm định: drive_008
- Mô tả một dự đoán trong `detect_result.jpg`:
    Mô hình nhận diện xe nhưng có độ tin cậy thấp, bị bỏ sót một số xe ở xa, hoặc dự đoán nhầm lớp phương tiện.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào?
    Về quy tắc nhận diện kích thước: Kiểm tra lại xem mô hình có xu hướng bỏ sót các phương tiện kích thước nhỏ ở xa hay không, từ đó xác định khoảng kích thước pixel tối thiểu mà mô hình có thể phát hiện được.
- Minh chứng nào có thể bác bỏ nhận định của bạn?
    Thời gian huấn luyện quá ngắn: chỉ chạy đúng 8 epochs trên CPU trong 21.87 giây, hàm mất mát (loss) chưa kịp hội tụ.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?
    Bản thân log đầu ra đã ghi rõ "purpose": "chỉ dùng để phản hồi và tìm lỗi dữ liệu" và "not_production_benchmark": true.
    4 ảnh đều cùng một góc camera và cùng điều kiện ánh sáng tĩnh, hoàn toàn không đại diện được cho các biến số phức tạp ngoài thực

## 6. Đối chiếu nhãn

- Số hộp ghép được:48
- IoU trung bình và trung vị: IoU trung bình:0.844836, IoU trung vị:0.866725
- Mức đồng thuận lớp:70.83%
- Số hộp phía bạn không ghép được:54
- Số hộp phía đối chiếu không ghép được:2
- Một điểm khác biệt cụ thể: Tôi gán nhãn nhiều hơn so với bộ đối chiếu
- Quy tắc hoặc hành động sửa phát sinh:Cần rà soát kỹ lại các vật thể ở xa, chỉ gán nhãn khi nhìn rõ và đủ bằng chứng phân lớp, tránh đoán hoặc vẽ bao phủ cả những khu vực không rõ ràng
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?
    Mức đồng thuận chỉ đo lường sự thống nhất và khả năng tái lập quy tắc giữa hai nguồn gán nhãn độc lập. Nếu cả hai bên cùng hiểu sai quy chuẩn gán nhãn từ đầu hoặc cùng mắc một sai lầm hệ thống, mức đồng thuận vẫn sẽ rất cao nhưng nhãn vẫn bị sai so với thực tế khách quan.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Việc chủ động gắn cờ needs_review cho các phương tiện ở rìa xa bị mờ nhòe trước khi đối chiếu, sau đó kết quả thực tế cho thấy cả mô hình YOLO lẫn bộ tham chiếu đều hoàn toàn bỏ sót các hộp này. Điều này chứng minh quy trình tự kiểm tra đã phát hiện chính xác ranh giới của dữ liệu mơ hồ và xác định đúng ngưỡng giới hạn kích thước mà mô hình chưa thể tiếp nhận.

Đối với các phương tiện bị che khuất một phần ở xa, có cần thiết phải nhận diện được không, hay chỉ cần đợi xe lại gần để đủ pixel và nhận diện đúng?