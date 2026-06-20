# Phân tích yêu cầu — vai Consumer

- Cặp đàm phán: Camera Stream → AI Vision
- Product: Camera Motion Detection / Campus AI Vision
- Consumer service: Camera Stream
- Provider service: AI Vision
- Người viết: Nhóm Consumer
- Ngày: 2026-06-20

---

## 1. Resource Consumer cần nhận/gửi

| Resource | Consumer dùng để làm gì? | Field bắt buộc với Consumer | Field có thể tùy chọn |
|---|---|---|---|
| VisionDetectRequest | Gửi dữ liệu motion tới AI Vision | requestId, sourceService, timestamp, payload | none |
| VisionDetection | Nhận kết quả detection | detectionId, status, objects, riskLevel, modelVersion | confidence, detail |
| ModelInfo | Kiểm tra model tương thích | modelId, modelName, version, supportedFormats, lastTrainedAt | none |

---

## 2. API Consumer cần gọi

| Method | Path | Lúc nào gọi? | Kỳ vọng response |
|---|---|---|---|
| POST | /vision/detect | Khi camera phát hiện chuyển động | 201 + detectionId và kết quả ban đầu |
| GET | /vision/detections/{detectionId} | Khi cần xác nhận chi tiết hoặc polling | 200 + detection detail |
| GET | /vision/models/info | Khi cần biết model hỗ trợ định dạng hoặc version | 200 + model metadata |
| GET | /health | Trước khi gửi request hoặc khi monitor | 200 + service healthy |

---

## 3. Error case Consumer cần xử lý

| Status | Consumer hiểu là gì? | Consumer sẽ xử lý thế nào? |
|---:|---|---|
| 400 | Payload hoặc schema không hợp lệ | Kiểm tra lại request body, sửa lỗi field |
| 401 | Thiếu token hoặc token hết hạn | Đăng nhập lại, cập nhật cấu hình auth |
| 403 | Không đủ quyền | Báo lỗi quyền truy cập, yêu cầu cấu hình lại |
| 404 | detectionId không tồn tại | Hiển thị không tìm thấy hoặc retry với id khác |
| 409 | Duplicate request hoặc replay | Dừng gửi lại, kiểm tra requestId |
| 422 | Vi phạm nghiệp vụ | Hiển thị thông tin validate từ provider |

---

## 4. Giả định bổ sung

- Camera Stream có thể gửi payload metadata nếu ảnh quá lớn.
- `requestId` dùng cho idempotency và replay protection.
- `confidence` có thể là null nếu detection đang xử lý.

---

## 5. Câu hỏi cho Provider

1. Provider có cần hỗ trợ cả Base64 image và image URL không?
2. Nếu detection không thể hoàn thành, status sẽ là FAILED và consumer có retry không?
3. Model version có thay đổi thường xuyên không, và consumer nên cache không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Provider đổi định dạng payload | Consumer parse lỗi | Chốt payload schema trong OpenAPI |
| Không nhận được ảnh qua URL | Detection fail | Test URL access trong mạng campus |
| Token không đúng | Giao tiếp thất bại | Chuẩn hóa header Authorization |
| Không đồng bộ trạng thái | Consumer hiểu sai status | Dùng enum rõ ràng PENDING/COMPLETED/FAILED |
