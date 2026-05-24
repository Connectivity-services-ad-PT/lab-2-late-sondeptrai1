# Biên bản đàm phán hợp đồng API

- Cặp đàm phán: 06 (IoT Ingestion ↔ Analytics)
- Product: Smart Campus
- Provider: Nhóm A1 (IoT Ingestion)
- Consumer: Nhóm A5 (Analytics)
- Phiên: v1.0
- Ngày: 24/05/2026

---

## Issue #1

- Raised by: Consumer (A5)
- Endpoint: Topic `iot.telemetry.ingested`
- Concern: A5 cần dữ liệu có kèm `zoneId` (khu vực) để có thể thống kê và vẽ biểu đồ theo từng tòa nhà thay vì chỉ từng thiết bị đơn lẻ.
- Proposal: A1 đính kèm trường `zoneId` vào payload của mỗi sự kiện telemetry gửi đi.
- Resolution: Rejected
- Rationale: A1 chỉ là gateway thu thập dữ liệu thô từ phần cứng (cảm biến). Các thiết bị cảm biến không lưu trữ thông tin không gian (`zoneId`). Việc ánh xạ từ `deviceId` sang `zoneId` thuộc về logic nghiệp vụ của Core Business. A5 nên lấy danh mục thiết bị từ Core Business để tự ánh xạ.
- Impact: Payload của A1 giữ nguyên sự tối giản, chỉ gửi `deviceId`. A5 phải tự xử lý logic mapping khu vực.

---

## Issue #2

- Raised by: Provider (A1)
- Endpoint: Topic `iot.telemetry.ingested`
- Concern: Cảm biến gửi dữ liệu liên tục mỗi giây. Nếu A1 đẩy từng event rời rạc vào queue, sẽ gây quá tải hệ thống message broker và hệ thống xử lý của A5.
- Proposal: A1 sẽ gom nhóm (batch) các event mỗi 10 giây hoặc 100 events trước khi đẩy vào queue.
- Resolution: Modified
- Rationale: Việc gom nhóm quá lâu (10s) có thể làm chậm quá trình cảnh báo thời gian thực của các hệ thống khác đọc cùng queue. Hai bên chốt: A1 không gom nhóm quá 2 giây, và A5 phải hỗ trợ xử lý cả 2 định dạng (single event và array of events).
- Impact: Payload có thể là một object hoặc một mảng các object. Queue throughput được tối ưu.

---

## Issue #3

- Raised by: Consumer (A5)
- Endpoint: Cơ chế Retry & Dead-letter
- Concern: Khi A5 gặp lỗi kết nối database và không thể ghi nhận telemetry, event có thể bị mất.
- Proposal: A1 cần lưu lại event và cho phép API gọi lại để lấy dữ liệu cũ (replay).
- Resolution: Rejected
- Rationale: A1 thiết kế theo mô hình "Fire and Forget" để đảm bảo hiệu năng. Trách nhiệm lưu trữ tạm thời thuộc về Message Broker (Kafka/RabbitMQ). Nếu A5 lỗi, hệ thống Broker phải tự động đưa vào Dead-letter Queue (DLQ).
- Impact: A5 tự cấu hình DLQ phía consumer. A1 không thiết kế thêm database lưu trữ event tạm thời.

---

## Issue #4

- Raised by: Provider (A1)
- Endpoint: Payload Schema
- Concern: Một số cảm biến cũ đôi khi gửi dữ liệu sai định dạng (ví dụ nhiệt độ là string thay vì number). A1 nên chuẩn hóa hay cứ đẩy thô?
- Proposal: A1 sẽ validate và drop (bỏ qua) các event sai định dạng trước khi đẩy vào Queue để làm sạch dữ liệu cho A5.
- Resolution: Accepted
- Rationale: Tránh việc data rác làm hỏng hệ thống phân tích của A5. A1 có bộ lọc (schema validation) ngay tại cổng Ingestion.
- Impact: A5 yên tâm dữ liệu nhận được luôn đúng chuẩn JSON Schema đã thống nhất.

---

# Chốt hợp đồng v1.0

Provider sign-off: Đại diện Nhóm A1  
Consumer sign-off: Đại diện Nhóm A5  
Witness (GV/TA): Giảng viên hướng dẫn  
Date: 24/05/2026

---

## Ghi chú warning nếu Spectral còn cảnh báo

| Warning | Lý do chấp nhận tạm thời | Kế hoạch sửa |
|---|---|---|
| (Không áp dụng cho Queue Async) | | |
