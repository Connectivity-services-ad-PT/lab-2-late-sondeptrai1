# Checklist Lab 02

> Cặp 06 (IoT Ingestion ↔ Analytics) là **Queue async** → chấm theo Rubric B.
> Nhóm vẫn viết thêm `openapi.yaml` cho luồng REST (health/backfill/test) nên các mục REST dưới đây đều áp dụng được.

- [x] Đã xác định đúng cặp đàm phán. (Cặp 06 — IoT Ingestion A1 ↔ Analytics A5)
- [x] Đã đọc đúng user story trong thư mục `user-stories/`. (`pair-06-iot-analytics-async.md`)
- [x] Provider đã điền `docs/analysis-provider.md`.
- [x] Consumer đã điền `docs/analysis-consumer.md`.
- [x] `openapi.yaml` khai báo `openapi: 3.1.0`.
- [x] Có tối thiểu 4 path. (`/health`, `/telemetry`, `/devices/{deviceId}/status`, `/telemetry/latest`)
- [x] Mỗi operation có `operationId`, `summary`, `description`, `tags`.
- [x] Schema lớn đã đưa vào `components/schemas`.
- [x] Có `oneOf` + `discriminator`. (`TelemetryPayload` theo `metricType`)
- [x] Có union type với `null`, không dùng `nullable: true`. (`batteryLevel: [number, "null"]`)
- [x] Có `Problem` schema cho response lỗi.
- [x] `spectral lint` không có severity error. (0 error / 0 warning)
- [x] Đã lưu `evidence/buoi-02/spectral-report.txt`.
- [x] Prism mock server chạy được ở port 4010. (minh chứng trong `mock-screenshots/`)
- [x] Có 5 ảnh request mẫu trong `mock-screenshots/`.
- [x] `negotiation-log.md` có tối thiểu 6 issue.
- [x] Có sign-off Provider, Consumer, Witness.
- [x] Đã hoàn thiện `VERSIONING.md` cho bài tập về nhà.

## Event contract (Rubric B — Queue async)

- [x] Đã ghi `docs/event-contract-pair06.md` (event name, topic, payload, ràng buộc).
- [x] Đã xác định Producer/Consumer và trigger phát event.
- [x] Có payload mẫu kèm `eventId`, `occurredAt`, `correlationId`, `source`.
- [x] Đã nêu xử lý duplicate/idempotent, retry, dead-letter.
- [x] Đã liệt kê issue chuyển tiếp sang Lab 03.
