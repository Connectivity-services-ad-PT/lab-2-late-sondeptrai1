# Event Contract sơ bộ — dùng cho dependency Queue async

> File này chỉ dùng cho các cặp Queue async ở Lab 02 để ghi nhận thỏa thuận ban đầu. Đặc tả chi tiết bằng AsyncAPI sẽ chuyển sang Lab 03.

## 1. Thông tin dependency

- Dependency số: 05
- Producer: Nhóm A1 (IoT Ingestion)
- Consumer: Nhóm A6 (Core Business)
- Cơ chế: Queue async
- Event/topic dự kiến: `sensor.reading.created` và `sensor.threshold.exceeded`
- Người ghi: Nhóm A6
- Ngày: 24/05/2026

## 2. Mục đích nghiệp vụ

Nhóm A1 thu thập dữ liệu và cảnh báo từ các cảm biến IoT, sau đó đẩy sự kiện vào hệ thống hàng đợi. Nhóm A6 (Core Business) đọc các sự kiện này để xử lý quy tắc nghiệp vụ (Rule Engine) và ra quyết định hành động như tạo cảnh báo an ninh (Alerts) hoặc điều khiển hệ thống toà nhà tự động.

## 3. Event name / topic

| Mục | Giá trị |
|---|---|
| Event name | `campus.iot.sensor.threshold.exceeded` |
| Topic/queue | `campus.iot.alerts.v1` |
| Producer | `service-iot-ingestion` (A1) |
| Consumer | `service-core-business` (A6) |

## 4. Payload tối thiểu

```json
{
  "eventId": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "eventType": "campus.iot.sensor.threshold.exceeded",
  "occurredAt": "2026-05-24T08:35:00Z",
  "correlationId": "tx-1234-xyz",
  "source": "service-iot-ingestion",
  "data": {
    "deviceId": "SENSOR-SMOKE-01",
    "sensorType": "smoke_detector",
    "value": 450.5,
    "threshold": 300.0,
    "unit": "ppm",
    "severity": "HIGH"
  }
}
```

## 5. Ràng buộc cần thống nhất

| Vấn đề | Quyết định tạm thời |
|---|---|
| Event id có bắt buộc không? | Có (UUID) |
| Có cần correlationId không? | Có |
| Có cho phép gửi trùng event không? | Có, Core Business phải dùng Idempotency filter bằng `eventId` |
| Thời gian sống của sự kiện (TTL) | Core Business sẽ tự drop các event `occurredAt` quá 5 phút so với hiện tại để tránh kích hoạt cảnh báo muộn. |
| Dead-letter queue | A6 cấu hình DLQ |

## 6. Issue chuyển sang Lab 03

1. Viết AsyncAPI Schema chi tiết cho các enum của `sensorType` và `severity`.
2. Định nghĩa cấu hình Retry-policy (Exponential Backoff).
