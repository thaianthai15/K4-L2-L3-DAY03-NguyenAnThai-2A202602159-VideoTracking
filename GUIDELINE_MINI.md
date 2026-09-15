# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn An Thái`
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

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới ... frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Duy trì tính liên tục của track` |
| Xe bị che lâu hơn ngưỡng trên | `Chỉ tạo ID mới nếu không thể xác định chắc chắn đó là xe cũ` | `Tránh gán nhầm ID` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `Xe không được quan sát liên tục` |
| Hai xe cắt nhau / chồng lên nhau | `Theo dõi dựa trên vị trí, hướng di chuyển và đặc điểm hình dáng` | `Hạn chế đổi ID giữa hai xe` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `Không tạo track chỉ dựa trên một vùng mờ hoặc vài pixel chưa chắc chắn.` |
| Xe đang đỗ, không di chuyển | `Vẫn giữ nguyên track ID và tiếp tục gán bbox qua các frame nếu xe còn xuất hiện. Không tạo ID mới chỉ vì xe đứng yên.` |
| Keyframe đặt dày ở đâu | `Đặt keyframe dày hơn tại các đoạn xe đổi hướng, tăng/giảm kích thước nhanh, bị che khuất, đi gần nhau, xuất hiện hoặc biến mất khỏi ảnh. Với đoạn chuyển động ổn định, có thể đặt keyframe thưa hơn và nội suy bbox.` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `1/11/1`
- Tình huống: `Track ID 1 chỉ xuất hiện từ frame 1 đến frame 11 rồi kết thúc. Đây có thể là xe đi ra khỏi khung hình, bị che khuất hoặc không còn đủ rõ để tiếp tục gán nhãn.`
- Quyết định: `Kết thúc track ID 1 tại frame 11, không kéo bbox sang frame 12 trở đi.`
- Lý do: `Không tự suy đoán vị trí xe ở các frame sau khi không còn quan sát rõ.`

### Ca 2
- Clip / frame / ID: `1/54/4`
- Tình huống: `Track ID 4 bắt đầu xuất hiện từ frame 54, trong khi các track khác đã xuất hiện trước đó. Đây là trường hợp xe mới đi vào khung hình hoặc trước đó quá nhỏ/mờ nên chưa đủ điều kiện tạo track.`
- Quyết định: `Bắt đầu track ID 4 tại frame 54.`
- Lý do: `Chỉ tạo ID khi xe đã được xác định rõ là xe bốn bánh. Không gán ngược bbox cho những frame trước đó nếu chưa quan sát được xe một cách chắc chắn.`

### Ca 3
- Clip / frame / ID: `2/15/6`
- Tình huống: `Track ID 6 bắt đầu từ frame 15 và kết thúc tại frame 39. Đây là trường hợp cần quyết định thời điểm bắt đầu/kết thúc track khi xe xuất hiện hoặc biến mất trong cảnh.`
- Quyết định: `Gán ID 6 từ frame 15 đến frame 39; không gán ID 6 ở frame 14 hoặc frame 40 nếu xe chưa đủ rõ hoặc đã không còn quan sát được.`
- Lý do: `Luật nhóm là bắt đầu từ frame đầu tiên xác định được xe và kết thúc ở frame cuối cùng còn xác định được xe.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

Sau khi đối chiếu với gold và kiểm tra chéo, cần làm rõ các luật sau:

- Quy định thời điểm bắt đầu track:
    Bắt đầu track tại frame đầu tiên xe được xác định rõ là xe bốn bánh. Không bắt đầu chỉ vì xuất hiện một vùng nhỏ hoặc mờ có khả năng là xe.
- Quy định thời điểm kết thúc track:
    Kết thúc track tại frame cuối cùng còn nhìn thấy và xác định được xe. Không kéo bbox sang các frame xe đã ra khỏi ảnh hoặc bị che hoàn toàn.
- Quy định khi xe bị che khuất:
    Nếu chỉ nhìn thấy một phần xe, bbox chỉ bao quanh phần nhìn thấy. Nếu xe bị che hoàn toàn, không vẽ bbox cho các frame đó trừ khi guideline của gold quy định rõ được phép nội suy qua vùng che khuất.
- Quy định giữ track ID:
    Khi xe xuất hiện lại sau một đoạn ngắn, cần ưu tiên giữ ID cũ nếu có đủ đặc điểm để xác định đó là cùng một xe. Không tạo ID mới chỉ vì xe tạm thời bị che hoặc đứng yên.
- Quy định đối với xe đang đỗ:
    Xe đứng yên vẫn là một object hợp lệ và phải giữ nguyên ID qua các frame mà xe còn xuất hiện.
- Quy định đặt keyframe:
    Cần đặt keyframe dày tại các đoạn có chuyển động nhanh, thay đổi kích thước bbox, xe bị che, xe giao nhau hoặc gần nhau. Những đoạn chuyển động ổn định có thể dùng nội suy.
- Quy định khi hai xe chồng lấn:
    Mỗi xe phải có bbox và track ID riêng. Không gộp hai xe thành một bbox duy nhất.
- Quy định về độ chính xác bbox:
    Bbox cần ôm sát phần xe nhìn thấy, hạn chế lấy dư quá nhiều nền hoặc phần của object khác.
