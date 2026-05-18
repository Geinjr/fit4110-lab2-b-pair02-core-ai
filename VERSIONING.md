# Versioning — AI Vision Service API

## 1. Chiến lược versioning

API này dùng **URL versioning** với prefix `/v{n}` khi có breaking change.

- Phiên bản hiện tại: `v1.0.0`
- URL base: `http://localhost:4010`
- Khi có breaking change sẽ chuyển sang: `http://localhost:4010/v2`

---

## 2. Backward-compatible change (không cần tăng major version)

Các thay đổi sau **không breaking**, Consumer không cần sửa code:

| Thay đổi                                       | Ví dụ                                                    |
| ---------------------------------------------- | -------------------------------------------------------- |
| Thêm field mới vào response                    | Thêm `processingTimeMs` vào `FaceMatchResult`            |
| Thêm endpoint mới                              | Thêm `GET /vision/models`                                |
| Thêm giá trị enum mới vào field không bắt buộc | Thêm `UNCERTAIN` vào `riskLevel`                         |
| Nới lỏng constraint                            | Tăng `maxLength` của `subjectId` từ 80 lên 120           |
| Thêm query parameter tùy chọn                  | Thêm `?modelVersion=v2` vào `GET /vision/results/recent` |

---

## 3. Breaking change (phải tăng major version)

Các thay đổi sau **có breaking**, phải tạo version mới và thông báo trước:

| Thay đổi                | Ví dụ                                  | Tác động            |
| ----------------------- | -------------------------------------- | ------------------- |
| Xóa field khỏi response | Xóa `traceId` khỏi `FaceMatchResult`   | Consumer parse lỗi  |
| Đổi tên field           | `confidence` → `score`                 | Consumer parse lỗi  |
| Đổi kiểu dữ liệu        | `confidence` từ `number` sang `string` | Consumer type error |
| Xóa endpoint            | Xóa `GET /vision/results/recent`       | Consumer gọi 404    |
| Thắt chặt constraint    | Giảm `maxLength` của `imageRef`        | Consumer bị 400     |
| Đổi enum value          | `MATCH` → `MATCHED`                    | Consumer logic sai  |

---

## 4. Deprecated field

Field hoặc endpoint đang trong quá trình loại bỏ sẽ được đánh dấu `deprecated: true` trong `openapi.yaml` và kèm header `Sunset` trong response.

Ví dụ trong `openapi.yaml`:

```yaml
/vision/results/recent:
  get:
    deprecated: true
    description: |
      DEPRECATED: Dùng GET /vision/results thay thế từ v2.0.
      Endpoint này sẽ bị xóa sau 2026-08-01.
```

Ví dụ header `Sunset` trong response:

```
Sunset: Sun, 01 Aug 2026 00:00:00 GMT
Deprecation: true
Link: <https://api.campus.local/v2/vision/results>; rel="successor-version"
```

---

## 5. Kế hoạch migration v1 → v2

| Mốc        | Hành động                                 |
| ---------- | ----------------------------------------- |
| 2026-06-01 | Phát hành v2 song song v1                 |
| 2026-07-01 | Gửi thông báo deprecation v1 cho Consumer |
| 2026-08-01 | Tắt v1, chỉ còn v2                        |

---

## 6. Changelog

### v1.0.0 — 2026-05-18

- Phát hành lần đầu
- Endpoints: `POST /vision/face-match`, `GET /vision/detections/{detectionId}`, `GET /vision/results/recent`, `GET /health`
- Hỗ trợ `ImageRefRequest` và `EmbeddingRequest` qua `oneOf` + `discriminator`
- Cursor-based pagination cho `GET /vision/results/recent`
- Webhook `detectionCompleted`
