# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603313 - Nguyễn Hữu Lộc

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 3 | Hàng nhô ra: box / mask xe có gồm hàng không? | Box **và** mask xe gồm hàng nằm trong thùng hoặc chất trực tiếp trên thùng. Chỉ phần hàng nhô ra ngoài hình chiếu thùng xe quá ~20% chiều dài xe mới bị loại (giữ 4.1.5). | img_008 (ba gác chở chuối) | CAL-03 |
| 7 | Có mức `conf` tối thiểu để chấp nhận box máy không cần sửa? | **Không.** `conf` chỉ dùng để sắp thứ tự duyệt; box điểm cao vẫn sai lớp (bán tải → `truck 0.73`). Mọi box phải được nhìn trên ảnh. | img_002, img_011 | CAL-05 |

Chưa chốt ở v1.1: câu 1 (xe đạp điện), 2 (khối xe máy dày gần camera), 4 (ngưỡng 20% người trong xe có mui), 5 (kính xe trong mask), 6 (box chồng ảnh đêm) — chưa gặp đủ ca trong gold / Part 2 để quyết; giữ nguyên v1.0 và tick `can_review` nếu gặp.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_002 | Tuk-tuk có mui kín, nhìn ngang giống ô tô nhỏ | YOLO26m: img_013 `car 0.42` (box phủ nửa xe), img_002 không có box; YOLOE bắt đúng img_002 (0.93) | Luôn `three_wheeler`; box `car`/`truck` của máy chồng lên xe 3 bánh phải **đổi lớp**, không chỉ chỉnh box | Mục 2, dòng `three_wheeler`: "xe có mui kín vẫn là `three_wheeler` nếu thấy 3 bánh hoặc kết cấu 1 bánh trước + 2 bánh sau" (CAL-01) |
| img_002, img_011 | Xe bán tải / xe mép ảnh | YOLO26m: `truck 0.73` (img_002), `truck 0.77` (img_011); YOLOE: bán tải img_002 là `three_wheeler 0.37` | `car` — lỗi máy, v1.0 đã rõ; chỉ bổ sung ảnh ví dụ | Mục 2, dòng `car`: thêm ảnh ví dụ img_002 (CAL-02) |
| img_013, img_010 | Mask xe ba bánh có người lái / người đạp ngồi trong khung | SAM 2.1 b (click): tô cả tài xế img_013 và áo người đạp img_010 dù có click âm; img_013 tô cả khoang trống + người phía sau | Mask `three_wheeler` **không** gồm người; người có mask `person` riêng; lỗ lớn trong khoang (thấy người / cảnh phía sau) bỏ khỏi mask | Mục 4.2, thêm **4.2.5**: "người lái / khách ngồi trong khung là vật phía trước → tách khỏi mask xe" (CAL-04) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 3 (hàng trên thùng thuộc box/mask xe) và câu 7 (không auto-accept theo `conf`); làm rõ tuk-tuk mui kín là `three_wheeler`; thêm 4.2.5 tách người khỏi mask xe ba bánh; thêm ảnh ví dụ bán tải = `car` | CAL-01 … CAL-05 |
