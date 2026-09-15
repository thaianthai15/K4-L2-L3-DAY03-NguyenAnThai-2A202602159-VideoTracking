# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Nguyễn An Thái
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `15` phút |
| Thời gian gán `clip_01` | `60` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `Khoảng 16 annotation thủ công/track` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Ô tô xuất hiện một phần hoặc bị che khuất bởi hàng rào, xe khác hoặc nằm sát mép ảnh. Tôi chỉ vẽ bounding box bao quanh phần nhìn thấy của xe, đồng thời điều chỉnh box ở các keyframe tiếp theo khi xe xuất hiện rõ hơn.

2. Một xe xuất hiện muộn và dễ bị tạo thành track/ID mới. Tôi tua lại các frame trước đó, kiểm tra chuyển động và nối xe về cùng một track nếu xác định đó là cùng một đối tượng.

3. Khi xe di chuyển nhanh, bounding box nội suy giữa các frame có thể lệch khỏi xe. Tôi kiểm tra các frame đầu, giữa và cuối, sau đó thêm keyframe hoặc chỉnh lại bounding box tại những frame bị lệch.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Kiểm tra ID của từng xe, phát hiện trường hợp một xe bị chia thành nhiều ID hoặc ID bị đổi giữa chừng.
- Lượt 2: Kiểm tra frame bắt đầu và frame kết thúc của từng track, phát hiện bbox xuất hiện quá sớm hoặc tồn tại quá lâu.
- Lượt 3: Kiểm tra các frame ở giữa track để phát hiện bbox lệch khỏi xe, nhảy sang xe khác hoặc không bám sát object.

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `19e16f4bd3e627351e1dff95aa033f82006821c657b665af1b3a84494868b4a` |
| Thời điểm khóa | `2026-09-15T03:53:01.255725+00:00` |
| Số row / frame / track trước khi mở reference | `599 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | | | | | | | | | | |
| Sau rework | | | | | | | | | | |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): có

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox xuất hiện trước thời điểm đối tượng tham chiếu xuất hiện | 80–100 | 5 | Kiểm tra lại frame bắt đầu xuất hiện của xe. Nếu bbox được vẽ quá sớm thì đặt `outside` tại frame phù hợp hoặc xóa các bbox thừa. |
| Bbox còn tồn tại sau khi xe rời khung | 149–151 | 4 | Đặt `outside` tại frame xe rời khung và xóa các bbox còn thừa sau thời điểm đó. |
| Bbox còn tồn tại sau khi xe rời khung | 169–171 | 8 | Đặt `outside` tại frame xe rời khung và xóa các bbox còn thừa sau thời điểm đó. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt` / `"bytetrack": { "label": "ByteTrack control", "tracker": "bytetrack.yaml"},
    "reid": { "label": "BoT-SORT + ReID treatment", "tracker": "/content/Day3-Lab/configs/trackers/botsort-reid.yaml"}`|
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / 2, 5, 7` |
| device | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.826 | 0.809 | 0.846 | 0.882 | 0.964 | 0.927 | 0.872 | 34 | 8 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 |54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.793 | 0.735 | 0.859 | 0.906 | 0.896 | 0.788 | 0.897 | 82 | 43 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**
MOTA của tôi thấp hơn IDF1:

- MOTA = 0.927
- IDF1 = 0.964

Điều này cho thấy annotation của tôi vừa có chất lượng detection/tracking tốt, vừa giữ ID khá nhất quán.

MOTA chủ yếu phản ánh các lỗi:

- False Positive — FP;
- False Negative — FN;
- ID Switch — IDSW.

Công thức khái quát:

`MOTA = 1 - (FN + FP + IDSW) / số object ground truth`

MOTA không phạt nặng lỗi ID vì IDSW chỉ được cộng như một thành phần lỗi đơn lẻ. Nếu một track bị đổi ID nhưng vẫn phát hiện đúng object trong nhiều frame, MOTA vẫn có thể cao. Ngược lại, IDF1 tập trung mạnh hơn vào việc ID dự đoán có nhất quán với identity ground truth trong toàn bộ chuỗi frame hay không.

Trong kết quả của tôi, IDF1 cao hơn MOTA và IDSW = 0. Điều này cho thấy lỗi còn lại chủ yếu liên quan đến bbox thừa hoặc bbox thiếu, không phải lỗi đổi ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**
So sánh:

| Metric | ByteTrack | BoT-SORT + ReID | Thay đổi |
| --- | ---: | ---: | ---: |
| IDF1 | 0.875 | 0.900 | ReID cao hơn 0.025 |
| AssA | 0.776 | 0.820 | ReID cao hơn 0.044 |
| IDSW | 2 | 2 | Không thay đổi |

BoT-SORT + ReID tốt hơn ByteTrack ở:

- IDF1: tăng từ 0.875 lên 0.900.
- AssA: tăng từ 0.776 lên 0.820.

Điều này cho thấy treatment có khả năng duy trì association giữa các frame tốt hơn. Appearance/ReID có thể hỗ trợ khi motion và IoU không đủ để phân biệt các xe gần nhau hoặc bị che khuất.

Tuy nhiên, IDSW của hai phương pháp đều bằng 2. Vì vậy ReID không loại bỏ hoàn toàn lỗi đổi ID.

Các lỗi cụ thể được notebook phát hiện:

- ByteTrack:
  - Frame 59: track gold 4 đổi từ ID 14 sang ID 15.
  - Frame 94: track gold 5 đổi từ ID 23 sang ID 32.

- BoT-SORT + ReID:
  - Frame 87: track gold 5 đổi từ ID 17 sang ID 18.
  - Frame 113: track gold 6 đổi từ ID 24 sang ID 31.

ReID tốt hơn về tổng thể association nhưng vẫn có lỗi ID switch. Ngoài ra, đây không phải một thí nghiệm cô lập hoàn toàn causal effect của ReID vì ByteTrack và BoT-SORT là hai tracker implementation khác nhau, không chỉ khác mỗi thành phần appearance.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**
So sánh ByteTrack với BoT-SORT + ReID:

| Metric | ByteTrack | ReID | Nhận xét |
| --- | ---: | ---: | --- |
| DetA | 0.649 | 0.711 | ReID tốt hơn |
| FP | 88 | 91 | ReID tăng 3 FP |
| FN | 54 | 26 | ReID giảm 28 FN |
| AssA | 0.776 | 0.820 | ReID tốt hơn |
| IDSW | 2 | 2 | Không đổi |

ReID có:

- DetA cao hơn: `0.711 > 0.649`.
- FN thấp hơn: `26 < 54`.
- AssA cao hơn: `0.820 > 0.776`.
- Nhưng FP cao hơn một chút: `91 > 88`.

Điều này cho thấy ReID giúp model giữ và phát hiện được nhiều đoạn xe hơn, đặc biệt là các đoạn bị che khuất hoặc detector có độ tin cậy thấp. Tuy nhiên, treatment cũng tạo thêm một số detection không khớp với gold.

Vì vậy lỗi còn lại đến từ cả hai phần:

- Detector/localization
   - FP vẫn còn.
   - Một số bbox bị lệch.
   - Một số xe vẫn bị bỏ sót.
   - ReID không thể tự sửa hoàn toàn detection sai.

- Association/tracking
   - IDSW vẫn bằng 2.
   - Một số track vẫn bị tách thành nhiều ID.
   - ReID cải thiện AssA nhưng không loại bỏ hoàn toàn lỗi association.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`...`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`...`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- Quy định rõ frame bắt đầu và kết thúc track
   - Bắt đầu tại frame đầu tiên object nhìn thấy.
   - Kết thúc tại frame cuối cùng object còn nhìn thấy.
   - Không để bbox tồn tại trước khi object xuất hiện hoặc sau khi object biến mất.

- Quy định về object bị che khuất
   - Vẽ bbox quanh phần nhìn thấy.
   - Không tự đoán phần bị che nếu guideline không yêu cầu.
   - Nếu chỉ còn một phần rất nhỏ hoặc không chắc đó là xe, đánh dấu để review.

- Quy định về Track ID
   - Một chiếc xe chỉ có một track ID trong toàn bộ khoảng thời gian xuất hiện.
   - Không tạo track mới nếu cùng chiếc xe đã có track trước đó.
   - Nếu track muộn, quay lại bổ sung keyframe hoặc merge track.

- Thêm checklist trước khi nộp
   - Kiểm tra số track.
   - Kiểm tra frame đầu/cuối của từng track.
   - Kiểm tra xe có bị chia thành nhiều ID không.
   - Kiểm tra ID có nhảy khi xe bị che không.
   - Kiểm tra bbox có bám sát object không.
   - Kiểm tra các frame có nhiều xe gần nhau.

- Tăng số lần review
   - Lượt 1: kiểm tra ID.
   - Lượt 2: kiểm tra frame đầu và cuối.
   - Lượt 3: kiểm tra frame giữa.
   - Lượt 4: kiểm tra các frame có occlusion hoặc nhiều xe giao nhau.

- Ghi lại các case khó thành ví dụ trong guideline
   - Xe xuất hiện ở rìa frame.
   - Xe bị che bởi hàng rào.
   - Xe biến mất sau vật cản.
   - Một xe bị tạo nhầm thành hai ID.
   - Hai xe gần nhau và dễ bị đổi ID.

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_bytetrack_clip_01.txt`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [X] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)
