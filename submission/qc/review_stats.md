# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 611; đối tượng sau duyệt: 593; ngưỡng accepted: IoU >= 0.9
- Đã **loại 28 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 588 | 588/611 = 96.2% | 611 box máy |
| edited (sửa) | 4 | 4/611 = 0.7% | 611 box máy |
| deleted (xoá) | 19 | 19/611 = 3.1% | 611 box máy |
| added (vẽ thêm) | 1 | 1/593 = 0.2% | 593 đối tượng sau duyệt |

Trong số deleted + added, **1 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 0.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 364 | 2 | 9 | 0 |
| bicycle | 11 | 0 | 1 | 0 |
| motorbike | 141 | 1 | 8 | 1 |
| car | 37 | 1 | 1 | 0 |
| bus | 3 | 0 | 0 | 0 |
| truck | 7 | 0 | 0 | 0 |
| three_wheeler | 25 | 0 | 0 | 0 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 69 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 63 | 16 | 6 | 79.7% (KTC 95%: 70–87%) | 91.3% (KTC 95%: 82–96%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - extra: 3/3
  - missing: 2/3
  - loose: 2/3
  - lọt: img_017.jpg (missing), img_028.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
