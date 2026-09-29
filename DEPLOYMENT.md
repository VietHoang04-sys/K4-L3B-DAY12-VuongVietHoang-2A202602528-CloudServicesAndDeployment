# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Vuong Viet Hoang |
| Mã học viên | 2A202602528 |
| Repo | https://github.com/VietHoang04-sys/K4-L3B-DAY12-VuongVietHoang-2A202602528-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | Chưa được cấp — hoàn tất sau khi chạy `railway domain` |
| Platform | Railway |
| Ngày deploy | Chưa deploy (chuẩn bị ngày 2026-09-29) |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis add-on của Railway |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Quy Trình Deploy Railway

1. Cài Railway CLI: `npm i -g @railway/cli`.
2. Chạy `railway login`, sau đó `railway init` trong thư mục repo này.
3. Tạo Redis: `railway add --database redis`.
4. Trong service agent, đặt các biến `AGENT_API_KEY`, `RATE_LIMIT_PER_MINUTE=10`,
   `MONTHLY_BUDGET_USD=10.0`, `LOG_LEVEL=INFO`. Gắn `REDIS_URL` do Redis service
   cung cấp và không ghi giá trị secret vào repo.
5. Chạy `railway up`, rồi `railway domain` để lấy public HTTPS URL.
6. Thay URL ở bảng trên, dán output kiểm tra vào mục bên dưới, rồi chạy
   `pytest tests/test_cp5.py -v`.

Railway tự cấp `PORT`; không đặt cứng PORT trên dashboard. Token Railway chỉ cần
cho CLI/CI và phải lưu trong GitHub Secrets (`RAILWAY_TOKEN`), không đưa vào
`.env` hoặc tài liệu.

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
Chưa có output public vì service chưa được deploy.
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

```
Không dùng phương án dự phòng; đang chờ tạo Railway project và public URL.
```
