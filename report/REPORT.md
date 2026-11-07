## Báo cáo bài thực hành Ngày 1 – Đọc nhãn từ đầu ra YOLO11
*Ngày chạy:* 2026-11-07
*Runtime Colab:* CPU
*Python / PyTorch / Ultralytics:* 3.13.18 / 2.14.1+cpu / 8.4.145
*Checkpoint:* yolo11n-cls.pt, yolo11n.pt, yolo11n-seg.pt
*Thay đổi so với notebook nguồn:* Không

## 1. Phân loại ảnh – prediction cấp ảnh
Nguồn evidence: `classification_predictions.json`, sample `traffic`.
- Record hạng 1 (`class_id`: 468, `class_name`: "cab", `rank`: 1, `score`: 0.510915, `taxonomy_name`: "ImageNet-1K"):
- Record này mô tả toàn ảnh như thế nào?: Đại diện cho nhãn phân loại tổng quan nhất có độ tin cậy cao nhất cho toàn bức ảnh giao thông.
- Ai định nghĩa class list mà checkpoint có thể dự đoán?: Nhóm nghiên cứu ImageNet định nghĩa thông qua tập 1000 lớp đối tượng ImageNet-1K.
- Vì sao cần giữ cả ID, tên lớp và tên taxonomy?: Đảm bảo tính minh bạch, tránh nhầm lẫn giữa các mã số ID khác nhau ở các bộ dataset và giúp dễ dàng tra cứu nghĩa của nhãn.
- Nếu ảnh có nhiều chủ thể, guideline cần quy định điều gì?: Cần quy định rõ cách ưu tiên nhãn cho chủ thể chínhếm diện tích lớn nhất hoặc chủ thể trung tâm trong khung hình.
- Vì sao model score không phải ground truth?: Score chỉ thể hiện mức độ tự tin của mô hình AI dựa trên xác suất thống kê, không phải nhãn chuẩn do con người xác thực chính xác.

## 2. Phát hiện vật thể – lớp và box cho từng object
Nguồn evidence: `detection_predictions.json` và `visuals/detection_predictions.png`, sample `kitchen`.
- Một record (`class_name`: "bus", `score`: 0.912557, `bbox_xyxy`: [93.17, 187.95, 223.01, 320.91], `bbox_width`: 129.84, `bbox_height`: 132.96):
- Diễn giải vị trí box bằng lời: Khung giới hạn xe buýt nằm ở khu vực góc trái giữa của bức ảnh với độ tin cậy rất cao.
- So sánh số prediction ở hai threshold: Khi giảm ngưỡng score (threshold), số lượng vật thể được phát hiện tăng lên do mô hình nhận diện thêm các đối tượng có độ tự tin thấp hơn.
- Điều gì thay đổi đối với độ bao phủ và khối lượng reviewer cần xem?: Ngưỡng thấp tăng độ bao phủ (giảm bỏ sót) nhưng làm tăng mạnh khối lượng công việc kiểm tra lại (rework/review) của con người do có nhiều dự đoán sai (false positive).
- Đề xuất một quy tắc box chặt: Box phải ôm khít phần biên ngoài cùng của vật thể, không để khoảng trống quá lớn và không cắt cụt phần thân chính.
- Với object bị che khuất/cắt mép, điều gì cần guideline hoặc escalation quyết định?: Cần quy định rõ ngưỡng diện tích tối thiểu (ví dụ: che khuất trên 50% hoặc cắt mép quá nửa thì loại bỏ hoặc vẫn gán nhãn phụ thuộc vào mục đích bài toán).

## 3. Phân đoạn theo từng đối tượng – polygon cho mỗi instance
Nguồn evidence: `segmentation_predictions.json` và `visuals/segmentation_prediction.png`, sample `kitchen`.
- Một record (`instance_id`: "traffic-001", `class_name`: "bus", `score`: 0.925745, số điểm polygon: 120):
- Polygon bổ sung chi tiết gì so với box?: Cung cấp đường viền đa giác bám sát hình dạng thực tế của đối tượng thay vì chỉ là hình chữ nhật bao ngoài.
- `instance_id` dùng để làm gì và không phải loại ID nào?: Dùng để phân biệt các thực thể riêng biệt trong cùng một lớp (ví dụ xe A và xe B), đây không phải là mã định danh vĩnh viễn toàn cục (global ID) mà chỉ mang tính cục bộ trong ảnh.
- Đề xuất một quy tắc biên mask: Các đỉnh polygon phải nằm chính xác trên biên giới chuyển tiếp giữa đối tượng và nền, không lấn sâu vào trong hay tràn ra ngoài.
- Với vùng mờ/tiếp xúc/che khuất, điều gì cần guideline hoặc escalation quyết định?: Cần hướng dẫn cách ước lượng biên tại các điểm bị khuất bóng hoặc mờ nhòe, hoặc chuyển lên cấp cao hơn (escalation) nếu ranh giới quá mơ hồ.

## 4. Vòng đời và kiểm tra chất lượng
ảnh thô → guideline → ground truth → huấn luyện → prediction → QC/rework
| Tác vụ | Đơn vị/định dạng ground truth | Lỗi hoặc điểm mơ hồ quan sát được | Annotator làm gì? | Reviewer xem gì? |
| --- | --- | --- | --- | --- |
| Phân loại ảnh | Image-level class | Nhầm lẫn giữa các loại xe tương tự | Gán nhãn lớp chính xác cho ảnh | Kiểm tra độ chính xác nhãn tổng quan |
| Phát hiện vật thể | Bounding box | Vật thể bị che khuất một phần | Vẽ box bao trọn vật thể | Kiểm tra độ khít và sót vật thể của box |
| Instance segmentation | Polygon mask | Biên vật thể phức tạp, mờ | Vẽ điểm đa giác chi tiết theo biên | Kiểm tra độ mịn và chính xác của viền mask |

## 5. An toàn dữ liệu
- Một quy tắc bảo vệ dữ liệu: Tuyệt đối không đưa dữ liệu nhạy cảm, thông tin cá nhân hoặc hình ảnh độc hại lên môi trường chung hay chia sẻ ngoài phạm vi bài học.
- Nếu thấy ảnh hoặc dữ liệu không đúng phạm vi, tôi sẽ dừng và báo cho: Giảng viên phụ trách hoặc ban quản lý kỹ thuật (mentor).

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
