# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Châu Nguyễn Trí Vũ (MSSV: 2A202602101)
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm: Các xe đang đỗ bên lề đường nhưng có mặt trong khung hình suốt clip vẫn phải được gán và duy trì track liên tục (không bỏ qua xe tĩnh). Xe đồ chơi, hình vẽ trên thân xe buýt không gán.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID cũ nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) | Quãng đời ngắn dưới 2 giây đủ để con người và tracker suy luận danh tính liên tục qua chuyển động quán tính |
| Xe bị che lâu hơn ngưỡng trên | Mở **track mới (cấp ID mới)** | Quá 2 giây độ bất định về chuyển động và hướng đi quá lớn, không thể đảm bảo cùng một đối tượng |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** | Quy ước chuẩn MOT: đối tượng rời khỏi tầm quan sát xem như kết thúc track |
| Hai xe cắt nhau / chồng lên nhau | Giữ nguyên ID của từng xe; xe ở trước che xe ở sau thì bbox xe sau chỉ ôm phần nhìn thấy | Tránh hoán đổi ID (ID switch) khi hai box giao nhau |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa ảnh, **không đoán** phần thân xe nằm ngoài ảnh |
| Xe bị xe khác che một phần | Bbox chỉ ôm sát phần **nhìn thấy được**, không vẽ bao trùm phần bị che |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **thấy rõ tối thiểu 1 phần đầu xe/bánh xe hoặc kích thước >= 20x20 pixel** |
| Xe đang đỗ, không di chuyển | Gán 1 keyframe đầu và 1 keyframe cuối, duy trì box cố định không đổi |
| Keyframe đặt dày ở đâu | Đặt dày (cứ 5–8 frame/keyframe) tại các khúc cua, xe giảm tốc/tăng tốc, hoặc lúc bắt đầu/kết thúc bị che khuất |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / frame 105–115 / ID 6
- Tình huống: Xe ID 6 đi qua đoạn có bóng râm và cành cây che khuất một phần mui và kính trước.
- Quyết định: Vẫn duy trì ID 6, co hẹp bbox theo từng frame để chỉ bao lấy phần thân xe màu trắng nhìn thấy bên dưới cành cây.
- Lý do: Xe bị che dưới 10 frame (< 1 giây) và vận tốc không đổi, mắt người nhận biết rõ ràng cùng một chiếc xe.

### Ca 2
- Clip / frame / ID: `clip_01` / frame 148–151 / ID 4
- Tình huống: Xe ID 4 đi chéo ra khỏi mép trái màn hình, chỉ còn một góc cản sau ở mép ảnh tại frame 148, frame 149 hầu như khuất hẳn.
- Quyết định: Đặt keyframe cuối cùng tại frame 148 và bấm `Outside` (phím O) ngay tại frame 149.
- Lý do: Nếu không bấm Outside tại frame 149, CVAT sẽ tiếp tục nội suy box trôi ra ngoài vùng nhìn thấy, tạo ra lỗi box treo (Ghost box) làm tăng FP.

### Ca 3
- Clip / frame / ID: `clip_01` / frame 87–91 / ID 5
- Tình huống: Xe ID 5 xuất hiện từ xa ở đường chân trời, frame 87 mới chỉ là một vệt pixel mờ mờ, frame 91 bắt đầu thấy rõ hình khối xe con.
- Quyết định: Bắt đầu track từ frame 91 (thay vì frame 87).
- Lý do: Tuân thủ quy tắc entry threshold: chỉ gán khi đã đủ chi tiết khẳng định là xe 4 bánh, tránh nguy cơ gán nhầm xe máy từ xa.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Bổ sung định lượng ngưỡng xuất hiện (Entry threshold):** Cần quy định rõ kích thước tối thiểu (20x20 pixel hoặc thấy rõ bánh/mũi xe) để tránh việc người gán quá sớm (gây FP) hoặc quá muộn (gây FN) như trường hợp xe ID 5.
- **Rà soát điểm kết thúc (Exit threshold):** Luôn tua chậm từng frame ở rìa ảnh trước khi export, đảm bảo frame đầu tiên xe khuất hẳn phải có thuộc tính `outside=1` để không bị phạt lỗi box treo.
