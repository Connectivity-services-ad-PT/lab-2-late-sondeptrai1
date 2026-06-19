# Known Issues — Lab 02

Ghi các lỗi còn tồn tại nếu chưa xử lý xong.

| Lỗi | Ảnh hưởng | Cách xử lý dự kiến | Người phụ trách |
|---|---|---|---|
| Chưa có đặc tả AsyncAPI đầy đủ cho topic `campus.iot.telemetry.v1` | Lab 02 chỉ có event contract sơ bộ, chưa validate được message theo schema broker | Viết AsyncAPI 3.0 + JSON Schema cho payload ở Lab 03 | Nhóm A1 |
| Cơ chế bảo mật khi kết nối Message Broker chưa định nghĩa | Chưa rõ cách xác thực producer/consumer với broker | Thống nhất SASL/mTLS + scope topic ở Lab 03 | Nhóm A1 + A5 |
| Kịch bản replay/khôi phục từ Dead-letter Queue chưa mô tả | Khi A5 lỗi và message rơi vào DLQ, chưa có quy trình xử lý lại | Thiết kế consumer replay từ DLQ ở Lab 03 | Nhóm A5 |

> Lưu ý: `spectral lint openapi.yaml` hiện **0 error / 0 warning** — không có lỗi tồn đọng ở phần REST contract.
