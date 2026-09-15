# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: SOLO
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

Bổ sung của nhóm (nếu có): không bổ sung

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 5 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Đảm bảo tính liên tục của cùng một đối tượng khi bị che khuất trong thời gian ngắn. |
| Xe bị che lâu hơn ngưỡng trên | tạo **track mới / ID mới** khi xe xuất hiện lại | Khi thời gian che khuất quá dài, khó đảm bảo đối tượng xuất hiện lại chính là cùng track; tách ID giúp giảm nguy cơ gán nhầm danh tính. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Khi xe đã hoàn toàn rời khỏi frame, lần xuất hiện lại được xem là một lần xuất hiện mới để tránh nối nhầm với track cũ. |
| Hai xe cắt nhau / chồng lên nhau | giữ nguyên ID của từng xe dựa trên vị trí, hình dạng và đặc điểm quan sát được trước/sau khi giao nhau; **không đổi ID chỉ vì hai xe chồng lấn** | Tránh ID switch khi các đối tượng giao nhau. Khi bị che một phần, bbox chỉ bao quanh phần xe thực sự nhìn thấy, không suy đoán phần bị che khuất. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được**, không suy đoán phần bị che khuất |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **chỉ gán khi đủ rõ để xác định chắc chắn là vehicle** |
| Xe đang đỗ, không di chuyển | vẫn giữ bbox và ID nếu xe còn xuất hiện trong frame; không bỏ track chỉ vì xe không chuyển động |
| Keyframe đặt dày ở đâu | đặt dày hơn tại các đoạn xe **bắt đầu/kết thúc xuất hiện, bị che khuất, giao nhau với xe khác, thay đổi hướng/chuyển động hoặc bbox thay đổi rõ rệt**; đoạn chuyển động ổn định có thể đặt thưa hơn |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: `clip_01 / frame 80 / ID 5`
- Tình huống: Xe bị che khuất một phần, làm cho hình dạng đầy đủ của xe không quan sát được và bbox có thể khác đáng kể so với teaching reference.
- Quyết định: Giữ nguyên ID và đặt bbox theo **phần xe nhìn thấy được**, không suy đoán phần bị che khuất.
- Lý do: Guideline yêu cầu bbox chỉ bao quanh phần phương tiện thực sự quan sát được. Việc xe bị che một phần không phải lý do để đổi ID nếu vẫn có thể xác định đó là cùng một xe.

### Ca 2

- Clip / frame / ID: `clip_01 / frame 103 / ID 6`
- Tình huống: ID 6 xuất hiện trong vùng có che khuất/chuyển tiếp, khiến vị trí và kích thước bbox khó xác định chính xác.
- Quyết định: Giữ nguyên ID 6 và điều chỉnh/kiểm tra bbox theo phần phương tiện thực sự nhìn thấy trong frame.
- Lý do: Ưu tiên tính liên tục của ID khi vẫn nhận diện được cùng phương tiện; không mở rộng bbox vào vùng bị che khuất chỉ để khớp với hình dạng giả định.

### Ca 3

- Clip / frame / ID: `clip_01 / frame 80–100 / ID 6`
- Tình huống: Có bbox của ID 6 xuất hiện sớm hơn thời điểm phương tiện thực sự đủ rõ để xác định trong frame.
- Quyết định: Điều chỉnh điểm bắt đầu của track ID 6, không duy trì bbox ở các frame trước khi xe thực sự xuất hiện rõ.
- Lý do: Track chỉ nên bắt đầu từ frame đầu tiên có thể xác định đáng tin cậy đó là một phương tiện bốn bánh; tránh tạo false positive ở phần đầu track.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Cần quy định rõ thời điểm bắt đầu và kết thúc track: chỉ bắt đầu khi xe đủ rõ để xác định là vehicle, và kết thúc khi xe thực sự rời khỏi frame; không để bbox xuất hiện trước khi xe xuất hiện hoặc tồn tại sau khi xe đã rời frame.

- Cần quy định rõ cách xử lý bbox khi xe bị che khuất: bbox chỉ bao quanh phần xe nhìn thấy được, không suy đoán phần bị che; khi xe bị che ngắn dưới ngưỡng quy định thì giữ nguyên ID, còn khi vượt ngưỡng hoặc rời khỏi frame rồi quay lại thì tạo track/ID mới.

- Cần kiểm tra riêng các điểm chuyển tiếp của track (track start/end), vì đây là nơi dễ tạo bbox thừa và false positive dù bbox ở các frame giữa track vẫn đúng.