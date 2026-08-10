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
| Public URL | https://day12-chat-84ca.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-08-10 |

Public URL dùng HTTPS và không có dấu `/` ở cuối.

## Biến Môi Trường Trên Cloud

| Biến | Nguồn giá trị |
|------|---------------|
| `PORT` | Render tự cung cấp |
| `API_TOKEN` | Secret tự sinh, đặt trong Environment của web service |
| `REDIS_URL` | Internal connection string của Render Key Value service |
| `BUCKET_CAPACITY` | `10` |
| `REFILL_PER_MINUTE` | `10` |
| `DAILY_BUDGET_USD` | `1.0` |
| `LOG_LEVEL` | `INFO` |

## Lệnh Kiểm Tra

Đặt URL và token trong terminal cục bộ; không commit hai giá trị này:

```bash
export DEPLOY_URL="https://day12-chat-84ca.onrender.com"
# DEPLOY_API_TOKEN được đọc từ .env local, không ghi giá trị vào tài liệu.

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

Kiểm tra ngày 2026-08-10:

```text
GET /healthz -> HTTP/2 200
{"status":"ok","service":"day12-chat-service","version":"1.0.0"}

GET /readyz -> HTTP/2 200
{"status":"ready","redis":true}

POST /chat (không có token) -> HTTP/2 401
www-authenticate: Bearer
{"detail":"invalid or missing bearer token"}

POST /chat (Bearer token hợp lệ) -> HTTP/2 200
Response có đầy đủ reply, client_id, turns_before, usd_cost và usage.
```

## Ảnh Chụp Màn Hình

Sau khi deploy, lưu hai file:

- `screenshots/dashboard.png`: Render deployment ở trạng thái thành công.
- `screenshots/healthz.png`: kết quả gọi Public URL `/healthz`.
