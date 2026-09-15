# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Nguyễn Xuân Quang`
Ngày: `2026-09-15`

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `15` phút |
| Thời gian gán `clip_01` | `45` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `2` |

Tình huống khó: 
1. ID 5 drift tại MOT 82–96;
2. ID 6 có bbox sớm MOT 79–100;
3. ID 8 sai boundary ở MOT 133–135 và 169–171. Các tình huống này cần kiểm midpoint hoặc Outside tại endpoint.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 | `682f868b937edff0c5d32277f15ea2162ed5383e251e2a50822d832f2b86f3ae` |
| Thời điểm khóa | `2026-09-15T08:57:14Z` |
| Row / frame / track | `620 / 190 / 8` |

| Bản | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Pre-gold | 0.795 | 0.779 | 0.814 | 0.873 | 0.956 | 0.908 | 0.858 | 50 | 3 | 0 |
| Sau rework | 0.795 | 0.779 | 0.814 | 0.873 | 0.956 | 0.908 | 0.858 | 50 | 3 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**


Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| -------- | ----- | -- | -------------- |
| Bbox thừa trước entry | MOT 79–100 / CVAT 78–99 | 6 | Chưa sửa annotation; cần kiểm frame đầu xe xác định được và đặt Outside đúng endpoint. |
| Bbox thừa/treo | MOT 51–53, 149–151 / CVAT 50–52, 148–150 | 4 | Chưa sửa annotation; cần kiểm entry/exit theo từng frame. |
| Interpolation drift | MOT 82–96 / CVAT 81–95 | 5 | Chưa sửa annotation; cần thêm keyframe quanh midpoint bị lệch. |

## 4. Kết quả model

Cấu hình từ `outputs/model_run_config.json`:

| Mục                                | Giá trị |
| ---------------------------------- | ------- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cpu` / `0.5.13` |
| weights / hai tracker              | `yolo26n.pt`; `bytetrack.yaml`; `botsort-reid.yaml` |
| conf / IoU / imgsz / classes       | `0.25` / `0.70` / `960` / `[2, 5, 7]` |
| device                             | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bạn vs gold | 0.795 | 0.779 | 0.814 | 0.873 | 0.956 | 0.908 | 0.858 | 50 | 3 | 0 |
| ByteTrack vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.859 | 91 | 26 | 2 |
| ReID vs bạn | 0.731 | 0.664 | 0.813 | 0.871 | 0.878 | 0.755 | 0.852 | 84 | 66 | 2 |

## 5. Phân tích

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**
1. MOTA 0.908 thấp hơn IDF1 0.956. IDSW = 0 cho thấy identity annotation tốt. Nếu MOTA cao nhưng IDF1 thấp, bbox có thể vẫn đúng nhưng association/ID sai; MOTA không phản ánh đầy đủ identity như IDF1.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**
2. ReID tăng IDF1 từ 0.875 lên 0.900 và AssA từ 0.776 lên 0.820; IDSW vẫn 2. ByteTrack switch gold ID 4 ở MOT 59 và gold ID 5 ở MOT 94. ReID còn switch gold ID 5 ở MOT 87 và gold ID 6 ở MOT 113. Đây là comparison hệ thống, không cô lập causal effect của ReID vì ByteTrack và BoT-SORT khác implementation.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**
3. ReID tăng DetA 0.649 → 0.711, giảm FN 54 → 26, nhưng tăng FP 88 → 91. Lỗi còn lại là cả detector (FP/FN) lẫn association (fragmentation, IDSW, ghost track).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**
4. ReID track 7 có ghost segment MOT 16–116 không khớp gold; annotation tay có IDSW = 0 và chỉ 3 FN, nên không sửa nhãn tay theo model ở đoạn này.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**
5. ReID khiến tôi xem lại ID 6; gold evaluation xác nhận nhãn tay có bbox sớm MOT 79–100. Model chỉ dùng để chọn frame cần soi vì bản thân nó vẫn có ghost/fragmentation/IDSW.

## 6. Nếu gán thêm 10 clip

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Tôi sẽ bắt buộc kiểm frame đầu/cuối của mọi track, kiểm midpoint đoạn interpolation dài, đặt keyframe ở đổi hướng/scale/occlusion, và làm peer review trước pre-gold lock.

## 7. Checklist artifact

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)






