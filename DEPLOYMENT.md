# Thông Tin Deploy — Checkpoint 5

> Không ghi giá trị `API_TOKEN` hoặc mật khẩu Redis vào file này.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Hoàng Long |
| Mã học viên | 2A202601134 |
| Repo | https://github.com/hoanglongvbhp0703-sys/K4-DAY12-2A202601134-NguyenHoangLong |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | PENDING_RAILWAY_URL |
| Platform | Railway |
| Ngày deploy | 2026-08-10 |

Sau khi Railway tạo domain, thay `PENDING_RAILWAY_URL` bằng URL HTTPS đầy đủ,
không thêm dấu `/` ở cuối.

## Biến Môi Trường Trên Cloud

| Biến | Nguồn giá trị |
|------|---------------|
| `PORT` | Railway tự cung cấp |
| `API_TOKEN` | Secret tự sinh, đặt trong Variables của service chat |
| `REDIS_URL` | Reference variable tới `REDIS_URL` của Railway Redis service |
| `BUCKET_CAPACITY` | `10` |
| `REFILL_PER_MINUTE` | `10` |
| `DAILY_BUDGET_USD` | `1.0` |
| `LOG_LEVEL` | `INFO` |

## Lệnh Kiểm Tra

Đặt URL và token trong terminal cục bộ; không commit hai giá trị này:

```bash
export DEPLOY_URL="https://domain-cua-service.up.railway.app"
export DEPLOY_API_TOKEN="token-da-set-tren-railway"

curl -i "$DEPLOY_URL/healthz"
curl -i "$DEPLOY_URL/readyz"

curl -i -X POST "$DEPLOY_URL/chat" \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'

curl -i -X POST "$DEPLOY_URL/chat" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $DEPLOY_API_TOKEN" \
  -H "X-Client-Id: sv-test" \
  -d '{"message":"Deploy là gì?"}'
```

## Kết Quả Chạy Thật

Chưa có — cập nhật sau khi Railway deploy thành công. Cần lưu bằng chứng cho:

- `/healthz`: HTTP 200, `status` là `ok`.
- `/readyz`: HTTP 200, `status` là `ready`.
- `/chat` không có token: HTTP 401 và có `WWW-Authenticate: Bearer`.
- `/chat` có token hợp lệ: HTTP 200 và response có `reply`.

## Ảnh Chụp Màn Hình

Sau khi deploy, lưu hai file:

- `screenshots/dashboard.png`: Railway deployment ở trạng thái thành công.
- `screenshots/healthz.png`: kết quả gọi Public URL `/healthz`.
