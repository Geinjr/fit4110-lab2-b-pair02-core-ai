# Phân tích yêu cầu — vai Provider

- Cặp đàm phán: Cặp 02
- Product: A / B
- Provider service: AI Vision
- Consumer service: Core Business
- Người viết: [Tên bạn]
- Ngày: [Ngày hôm nay]

---

## 1. Resource chính

| Resource          | Mô tả                          | Thuộc tính bắt buộc                                 | Thuộc tính tùy chọn                   |
| ----------------- | ------------------------------ | --------------------------------------------------- | ------------------------------------- |
| `FaceMatchResult` | Kết quả so khớp khuôn mặt      | `matchId`, `confidence`, `decision`, `timestamp`    | `subjectId`, `reason`, `modelVersion` |
| `Detection`       | Kết quả phân tích/detect chung | `detectionId`, `objects`, `confidence`, `timestamp` | `traceId`, `riskLevel`                |

---

## 2. Action/API dự kiến

| Method | Path                               | Mục đích                               | Consumer gọi khi nào?                            |
| ------ | ---------------------------------- | -------------------------------------- | ------------------------------------------------ |
| POST   | `/vision/face-match`               | Core gửi ảnh hoặc embedding để so khớp | Khi cần xác minh danh tính                       |
| GET    | `/vision/detections/{detectionId}` | Core lấy kết quả detect theo id        | Sau khi Camera gửi frame và AI Vision xử lý xong |
| GET    | `/vision/results/recent`           | Core lấy danh sách kết quả gần đây     | Khi cần audit hoặc dashboard                     |
| GET    | `/health`                          | Kiểm tra service còn sống              | Trước khi gọi các endpoint chính                 |

---

## 3. Error case

| Status | Tình huống                                            | Response body dự kiến |
| -----: | ----------------------------------------------------- | --------------------- |
|    400 | Payload sai định dạng, thiếu field bắt buộc           | `Problem`             |
|    401 | Thiếu Bearer token                                    | `Problem`             |
|    403 | Token hợp lệ nhưng Core không có quyền gọi face-match | `Problem`             |
|    404 | `detectionId` không tồn tại trong hệ thống            | `Problem`             |
|    422 | Ảnh đúng format nhưng chất lượng quá thấp để detect   | `Problem`             |
|    500 | Model AI lỗi hoặc downstream timeout                  | `Problem`             |

---

## 4. Giả định bổ sung

- Giả định 1: Core gửi URL ảnh (imageRef) thay vì upload file trực tiếp.
- Giả định 2: Confidence là số thực từ 0.0 đến 1.0.
- Giả định 3: AI Vision không lưu ảnh gốc, chỉ lưu kết quả detect.

---

## 5. Câu hỏi cho Consumer

1. Core gửi `imageRef` (URL) hay `faceEmbedding` (vector) vào `/vision/face-match`?
2. Ngưỡng confidence bao nhiêu thì Core coi là match hợp lệ?
3. Khi model không chắc chắn, Core muốn nhận `200` kèm trạng thái `low_confidence` hay `422`?

---

## 6. Rủi ro tích hợp

| Rủi ro                                             | Tác động                | Đề xuất xử lý                                        |
| -------------------------------------------------- | ----------------------- | ---------------------------------------------------- |
| Core và AI Vision hiểu khác nhau về format đầu vào | Request bị 400 liên tục | Chốt rõ schema FaceMatchRequest trong openapi.yaml   |
| Confidence dùng thang đo khác nhau                 | Core ra quyết định sai  | Thống nhất kiểu `number`, `minimum: 0`, `maximum: 1` |
| AI Vision trả lỗi 500 khi model quá tải            | Core bị block           | Thống nhất timeout và fallback behavior              |
