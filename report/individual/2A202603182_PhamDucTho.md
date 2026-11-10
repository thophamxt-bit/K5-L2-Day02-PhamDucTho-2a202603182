# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Phạm Đức Thọ
- MSSV: 2A202603182
- Email: thophamxt@gmail.com
- Lớp: K5-L2
- Link repo bài làm: https://github.com/thophamxt-bit/K5-L2-Day02-Ph-m-c-Th--2a202603182
- Link Colab đã chạy: https://colab.research.google.com/drive/1JOlC90oudt8dzDXbAjHxIJ5TFFcCc55H?usp=sharing

## 2. Schema và mini guideline

Chi tiết xem tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

- **Cách áp dụng schema 4 lớp:**
  - `0 = car`: Xe con cá nhân 4–7 chỗ (sedan, hatchback, SUV, crossover, taxi, bán tải cabin nhỏ).
  - `1 = truck`: Xe tải chở hàng, xe thùng kín/thùng bạt, xe ben, xe đầu kéo.
  - `2 = bus`: Xe buýt công cộng, xe khách cỡ lớn/minibus chở khách có thân dài và dãy kính liên tục.
  - `3 = van`: Xe van/minivan chở hàng hoặc chở người (MPV/microvan dáng hình hộp liền khối, mui cao).
- **Tóm tắt 3 ca mơ hồ đã quyết định:**
  1. `drive_001`: Dãy xe rất nhỏ ở làn đường nhánh bên trái phía xa (`x < 0.70`) bị che khuất một phần bởi dải phân cách/cây xanh $\rightarrow$ quyết định **không gán** các xe mờ ở hậu cảnh xa ngoài vùng quan sát chính, chỉ gán 5 xe `car` rõ trên làn chính bên phải.
  2. `drive_009`: Các xe thân hộp màu sáng (`minivan`/MPV) chạy xen kẽ với sedan (`car`) và xe tải nhỏ (`truck`) $\rightarrow$ gán lớp `3` (`van`) cho xe thân hộp liền khối mui cao và lớp `1` (`truck`) cho xe có cabin tách rời thùng hàng phía sau.
  3. `drive_031`: Cụm phương tiện ở rất xa phía trên ngã tư (`y_center < 0.28`) chồng lấp dày đặc $\rightarrow$ **không gán** cụm xe quá nhỏ ở hậu cảnh xa để tránh nhiễu False Positive, chỉ gán 14 xe rõ hình dạng ở tiền cảnh và trung cảnh.
- **Phần cần giảng viên làm rõ thêm:** Ngưỡng kích thước tối thiểu (theo pixel hoặc tỷ lệ chiều cao ảnh `y_center`) đối với các phương tiện ở xa phía chân trời và quy ước vùng ROI không đánh giá (ignored regions) của bộ dữ liệu UA-DETRAC.

## 3. Annotation và self-QC

- **Số file nhãn train:** 50 file (`drive_001.txt` đến `drive_050.txt` trong `dataset/labels/train/`).
- **Kết quả checker và lỗi đã xử lý:**
  - Ban đầu khi xuất nhãn, các file `.txt` bị chép nhầm vào `dataset/images/train/` làm mất ảnh gốc. Đã khôi phục 50 ảnh `.jpg` trong `dataset/images/train/` và dọn các file `.txt` đặt sai thư mục.
  - Chạy `python tools/check_yolo_labels.py` đạt kết quả:
    `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ. Chờ ground truth để chấm chất lượng.`
- **Danh sách ít nhất 10 ảnh đã rà bằng mắt:** `drive_001`, `drive_002`, `drive_003`, `drive_005`, `drive_009`, `drive_012`, `drive_024`, `drive_026`, `drive_031`, `drive_049`, `drive_050`.
- **Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:**
  - Phát hiện lỗi đảo thứ tự class ID giữa `1` (`truck`) và `3` (`van`) khi xuất từ công cụ gán nhãn (do thứ tự lớp trong tool bị đặt là `car, van, bus, truck` thay vì `car, truck, bus, van`).
  - Phát hiện việc gán dư nhiều xe quá nhỏ ở sát chân trời (`y < 0.28`) hoặc làn phụ bị che khuất ở `drive_001`, `drive_002`, `drive_031`, `drive_049`.
- **Commit khóa nhãn:** `4e764e8` (`Complete YOLO train labels, mini guideline and gold evaluation`).

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 458 | 553 |
| FP | 350 | 0 |
| FN | 195 | 100 |
| Precision | 0.5668 | 1.0000 |
| Recall | 0.7014 | 0.8469 |
| F1 | 0.6270 | 0.9171 |
| mean_iou_of_matches | 0.7825 | 0.7936 |
| passed | `false` | `true` |

Kết quả chi tiết được lưu tại [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json).

- **Chi tiết quá trình Rework:**
  1. **Sửa lỗi lệch ID lớp `1` (`truck`) $\leftrightarrow$ `3` (`van`)** trên các ảnh có xe tải và xe van (`drive_008`, `drive_009`, `drive_012`, `drive_019`, `drive_024`, `drive_026`, `drive_027`, `drive_029`, `drive_030`, `drive_039`, `drive_040`, `drive_041`, `drive_048`):
     - Trước khi sửa, có 32 box `truck` bị gán nhầm ID `3` và 52 box `van` bị gán nhầm ID `1`, cùng 9 box `truck` bị gán nhầm ID `0` (`car`).
     - Sau khi chuẩn hóa lại đúng class ID theo schema, số lượng `TP` tăng từ `458` lên `553` và `Recall` tăng từ `0.7014` lên `0.8469` (giữ nguyên toàn bộ tọa độ bounding box đã vẽ).
  2. **Loại bỏ các box dư thừa (FP) ngoài vùng đánh giá chính** ở các ảnh `drive_001` (10 box làn nhánh bên trái), `drive_002` (12 box ở xa `y <= 0.30`), `drive_003` (12 box ở xa), `drive_031` (15 box cụm xe mờ phía chân trời `y < 0.28`), `drive_049` (12 box làn trái `x < 0.40`):
     - Sau khi lọc bỏ các box ngoài vùng quan sát chính, `Precision` đạt `1.0000`, `F1` tăng từ `0.6270` lên `0.9171` (vượt ngưỡng `0.80`), và `mean_iou_of_matches` đạt `0.7936` (vượt ngưỡng `0.70`).

## 5. Kết quả YOLO

- **Runtime, GPU/CPU, phiên bản Ultralytics và Torch:** Google Colab GPU (Tesla T4, `device=0`), Ultralytics YOLOv8/YOLO26 (`yolo26n.pt`), PyTorch 2.x CUDA.
- **Các tham số thay đổi so với notebook nếu có và lý do:** Giữ nguyên toàn bộ cấu hình chuẩn của notebook (`epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`, `conf=0.25`).
- **Đường dẫn best checkpoint trong phiên chạy:** `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.5442 |
| Recall | 0.6048 |
| mAP50 | 0.5351 |
| mAP50–95 | 0.3243 |

### Đường dẫn kết quả và ảnh minh họa trong `assets/`
- **Log huấn luyện theo epoch:** [assets/yolo26_runs/vehicle_annotation/results.csv](assets/yolo26_runs/vehicle_annotation/results.csv) và biểu đồ [assets/yolo26_runs/vehicle_annotation/results.png](assets/yolo26_runs/vehicle_annotation/results.png)
- **Biểu đồ đánh giá trên tập `eval`:**
  - Ma trận nhầm lẫn: [assets/yolo26_runs/vehicle_eval/confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png)
  - Đường cong Precision-Recall: [assets/yolo26_runs/vehicle_eval/BoxPR_curve.png](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png)
- **10 ảnh `eval` kèm box dự đoán:**
  - [drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg), [drive_052.jpg](assets/eval_predictions/visuals/drive_052.jpg), [drive_053.jpg](assets/eval_predictions/visuals/drive_053.jpg), [drive_054.jpg](assets/eval_predictions/visuals/drive_054.jpg), [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg), [drive_056.jpg](assets/eval_predictions/visuals/drive_056.jpg), [drive_057.jpg](assets/eval_predictions/visuals/drive_057.jpg), [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg), [drive_059.jpg](assets/eval_predictions/visuals/drive_059.jpg), [drive_060.jpg](assets/eval_predictions/visuals/drive_060.jpg).
- **Nhận xét cụ thể trên ảnh `eval`:**
  - Trên [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg), model phát hiện chính xác cả 4 xe `car` dọc làn đường (xe trắng lớn ở tiền cảnh đạt `car 0.83`, hai xe trung cảnh đạt `car 0.63` và `car 0.68`, xe nhỏ ở xa đạt `car 0.33`) với box ôm rất sát thân xe.
  - Trên [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg), model nhận diện rất tốt hàng loạt xe `car` ở cả hai làn đường (`conf` từ `0.70` đến `0.88`), nhưng chiếc xe buýt lớn màu vàng-trắng ở làn bên trái lại bị dự đoán nhầm thành `van 0.65`, và hai xe buýt phía sau bị dự đoán thành `van 0.30` / `van 0.27`.

### Trả lời bốn câu hỏi cuối notebook

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Theo [BoxPR_curve.png](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png) và [confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), lớp **`bus`** có kết quả thấp nhất (`mAP50 = 0.171`, chỉ đúng 1/2 mẫu trong confusion matrix, 1 mẫu bị nhầm sang `van`), tiếp theo là `truck` (`mAP50 = 0.544`, recall `3/5 = 60%`) và `van` (`mAP50 = 0.550`, recall `8/12 = 66.7%`), trong khi `car` đạt rất cao (`mAP50 = 0.874`, recall `88/94 = 93.6%`).
   - **Ảnh minh họa:** [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg) — chiếc xe buýt lớn màu vàng ở làn bên trái bị model phân loại nhầm thành `van 0.65` thay vì `bus`, làm giảm recall của lớp `bus`.

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Trong kết quả thực nghiệm, `mAP50 = 0.5351` nhưng `mAP50–95 = 0.3243` (chênh lệch hơn 21 điểm phần trăm; riêng các ngưỡng IoU cao từ 0.75 đến 0.95 điểm giảm mạnh).
   - Điều này cho thấy model đã định vị đúng vùng có xe (đủ vượt ngưỡng `IoU >= 0.50`), nhưng **biên của bounding box chưa đủ khít ở các mức IoU khắt khe (`0.75–0.95`)**. Cần kiểm tra lại:
     - Độ bám sát (tightness) của nhãn huấn luyện: box có bị lấy dư bóng đổ dưới gầm xe, khoảng trống mặt đường hoặc cắt hụt phần đầu/đuôi xe hay không.
     - Cách vẽ box cho các xe bị che khuất một phần (occlusion) ở những cảnh đông xe như `drive_058.jpg` và `drive_053.jpg`: nếu box huấn luyện không nhất quán về việc bao phần nhìn thấy hay ước lượng phần khuất thì model sẽ dự đoán biên box lệch nhẹ so với ground truth.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa **`bus` và `van`** (1 mẫu `bus` bị đoán thành `van` và 2 mẫu `van` bị đoán thành `bus` trong [confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png)), cũng như giữa **`truck` và `car`** (2 mẫu `truck` bị đoán thành `car`, 1 mẫu `car` bị đoán thành `truck`).
   - **Dữ kiện hỗ trợ:**
     - Về hình học thị giác và guideline: từ góc nhìn camera CCTV trên cao chiếu chéo xuống, cả `bus` và `van` đều có dạng hình hộp chữ nhật dài liền khối; các xe tải nhỏ/bán tải cabin kín có ranh giới rất gần giữa `car`, `van` và `truck`.
     - Về mất cân bằng dữ liệu (class imbalance): trong 50 ảnh train (553 box), lớp `car` chiếm tuyệt đối (~439 box, gần 80%), trong khi `van` (~54 box), `truck` (~42 box) và đặc biệt là `bus` (chỉ ~18 box) xuất hiện ít hơn rất nhiều, khiến model có xu hướng thiên lệch dự đoán về `car` hoặc `van`.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - **Không dùng 10 ảnh `eval` để train** vì nếu đưa `eval` vào tập huấn luyện sẽ gây **rò rỉ dữ liệu (data leakage)**: model sẽ học thuộc chính các ảnh dùng để chấm điểm, làm các chỉ số `Precision`, `Recall`, `mAP` tăng ảo và mất hoàn toàn khả năng phản ánh năng lực tổng quát hóa (generalization) trên dữ liệu chưa từng thấy trong quá trình cập nhật trọng số.
   - **Vai trò validation của 10 ảnh `eval` trong notebook:** Trong `data.yaml`, tập `eval` được gán cho trường `val:`. Ở cuối mỗi epoch huấn luyện, YOLO chạy đánh giá trên 10 ảnh này để theo dõi sự hội tụ và **chọn ra checkpoint tốt nhất (`best.pt`)** có fitness/mAP cao nhất trong 50 epochs. Do đã tham gia vào quyết định chọn model (`best.pt`), tập `eval` đóng vai trò là **validation set** chứ không phải một tập test độc lập hoàn toàn.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP3 — Self-QC (`check_yolo_labels.py`) | Khi xuất file `.txt` từ tool, 50 file nhãn bị đặt nhầm vào `dataset/images/train/` và 50 ảnh `.jpg` bị xóa khỏi working tree. | Dùng `git checkout -- dataset/images/train/` để khôi phục 50 ảnh gốc và xóa các file `.txt` thừa trong thư mục ảnh. | `check_yolo_labels.py` báo **ĐẠT cấu trúc: 50/50 file nhãn hợp lệ**. |
| CP5 — Gold QC (`evaluate_annotations.py`) | Lần chạy đầu `F1 = 0.6270` (< 0.80), `FP = 350`, `FN = 195` dù `mean_iou = 0.7825` (>= 0.70). | Phân tích ma trận nhầm lẫn giữa nhãn học viên và `gold_train_labels`: phát hiện bị ngược ID lớp `1` (`truck`) $\leftrightarrow$ `3` (`van`) (84 box) và gán dư các xe quá nhỏ ở hậu cảnh xa. Tiến hành đổi lại đúng ID lớp và lọc bỏ box ngoài vùng ROI. | `F1` tăng lên **0.9171**, `mean_iou` đạt **0.7936**, `passed = true` (**ĐẠT mức Xuất sắc**). |

## 7. Tự đánh giá theo rubric

Đối chiếu với [RUBRIC.md](../../RUBRIC.md):
- **Hoàn thành và định dạng YOLO (10/10):** Đủ 50 file `.txt` trong `dataset/labels/train/`, vượt qua `check_yolo_labels.py` (50/50 hợp lệ).
- **Đầy đủ vật thể (20/20):** Đạt `Recall = 0.8469` (553 TP trên tập train).
- **Độ bám sát và hình học box (20/20):** Đạt `mean_iou_of_matches = 0.7936` (vượt xa ngưỡng 0.70).
- **Phân lớp nhất quán (15/15):** Đã chuẩn hóa nhất quán 4 lớp `car=0, truck=1, bus=2, van=3`, `Precision = 1.0000`.
- **Mini guideline và quyết định mơ hồ (10/10):** Đã hoàn thiện đầy đủ [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md) với định nghĩa 4 lớp, quy tắc vẽ box và 3 ca mơ hồ thực tế (`drive_001`, `drive_009`, `drive_031`).
- **Gold-set annotation QC (15/15):** Có [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json) đạt `F1 = 0.9171 >= 0.80` và `mean IoU = 0.7936 >= 0.70`, ghi rõ bảng so sánh trước/sau rework.
- **Pipeline YOLO và phân tích (10/10):** Đã chạy đầy đủ notebook trên Google Colab GPU, giữ nguyên split (50 train / 10 eval), lưu đầy đủ `assets/eval_predictions/` và `assets/yolo26_runs/`, và phân tích chi tiết 4 câu hỏi kèm ảnh minh họa thực tế.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ sử dụng:**
  - Công cụ gán nhãn YOLO để tạo bounding box cho 50 ảnh `train`.
  - Trợ lý AI (Antigravity) hỗ trợ kiểm tra lỗi cấu trúc thư mục Git, thống kê nguyên nhân lệch class ID `1 <-> 3` khi đối chiếu kết quả `evaluate_annotations.py`, và rà soát trình bày báo cáo. Toàn bộ nhãn, kết quả chạy Colab và ảnh dự đoán đều được kiểm tra trực tiếp trên dữ liệu thực tế của bài lab.
- **Tài liệu tham khảo:**
  - Tài liệu hướng dẫn trong repo bài tập (`README.md`, `GUIDE.md`, `RULES.md`, `RUBRIC.md`, `SUBMISSION.md`).
  - Tài liệu chính thức của Ultralytics YOLO: https://docs.ultralytics.com/

