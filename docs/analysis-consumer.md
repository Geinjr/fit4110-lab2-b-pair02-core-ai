# Phân tích yêu cầu — vai Consumer

- Cặp đàm phán: Cặp 02
- Product: A / B
- Consumer service: Core Business
- Provider service: AI Vision
- Người viết: Nguyễn Công Hiệp
- Ngày: 18/05/2026

---

## 1. Resource Consumer cần nhận/gửi

| Resource          | Consumer dùng để làm gì?         | Field bắt buộc với Consumer                   | Field có thể tùy chọn       |
| ----------------- | -------------------------------- | --------------------------------------------- | --------------------------- |
| `FaceMatchResult` | Ra quyết định xác minh danh tính | `decision`, `confidence`, `matchId`           | `subjectId`, `reason`       |
| `Detection`       | Audit và dashboard bất thường    | `detectionId`, `objects`, `overallConfidence` | `riskLevel`, `modelVersion` |

---

## 2. API Consumer cần gọi

| Method | Path                               | Lúc nào gọi?                   | Kỳ vọng response               |
| ------ | ---------------------------------- | ------------------------------ | ------------------------------ |
| POST   | `/vision/face-match`               | Khi cần xác minh danh tính     | 200 với decision và confidence |
| GET    | `/vision/detections/{detectionId}` | Khi cần lấy chi tiết detection | 200 với objects và riskLevel   |
| GET    | `/vision/results/recent`           | Khi cần audit hoặc dashboard   | 200 với danh sách kết quả      |
| GET    | `/health`                          | Trước khi gọi API chính        | 200 status ok                  |

---

## 3. Error case Consumer cần xử lý

| Status | Consumer hiểu là gì?      | Consumer sẽ xử lý thế nào?         |
| -----: | ------------------------- | ---------------------------------- |
|    400 | Request sai schema        | Kiểm tra lại payload trước khi gửi |
|    401 | Thiếu token               | Refresh token rồi retry            |
|    403 | Không đủ quyền            | Báo lỗi quyền truy cập cho admin   |
|    404 | detectionId không tồn tại | Hiển thị trạng thái không tìm thấy |
|    422 | Ảnh chất lượng thấp       | Yêu cầu chụp lại ảnh               |
|    500 | AI Vision lỗi             | Retry sau 30 giây hoặc fallback    |

---

## 4. Giả định bổ sung

- Giả định 1: AI Vision luôn trả response trong vòng 3 giây.
- Giả định 2: confidence dùng thang float 0.0 đến 1.0.
- Giả định 3: Core tự quyết định ngưỡng confidence để coi là MATCH.

---

## 5. Câu hỏi cho Provider

1. AI Vision có cache kết quả theo traceId không?
2. Khi model đang cập nhật thì API có downtime không?
3. Giới hạn số request mỗi phút là bao nhiêu?

---

## 6. Rủi ro tích hợp

| Rủi ro                       | Tác động                      | Đề xuất xử lý                |
| ---------------------------- | ----------------------------- | ---------------------------- |
| AI Vision trả LOW_CONFIDENCE | Core không ra được quyết định | Thống nhất fallback behavior |
| modelVersion thay đổi        | Kết quả không nhất quán       | Ghi modelVersion vào log     |
