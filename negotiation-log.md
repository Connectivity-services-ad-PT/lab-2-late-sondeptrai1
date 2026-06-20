# Biên bản đàm phán hợp đồng API

- Cặp đàm phán: Camera Stream → AI Vision
- Product: Camera Motion Detection / Campus AI Vision
- Provider: AI Vision
- Consumer: Camera Stream
- Phiên: v1.0
- Ngày: 2026-06-20

---

## Issue #1

- Raised by: Provider
- Endpoint: POST /vision/detect
- Concern: Consumer muốn gửi cả raw frame và metadata, nhưng provider cần kiểm soát kích thước payload.
- Proposal: Dùng `oneOf` với `payloadType` và hỗ trợ `IMAGE_FRAME` hoặc `IMAGE_METADATA`.
- Resolution: Accepted
- Rationale: Giữ contract linh hoạt và cho phép consumer chuyển sang metadata nếu ảnh quá lớn.
- Impact: Provider cần validate cả hai biến thể, consumer phải gửi `payloadType` rõ ràng.

---

## Issue #2

- Raised by: Consumer
- Endpoint: POST /vision/detect
- Concern: Consumer cần nhận response nhanh với detectionId và confidence.
- Proposal: Response trả `detectionId`, `status`, `objects`, `riskLevel`, `confidence` và `modelVersion`.
- Resolution: Accepted
- Rationale: Thông tin này đủ để consumer quyết định xử lý tiếp hoặc cảnh báo.
- Impact: Provider phải trả kết quả tổng quan ngay cả khi status là PENDING.

---

## Issue #3

- Raised by: Provider
- Endpoint: POST /vision/detect
- Concern: `confidence` có thể không luôn có giá trị nếu detection chưa xong.
- Proposal: Dùng `type: [number, "null"]` cho trường `confidence`.
- Resolution: Accepted
- Rationale: Tránh ép buộc giá trị confidence khi service trả kết quả pending.
- Impact: Consumer phải xử lý giá trị null.

---

## Issue #4

- Raised by: Consumer
- Endpoint: POST /vision/detect
- Concern: Retry hoặc duplicate request có thể gây phát hiện trùng lặp.
- Proposal: Thêm field `requestId` và trả 409 nếu duplicate.
- Resolution: Accepted
- Rationale: Hỗ trợ idempotency và giảm lỗi do retry từ camera stream.
- Impact: Provider cần lưu `requestId` đã xử lý, consumer cần tạo UUID duy nhất cho mỗi request.

---

## Issue #5

- Raised by: Provider
- Endpoint: GET /vision/models/info
- Concern: Consumer cần biết model hỗ trợ định dạng ảnh và version.
- Proposal: Thêm endpoint model info trả `supportedFormats` và `version`.
- Resolution: Accepted
- Rationale: Consumer có thể kiểm tra khả năng tương thích trước khi gửi payload.
- Impact: Tăng tính minh bạch về phiên bản model.

---

## Issue #6

- Raised by: Consumer
- Endpoint: Tất cả lỗi 4xx/5xx
- Concern: Consumer cần xử lý lỗi đồng nhất.
- Proposal: Dùng `application/problem+json` và schema `Problem` cho tất cả lỗi.
- Resolution: Accepted
- Rationale: Chuẩn hóa cách parse lỗi và tránh hiểu nhầm giữa client/server.
- Impact: Provider phải trả Problem Details với `type`, `title`, `status`.

---

# Chốt hợp đồng v1.0

Provider sign-off: AI Vision Team
Consumer sign-off: Camera Stream Team
Witness (GV/TA):  
Date: 2026-06-20

---

## Ghi chú warning nếu Spectral còn cảnh báo

| Warning | Lý do chấp nhận tạm thời | Kế hoạch sửa |
|---|---|---|
|  |  |  |
