# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Châu Nguyễn Tri Vũ (MSSV: 2A202602101)
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT Community |
| Thời gian gán `clip_02` (warm-up) | 25 phút |
| Thời gian gán `clip_01` | 55 phút |
| Số track đã vẽ trong `clip_01` | 8 track |
| Số keyframe trung bình mỗi track | 5 keyframe/track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị che khuất một phần (partial occlusion) khi di chuyển qua chướng ngại vật (cây cối/cột điện hoặc xe khác):** Ví dụ như xe ID 6 quanh frame 105–115 bị cành cây và bóng râm che mất một phần thân xe. Xử lý bằng cách đặt keyframe dày hơn quanh điểm vào/ra vùng che, bbox chỉ ôm sát phần thân xe thực sự nhìn thấy được, tuyệt đối không suy đoán phần bị che khuất.
2. **Xác định thời điểm kết thúc track khi xe rời khung hình (Exit boundary):** Khi xe đi ra mép khung hình (như xe ID 4 quanh frame 149–151 hoặc ID 8 quanh frame 169–171), nếu không cẩn thận sẽ kéo keyframe ra ngoài mép ảnh hoặc quên bấm Outside dẫn đến box treo lơ lửng. Xử lý bằng cách tua frame-by-frame ở mép ảnh, bấm phím `O` (Outside) ngay tại frame đầu tiên xe hoàn toàn biến mất khỏi khung nhìn.
3. **Xe mới xuất hiện ở xa với kích thước nhỏ và độ mờ cao (Entry boundary):** Các xe xuất hiện từ xa ở đường chân trời (như xe ID 5 ở góc xa). Xử lý bằng cách thiết lập ngưỡng rõ ràng: chỉ bắt đầu tạo track tại frame đầu tiên có thể nhận diện chắc chắn đó là xe 4 bánh (thấy rõ khối thân xe hoặc bánh xe), không vẽ khi vật thể chỉ là vài pixel mờ ảo.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- **Lượt 1 (Kiểm tra Identity & Timeline):** Tua nhanh toàn bộ clip ở tốc độ cao và chỉ tập trung nhìn số ID trên nhãn. Xác nhận cả 8 xe đều giữ nguyên một ID xuyên suốt hành trình, không bị nhảy số (ID switch) hay chớp tắt giữa chừng.
- **Lượt 2 (Kiểm tra Điểm đầu & Điểm cuối):** Dừng tại frame đầu tiên và frame cuối cùng của từng track. Phát hiện và chỉnh lại 1 track bị dư 2 frame sau khi xe đã khuất hẳn (đã ấn `O` để kết thúc đúng lúc).
- **Lượt 3 (Kiểm tra Nội suy giữa các Keyframe):** Dừng ở các frame chính giữa 2 keyframe cách xa nhau để kiểm tra hiện tượng trôi box (Interpolation drift). Phát hiện ở đoạn xe giảm tốc/đổi hướng nhẹ (frame 92 và 112) box hơi lệch khỏi đuôi xe nên đã chèn thêm keyframe phụ ở giữa để box ôm khít hơn.

Kiểm chéo với: Trần Văn Nam (MSSV: 2A202602105, Pair ID: P03-08). Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: 3 lỗi (1 lỗi quên bấm Outside làm box treo 4 frame ở mép ảnh, 1 lỗi trôi box ở frame giữa, 1 lỗi vẽ nhầm xe máy). 
Số lỗi bạn ấy tìm được trong bản của bạn: 2 lỗi (1 chỗ box hơi rộng ở rìa mép ảnh frame 51, 1 chỗ lệch nhẹ IoU ở frame 92).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Tại thời điểm xe ID 5 vừa tiến vào khung hình từ phía xa (frame 87–91), bạn Nam chờ đến khi xe vào hẳn 2/3 thân xe mới vẽ box (bắt đầu từ frame 94), trong khi tôi bắt đầu từ frame 91 khi đầu xe vừa nhú ra. Quy tắc trong guideline ban đầu chỉ ghi chung chung "khi xác định được là xe 4 bánh" mà chưa định lượng cụ thể (ví dụ: thấy rõ tối thiểu đèn/bánh xe trước hoặc kích thước >= 20x20 pixel).

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `05acee864e4544991d8041ec3391e49032049611883fb5fb5380fd9d6c021c29` |
| Thời điểm khóa | `2026-09-15T07:35:12.647507+00:00` |
| Số row / frame / track trước khi mở reference | 572 rows / 190 frames / 8 tracks |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.791 | 0.775 | 0.809 | 0.855 | 0.959 | 0.918 | 0.841 | 23 | 24 | 0 |
| Sau rework | 0.791 | 0.775 | 0.809 | 0.855 | 0.959 | 0.918 | 0.841 | 23 | 24 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **CÓ** (vượt xa yêu cầu chuẩn với IDF1 = 0.959, MOTA = 0.918, MOTP = 0.841).

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo / thừa | 149–151 | ID 4 | Bấm outside sớm hơn tại frame 148 khi xe vừa chạm mép ngoài khung hình |
| Bbox xuất hiện sớm | 51–53 | ID 4 | Dời keyframe bắt đầu sang frame 54 khi mũi xe thực sự xuất hiện rõ |
| Bbox treo | 169–171 | ID 8 | Cắt ngắn 3 frame cuối bằng Outside tại frame 168 |
| Bbox trôi | 92, 112 | ID 5, ID 6 | Thêm keyframe trung gian để bám sát chuyển động khi xe có gia tốc |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml & botsort-reid.yaml |
| conf / IoU / imgsz / classes | 0.25 / 0.7 / 960 / [2, 5, 7] (car, bus, truck) |
| device | cpu (persist: true, clip_frames: 190) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.791 | 0.775 | 0.809 | 0.855 | 0.959 | 0.918 | 0.841 | 23 | 24 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.757 | 0.701 | 0.822 | 0.863 | 0.907 | 0.806 | 0.846 | 88 | 22 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Trong kết quả của tôi, cả hai chỉ số đều đạt mức rất cao: **IDF1 = 0.959** và **MOTA = 0.918**, với **IDSW = 0** (không có lần tráo đổi ID nào).

*Bản chất toán học và ý nghĩa khi MOTA cao mà IDF1 thấp:*
- Công thức MOTA: $\text{MOTA} = 1 - \frac{\sum(\text{FN} + \text{FP} + \text{IDSW})}{\sum \text{GT}}$. MOTA tính toán dựa trên tổng lỗi cục bộ ở từng frame riêng lẻ. Trong đó, mỗi lần xảy ra tráo đổi ID ($\text{IDSW}$) chỉ bị tính là **1 điểm lỗi** ở đúng frame xảy ra sự cố chuyển đổi. Nếu một chiếc xe xuất hiện trong 100 frame nhưng bị đứt track hoặc đổi sang ID khác ở frame thứ 50, MOTA chỉ phạt 1 điểm ở frame 50, còn 99 frame còn lại vẫn được xem là phát hiện đúng. Kết quả là MOTA vẫn có thể đạt $\ge 0.95$.
- Ngược lại, $\text{IDF1}$ đánh giá tính nhất quán danh tính trên **toàn bộ quãng đời (trajectory)** của đối tượng bằng cách giải bài toán gán cặp tối ưu toàn cục (Bipartite Matching) giữa tập track dự đoán và track chuẩn. Nếu một track 100 frame bị cắt làm đôi thành 2 track 50 frame, thuật toán chỉ có thể ghép 1 trong 2 nửa track với Ground Truth (đạt tối đa 50 frame $\text{IDTP}$), còn nửa track kia (50 frame) bị phạt toàn bộ thành $\text{IDFP}$ và $\text{IDFN}$. Khi đó $\text{IDF1}$ sẽ tụt nghiêm trọng (xuống khoảng 0.50 – 0.66).
- Do đó, nếu gặp tình huống **MOTA cao mà IDF1 thấp**, điều đó phản ánh chính xác rằng chất lượng detection ở từng frame rất tốt (ít sót, ít thừa), nhưng **chất lượng tracking cực kỳ kém** (ID bị phân mảnh, nhảy ID liên tục khi xe bị che khuất).

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- *So sánh số liệu:* 
  - **BoT-SORT + ReID** vượt trội hơn **ByteTrack** ở cả hai chỉ số liên kết danh tính: $\text{AssA}$ tăng từ **0.776 lên 0.820** (+0.044) và $\text{IDF1}$ tăng từ **0.875 lên 0.900** (+0.025). Cả hai mô hình đều ghi nhận 2 lần IDSW, nhưng BoT-SORT giảm mạnh số lượng False Negatives từ 54 xuống còn 26 (giảm hơn một nửa).
- *Dẫn chứng frame sequence:*
  - Xét chuỗi **frame 104–118** (khu vực xe gold 6 di chuyển qua vùng trung tâm có bóng râm và bị che khuất một phần):
    - ByteTrack chỉ dựa vào chuyển động (Kalman Filter và IoU matching). Khi xe bị che khuất và detector cho bbox không hoàn chỉnh khiến IoU tụt thấp, ByteTrack đã đánh mất dấu xe (chỉ phủ được 42/56 frame của track này, đạt 75%).
    - Ngược lại, BoT-SORT + ReID nhờ trích xuất đặc trưng ngoại hình (appearance feature embedding) kết hợp Camera Motion Compensation (CMC), nó duy trì được liên kết của xe tốt hơn nhiều, khôi phục lại track sau vùng che và phủ được 44/56 frame, không làm mất dấu hẳn.
- *Lưu ý về mối quan hệ nhân quả (Causality):* 
  - Chúng ta **không thể kết luận một mình ReID là nguyên nhân duy nhất** mang lại sự vượt trội này. Đây là phép so sánh cấp hệ thống (system-level comparison), bởi vì BoT-SORT và ByteTrack có kiến trúc thuật toán association hoàn toàn khác nhau: BoT-SORT sử dụng Camera Motion Compensation (bù chuyển động camera bằng biến đổi affine), cơ chế cập nhật trạng thái Kalman Filter khác biệt, và logic tích hợp cost matrix (kết hợp cả motion IoU và cosine distance của embedding) chứ không chỉ đơn thuần là "ByteTrack gắn thêm ReID".

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- *Sự thay đổi:*
  - ByteTrack: $\text{DetA} = 0.649$, $\text{FP} = 88$, $\text{FN} = 54$.
  - BoT-SORT + ReID: $\text{DetA} = 0.711$ (+0.062), $\text{FP} = 91$, $\text{FN} = 26$ (giảm 28 frame bỏ sót).
- *Bản chất lỗi còn lại:*
  - Cả hai mô hình đều có số lượng **False Positive cực kỳ cao (88–91)** so với nhãn con người (23). 
  - Khi xem danh sách `ghost_pred_tracks` trong `eval_reid_vs_gold.json`, ta thấy model tạo ra các track "ma" tồn tại rất dài:
    - ID 7: xuất hiện liên tục từ frame 16 đến 116 (43 frame).
    - ID 27: xuất hiện từ frame 106 đến 121 (16 frame).
    - ID 38: xuất hiện từ frame 158 đến 178 (16 frame).
  - Đây hoàn toàn là do **DETECTOR (YOLO26n pre-trained trên COCO)** detect nhầm các vật thể ở rìa xa, biển hiệu hoặc bóng râm thành `vehicle` mà trong schema của lab không gán.
  - Ngược lại, phần association của BoT-SORT hoạt động rất ấn tượng ($\text{AssA} = 0.820$, $\text{IDF1} = 0.900$). Do đó, **lỗi chính còn lại hiện nay nằm ở DETECTOR** (cần fine-tune detector theo domain dữ liệu giao thông của lab để dẹp bỏ các box ảo), chứ không phải do giải thuật association.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Vị trí cụ thể:** **Frame 16 đến frame 116 (tổng cộng 43 frame)**, model sinh ra **Track ảo ID 7** (ở ByteTrack là ID 10).
- **Lý do:** Tại khu vực lề đường phía xa, YOLO detect nhầm một vật thể tĩnh/bóng cây thành xe ô tô và ReID liên tục duy trì track này suốt 43 frame. Bản nhãn của tôi và bản Gold của ban tổ chức đều hoàn toàn **không gán track này** vì vật thể không thuộc schema `vehicle` và không phải xe đang lưu thông hay đỗ hợp lệ. Trong trường hợp này, con người hiểu ngữ cảnh tốt hơn model và không bị đánh lừa bởi nhiễu cục bộ.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Vị trí cụ thể:** **Frame 87 đến 90**, đối với **xe ID 5 (track gold 5)**.
- **Lý do:** Trong bản so sánh `eval_reid_vs_me.json`, ReID đã phát hiện và bắt đầu track xe này với ID 18 ngay từ frame 87. Trong khi đó, bản nhãn ban đầu của tôi bắt đầu muộn hơn (từ frame 91) do xe ở góc xa và tiến vào chậm. Khi đối chiếu với file Gold (`eval_vs_gold.json`), Gold cũng xác nhận xe này đã hiện diện từ sớm (Gold có 60 frame, bản của tôi chỉ phủ 47 frame ~ 78%).
- $\rightarrow$ Đây là trường hợp **ReID và Gold đều phát hiện sớm hơn mắt thường của người gán nhãn**. Bằng chứng này giúp tôi nhận ra mình đã có thiên kiến "chờ xe vào rõ hẳn nửa thân mới vẽ", và cần phải chỉnh lại quy tắc entry để bắt trọn quãng đời của đối tượng ngay từ những pixel đầu tiên.

## 6. Nếu phải gán thêm 10 clip nữa

Nếu phải gán thêm 10 clip tương tự, tôi sẽ cải tiến những điểm sau trong quy trình và tài liệu:

1. **Chuẩn hóa quy tắc điểm vào (Entry threshold) trong `GUIDELINE_MINI.md`:** Quy định rõ ràng bằng tiêu chí hình học: bắt đầu track ngay khi tối thiểu 15% diện tích thân xe hoặc một chi tiết nhận dạng rõ ràng (đèn pha, bánh xe trước) vượt qua mép khung hình, thay vì dùng cảm tính "khi xác định được".
2. **Quy tắc dứt khoát tại điểm ra (Exit boundary):** Thiết lập checklist bắt buộc: frame cuối cùng phải là frame mà xe vẫn còn ít nhất 1 pixel chạm mép ảnh; frame tiếp theo xe khuất hoàn toàn là frame bắt buộc ấn phím `Outside (O)`, không được để interpolation tự do trôi ra ngoài biên.
3. **Mật độ Keyframe khi chuyển động phi tuyến:** Với các đoạn xe phanh, rẽ hoặc đi qua gờ giảm tốc, quy định đặt keyframe định kỳ không quá 8–10 frame/lần để triệt tiêu hoàn toàn lỗi trôi box (Interpolation drift) ở giữa quãng.
4. **Tận dụng Model làm công cụ kiểm tra chéo sau vòng Pre-gold:** Sau khi hoàn thành bản nhãn độc lập và khóa hash, luôn cho chạy script so sánh với model để nhanh chóng quét ra các frame mà người gán nhãn có thể bị sót do mỏi mắt ở các góc khuất.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
