# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Lê Danh Trung<br>
**MSSV:** 2A202602317<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: `drive_008`, `drive_022`, `drive_033`, `drive_038`
- Số vật thể thực tế: 77
- Mã SHA-256 của gói YOLO của bạn: `d7456e12edf80751ddd4e478ea59516d576e44ebc4ddd057f68395e65d90d836`
- Mã SHA-256 của gói CVAT gốc của bạn: `57cfbf164cd0be450df5b9715c1f0cd9d3857d94f400ef9237251b76f0eca5a1`
- Nguồn đối chiếu: bạn cùng cặp hoặc bộ tham chiếu do người hướng dẫn thực hành cấp: bộ tham chiếu do Lab Coach cấp (teaching_reference)
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: Nhận trực tiếp từ Lab Coach sau khi đã chốt mã SHA-256 cá nhân trên Colab.

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

Mã băm SHA-256 của hai gói YOLO và CVAT gốc do tôi tự làm đã được sinh ra và ghi nhận trong hệ thống trước khi tôi tải gói tham chiếu (có SHA-256 khác biệt hoàn toàn). Sự bất biến của mã băm này chứng minh tôi không sao chép dữ liệu và đã hoàn thành việc gán nhãn một cách độc lập.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_008` (VAN 9) | `van` | Thân hộp kín, phần nóc phẳng, kích thước ngắn, không có dãy cửa sổ dài. | Phân biệt với `bus` (cần thân dài, nhiều cửa sổ) và `truck` (cần có sàn hàng lộ). |
| `drive_038` (TRUCK 77) | `truck` | Khoang lái tách biệt, có sàn chở hàng phía sau. | Lớp `truck` dành cho xe có sàn hàng. |
| `drive_033` (BUS 52) | `bus` | Bị mép ảnh dưới cắt ngang, chỉ thấy nóc có cục điều hòa và bề ngang lớn. | Đủ bằng chứng hình học đặc trưng để phân lớp, đánh dấu `truncated`. |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Chiếc xe cứu hộ (TRUCK 77) mang lớp cố định là `truck` do bản chất thiết kế sàn chở hàng của nó. Tuy nhiên, thuộc tính `visibility` của nó lại thay đổi linh hoạt từ `clear` sang `occluded` vì nó bị một chiếc ô tô con màu trắng đi phía trước che khuất cản trước và bánh xe.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| Vẽ và gán nhãn nhầm loại xe | phạm vi/hình học/gán nhãn | Phóng to 100% và rà soát chéo ảnh đối chiếu `comparison_overlay.png` và quy tắc "Một xe - một hộp". | Điều chỉnh lại cho đúng nhãn |
| Gán `clear` cho xe bị lấp | thuộc tính | Phóng to 100% để kiểm tra độ che khuất (overlap) giữa các xe. | Đổi `visibility` thành `occluded` vì xe bị che mất cản trước. |

- Số hộp `needs_review` trước và sau khi kiểm: 4 hộp trước khi kiểm (phân vân các xe ở xa), 0 hộp sau khi đối chiếu chéo và chốt quyết định.
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: Đối với các xe ở tít góc xa bị mờ thành một vệt xám, tôi đánh dấu `needs_review` và hỏi ý kiến Lab Coach để xác nhận rằng nếu không thấy rõ đặc trưng (bánh xe, hình khối) thì nên xóa box, không đoán mò.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `0 0.651563 0.523438 0.125000 0.089063`
- Tên lớp và tọa độ điểm ảnh `xyxy`: `car`, `[377, 305, 457, 362]`
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

Định dạng text của YOLO chỉ lưu các con số vô tri (mã lớp từ 0-3 và tọa độ chuẩn hóa). Nếu người gán nhãn chọn nhầm lớp (nhìn `van` thành `bus`), hoặc vẽ một bounding box quá rộng bao trọn cả xe bên cạnh, thì file TXT vẫn hợp lệ về mặt cú pháp máy tính nhưng lại sai lệch hoàn toàn về ý nghĩa vật lý.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định: `drive_008`
- Mô tả một dự đoán trong `detect_result.jpg`: Mô hình (train 8 epoch) dự đoán được các xe `car` ở khu vực gần trung tâm khá tốt, nhưng box vẽ hơi lỏng lẻo và có dấu hiệu bỏ sót (false negative) các xe `van`, `bus` ở khoảng cách xa.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Gợi ý rằng dữ liệu huấn luyện cho lớp `van` và `bus` đang quá ít (mất cân bằng lớp học) và thiếu các ví dụ ở nhiều tỷ lệ kích thước khác nhau.
- Minh chứng nào có thể bác bỏ nhận định của bạn? Kích thước tập dữ liệu quá nhỏ (4 ảnh, train trong 42 giây) và số epoch quá ít (8) không đủ cơ sở thống kê vững chắc để kết luận do chất lượng nhãn hay do mô hình chưa hội tụ.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?

Khối lượng 4 ảnh chỉ mang tính chất chẩn đoán kỹ thuật để kiểm tra xem luồng dữ liệu (pipeline) từ CVAT sang PyTorch có bị gãy hay không. Nó thiếu đi sự đa dạng (variance) về góc chụp, ánh sáng và thời tiết để có thể đo lường chỉ số mAP thực tế.

## 6. Đối chiếu nhãn

- Số hộp ghép được: 48
- IoU trung bình và trung vị: 0.861465 và 0.881555
- Mức đồng thuận lớp: 72.9% (0.729167)
- Số hộp phía bạn không ghép được: 29
- Số hộp phía đối chiếu không ghép được: 2
- Một điểm khác biệt cụ thể: Số lượng hộp tôi gán nhưng không thể ghép được lên tới 29 hộp. Trên `comparison_overlay.png`, sự khác biệt lớn nhất là tôi đã gán nhãn rất chi tiết cả những chiếc xe ở xa trên đỉnh dốc hoặc lấp ló phía viền ảnh, trong khi bộ tham chiếu không gán những vật thể này.
- Quy tắc hoặc hành động sửa phát sinh: Áp dụng lại quy tắc "vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp: không đoán". Tôi sẽ rà soát lại 29 box này; nếu xe thực sự chỉ là một vệt điểm ảnh không rõ đặc trưng, tôi sẽ xóa để không đưa nhiễu vào mô hình.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?

Hai người làm có thể đạt IoU cao và đồng thuận lớp 100% không phải vì họ làm đúng quy tắc, mà vì họ cùng mắc một lỗi hệ thống giống hệt nhau (ví dụ: cả hai cùng hiểu sai tài liệu và gộp chung xe tải với rơ-moóc). Đồng thuận chỉ đo mức độ giống nhau, không đo sự chính xác tuyệt đối.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Minh chứng mạnh nhất là bộ file audit json (`input_pool_audit.json`, `my_export_audit.json`, `my_native_export_audit.json`) lưu lại toàn bộ vết tích thời gian và SHA-256 xác thực quy trình làm bài độc lập. Câu hỏi cho Coach: *"Với các camera giao thông có độ sâu trường ảnh lớn, làm thế nào để quy định một ngưỡng kích thước pixel tối thiểu (min bounding box area) thống nhất cho toàn đội gán nhãn, tránh tình trạng người gán xe ở xa, người lại bỏ qua?"*
