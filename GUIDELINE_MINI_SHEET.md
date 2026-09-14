# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Lê Danh Trung<br>
**MSSV:** 2A202602317<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO — ghi `SOLO` nếu làm cá nhân

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

- Ảnh và mã vật thể: Ảnh 1 (`drive_008.jpg`), mã VAN 9
- Dấu hiệu nhìn thấy: Xe có hình dáng thân hộp kín, ngắn, phần nóc phẳng và không có dãy cửa sổ dài dọc thân xe đặc trưng của xe khách.
- Quy tắc áp dụng: Lớp `van` dành cho thân hộp nhỏ, kín; trong khi `bus` đòi hỏi thân dài và nhiều cửa sổ/hàng ghế.
- Quyết định: Gán lớp `van`
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Bật thuộc tính `review_state` thành `needs_review` để ghi nhận sự không chắc chắn, tuyệt đối không tự đoán mò.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: Ảnh 3 (`drive_038.jpg`), mã TRUCK 77
- Dấu hiệu nhìn thấy: Xe có khoang lái và phần sàn chở hàng phía sau tách biệt rõ ràng.
- Quy tắc áp dụng: Quy định lớp `truck` khi nhìn thấy sàn chở hàng/thiết bị công vụ rõ ràng.
- Quyết định: Gán lớp `truck` cho chiếc xe.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Chọn `visibility` là `unclear` và đánh dấu `needs_review` để tiến hành đối chiếu lại sau.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: Ảnh 2 (`drive_033.jpg`), mã BUS 52 (sát mép dưới cùng)
- Dấu hiệu nhìn thấy khi phóng 100%: Phần đầu xe bị cắt ngang bởi mép dưới của ảnh, chỉ hiển thị phần nóc với cục điều hòa cùng một phần nhỏ thân sau.
- Giá trị `visibility`: clear
- Giá trị `boundary`: truncated
- Trạng thái `review_state`: confident
- Lý do: Kích thước bề ngang lớn, cấu trúc nóc đặc trưng và tỷ lệ khung gầm còn lại đủ bằng chứng vững chắc để phân lớp là `bus`.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [x] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [x] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 77 — 40–60 là mục tiêu khối lượng, không phải điểm cắt.
