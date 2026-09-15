# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Hữu Tài / SOLO`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | 45 phút |
| Thời gian gán `clip_01` | 75 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | 8 |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che khuất một phần trong thời gian ngắn: tôi giữ nguyên ID khi vẫn có thể xác định đó là cùng một xe và thời gian bị che không vượt quá 5 frame.

2. Hai xe cắt nhau hoặc chồng lấn bounding box: tôi giữ hai ID riêng biệt và dựa vào vị trí, hướng di chuyển và quỹ đạo trước/sau khi cắt nhau để tránh đổi ID.

3. Xe có tốc độ cao và ở nhiều góc quay khác nhau tạo ra viền mờ hay thậm chí hình dáng lạ xung quanh: dựa vào nhận thức logic để boxing hình dáng thực tế.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Kiểm tra toàn bộ các ID và sự liên tục của từng track, đặc biệt chú ý các trường hợp ID bị đổi hoặc nhảy.
- Lượt 2: Kiểm tra frame đầu và frame cuối của từng track để phát hiện xe bị bỏ sót, xuất hiện/biến mất bất thường hoặc bbox không đúng.
- Lượt 3: Phát hiện lỗi theo dõi ở giữa clip, đặc biệt khi xe che khuất hoặc hai xe đi gần/cắt nhau.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | 5B94C6535B5FE3B857DD2172401A7D8ABBD0034DC4AFD68A94D30B5D5A75DC52 |
| Thời điểm khóa | 11:20 |
| Số row / frame / track trước khi mở reference | 611 / 190 / 8 |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.820 | 0.804 | 0.837 | 0.882 | 0.965 | 0.927 | 0.870 | 40 | 2 | 0 |
| Sau rework | 0.835 | 0.821 | 0.850 | 0.880 | 0.963 | 0.925 | 0.872 | 28 | 15 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): có

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox xuất hiện quá sớm | 80–100 | 6 | Xóa bbox ID 6 trước thời điểm track 6 thực sự xuất hiện |
| Bbox còn tồn tại sau khi xe rời frame | 149–151 | 4 | Xóa bbox ID 4 sau khi xe đã rời khỏi khung hình |
| Bbox còn tồn tại sau khi xe rời frame | 169–171 | 8 | Xóa bbox ID 8 sau khi xe đã rời khỏi khung hình |
| Bbox lệch so với teaching reference | 55 | 4 | Kiểm tra và điều chỉnh bbox theo đúng phần phương tiện quan sát được, không suy đoán phần bị che khuất |
| Bbox lệch so với teaching reference | 80–81 | 5 | Kiểm tra và điều chỉnh bbox theo phần phương tiện nhìn thấy được trong từng frame |
| Bbox lệch so với teaching reference | 101–105 | 6 | Kiểm tra và điều chỉnh bbox theo phần phương tiện quan sát được, đảm bảo bbox bám sát vật thể qua các frame liên tiếp |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `Python 3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / ByteTrack control, BoT-SORT + ReID treatment` |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / [2, 5, 7]` |
| device | `CPU` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.820 | 0.804 | 0.837 | 0.882 | 0.965 | 0.927 | 0.870 | 40 | 2 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.805 | 0.744 | 0.873 | 0.930 | 0.879 | 0.754 | 0.928 | 88 | 61 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi (0.927) thấp hơn IDF1 (0.965). Điều này cho thấy annotation có khả năng phát hiện/duy trì đối tượng khá tốt và đặc biệt giữ nhất quán ID rất tốt. IDSW = 0 nên không có lỗi đổi ID khi so với gold. MOTA không phạt nặng lỗi ID vì IDSW chỉ xuất hiện như một thành phần cộng thêm trong tử số của công thức MOTA, cùng với FP và FN; do đó nếu FP/FN không lớn thì một số lỗi ID vẫn có thể khiến MOTA giảm tương đối ít. IDF1 tập trung trực tiếp hơn vào chất lượng association và độ nhất quán identity.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

ByteTrack control có IDF1 = 0.875, AssA = 0.776 và IDSW = 2; trong khi BoT-SORT + ReID có IDF1 = 0.900, AssA = 0.820 và IDSW = 2. Như vậy ReID treatment cải thiện IDF1 và AssA, nhưng không làm thay đổi số IDSW tổng thể. Một sequence đáng chú ý là quanh frame 87 của BoT-SORT + ReID, gold track 5 bị chuyển từ ID 17 sang ID 18; do đó vẫn xuất hiện ID switch. Với ByteTrack, ID switch của track 5 xảy ra ở frame 94 (ID 23 -> 32). Kết quả cho thấy ReID giúp association tốt hơn tổng thể và giảm lỗi đứt/mất liên kết trong clip này, nhưng không loại bỏ hoàn toàn ID switch. Đây không phải causal effect riêng của ReID vì ByteTrack và BoT-SORT là hai tracker implementation khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA của ByteTrack là 0.649 với FP = 88 và FN = 54, trong khi BoT-SORT + ReID đạt DetA = 0.711 với FP = 91 và FN = 26. Như vậy DetA tăng 0.062; FP tăng nhẹ 3 nhưng FN giảm mạnh 28. Điều này cho thấy BoT-SORT + ReID có khả năng duy trì/phát hiện các đối tượng tốt hơn, đặc biệt giảm số đối tượng bị bỏ sót. Vì IDF1 và AssA cũng tương đối cao, lỗi còn lại không chỉ nằm ở association; cả detector/detection coverage và association đều còn lỗi. Tuy nhiên, số FN khá lớn của các tracker model cho thấy detection/track coverage vẫn là một nguồn lỗi đáng kể.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 87, track 5: annotation của tôi vẫn giữ ID 5 nhất quán, trong khi BoT-SORT + ReID chuyển ID từ 17 sang 18. Do đó tại frame này model bị ID switch còn annotation của tôi không bị switch. Kết quả evaluation cũng xác nhận annotation của tôi có IDSW = 0 khi so với gold, trong khi ReID có IDSW = 2 khi so với gold.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Frame 115, track 6: BoT-SORT + ReID và annotation của tôi có khác biệt về bbox, với IoU =  0.51. Kết quả này khiến tôi kiểm tra lại bbox của track 6 trong vùng frame 115–117. Tuy nhiên, vì annotation của tôi được đặt theo phần phương tiện thực sự quan sát được và không suy đoán phần bị che khuất, IoU thấp với model không đủ để kết luận annotation của tôi sai. Vì vậy tôi giữ nguyên bbox nếu kiểm tra trực quan cho thấy nó phù hợp với phần visible của phương tiện.

## 6. Nếu phải gán thêm 10 clip nữa

Tôi sẽ bổ sung vào `GUIDELINE_MINI.md` quy tắc rõ hơn về thời điểm bắt đầu/kết thúc track và cách xử lý vật thể bị che khuất. Cụ thể, cần quy định rõ: không tạo bbox trước khi phương tiện đủ rõ để xác định là vehicle; khi phương tiện rời khỏi khung hình phải kết thúc track ngay; với phương tiện bị che khuất thì bbox chỉ bao quanh phần thực sự quan sát được, không suy đoán phần bị che khuất. Đồng thời, tôi sẽ bổ sung hướng dẫn cho các trường hợp hai phương tiện giao nhau/chồng lấn để tránh ID switch.

Về quy trình làm việc, tôi sẽ chia việc kiểm tra thành ba lượt: lượt đầu kiểm tra tính liên tục của ID và thời điểm bắt đầu/kết thúc track; lượt hai kiểm tra bbox ở các frame có che khuất, giao nhau hoặc phương tiện nhỏ/khó nhìn; lượt ba kiểm tra lại các đoạn chuyển động liên tiếp để phát hiện bbox bị lệch hoặc trôi. Tôi cũng sẽ khóa bản pre-gold trước khi xem reference và ghi lại các trường hợp mơ hồ thay vì sửa bbox theo trực giác. Sau khi hoàn thành, tôi sẽ chạy evaluation để kiểm tra lại HOTA, IDF1, MOTA và các lỗi FP/FN/IDSW.





## Thí nghiệm độ nhạy với `appearance_thresh`

Để đánh giá ảnh hưởng của ngưỡng tương đồng appearance trong BoT-SORT + ReID, tôi thực hiện thí nghiệm với ba giá trị `appearance_thresh = 0.70, 0.80, 0.90`, trong khi giữ nguyên các tham số khác.

| Cấu hình | HOTA | DetA | AssA | IDF1 | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| ReID appearance = 0.70 | 0.763 | 0.711 | 0.820 | 0.900 | 91 | 26 | 2 |
| ReID appearance = 0.80 | 0.763 | 0.711 | 0.820 | 0.900 | 91 | 26 | 2 |
| ReID appearance = 0.90 | 0.763 | 0.710 | 0.820 | 0.899 | 91 | 27 | 2 |

### Nhận xét

Kết quả cho thấy việc thay đổi `appearance_thresh` trong khoảng từ 0.70 đến 0.90 không tạo ra khác biệt đáng kể trên `clip_01`. Hai cấu hình 0.70 và 0.80 cho kết quả hoàn toàn giống nhau trên tất cả các chỉ số được đánh giá. Khi tăng ngưỡng lên 0.90, hiệu năng giảm rất nhẹ: DetA giảm từ 0.711 xuống 0.710, IDF1 giảm từ 0.900 xuống 0.899 và FN tăng từ 26 lên 27. Trong khi đó, HOTA, AssA, FP và IDSW không thay đổi.

Đặc biệt, số ID switch vẫn bằng 2 ở cả ba cấu hình. Do đó, việc thay đổi `appearance_thresh` trong phạm vi thí nghiệm này không giải quyết được các trường hợp ID switch của tracker.

Tuy nhiên, kết quả này chỉ cho thấy `appearance_thresh` không có ảnh hưởng đáng kể đối với `clip_01` và cấu hình thí nghiệm hiện tại; không thể kết luận rằng tham số này luôn không quan trọng đối với các video hoặc bối cảnh khác.







## 7. Tệp đã nộp

- [ ] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
