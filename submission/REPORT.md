# Báo cáo Day 08 — 2A202603313 - Nguyễn Hữu Lộc

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / motorbike / car / bus / truck | YOLO26m | 0.3 (mặc định) | Sweep không tìm được ngưỡng ≤ 0.5 nào cho person, motorbike, car đạt recall 90%; bus và truck gần như không có trong gold. Ở r03 (conf 0.5) máy sót nhiều, ví dụ img_007 person còn 6/15, motorbike còn 5/11, nên em giữ ngưỡng thấp. |
| bicycle | YOLO26m | 0.2 | Sweep cho recall 6/6, precision 6/7 trên gold. Ở r03 img_019 chỉ còn 1/4 xe đạp. |
| three_wheeler | YOLOE-26s, prompt "tuk-tuk", "cyclo", "cargo tricycle" | 0.15 | YOLO26 không có lớp này: ở img_002 nó bỏ qua chiếc tuk-tuk, ở img_013 thì gọi là car (r02). YOLOE tìm được tuk-tuk img_002 (0.93) và xích lô img_032 (0.92) (r04). Sweep cho recall 7/7, precision 78%. |

- Lệnh merge em đã dùng (ra 28 ảnh, 610 box):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "bicycle=0.2,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
  Sau đó cài bẫy bằng `python scripts/inject_traps.py --pre work/prelabel/merged_review/pre.coco.json --student "2A202603313"`.
- Phiên bản SAM: em chọn SAM 2.1 base (`sam2.1_b.pt`). Máy em chạy CPU nên không dùng được SAM 3; SAM 2.1 b vẫn chạy được, khoảng 7.9 giây/ảnh (r05), và mask bám biên tốt hơn bản mobile.
- Lỗi không sửa được bằng ngưỡng: YOLO26 gọi sai lớp xe ba bánh (img_013 tuk-tuk thành car 0.42, img_002 tuk-tuk không có box) và gọi xe bán tải img_002 là truck 0.73. Em tăng ngưỡng từ 0.25 lên 0.5 (r02, r03) thì các box này vẫn y nguyên. Cách sửa là dùng thêm YOLOE cho three_wheeler, khi merge thì ưu tiên three_wheeler nếu chồng với car/truck, phần còn lại em sửa tay lúc review.
- Box điểm cao mà sai: xe bán tải img_002 bị gọi truck 0.73 (đúng ra là car), xe ở mép trái img_011 bị gọi truck 0.77. Box điểm thấp mà đúng: xích lô bên trái img_032 (0.48), nhiều người trong đám đông img_002 chỉ được 0.27–0.36 nhưng đều là người thật.
- Giới hạn: em chọn ngưỡng trên chính 4 ảnh gold (80 đối tượng, trong đó 76 không tính ignore_region) rồi lại chấm máy trên đúng 4 ảnh đó, nên precision/recall trên gold sẽ lạc quan hơn thực tế. Các lớp ít vật (bicycle 6, three_wheeler 7, truck 1, bus 0) chỉ cần sai 1 vật là recall đã đổi khoảng 15%, nên ngưỡng 0.2 và 0.15 chỉ là ước lượng thô.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 463 | 88.9% | 521 box máy |
| edited | 16 | 3.1% | 521 box máy |
| deleted | 42 | 8.1% | 521 box máy |
| added | 25 | 5.0% | 504 đối tượng sau duyệt |

*(Đã loại 23 đối tượng quanh bẫy khỏi bảng; 8 ca deleted + added thực ra là đổi lớp, nên số vật máy sót thật khoảng 25 − 8 = 17. Nguồn: `qc/review_stats.md`.)*

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? three_wheeler: em phải xoá 24 box và chỉ giữ nguyên 9. YOLOE hay tạo nhiều box trùng cho cùng một xe (img_013, img_023) và bắt nhầm xe khác thành xe ba bánh, ví dụ xe bán tải ở img_002 (model_log r04, r07).
- So với gold: precision của máy là 60/74 = 81.1% (KTC 95%: 71–88%), recall là 60/76 = 78.9% (KTC 95%: 69–87%). Mẫu số recall là 76 đối tượng gold trong 4 ảnh, mẫu số precision là 74 box máy (60 đúng, 14 sai).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: em bắt được 6/12 = 50% (KTC 95%: 25–75%), thấp hơn ngưỡng 80%. Các bẫy lọt: cả 3/3 bẫy loose (img_021, img_022, img_024), 1/3 missing (img_018), 1/3 extra (img_010), 1/3 wrong_class (img_031). Vì sao: cả 3 bẫy box lỏng đều lọt, nghĩa là em gần như bỏ qua bước 4 (kiểm box lỏng) của checklist.
- Accepted cao mà bắt bẫy thấp nói gì về cách duyệt: accepted lên tới 88.9% nhưng em chỉ bắt được 6/12 bẫy, tức là em bấm chấp nhận nhiều hơn là thật sự kiểm từng box.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 15 phút / 4 ảnh = 3.75 phút/ảnh, duyệt có pre-label 60 phút / 24 ảnh = 2.5 phút/ảnh, nhanh hơn khoảng 33%. Thiên lệch: em gán gold trước nên đã quen kiểu cảnh này, và gold chỉ có 4 ảnh nên con số thời gian gán tay chưa chắc chắn.
- Kết luận: có, vì duyệt nhanh hơn khoảng 33% mỗi ảnh. Nhưng máy chỉ đáng dùng nếu em duyệt kỹ hơn: recall của máy trên gold chỉ 78.9% và máy vẫn sót khoảng 17 vật trong batch.
- Ba quy tắc giảm thiên kiến của em (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Kiểm cả 4 cạnh của từng box trước khi Accept. Kiểm tra: vòng duyệt sau phải bắt được ít nhất 2/3 bẫy loose.
  2. Ẩn nhãn máy 15 giây đầu mỗi ảnh để tự đếm vật trước. Kiểm tra: ghi số vật tự đếm cho từng ảnh rồi so với số box sau duyệt.
  3. Nghỉ 3–5 phút sau mỗi 10 ảnh. Kiểm tra: ghi các lần nghỉ vào `time_log.csv` (dòng `break`).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk có tài xế) | SAM 2.1 b | 3 (2 dương ở thân xe và mui, 1 âm ở tài xế) | Có lẫn người. Em đã click âm vào tài xế nhưng mask vẫn tô gần hết người tài xế, cả người ngồi phía sau nhìn qua khung xe cũng bị tô. | Mask chiếm 44% ảnh, lấy luôn khoảng trống trong khoang xe. Sai mục 4.2.2 (lỗ lớn phải bỏ) và 4.2.4 (người là vật riêng). | `masks/tuktuk_img013.jpg` |
| img_008 (ba gác chở chuối) | SAM 2.1 b | 3 (2 dương ở thùng xe và chuối trong thùng, 1 âm ở mặt người bán) | Có một phần hàng: chuối trong thùng được tô, chuối chất cao phía trên thì không. Người bán không bị lẫn. | Mask cắt ngang đống chuối, không rõ phần nào tính là hàng của xe. | `masks/cargo_tricycle_img008.jpg` |
| img_010 (xích lô nhìn từ sau, người đạp che) | SAM 2.1 b | 4 (3 dương ở bánh sau, mui, bánh trái; 1 âm ở xích lô bên cạnh) | Có lẫn người đạp: áo khoác bị tô gần hết, quần thì không. | Mask lấn sang mui và càng của xích lô thứ hai dù đã click âm. Sai mục 4.2.4 (hai vật chạm nhau). | `masks/cyclo_img010.jpg` |

- Đối chiếu guideline 4.2: mục 4.2.4 giúp em biết phải đặt click âm, nhưng một click âm không đủ khi người ngồi ngay trong khung xe (img_013, img_010). Guideline v1.0 còn thiếu hai chỗ: mask xe ba bánh có trừ phần người lái không (v1.0 mới nói cho box ở 4.1.4), và hàng chất trong thùng ba gác (img_008) tính vào mask đến đâu (câu hỏi mở số 3). Em chốt hai chỗ này ở v1.1 (CAL-03, CAL-04).
- Interactor SAM trong CVAT: em chưa thử vì CVAT chạy Docker trên máy CPU, chưa deploy function SAM.

## 6. Calibration (Part 6)

- Số quyết định: 5 (CAL-01 đến CAL-05 trong `calibration_decisions.csv`, gồm 2 `loi_may` và 3 `cau_hoi_mo`). Quan trọng nhất là CAL-01 (tuk-tuk và các xe ba bánh luôn gán `three_wheeler`), vì three_wheeler là lớp máy làm tệ nhất (24 deleted so với 9 accepted trong `review_stats.md`), và đây là lỗi không sửa được bằng đổi ngưỡng.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: tuk-tuk mui kín vẫn là `three_wheeler` (CAL-01); hàng nằm trong hoặc trên thùng tính vào box và mask xe, chỉ bỏ phần nhô ra quá 20% (CAL-03); thêm mục 4.2.5 tách người lái / khách ra khỏi mask xe ba bánh (CAL-04); không tự chấp nhận box theo `conf` (CAL-05); thêm ảnh ví dụ xe bán tải là `car` (CAL-02).
- Nguyên nhân gốc: chủ yếu là do máy. YOLO26 không có lớp xe ba bánh. Với CAL-02, v1.0 đã ghi rõ xe bán tải là car, nên đây là máy sai chứ không phải guideline mơ hồ. CAL-03/04/05 là do guideline mơ hồ, còn lỗi box lỏng là do em bỏ bước 4 của checklist.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ảnh ngẫu nhiên, seed 42: img_003, img_006, img_014, img_031, img_035; 98 đối tượng). Người kiểm là em, nên có hạn chế: em vừa là người duyệt vừa là người kiểm.
- Lỗi theo đối tượng: 2/98 = 2.0%. Lỗi theo ảnh: 2/5 = 40.0% (ngưỡng Release ≤ 2%).
- Top lỗi: sai vị trí/kích thước 1 lỗi (img_014, object 141: box person kéo quá rộng); sai lớp 1 lỗi (img_031, object 441: đúng phải là person).
- Giới hạn mẫu nhỏ: có 2/5 ảnh lỗi, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật lên tới 77% (`qc_report_auto.md`). Với mẫu chỉ 5 ảnh thì không thể chứng minh lô đạt ngưỡng ≤ 2%.
- Script kiểm được / không kiểm được: script kiểm được những gì đếm được bằng số, gồm tỉ lệ lỗi theo ảnh và theo đối tượng từ phiếu `qc_review.csv`, tỉ lệ bắt bẫy 6/12, accepted/edited/deleted/added so với box máy và precision/recall so với gold. Script không kiểm được lỗi mà em không ghi vào phiếu, vật mà cả máy và em cùng bỏ sót, box có thật sự ôm sát hay không, và em có thật sự làm đúng checklist hay không.

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): không cần Hold vì các câu hỏi mở chính đã chốt ở v1.1; nhưng ảnh lỗi 2/5 = 40% vượt ngưỡng 2%, và bắt bẫy chỉ 6/12 = 50%, thấp hơn ngưỡng 80%, nên lô này phải Rework.

Việc cần làm tiếp:
1. Duyệt lại cả 24 ảnh, tập trung kiểm box lỏng (bước 4) vì cả 3 bẫy loose đều lọt.
2. Sửa 2 lỗi QC: box person 141 ở img_014 và lớp của object 441 ở img_031, export ra `submission/labels/qc_fixed.json`.
3. Chạy lại `build_release.py` rồi `qc_sample.py --force`, kiểm một mẫu mới và ghi kết quả vòng 2 vào báo cáo.
4. Áp dụng 3 quy tắc giảm thiên kiến ở mục 4 khi duyệt lại.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude Code (trợ lý lập trình trong VS Code), dùng **sau khi** đã xong gold và Part 2–3: (1) chạy lại `merge_prelabels.py` để xác nhận lệnh merge ghi ở mục 1 tái tạo đúng 610 box; (2) chạy `sam2_points.py` cho 3 mask Part 5 (toạ độ click chọn từ ảnh gốc); (3) soạn nháp nhận xét `model_log.csv`, `calibration_decisions.csv`; gợi ý ý chính cho mục 2–4, 6. AI **không** mở `traps_key.json`, không tạo `batch_reviewed.json`, không điền `time_log.csv`.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không.
- Dùng cache `data/cache/` thay vì tự chạy? Không, tự chạy `y26_all` (r06) và `yoloe_all` (r07) trên CPU.
- Lỗi / bug script đã gặp (nếu có): không sửa script nào.
