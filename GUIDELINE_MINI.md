# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con cá nhân 4–7 chỗ cỡ nhỏ/vừa: sedan, hatchback, SUV, crossover, taxi, xe bán tải cabin nhỏ. | Có nắp capo và cốp sau rõ hoặc dáng thấp; phân biệt với `van` ở phần đuôi/mui không vuông cao dạng hộp chở khách/hàng. |
| `truck` | Xe tải chở hàng, xe thùng kín/thùng bạt, xe ben, xe đầu kéo. | Có cabin tách biệt với thùng hàng phía sau hoặc khung gầm cao, kích thước lớn hơn xe con/bán tải. |
| `bus` | Xe buýt công cộng, xe khách cỡ lớn hoặc minibus chở khách nhiều hàng ghế. | Thân dài, cao, dãy cửa kính liên tục dọc hai bên hông xe. |
| `van` | Xe van/minivan chở người hoặc chở hàng (MPV/microvan thân hộp). | Dáng hình hộp liền khối (one-box/two-box), mui cao thẳng về phía đuôi, ngắn hơn `bus` và cao/vuông hơn `car`. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: 4 cạnh bounding box ôm sát phần thân xe (kể cả bánh xe và gương chiếu hậu nếu nhìn thấy rõ), không lấy dư mặt đường, bóng đổ hay khoảng trống xung quanh.
- Vật thể bị che khuất: Chỉ vẽ 1 box bao quanh đúng phần diện tích nhìn thấy được của xe nếu phần đó đủ rõ để xác định chắc chắn lớp (`car`/`truck`/`bus`/`van`); không ước lượng phần bị che.
- Vật thể bị cắt bởi rìa ảnh: Kéo cạnh box chạm sát biên ảnh (`0.0` hoặc `1.0`), không vẽ tràn ra ngoài khung hình; nếu chỉ còn mảnh quá nhỏ sát mép không rõ loại xe thì bỏ qua.
- Vật thể rất nhỏ/mơ hồ: Các xe ở quá xa phía chân trời/nền hậu cảnh hoặc bị lóa đèn/mờ không thể phân biệt rõ loại xe hoặc nằm ngoài vùng quan sát chính thì không gán để tránh nhiễu (False Positive).

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001` | Dãy xe rất nhỏ ở làn đường nhánh bên trái phía xa (`x < 0.70`, kích thước nhỏ và bị che bởi dải phân cách/cây) | Không gán nhãn các xe ở vùng xa/ngoài làn chính; chỉ gán 5 xe `car` rõ trên làn chính bên phải | Theo quy tắc vật thể rất nhỏ/mơ hồ ở hậu cảnh xa không thuộc vùng quan sát chính và khó xác định chính xác biên box. |
| `drive_009` | Các xe thân hộp màu sáng/bạc (`minivan`/MPV) chạy xen kẽ với sedan (`car`) và xe tải nhỏ (`truck`) | Gán lớp `3` (`van`) cho xe thân hộp liền khối mui cao; gán `1` (`truck`) cho xe có thùng hàng phía sau | Tuân thủ đúng ID schema bắt buộc (`car=0, truck=1, bus=2, van=3`), phân biệt `van` (thân hộp liền) và `truck` (thùng hàng rời). |
| `drive_031` | Nhóm phương tiện tụ tập ở rất xa phía trên cầu/ngã tư (`y_center < 0.28`), bị chồng lấp dày đặc | Không gán các cụm xe mờ ở xa (`y_center < 0.28`), chỉ gán 14 xe rõ hình dạng ở tiền cảnh và trung cảnh | Tránh đoán mò lớp và vẽ box chồng chéo thiếu chính xác đối với vật thể quá nhỏ dưới ngưỡng nhận diện. |
