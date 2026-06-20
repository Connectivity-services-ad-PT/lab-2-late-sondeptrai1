# VERSIONING

## Contract version

- Phiên bản hiện tại: `1.0.0`
- Mỗi thay đổi breaking trên OpenAPI contract sẽ tăng major version.
- Thêm endpoint hoặc thêm field optional sẽ tăng minor version.
- Chỉ sửa lỗi tài liệu hoặc examples không ảnh hưởng contract functional sẽ tăng patch.

## Quy trình nâng cấp

1. Thảo luận Consumer/Provider và chốt trong `negotiation-log.md`.
2. Cập nhật `openapi.yaml` theo hợp đồng mới.
3. Chạy `npm run lint` và `npm run lint:report`.
4. Chạy Prism mock và kiểm thử request.
5. Commit với message: `chore(contract): camera-ai-vision v1.0 signed-off`

## Lịch sử phiên bản

- `1.0.0` — Initial contract cho Camera Stream → AI Vision.
  - Pair: Camera + AI Vision
  - Endpoints: /health, /vision/detect, /vision/detections/{id}, /vision/models/info
  - Signed off: 2026-06-20
  - Spectral lint: PASS
  - Mock server: PASS (5 request samples tested)
