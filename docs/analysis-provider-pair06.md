# Phân tích yêu cầu — vai Provider

- Cặp đàm phán: 06 (IoT Ingestion ↔ Analytics)
- Product: Smart Campus
- Provider service: IoT Ingestion (A1)
- Consumer service: Analytics (A5)
- Người viết: Nhóm A1
- Ngày: 24/05/2026

---

## 1. Resource chính (Event Stream)

| Resource | Mô tả | Thuộc tính bắt buộc | Thuộc tính tùy chọn |
|---|---|---|---|
| `Telemetry Event` | Dữ liệu đo lường định kỳ từ cảm biến | `deviceId`, `metric`, `value`, `timestamp` | `batteryLevel` |
| `Device Status Event` | Sự thay đổi trạng thái của cảm biến | `deviceId`, `status` (ONLINE/OFFLINE), `timestamp` | `errorCode` |

---

## 2. Action/API dự kiến (Topic Message)

*Lưu ý: Vì đây là Queue Async nên Method/Path được thay bằng Message Topic.*

| Topic | Event | Mục đích | Consumer (A5) gọi khi nào? |
|---|---|---|---|
| `campus.iot.telemetry.v1` | `telemetry.ingested` | Cung cấp dữ liệu đo lường thô | A5 subscribe để nhận liên tục |
| `campus.iot.device.v1` | `device.status_changed` | Báo cáo trạng thái thiết bị | A5 subscribe để bỏ qua dữ liệu thiết bị hỏng |

---

## 3. Error case (Phía Provider)

*Lưu ý: Đối với Queue Async, các lỗi chủ yếu nằm ở khâu validate dữ liệu đầu vào hoặc lỗi broker.*

| Lỗi / Tình huống | Hướng xử lý của Provider (A1) | Gửi cho Consumer không? |
|---|---|---|
| Payload từ thiết bị sai định dạng | Drop event, ghi log lỗi ở Gateway | Không |
| Mất kết nối Message Broker | Retry đẩy event lên broker, nếu quá hạn thì drop | Không (hoặc trễ) |
| Lỗi dữ liệu logic (nhiệt độ > 1000) | Gắn cờ (flag) `isAnomalous: true` | Có, Consumer tự quyết định |

---

## 4. Giả định bổ sung

Ghi rõ những điểm user story chưa nói nhưng Provider cần giả định.

- Giả định 1: Các thiết bị IoT có khả năng tự gán timestamp (thời gian đo thực tế). Nếu không có, A1 sẽ lấy giờ hệ thống lúc tiếp nhận.
- Giả định 2: A1 sẽ không map `deviceId` sang `zoneId` hay `buildingId`, việc này A5 tự thực hiện với Core.
- Giả định 3: A1 có thể gộp (batch) nhiều bản ghi telemetry vào một mảng để tối ưu đường truyền.

---

## 5. Câu hỏi cho Consumer (A5)

1. Các bạn có xử lý được payload dạng mảng (Array of events) hay yêu cầu chúng tôi bóc tách thành từng event đơn lẻ?
2. Tần suất cập nhật (throughput) có thể lên đến hàng ngàn event/giây, hệ thống của các bạn có đảm bảo không bị thắt cổ chai không?
3. Các bạn có cần thông tin về mức pin (`batteryLevel`) của cảm biến để làm thống kê bảo trì không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| A5 hiểu sai đơn vị đo lường | Báo cáo/thống kê bị sai lệch | A1 sẽ gửi kèm trường `unit` trong mọi event |
| Lưu lượng event quá lớn (Spike) | Consumer (A5) bị lag | A5 phải cấu hình Auto-scaling consumer và A1 gom batch |
