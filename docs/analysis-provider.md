# Phân tích yêu cầu — vai Provider

- Cặp đàm phán: Camera Stream → AI Vision
- Product: Camera Motion Detection / Campus AI Vision
- Provider service: AI Vision
- Consumer service: Camera Stream
- Người viết: Nhóm Provider
- Ngày: 2026-06-20

---

## 1. Resource chính

| Resource | Mô tả | Thuộc tính bắt buộc | Thuộc tính tùy chọn |
|---|---|---|---|
| VisionDetection | Kết quả detection của AI Vision | detectionId, status, objects, riskLevel, modelVersion | confidence, detail, receivedAt, analyzedAt |
| ModelInfo | Metadata về model AI Vision | modelId, modelName, version, supportedFormats, lastTrainedAt |  |

---

## 2. Action/API dự kiến

| Method | Path | Mục đích | Consumer gọi khi nào? |
|---|---|---|---|
| POST | /vision/detect | Gửi frame hoặc metadata để AI Vision phân tích | Khi camera phát hiện motion và muốn nhận detection nhanh |
| GET | /vision/detections/{detectionId} | Lấy chi tiết detection nếu cần polling hoặc xác nhận lại | Khi consumer cần xác nhận kết quả hoặc re-check |
| GET | /vision/models/info | Kiểm tra phiên bản model và định dạng ảnh hỗ trợ | Trước khi gửi hoặc khi cần kiểm tra tương thích |
| GET | /health | Kiểm tra service có sống hay không | Trước khi sử dụng API, hoặc health check hàng ngày |

---

## 3. Error case

Tối thiểu 5 case.

| Status | Tình huống | Response body dự kiến |
|---:|---|---|
| 400 | Payload thiếu trường bắt buộc hoặc schema sai | Problem Details với errors list |
| 401 | Thiếu Bearer token | Problem Details, type unauthorized |
| 403 | Token hợp lệ nhưng không đủ quyền | Problem Details, type forbidden |
| 404 | detectionId không tồn tại | Problem Details, type not-found |
| 409 | Duplicate requestId hoặc replay request | Problem Details, type conflict |
| 422 | Payload đúng JSON nhưng vi phạm nghiệp vụ | Problem Details, type validation |

---

## 4. Giả định bổ sung

- AI Vision provider có thể nhận cả raw frame lẫn metadata.
- Camera Stream chịu trách nhiệm cung cấp `sourceService` và `requestId` duy nhất.
- `confidence` có thể null khi kết quả chưa hoàn chỉnh hoặc đang chờ xử lý.
- `motionRegion` metadata có thể null nếu camera không xác định được vùng.

---

## 5. Câu hỏi cho Consumer

1. Camera Stream muốn gửi ảnh dưới dạng Base64 hay URL của hệ thống lưu trữ?
2. Giá trị `requestId` có cần idempotency cùng request body tương đương?
3. Consumer mong muốn liệu có trạng thái `PENDING` cho xử lý async không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Sai định dạng image URL | Provider không thể truy cập | Thống nhất URL phải là HTTPS và truy cập được trong campus network |
| Payload quá lớn | Mock hoặc thực tế timeout | Giới hạn kích thước ảnh và ưu tiên metadata khi cần |
| Base64 bị lỗi | AI Vision parse thất bại | Validator client/consumer kiểm tra trước khi gửi |
| Thiếu token | Yêu cầu thất bại 401 | Chuẩn hóa bearer token header |
| fields mapping khác nhau | Consumer parse lỗi | Chốt tên field theo OpenAPI contract |
