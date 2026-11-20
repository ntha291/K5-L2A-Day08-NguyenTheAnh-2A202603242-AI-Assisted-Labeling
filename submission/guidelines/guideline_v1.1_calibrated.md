# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603242 - Nguyễn Thế Anh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 và thêm quy tắc mới rút ra từ lỗi của máy / bẫy lọt trong quá trình review. Mỗi ca khó có dẫn ảnh ví dụ (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Phân định dựa trên **sự hiện diện của bàn đạp cơ học**: Nếu xe có bàn đạp trợ lực chân rõ ràng -> gán `bicycle`. Nếu không có bàn đạp (chỉ có sàn để chân cố định), bánh xe lớn, đèn pha xe máy -> gán `motorbike`. | img_018.jpg, img_021.jpg | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc hoặc ùn tắc gần camera: từng xe hay `ignore_region`? | Nếu cụm từ 5 xe trở lên nằm chen chúc, che khuất nhau > 50% diện tích nhìn thấy hoặc không phân biệt rõ ranh giới bánh/thân xe -> vẽ **một** `ignore_region` bao trùm cụm. Các xe ở rìa ngoài còn nhìn thấy rõ > 50% diện tích vẫn gán box `motorbike` riêng. | img_005.jpg, img_028.jpg | CAL-03 |
| 3 | Hàng hoá nhô ra khỏi xe: box xe có gồm hàng không? | Hàng hoá gắn cố định trên giỏ/giá chở hàng của xe máy/xe đạp thuộc về box xe. Nếu hàng hóa nhô dài ra ngoài > 20% chiều dài xe (ví dụ chở thanh tre, ống nhựa dài), box chỉ ôm khung xe, phần nhô ra không tính vào box để tránh làm loãng IoU phát hiện thân xe. | img_004.jpg, img_034.jpg | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg, img_002.jpg | Tuk-tuk, xích lô, xe ba gác trên đường phố | YOLO26 gọi tuk-tuk là `car` hoặc `truck` (conf 0.72 - 0.88) vì tập COCO không có nhãn ba bánh | Không dùng YOLO26 gán xe ba bánh; bắt buộc dùng YOLOE hoặc gán tay `three_wheeler`. Người lái/khách ngồi bên trong có box `person` riêng nếu nhìn thấy >= 20% diện tích cơ thể. | Bổ sung Mục 2 & Mục 4.1.4: Phân loại triệt để xe ba bánh địa phương vào `three_wheeler`, tách biệt hoàn toàn với `car`/`truck`. |
| img_017.jpg | Bẫy missing person ở vùng hậu cảnh vỉa hè bị lọt | Script gài bẫy xoá mất 1 box `person` ở xa; người duyệt bị thiên kiến tự động hóa, chỉ nhìn các box máy gợi ý mà không rà soát người đi bộ ở hậu cảnh | Thêm bước kiểm tra bắt buộc trong quy trình review: Trước khi bật layer nhãn máy, phải tắt nhãn và quét mắt độc lập 10 giây qua các vùng vỉa hè, lề đường để đếm người đi bộ. | Bổ sung Quy trình Review: Bước rà soát hậu cảnh độc lập nhằm triệt tiêu thiên kiến xác nhận từ nhãn máy. |
| img_020.jpg, img_036.jpg | Người lái và hành khách trên xe máy / xích lô bị che khuất một phần | Detector bỏ sót hành khách ngồi sau hoặc chỉ phát hiện 1 box gộp cả người và xe | Tách thành 2 box riêng biệt: 1 box `person` (ôm từ đầu đến phần chân/thân nhìn thấy) và 1 box phương tiện (`motorbike`/`three_wheeler`). Hai box chồng lấp nhau là hoàn toàn hợp lệ. | Bổ sung Mục 4.1.4: Quy tắc gán tách người ngồi trên phương tiện ngay cả khi diện tích nhìn thấy chỉ đạt ngưỡng tối thiểu 20%. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản khởi tạo của giảng viên, để ngỏ 7 câu hỏi và chưa phân định rõ ranh giới cụm xe/xe điện | — |
| v1.1 | Chốt quy tắc xe điện (dựa vào bàn đạp); quy tắc ngưỡng che khuất > 50% cho `ignore_region` cụm xe máy; chốt triệt để phân loại xe ba bánh địa phương tách khỏi `car`/`truck`; siết quy trình review chống lọt người hậu cảnh | CAL-01, CAL-02, CAL-03, CAL-04 (dựa trên quan sát từ `img_002`, `img_005`, `img_013`, `img_017`, `img_018`, `img_021`, `img_028`) |
