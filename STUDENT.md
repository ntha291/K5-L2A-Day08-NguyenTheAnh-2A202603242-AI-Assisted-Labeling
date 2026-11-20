# Thông tin học viên

- Họ và tên: Nguyễn Thế Anh
- MSHV: 2A202603242
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: GlennQuagmiree
- Repo bài nộp (URL): https://github.com/GlennQuagmiree/K5-L23-DAY08-2A202603242-NguyenTheAnh-AIPrelabel
- Commit chốt bài (hash): e352f81
- Máy: Windows, CPU (Intel Core i5/i7)
- CVAT (Docker): Không cài Docker / Quy trình offline COCO 1.0 chuẩn CVAT

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 588 / 4 / 19 (mẫu số: 611 box máy); added: 1 (mẫu số: 593 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 79.7% / 91.3% (mẫu số: 69 đối tượng gold)
- Bắt bẫy: 10/12 = 83.3%
- Thời gian: gán tay 6.1 phút/ảnh; duyệt có pre-label 3.0 phút/ảnh
- Lỗi QC: 12.5% theo ảnh (1/8), 0.6% theo đối tượng (1/165)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ ảnh có lỗi 1/8 = 12.5% vượt ngưỡng cho phép 2%, cần rà soát lại người đi bộ ở hậu cảnh trước khi Release.
