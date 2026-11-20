# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 165 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 1/165 = 0.6% | 165 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 1/8 = 12.5% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 588/611 = 96.2% / 4/611 = 0.7% / 19/611 = 3.1% | 611 box máy |
| Máy: added | 1/593 = 0.2% | 593 đối tượng sau duyệt |
| Bắt lỗi bẫy | 10/12 = 83.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 1 | img_017.jpg |

## Giới hạn thống kê
- 1 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **47%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 12.5% > 2%

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
