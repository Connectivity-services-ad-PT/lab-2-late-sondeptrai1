# Phân tích yêu cầu — vai Consumer

- Cặp đàm phán: 06 (IoT Ingestion ↔ Analytics)
- Product: Smart Campus
- Consumer service: Analytics (A5)
- Provider service: IoT Ingestion (A1)
- Người viết: Nhóm A5
- Ngày: 24/05/2026

---

## 1. Resource Consumer cần nhận/gửi

| Resource | Consumer dùng để làm gì? | Field bắt buộc với Consumer | Field có thể tùy chọn |
|---|---|---|---|
| `Telemetry Event` | Tổng hợp, vẽ biểu đồ và phát hiện bất thường theo thời gian | `deviceId`, `metricType`, `value`/`smokeLevel`, `unit` | `batteryLevel` |
| `Device Status Event` | Loại bỏ dữ liệu của thiết bị OFFLINE/MAINTENANCE khỏi thống kê | `deviceId`, `status` | `errorCode`, `batteryLevel` |

---

## 2. API Consumer cần gọi

| Method | Path | Lúc nào gọi? | Kỳ vọng response |
|---|---|---|---|
| GET | `/health` | Trước khi bắt đầu phiên đồng bộ, kiểm tra Gateway còn sống | `200` + `status: ok` |
| GET | `/telemetry/latest?deviceId={id}` | Khi cần đối chiếu/backfill bản ghi mới nhất của 1 thiết bị | `200` + `TelemetryPayload` (oneOf theo `metricType`) |
| POST | `/telemetry` | (Chỉ dùng khi test/replay) đẩy lại 1 bản ghi để kiểm thử pipeline | `202` + `eventId` |
| POST | `/devices/{deviceId}/status` | Không trực tiếp gọi; chỉ đọc trạng thái để lọc dữ liệu | `200` |

> Luồng chính của A5 là **subscribe** stream telemetry do A1 đẩy vào Message Broker. Các REST endpoint ở trên dùng cho health-check, backfill và kiểm thử.

---

## 3. Error case Consumer cần xử lý

Tối thiểu 5 case.

| Status | Consumer hiểu là gì? | Consumer sẽ xử lý thế nào? |
|---:|---|---|
| 400 | Request sai schema (sai `deviceId` pattern, thiếu field) | Log lỗi, sửa query/payload, không retry mù |
| 401 | Thiếu hoặc hết hạn Bearer token | Refresh token rồi gọi lại |
| 404 | Không có bản ghi telemetry cho `deviceId` | Hiển thị "chưa có dữ liệu", bỏ qua thiết bị trong batch |
| 422 | JSON đúng cấu trúc nhưng sai logic nghiệp vụ | Đọc `detail` trong Problem, ghi vào báo cáo data-quality |
| 5xx | Gateway/Broker lỗi tạm thời | Backoff + retry có giới hạn, sau đó đẩy vào DLQ phía consumer |

---

## 4. Giả định bổ sung

- Giả định 1: Mọi event telemetry đều kèm `unit` rõ ràng, A5 không tự suy đoán đơn vị đo.
- Giả định 2: A1 đã validate và drop dữ liệu rác ở cổng Ingestion, nên dữ liệu A5 nhận luôn đúng JSON Schema đã thống nhất.
- Giả định 3: A5 tự ánh xạ `deviceId → zoneId/buildingId` bằng danh mục thiết bị lấy từ Core Business.

---

## 5. Câu hỏi cho Provider

1. Payload telemetry có thể tới dưới dạng mảng (batch) hay luôn là object đơn lẻ? Kích thước batch tối đa là bao nhiêu?
2. Khi thiết bị chuyển OFFLINE, A1 có gửi event `device.status` riêng để A5 ngừng tính dữ liệu của thiết bị đó không?
3. `batteryLevel` có thể là `null` (thiết bị cắm điện) — A5 nên hiểu `null` là "không áp dụng" hay "chưa đo được"?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Provider đổi kiểu dữ liệu (`value` number → string) | Consumer parse lỗi, biểu đồ sai | Chốt type/format/pattern trong contract + version hợp đồng |
| Provider thiếu mã lỗi chuẩn | Consumer khó phân loại lỗi | Chuẩn hóa Problem Details (`type`, `title`, `status`, `detail`) |
| `metricType` mới xuất hiện ngoài enum đã chốt | A5 không nhận diện được sensor mới | Dùng `oneOf`+`discriminator`, A5 bỏ qua type lạ và cảnh báo, không crash |
