# Biên bản đàm phán hợp đồng API

- Cặp đàm phán: Cặp 02
- Product: A / B
- Provider: AI Vision
- Consumer: Core Business
- Phiên: v1.0
- Ngày: 18/05/2026

---

## Issue #1

- Raised by: Consumer
- Endpoint: POST /vision/face-match
- Concern: Core Business chưa biết nên gửi imageRef (URL) hay faceEmbedding (vector)
- Proposal: Dùng discriminator requestType để hỗ trợ cả hai dạng
- Resolution: Accepted
- Rationale: Linh hoạt cho nhiều use case, discriminator giúp validate rõ ràng
- Impact: Schema FaceMatchRequest dùng oneOf gồm ImageRefRequest và EmbeddingRequest

---

## Issue #2

- Raised by: Consumer
- Endpoint: POST /vision/face-match
- Concern: Ngưỡng confidence bao nhiêu thì Core coi là match hợp lệ?
- Proposal: AI Vision trả decision gồm MATCH / NO_MATCH / LOW_CONFIDENCE, Core tự quyết định ngưỡng
- Resolution: Accepted
- Rationale: AI Vision không nên hard-code ngưỡng nghiệp vụ của Core
- Impact: Field decision thêm giá trị LOW_CONFIDENCE, Core xử lý riêng case này

---

## Issue #3

- Raised by: Provider
- Endpoint: POST /vision/face-match
- Concern: Khi model không chắc chắn, trả 200 low_confidence hay 422?
- Proposal: Trả 200 với decision LOW_CONFIDENCE thay vì 422
- Resolution: Accepted
- Rationale: LOW_CONFIDENCE là kết quả hợp lệ của model, không phải lỗi nghiệp vụ
- Impact: Core phải xử lý thêm case decision = LOW_CONFIDENCE trong response 200

---

## Issue #4

- Raised by: Consumer
- Endpoint: POST /vision/face-match
- Concern: confidence dùng thang đo float 0.0-1.0 hay integer 0-100?
- Proposal: Dùng float 0.0 đến 1.0
- Resolution: Accepted
- Rationale: Chuẩn phổ biến trong ML, dễ so sánh với ngưỡng
- Impact: Schema field confidence dùng type number, minimum 0, maximum 1

---

## Issue #5

- Raised by: Consumer
- Endpoint: GET /vision/detections/{detectionId}
- Concern: modelVersion trả trong response body hay trong header?
- Proposal: Trả trong response body để dễ log và audit
- Resolution: Accepted
- Rationale: Header dễ bị bỏ qua khi log, body đảm bảo luôn có trong record
- Impact: Field modelVersion có trong schema Detection và FaceMatchResult

---

## Issue #6

- Raised by: Provider
- Endpoint: POST /vision/face-match
- Concern: Nếu Core gửi cùng traceId hai lần thì AI Vision trả kết quả mới hay cached?
- Proposal: AI Vision xử lý lại, không cache theo traceId
- Resolution: Accepted
- Rationale: traceId dùng để trace/audit, không phải idempotency key
- Impact: Core không được dùng traceId để deduplicate request

---

# Chốt hợp đồng v1.0

Provider sign-off: Đỗ Trung Kiên
Consumer sign-off: Nguyễn Công Hiệp
Witness (GV/TA):
Date: 18/05/2026

---

## Ghi chú warning nếu Spectral còn cảnh báo

| Warning          | Lý do chấp nhận tạm thời | Kế hoạch sửa |
| ---------------- | ------------------------ | ------------ |
| Không có warning | Pass hoàn toàn           | -            |
