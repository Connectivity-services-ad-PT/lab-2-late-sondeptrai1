# Event Contract sơ bộ — dùng cho dependency Queue async

> File này chỉ dùng cho các cặp Queue async ở Lab 02 để ghi nhận thỏa thuận ban đầu. Đặc tả chi tiết bằng AsyncAPI sẽ chuyển sang Lab 03.

## 1. Thông tin dependency

- Dependency số: 06
- Producer: Nhóm A1 (IoT Ingestion)
- Consumer: Nhóm A5 (Analytics)
- Cơ chế: Queue async
- Event/topic dự kiến: `iot.telemetry.ingested` và `iot.device.status_changed`
- Người ghi: Nhóm A1
- Ngày: 24/05/2026

## 2. Mục đích nghiệp vụ

Nhóm A1 thu thập dữ liệu đo lường liên tục từ các thiết bị cảm biến (nhiệt độ, độ ẩm, chuyển động,...) và đẩy vào queue. Nhóm A5 nhận dữ liệu này theo thời gian thực để tổng hợp, tính toán trung bình theo giờ/ngày và cập nhật lên Dashboard điều hành Smart Campus.

## 3. Event name / topic

| Mục | Giá trị |
|---|---|
| Event name | `campus.iot.telemetry.ingested` |
| Topic/queue | `campus.iot.telemetry.v1` |
| Producer | `service-iot-ingestion` (A1) |
| Consumer | `service-analytics` (A5) |

## 4. Payload tối thiểu

```json
{
  "eventId": "0196fb3d-4ad7-7d1e-9f49-5d5148d2babc",
  "eventType": "campus.iot.telemetry.ingested",
  "occurredAt": "2026-05-24T08:30:00Z",
  "correlationId": "tx-9981-abc",
  "source": "service-iot-ingestion",
  "data": {
    "deviceId": "SENSOR-001",
    "metric": "temperature",
    "value": 36.5,
    "unit": "celsius"
  }
}
```

## 5. Ràng buộc cần thống nhất

| Vấn đề | Quyết định tạm thời |
|---|---|
| Event id có bắt buộc không? | Có (Định dạng UUID v4) |
| Có cần correlationId không? | Có (Dùng để truy vết chuỗi hành động) |
| Có cho phép gửi trùng event không? | Có thể xảy ra do network, A5 (Consumer) phải xử lý idempotent dựa vào `eventId`. |
| Retry khi lỗi | Delegate cho Message Broker (không retry ở tầng ứng dụng của A1). Ghi rõ ở Lab 03. |
| Dead-letter queue | A5 tự cấu hình DLQ phía consumer cho các message không thể parse. |

## 6. Issue chuyển sang Lab 03

1. Định nghĩa chi tiết Data Schema bằng JSON Schema / AsyncAPI (kiểm tra kiểu dữ liệu của `value` và ràng buộc của `metric`).
2. Cơ chế bảo mật và xác thực khi kết nối vào Message Broker.
3. Kịch bản khôi phục dữ liệu từ Dead-letter Queue khi A5 khắc phục xong lỗi.
