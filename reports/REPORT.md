## Báo cáo bài thực hành Ngày 1 – Đọc nhãn từ đầu ra YOLO11
*Ngày chạy:* 2026-11-07
*Runtime Colab:* CPU
*Python / PyTorch / Ultralytics:* 3.13.18 / 2.14.1+cpu / 8.4.145
*Checkpoint:* yolo11n-cls.pt, yolo11n.pt, yolo11n-seg.pt
*Thay đổi so với notebook nguồn:* Không

## 1. Phân loại ảnh – prediction cấp ảnh
- Record hạng 1: class_id=468, class_name="cab", rank=1, score=0.510915, taxonomy_name="ImageNet-1K"
- Mô tả: Dự đoán phân loại tổng quan cho toàn bộ bức ảnh.
- Nguồn taxonomy: ImageNet-1K do mô hình ImageNet định nghĩa.
- Lý do giữ ID và tên: Đảm bảo tính nhất quán và rõ nghĩa khi tra cứu.

## 2. Phát hiện vật thể – lớp và box cho từng object
- Record: class_name="bus", score=0.912557, bbox_xyxy=[93.17, 187.95, 223.01, 320.91]
- Diễn giải vị trí: Khung giới hạn xe buýt nằm ở góc trái giữa ảnh.

## 3. Phân đoạn theo từng đối tượng – polygon cho mỗi instance
- Record: instance_id="traffic-001", class_name="bus", score=0.925745, polygon_point_count=120

## 4. Vòng đời và kiểm tra chất lượng
| Tác vụ | Đơn vị/định dạng ground truth | Lỗi hoặc điểm mơ hồ | Annotator làm gì? | Reviewer xem gì? |
| --- | --- | --- | --- | --- |
| Phân loại ảnh | Image-level class | Nhầm lẫn giữa các loại xe | Gán nhãn lớp chính | Kiểm tra độ chính xác nhãn |
| Phát hiện vật thể | Bounding box | Vật thể bị che khuất | Vẽ box bao trọn vật thể | Kiểm tra độ khít của box |
| Instance segmentation | Polygon mask | Biên vật thể phức tạp | Vẽ điểm đa giác chi tiết | Kiểm tra độ mịn của biên |

## 5. An toàn dữ liệu
- Quy tắc bảo vệ: Không đưa dữ liệu nhạy cảm hay thông tin cá nhân lên môi trường chung.
- Báo cáo sự cố: Dừng xử lý và báo cáo mentor nếu phát hiện dữ liệu bất thường.

## 6. Danh sách bằng chứng
- [x] classification_predictions.json
- [x] detection_predictions.json
- [x] segmentation_predictions.json
- [x] IMAGE_ATTRIBUTION.md
- [x] visuals/classification_top5.png
- [x] visuals/detection_predictions.png
- [x] visuals/segmentation_prediction.png
- [x] Ô validation cuối notebook báo PASS.
- [x] Không có họ tên, MSSV hoặc dữ liệu nhạy cảm trong báo cáo/output.
