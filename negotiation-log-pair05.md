# Biên bản đàm phán hợp đồng API

- Cặp đàm phán: 05 (IoT Ingestion ↔ Core Business)
- Product: Smart Campus
- Provider: Nhóm A1 (IoT Ingestion)
- Consumer: Nhóm A6 (Core Business)
- Phiên: v1.0
- Ngày: 24/05/2026

---

## Issue #1

- Raised by: Consumer (A6)
- Endpoint: Queue `campus.iot.sensor.v1`
- Concern: Lượng dữ liệu raw (reading.created) từ hàng ngàn cảm biến gửi mỗi giây có thể làm quá tải (overload) hệ thống xử lý Rule Engine của Core Business.
- Proposal: Đề nghị A1 (IoT) tự cấu hình ngưỡng báo động và chỉ gửi sự kiện `sensor.threshold.exceeded` về cho A6, ngừng gửi dữ liệu thô (raw data).
- Resolution: Modified
- Rationale: Nhiệm vụ của A1 là truyền tải dữ liệu, việc quyết định ngưỡng an toàn nên nằm ở Core Business (A6). Tuy nhiên, để giảm tải, hai bên thống nhất A1 sẽ vẫn cấp cả 2 luồng dữ liệu (2 topic riêng biệt). A6 có quyền chỉ Subscribe vào topic cảnh báo (`alerts.v1`), không bắt buộc phải đọc topic dữ liệu thô.
- Impact: A6 không bị quá tải. A1 thiết kế 2 topic tách biệt.

---

## Issue #2

- Raised by: Consumer (A6)
- Endpoint: Payload của sự kiện cảm biến
- Concern: A6 cần truy xuất nhanh vị trí của thiết bị để tạo cảnh báo cháy, nếu phải gọi database để lấy `locationId` thì rất mất thời gian.
- Proposal: A1 đính kèm trường `locationId` (Mã phòng/Toà nhà) vào payload.
- Resolution: Rejected
- Rationale: Thiết bị IoT không tự biết nó đang được lắp ở đâu. Trách nhiệm lưu trữ sơ đồ toà nhà là của Core Business. Để tối ưu, A6 nên cache (Redis) bản đồ thiết bị để truy xuất nhanh, không được bắt A1 truyền dữ liệu ngoài viền.
- Impact: Payload tối giản, chỉ có `deviceId`. A6 áp dụng bộ nhớ đệm (Cache) để ánh xạ ID.

---

## Issue #3

- Raised by: Provider (A1)
- Endpoint: Độ trễ sự kiện (Delayed Events)
- Concern: Khi thiết bị mất mạng và có mạng lại, nó sẽ gửi ồ ạt các sự kiện trong quá khứ.
- Proposal: A6 cần sử dụng trường `occurredAt` trong payload thay vì giờ của hệ thống để đánh giá sự kiện, và tự quyết định xem có nên xử lý cảnh báo trễ hay không.
- Resolution: Accepted
- Rationale: Cảnh báo cháy từ 1 tiếng trước không còn giá trị kích hoạt còi báo động hiện tại. A6 sẽ cấu hình rule bỏ qua các event có `occurredAt` cũ hơn 5 phút.
- Impact: Hệ thống tránh được các cảnh báo sai (False Alerts) do độ trễ mạng.

---

# Chốt hợp đồng v1.0

Provider sign-off: Đại diện Nhóm A1  
Consumer sign-off: Đại diện Nhóm A6  
Witness (GV/TA): Giảng viên hướng dẫn  
Date: 24/05/2026

---

## Ghi chú warning nếu Spectral còn cảnh báo

| Warning | Lý do chấp nhận tạm thời | Kế hoạch sửa |
|---|---|---|
| (Không áp dụng cho Queue Async) | | |
