# Thông tin học viên

- Họ và tên: Nguyễn Hữu Lộc
- MSHV: 2A202603313
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: nguyenloc2004
- Repo bài nộp (URL): https://github.com/nguyenloc2004/K5-L23-DAY08-2A202603313-NguyenHuuLoc-AIPrelabel
- Commit chốt bài (hash): <<>>
- Máy: Windows 11, GPU GTX 1650 4 GB nhưng chạy model trên CPU (torch 2.15.0+cpu, conda env lab-vin, Python 3.11)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 463 / 16 / 42 = 88.9% / 3.1% / 8.1% (mẫu số: 521 box máy); added: 25 = 5.0% (mẫu số: 504 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 60/74 = 81.1% / 60/76 = 78.9%
- Bắt bẫy: 6/12 = 50%
- Thời gian: gán tay 3.75 phút/ảnh (15 phút / 4 ảnh); duyệt có pre-label 2.5 phút/ảnh (60 phút / 24 ảnh)
- Lỗi QC: 40% theo ảnh (2/5), 2.0% theo đối tượng (2/98)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 2/5 = 40% vượt ngưỡng 2% và bắt bẫy 6/12 = 50% dưới ngưỡng 80%.
