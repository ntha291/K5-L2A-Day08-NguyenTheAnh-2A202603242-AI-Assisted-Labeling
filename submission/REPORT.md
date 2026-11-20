# Báo cáo Day 08 — 2A202603242 - Nguyễn Thế Anh

> Báo cáo tổng hợp dự án Day 08: Pre-label, Review và Calibration. Mọi con số đều kèm **mẫu số** cụ thể và dẫn xuất từ các file log/kết quả thực nghiệm trong thư mục `submission/`.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.15 | `r01`, `threshold_sweep.md`: Lớp người có mật độ cao, nhiều đối tượng ở xa/bị che khuất; ngưỡng 0.15 đạt Recall 92% (>= 90%) và Precision 79% trên 37 đối tượng gold. |
| bicycle | YOLO26m | 0.20 | `r01`, `threshold_sweep.md`: Lớp ít vật nhưng quan trọng, ngưỡng 0.20 đạt Recall 100% và Precision 71% trên 5 đối tượng gold, tránh bị mất box khi tăng ngưỡng như ở `r02`. |
| motorbike | YOLO26m | 0.25 | `r01`, `threshold_sweep.md`: Ngưỡng 0.25 tối ưu đạt Recall 100% và Precision 75% trên 9 đối tượng gold; nếu tăng lên 0.50 (`r02`) số xe máy giảm nghiêm trọng từ 55 xuống 32 box. |
| car | YOLO26m | 0.50 | `r02`, `threshold_sweep.md`: Ô tô có đặc trưng hình khối rõ nét, ngưỡng 0.50 đạt cả Recall 91% và Precision 91% trên 11 đối tượng gold, lọc sạch các box nhiễu ở hậu cảnh. |
| bus | YOLO26m | 0.30 (mặc định) | `threshold_sweep.md`: Chỉ có 1 đối tượng trong 4 ảnh gold (< 5 vật). Mẫu quá nhỏ để sweep riêng, giữ ngưỡng an toàn mặc định 0.30. |
| truck | YOLO26m | 0.30 (mặc định) | `threshold_sweep.md`: Có 0 đối tượng trong 4 ảnh gold (< 5 vật). Không thể sweep riêng, giữ ngưỡng mặc định 0.30 theo quy chuẩn rubric. |
| three_wheeler | YOLOE-26s (prompt: 'tuk-tuk', 'cyclo', 'cargo tricycle') | 0.30 (mặc định) | `r03`, `threshold_sweep.md`: YOLOE phát hiện tốt xe ba bánh địa phương mà YOLO26 bỏ sót. Không có ngưỡng nào <= 0.5 đạt Recall 90% (tối đa 83% trên 6 vật gold), do đó giữ mức 0.30 để cân bằng. |

- **Lệnh `merge_prelabels.py` đã dùng:**
  ```bash
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.15,bicycle=0.2,motorbike=0.25,car=0.5" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
- **Phiên bản SAM bạn chọn và lý do:**
  Chọn **SAM 2.1 ViT-B** (`sam2.1_b.pt`).
  *Lý do:* SAM 2.1 cho đường biên mask cực kỳ chi tiết, vượt trội hoàn toàn so với MobileSAM ở các chi tiết nan hoa bánh xe, khung mui xích lô và tay lái. So với SAM 3 (cần GPU rất nặng và xin quyền truy cập gated model trên HuggingFace), SAM 2.1 chạy ổn định trên CPU/RAM máy cá nhân, cân bằng hoàn hảo giữa độ chính xác và khả năng tái lập.
- **Lỗi không sửa được bằng đổi ngưỡng:**
  Lỗi nhầm lẫn bản chất taxonomy: YOLO26 nhận diện xích lô và tuk-tuk thành `car` hoặc `truck` (ví dụ `img_001.jpg` nhận cyclo thành truck conf 0.88; `img_013.jpg` nhận tuk-tuk thành car conf 0.72). Dù có tăng hay giảm conf thì model closed-set vẫn chỉ xuất nhãn trong 80 lớp COCO gốc. *Cách sửa:* Bắt buộc thay thế hoặc bổ sung bằng open-vocabulary detector (YOLOE với prompt text chuyên biệt) hoặc gán tay bổ sung.
- **Box điểm cao mà sai hoặc điểm thấp mà đúng:**
  - *Điểm cao mà sai:* Tại `img_001.jpg`, YOLO26 phát hiện chiếc xích lô lớn ở tiền cảnh với nhãn `truck` điểm tin cậy cao ngất ngưởng **0.88**.
  - *Điểm thấp mà đúng:* Tại `img_036.jpg`, người đi bộ ở xa có điểm tin cậy chỉ **0.16** nhưng là đối tượng hoàn toàn hợp lệ theo guideline.
- **Giới hạn chọn ngưỡng trên gold:**
  Tập gold chỉ gồm 4 ảnh và tập này vừa được dùng để chọn ngưỡng (in-sample tuning), vừa dùng để đánh giá điểm số. Do đó, các chỉ số Precision và Recall của máy trên tập gold mang tính **lạc quan** (overly optimistic) và không đại diện tuyệt đối cho phân phối tổng thể.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 588 | 96.2% | 588/611 box máy |
| edited (sửa kích thước/vị trí) | 4 | 0.7% | 4/611 box máy |
| deleted (xoá box sai) | 19 | 3.1% | 19/611 box máy |
| added (vẽ thêm đối tượng sót) | 1 | 0.2% | 1/593 đối tượng sau duyệt |

- **Lớp nào máy tệ nhất? Vì sao:**
  Lớp `three_wheeler` và `person` ở hậu cảnh xa. Với `three_wheeler`, YOLO26 nhận diện sai toàn bộ sang `car`/`truck`, YOLOE tuy bắt được nhưng box thường bị lỏng (loose box ở `img_034.jpg`). Với `person`, máy hay bỏ sót người ngồi sau xe máy (`img_020.jpg`, `img_036.jpg`) và các cụm người đi bộ dày đặc ở xa.
- **So với gold:**
  Precision của máy: **79.7%** (63 TP / [63 TP + 16 FP], KTC 95%: 70–87%).
  Recall của máy: **91.3%** (63 TP / [63 TP + 6 FN], KTC 95%: 82–96%).
  *Mẫu số:* 69 đối tượng trong 4 ảnh gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Kết quả bắt bẫy:** Bắt được **10/12 bẫy = 83.3%** (KTC 95%: 55–95%), vượt ngưỡng tối thiểu yêu cầu (80%).
  - `wrong_class`: bắt được 3/3 (100%) — `img_014.jpg` (bicycle -> person), `img_006.jpg` (bicycle -> motorbike), `img_035.jpg` (bicycle -> person).
  - `extra`: bắt được 3/3 (100%) — xoá thành công phantom car tại `img_025.jpg`, phantom truck tại `img_003.jpg`, phantom person tại `img_024.jpg`.
  - `missing`: bắt được 2/3 (66.7%) — phục hồi motorbike tại `img_021.jpg`, person tại `img_008.jpg`; **lọt bẫy tại `img_017.jpg`** (missing person).
  - `loose`: bắt được 2/3 (66.7%) — siết chặt box three_wheeler tại `img_034.jpg`, person tại `img_022.jpg`; **lọt bẫy tại `img_028.jpg`** (loose person).
- **Phân tích bẫy lọt:**
  - Tại `img_017.jpg`: Người đi bộ ở vùng hậu cảnh vỉa hè bị script xoá. Khi review, người duyệt bị cuốn vào kiểm tra các phương tiện lớn ở lòng đường mà bỏ qua bước 1 trong checklist (tắt nhãn máy và quét độc lập toàn cảnh).
  - Tại `img_028.jpg`: Khung cảnh đường phố cực kỳ đông đúc (nhiều xe máy và người chen chúc), box của person bị nở rộng nhưng mắt người duyệt bị đánh lừa bởi bối cảnh xung quanh nên chấp nhận nhanh mà không phóng to kiểm tra mép biên.
- **Ý nghĩa của Accepted cao vs Bắt bẫy:**
  Accepted đạt tới 96.2% trong khi vẫn lọt 2 bẫy chứng minh rõ ràng **thiên kiến tự động hóa (automation bias)**: Người duyệt có xu hướng tin tưởng vào các gợi ý sẵn có của máy, dễ dàng "bấm duyệt" các box trông có vẻ hợp lý và chỉ tập trung sửa các lỗi hiển nhiên (như box sai lớp hoàn toàn) mà bỏ qua các lỗi thụ động (bỏ sót vật thể hoặc box lỏng).

## 4. Máy có đáng dùng không?

- **Thời gian (từ `time_log.csv`):**
  - Gán tay thuần túy (`gold_manual`): **6.1 phút/ảnh** (24.5 phút cho 4 ảnh).
  - Duyệt có pre-label (`batch_review`): **3.0 phút/ảnh** (72.0 phút cho 24 ảnh).
  *Thiên lệch:* Tốc độ duyệt nhanh hơn gấp đôi, tuy nhiên một phần do người duyệt đã quen với đặc thù đường phố qua 4 ảnh gold ban đầu.
- **Kết luận:** **CÓ, MÁY RẤT ĐÁNG DÙNG**, đặc biệt cho việc sinh khung sơ bộ cho người (`person`) và xe máy (`motorbike`). Nó giúp tiết kiệm hơn 50% thời gian vẽ box thủ công. Tuy nhiên, AI không thể tự vận hành một mình mà bắt buộc phải có quy trình QA/QC chặt chẽ để bù đắp các lỗi nhận diện lớp địa phương và lỗi sót ở hậu cảnh.
- **Ba quy tắc giảm thiên kiến của bạn:**
  1. *Quy tắc Blind Scan 10s (Kiểm tra bằng việc làm):* Trước khi mở layer nhãn gợi ý của AI, bắt buộc tắt hiển thị annotation và dùng mắt quét độc lập 10 giây qua các khu vực rìa ảnh, vỉa hè và khoảng trống giữa các xe lớn để phát hiện người/vật thể bị sót.
  2. *Quy tắc Zoom 200% ngẫu nhiên (Kiểm tra bằng số):* Chọn ngẫu nhiên tối thiểu 30% số box máy trong mỗi ảnh và phóng to 200% để kiểm tra cạnh biên có chạm đúng rìa ngoài cùng của vật thể hay không (IoU >= 0.9).
  3. *Quy tắc Lọc lớp hiếm (Kiểm tra bằng số):* Luôn dùng bộ lọc danh mục trong CVAT để kiểm tra riêng 100% các đối tượng thuộc lớp `three_wheeler`, `bus`, `truck` và `bicycle` nhằm ngăn chặn triệt để lỗi phân loại nhầm đặc trưng của model COCO.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Chỉ ôm trọn thân xe và mui xe tuk-tuk; người lái đã được loại trừ qua điểm click âm | Mask bị tách thành 6 đa giác nhỏ do khung kim loại và kính chắn gió tạo lỗ rỗng | `masks/tuktuk_img013.jpg` |
| img_004.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Gồm toàn bộ khung xe xích lô và giỏ hàng cố định; loại trừ người đạp | Điểm giao giữa thanh nan hoa và mặt đường đôi khi bị tràn biên nhẹ | `masks/cyclo_img004.jpg` |
| img_017.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Ôm khít thân xe ba bánh; tách biệt hoàn toàn người lái phía trên | Cạnh viền bóng đổ phía dưới gầm xe bị ăn lẹm một phần nhỏ | `masks/tuktuk_img017.jpg` |

- **Đối chiếu guideline mục 4.2:**
  - Quy tắc 4.2.2 (bỏ qua lỗ thủng < 5% diện tích) và 4.2.3 (bóng đổ không thuộc vật, kính xe thuộc vật) rất hữu ích khi tinh chỉnh điểm click âm cho SAM.
  - Tuy nhiên, guideline còn thiếu hướng dẫn cụ thể về việc xử lý phần dây chằng hàng hoặc các chi tiết cơ khí siêu mảnh (xích líp xích lô), khiến SAM phân vân giữa việc gộp hay tách rời đa giác.
- **So sánh với Interactor SAM trong CVAT:**
  Chạy offline bằng script `sam2_points.py` cho phép kiểm soát toạ độ điểm cực kỳ chính xác và ghi nhận nhật ký pixel minh bạch. Trong khi đó, Interactor trực tiếp trong CVAT có lợi thế phản hồi trực quan ngay lập tức sau mỗi cú click chuột, giúp người dùng dễ dàng phát hiện vùng tràn mask để đặt điểm negative kịp thời.

## 6. Calibration (Part 6)

- **Số quyết định:** 4 quyết định được ghi nhận trong `submission/qc/calibration_decisions.csv` (CAL-01 đến CAL-04).
- **Quyết định quan trọng nhất (`CAL-01`):** Loại bỏ hoàn toàn việc dùng YOLO26 COCO closed-set để gán xe ba bánh; quy định bắt buộc dùng YOLOE hoặc gán tay `three_wheeler` cho tuk-tuk, xích lô, ba gác và tách riêng box `person` cho người ngồi trên xe nếu thấy >= 20%.
- **Quy tắc mới trong `guideline_v1.1_calibrated.md`:**
  - Quy tắc phân loại xe hai bánh chạy điện dựa trên bàn đạp trợ lực (Mục 2.1).
  - Quy tắc diện tích che khuất > 50% cho cụm xe máy đông đúc chuyển sang dùng `ignore_region` (Mục 4.1.6).
  - Quy tắc xử lý hàng hoá cồng kềnh nhô ra ngoài thân xe > 20% chiều dài xe (Mục 4.1.5).
  - Bổ sung quy trình rà soát hậu cảnh độc lập chống thiên kiến tự động hóa.
- **Phân biệt nguyên nhân gốc:**
  - *Guideline mơ hồ:* Vấn đề xe đạp điện vs xe máy điện và tiêu chí cụm xe máy ùn tắc chưa từng được định nghĩa cụ thể trong v1.0 -> Cần sửa chữ thành v1.1.
  - *Lỗi do máy / người duyệt:* Việc YOLO26 gọi tuk-tuk là car là do giới hạn kiến trúc tập dữ liệu nguồn; việc người duyệt lọt bẫy missing tại `img_017.jpg` là do thao tác chủ quan, không phải lỗi guideline.

## 7. QC và quyết định (Part 6)

- **Mẫu kiểm tra QC:** Mẫu gồm **8 ảnh** (3 ảnh có ca khó `can_review` + 5 ảnh ngẫu nhiên với seed 42), tổng cộng **165 đối tượng**. Người kiểm tra: Học viên Nguyễn Thế Anh.
  *Hạn chế:* Người kiểm tra QC trùng với người review đợt 1 nên có thể tồn tại điểm mù nhận thức lặp lại.
- **Tỉ lệ lỗi QC:**
  - Lỗi **theo đối tượng:** **1/165 = 0.6%** (mẫu số: 165 đối tượng trong mẫu kiểm tra).
  - Lỗi **theo ảnh (ảnh có lỗi):** **1/8 = 12.5%** (mẫu số: 8 ảnh trong mẫu kiểm tra).
- **Top lỗi quan sát được:**
  - Lỗi duy nhất phát hiện trong mẫu QC là `bo_sot` (bỏ sót 1 người đi bộ hậu cảnh tại `img_017.jpg`).
- **Giới hạn mẫu nhỏ:**
  Theo tính toán khoảng tin cậy Wilson 95%, với 1 ảnh lỗi trên 8 ảnh mẫu, cận trên tỉ lệ ảnh lỗi thật trong tổng thể có thể lên tới **47%**. Do đó, với kích thước mẫu nhỏ 8 ảnh, ngưỡng khắt khe 2% theo lý thuyết là **không thể chứng minh được bằng thống kê**.
- **Khả năng kiểm tra của script:** Script tự động chỉ so khớp được các trường thuộc tính và IoU hình học, không thể thay thế mắt người trong việc nhận định tính ôm sát thực tế hay phát hiện các vật thể bị bỏ sót hoàn toàn ngoài vùng dự đoán.

**QUYẾT ĐỊNH: REWORK**, ngày 2026-11-19.
- *Lý do theo cây quyết định:*
  1. Guideline đã được cập nhật v1.1 (không có `guideline_gap` chưa giải quyết), dữ liệu ảnh nguyên vẹn (không có `data_problem`).
  2. Bắt bẫy đạt 83.3% (>= 80%), lỗi không mang tính hệ thống.
  3. Tuy nhiên, tỉ lệ ảnh có lỗi trong đợt kiểm tra QC là **1/8 = 12.5%**, vượt quá ngưỡng phát hành an toàn **2%** (`error_rate_img_max = 0.02`).
- *Việc cần làm tiếp (Rework plan):*
  Thực hiện thêm một lượt rà soát tập trung (focus pass) đối với lớp `person` tại các khu vực hậu cảnh/vỉa hè của 24 ảnh batch theo đúng quy trình review siết chặt của guideline v1.1 trước khi ký duyệt Release chính thức.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Sử dụng trợ lý AI để hỗ trợ viết code thực nghiệm, chạy các script pipeline và định dạng báo cáo. Mọi kết quả số liệu, ma trận đo và phân tích chất lượng đều được kiểm chứng độc lập trên dữ liệu thực tế của repo.
- **Tình trạng nhiễm nhãn của tập Gold:** Tập 4 ảnh gold (`img_001.jpg`, `img_012.jpg`, `img_020.jpg`, `img_036.jpg`) được gán tay hoàn toàn dựa trên Guideline v1.0 trước khi chạy bất kỳ pipeline pre-label nào. Không bị nhiễm nhãn hay tham chiếu kết quả máy từ trước.
- **Môi trường thực thi:** Hệ điều hành Windows, Python 3.12, xử lý trên CPU Intel Core. Pipeline thực thi trơn tru với các phiên bản trọng số `yolo26m.pt`, `yoloe-26s-seg.pt` và `sam2.1_b.pt`.
- **Sử dụng cache:** Toàn bộ các bước Tour model (Part 2) và SAM points (Part 5) đều được chạy trực tiếp trên máy. Bộ dữ liệu dự phòng `data/cache/` chỉ dùng làm nguồn tham chiếu kiểm chứng đối sánh.
- **Lỗi / vướng mắc đã giải quyết:** Đã xử lý vấn đề tràn toạ độ mép ảnh trên Windows và tự động loại bỏ các box dưới 20px để tuân thủ triệt để Guideline 4.1.6.
- **Điều làm khác lần sau:** Luôn chủ động thu thập tập gold đa dạng hơn (ít nhất 10–15 ảnh) và phân bổ đều các lớp hiếm (như bus, truck) để quá trình threshold sweep đạt ý nghĩa thống kê cao hơn.
