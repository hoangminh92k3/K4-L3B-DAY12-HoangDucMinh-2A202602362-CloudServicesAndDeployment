# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Hoàng Đức Minh |
| Mã học viên | 2A202602362 |
| Repo | https://github.com/hoangminh92k3/K4-L3B-DAY12-HoangDucMinh-2A202602362-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-6le4.onrender.com |
| Platform | Railway / Render / Cloud Run — Render |
| Ngày deploy | 29/09/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | redis://red-datkgo2d0e5s73cfukq0:6379 |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `https://day12-agent-6le4.onrender.com` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-agent-6le4.onrender.com/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-agent-6le4.onrender.com/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-6le4.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-agent-6le4.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-6le4.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
GET /health
HTTP/1.1 200 OK
{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready
HTTP/1.1 200 OK
{"status":"ready","redis":true}

POST /ask không gửi API key
HTTP/1.1 401 Unauthorized
{"detail":"invalid or missing API key"}

answer         : Ngáº¯n gá»n: Deploy la gi phá»¥ thuá»c vÃo ba yáº¿u tá» â cáº¥u hÃ¬nh qua biáº¿n 
                 mÃ´i trÆ°á»á» orchestrator biáº¿t tráº¡ng thÃ¡i, vÃ giá» háº¡n 
                 tÃi nguyÃªn.
user_id        : sv-test
history_length : 0
cost_usd       : 2.265E-05
tokens         : @{in=3; out=37}

(.venv) PS C:\Users\52100\OneDrive\Document\Day12\K4-L3B-DAY12-HoangDucMinh-2A202602362-CloudServicesAndDeployment> 1..15 | ForEach-Object {
>>   curl.exe -s -o NUL -w "%{http_code} " -X POST "$base/ask" `
>>     -H "Content-Type: application/json" `
>>     -H "X-API-Key: $apiKey" `
>>     -H "X-User-Id: rate-test-01" `
>>     -d '{\"question\":\"test\"}'
>> }
>> Write-Host
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429 
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---
