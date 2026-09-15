

Họ tên / nhóm: Nguyễn Đăng Vĩ Anh / 2A202602149
Ngày: 15/09/2026



## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | 30 phút |
| Thời gian gán `clip_01` | 90 phút |
| Số track đã vẽ trong `clip_01` | 8 track |
| Số keyframe trung bình mỗi track | ~76 keyframe/track (603 row / 8 track) |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Vật thể bị che khuất (Occlusion) một phần hoặc toàn phần:** Xử lý bằng cách bật cờ `Occluded` trên CVAT ngay khi vật thể bị che, dự đoán nội suy (interpolate) hình dáng thật của xe sau vật cản thay vì chỉ bọc phần xe bị lộ ra.
2. **Xe tiến ra sát mép và biến mất khỏi khung hình:** Khó xác định frame cuối cùng. Xử lý bằng cách neo Keyframe và bật cờ `Outside` ngay tại đúng frame mà xe hoàn toàn không còn điểm ảnh nào trên màn hình.
3. **Hai xe đi cắt ngang qua nhau:** Rất dễ bị nhầm lẫn và nhảy ID (ID Switch). Xử lý bằng cách chú ý danh sách Track hoạt động bên sidebar, gắn keyframe neo chặt cho từng xe trước và ngay sau khi cắt nhau để CVAT không nối nhầm.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

* Lượt 1: Tập trung hoàn toàn vào theo dõi ID. Phát hiện ra một số xe bị nhảy ID thành xe mới khi đi qua cột đèn và đã gộp track lại.
* Lượt 2: Soi kỹ các frame lúc xe mới vào và đi ra khỏi hình để tắt/bật cờ `Outside` cho chuẩn xác, bỏ các bbox treo (xe chưa vào hoặc đã ra).
* Lượt 3: Kiểm tra độ khít của Bounding Box ở các frame giữa (chỉnh lại cho sát mép xe, giảm thiểu lỗi BBOX TRÔI).

Kiểm chéo với: [Điền tên partner]. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: [Điền số lượng]. Số lỗi bạn ấy tìm được trong bản của bạn: [Điền số lượng].

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Hai người quyết định khác nhau về việc khoanh Bounding box khi xe ở quá xa (kích thước quá nhỏ) và các xe đang đỗ tĩnh ven đường. Luật còn thiếu cần thêm vào `GUIDELINE_MINI.md`: "Chỉ gán nhãn cho xe 4 bánh đang di chuyển trong dòng phương tiện, bỏ qua hoàn toàn các xe đỗ tĩnh trên vỉa hè hoặc có kích thước dưới 15x15 pixel".

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `dd3b55e5c09a6b4b368aaedce51742ae545ac3b2eaf31f728a0c141f3794f2f2` |
| Thời điểm khóa | 2026-09-15T07:57:00 UTC (14:57 ICT) |
| Số row / frame / track trước khi mở reference | 603 row / 190 frame / 8 track |

|  | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Bản pre-gold | [Điền từ eval_vs_gold.json pre-gold] | [Điền] | [Điền] | [Điền] | [Điền] | [Điền] | [Điền] | [Điền] | [Điền] | [Điền] |
| Sau rework | 0.756 | 0.735 | 0.785 | 0.820 | 0.971 | 0.941 | 0.792 | 32 | 2 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo | 86-100 | 6 | Đã có bbox trước khi track 6 xuất hiện. Sửa bằng cách bấm cờ Outside ở frame 85. |
| Bbox thừa | 149-151 | 4 | Còn bbox sau khi xe 4 đã rời khung. Sửa bằng cách bấm cờ Outside đúng ở frame 149. |
| Bbox trôi | 80-96 | 5 | IoU giảm xuống mức 0.51 - 0.56. Sửa bằng cách thêm các keyframe neo tại đây và căn chỉnh lại độ khít của box. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml & botsort-reid.yaml |
| conf / IoU / imgsz / classes | 0.25 / 0.7 / 960 / [2, 5, 7] |
| device | cpu |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| bạn vs gold | 0.756 | 0.735 | 0.785 | 0.820 | 0.971 | 0.941 | 0.792 | 32 | 2 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.696 | 0.634 | 0.774 | 0.824 | 0.880 | 0.756 | 0.799 | 90 | 55 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Của tôi MOTA (0.941) thấp hơn IDF1 (0.971). Tuy nhiên, nếu MOTA cao mà IDF1 thấp, điều đó cho thấy người gán nhãn có khả năng nhận diện/bắt vị trí vật thể (Detection) rất tốt (ít lỗi FP/FN) nhưng lại làm việc duy trì định danh xuyên suốt (Association) kém, dẫn đến lỗi nhảy ID (IDSW) liên tục. MOTA không phạt nặng lỗi ID vì công thức của nó cộng dồn tổng lỗi FP, FN và IDSW chia cho tổng Ground Truth. Trong hàng trăm frame, các lỗi FP/FN tích lũy rất nhanh, trong khi một lần nhảy IDSW làm hỏng hoàn toàn 1 track nhưng chỉ bị tính là 1 lỗi duy nhất về mặt toán học.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

Đối với BoT-SORT + ReID, IDF1 (0.900) và AssA (0.820) đều cao hơn hẳn so với ByteTrack (0.875 và 0.776). IDSW của cả 2 đều là 2. ReID đã làm tốt hơn trong việc nối vết các object. Ví dụ, ByteTrack gặp lỗi tách track gold 4 thành ID 14 và 15 (ở frame 59), nhưng ReID đã xử lý tốt đoạn này không bị đứt. Tuy nhiên, sự khác biệt này không hoàn toàn cô lập causal effect (tác động nhân quả) của riêng module ReID, vì ByteTrack và BoT-SORT có cách implementation thuật toán theo dõi khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA của ReID (0.711) tăng mạnh so với ByteTrack (0.649). FP tăng rất nhẹ (từ 88 lên 91), nhưng FN lại giảm ngoạn mục xuống còn một nửa (từ 54 xuống 26). Điều này cho thấy lỗi chủ yếu nằm ở khâu Association (nối vết) chứ không phải do Detector. Detector vẫn tìm ra hộp bắt hình (bbox), nhưng ByteTrack không nối được các bbox điểm thấp nên vứt bỏ và biến chúng thành lỗi FN (False Negative), trong khi BoT-SORT với sự trợ giúp của ReID đã nối kết thành công.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 16 đến 116, ReID bắt thừa (FP) ID 7 kéo dài tới 43 frame mà không khớp với track tham chiếu nào. Lý do: ReID bắt nhầm vào một vật thể tĩnh/không phải xe cộ thật (ví dụ xe đỗ ven đường hoặc biển báo) vì nó đứng im không nhúc nhích. Nhãn tay của tôi đã bỏ qua vật thể này nên nhãn của tôi là đúng.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Từ frame 107 đến 119, ID 6 (track bản A của tôi) bị model ReID ghi nhận là có mức độ lệch bbox so với model khá lớn (IoU chỉ quanh mức 0.56 - 0.58). Điều này làm tôi phải xem lại annotation vì có thể ở chuỗi frame này tôi đã gán bounding box bị lỏng, chưa ép sát vào mép xe thực tế hoặc bị trôi giữa các keyframe.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

* **Trong GUIDELINE_MINI.md:** Cập nhật rõ định nghĩa "Xe 4 bánh hợp lệ" (Loại trừ hoàn toàn xe đang đỗ tĩnh, xe ở quá xa rìa camera). Quy định rõ phải bật cờ `Occluded` ngay khi xe bị che khuất trên 30% diện tích.
* **Quy trình làm việc:** Áp dụng phương pháp làm việc 2 pass (2 vòng). Vòng 1 chỉ chuyên gán Keyframe ở các điểm rẽ và đặt cờ Outside. Vòng 2 rà soát lại toàn bộ độ khít (IoU) của các Bounding box nội suy để tránh lỗi "Bbox trôi" (khắc phục các IoU thấp ~0.50).

## 7. Tệp đã nộp

* [x] `annotations/clip_01/gt.txt`
* [x] `annotations/clip_02/gt.txt`
* [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
* [x] `GUIDELINE_MINI.md` đã điền
* [x] `outputs/eval_vs_gold.json`
* [x] `outputs/model_bytetrack_clip_01.txt`
* [x] `outputs/model_reid_clip_01.txt`
* [x] `outputs/model_run_config.json`
* [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
* [ ] `reports/review_partner.md`
* [x] `reports/REPORT.md` (file này)
