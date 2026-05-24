# Phân tích yêu cầu — vai Provider

- Cặp đàm phán: 05 (IoT Ingestion ↔ Core Business)
- Product: Smart Campus
- Provider service: IoT Ingestion (A1)
- Consumer service: Core Business (A6)
- Người viết: Nhóm A1
- Ngày: 24/05/2026

---

## 1. Resource chính (Event Stream)

| Resource | Mô tả | Thuộc tính bắt buộc | Thuộc tính tùy chọn |
|---|---|---|---|
| `Sensor Reading Event` | Dữ liệu đo lường liên tục từ cảm biến | `deviceId`, `sensorType`, `value`, `unit`, `timestamp` | `batteryLevel` |
| `Threshold Exceeded Event` | Sự kiện phát ra khi một chỉ số vượt ngưỡng an toàn do hệ thống cấu hình sẵn | `deviceId`, `sensorType`, `value`, `threshold`, `timestamp` | `severity` |

---

## 2. Action/API dự kiến (Topic Message)

*Lưu ý: Cơ chế Queue Async.*

| Topic | Event | Mục đích | Consumer (A6) gọi khi nào? |
|---|---|---|---|
| `campus.iot.sensor.v1` | `sensor.reading.created` | Báo cáo số liệu đo đạc định kỳ | A6 subscribe để giám sát hoặc chạy rule engine |
| `campus.iot.alerts.v1` | `sensor.threshold.exceeded` | Cảnh báo bất thường từ phần cứng | A6 subscribe để tạo Alert hoặc kích hoạt policy |

---

## 3. Error case (Phía Provider)

| Lỗi / Tình huống | Hướng xử lý của Provider (A1) | Gửi cho Consumer không? |
|---|---|---|
| Payload thiết bị gửi lên bị lỗi | Drop event, ghi log ở Gateway | Không |
| Mất kết nối Message Broker | Lưu buffer tạm, retry khi có mạng | Gửi trễ (Delayed) |
| Cảm biến báo lỗi phần cứng | Phát event `device.error` | Có |

---

## 4. Giả định bổ sung

- Giả định 1: Các thiết bị IoT có đồng hồ nội bộ. Nếu thiết bị không gửi kèm thời gian, A1 sẽ gán thời gian nhận (`ingestedAt`).
- Giả định 2: A1 không chứa thông tin toà nhà/vị trí. Việc mapping thiết bị nằm ở phòng nào là do Core Business (A6) quản lý.

---

## 5. Câu hỏi cho Consumer (A6)

1. Các bạn (Core Business) muốn nhận tất cả luồng dữ liệu (reading.created) hay chỉ muốn nhận khi có sự cố vượt ngưỡng (threshold.exceeded) để giảm tải?
2. Nếu cảm biến chập chờn, phát cảnh báo vượt ngưỡng liên tục trong 1 phút, Core Business có cơ chế chống spam (debounce) chưa hay A1 phải tự làm?
3. Core xử lý event trễ quá bao lâu thì bỏ qua (VD: event xảy ra hôm qua nhưng nay mới có mạng gửi lại)?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| A6 áp dụng sai Rule Engine do nhầm đơn vị đo | Kích hoạt báo cháy giả | A1 luôn gửi kèm trường `unit` rõ ràng |
