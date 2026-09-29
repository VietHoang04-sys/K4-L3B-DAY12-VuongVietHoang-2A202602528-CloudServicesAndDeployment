# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng mẫu trong mỗi câu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Vuong Viet Hoang  Mã học viên: 2A202602528

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Nếu image production thiếu `AGENT_API_KEY`, deploy sẽ fail ngay lúc khởi động thay
vì chạy với khóa `changeme`. Nhờ vậy tôi phát hiện lỗi cấu hình trước khi service
nhận request và phát sinh chi phí LLM.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Ví dụ: `{"event":"ask_completed","level":"info","timestamp":"2026-09-29T07:00:00+00:00","user_id":"sv-test","tokens_in":3,"tokens_out":42,"cost_usd":0.00003}`.
Từ đó có thể lọc theo user để tìm chi phí cao nhất và đếm tỷ lệ lỗi theo thời
gian/event; câu `print` không có schema để truy vấn hoặc tổng hợp ổn định.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ... MB |
| Multi-stage | ... MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

| 1 stage (bản đầu) | 1.7 GB |
| Multi-stage | 310 MB |

Multi-stage chỉ đưa virtualenv và mã chạy sang runtime slim; compiler, cache
build và phần hệ thống của image đầy đủ không đi vào image cuối.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Khi chỉ sửa `app/main.py`, layer cài dependency vẫn được cache; các layer copy
source và các layer sau đó phải chạy lại. Nếu `COPY . .` đứng trước
`pip install`, mọi thay đổi code làm mất cache của layer install và pip phải cài
lại toàn bộ dependency.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Nếu process bị khai thác, quyền root trong container có thể cho phép kẻ tấn công
đọc/sửa mọi file trong container và tận dụng lỗ hổng runtime để leo thang sang
host. `USER appuser` chạy app bằng UID không đặc quyền, nên ngay cả khi thoát
container hoặc khai thác thêm, điểm bắt đầu không còn là root.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

Có thể gửi 20 request trong 2 giây: 10 request ngay trước giây 00 và 10 request
ngay sau giây 00. Hai nhóm thuộc hai phút đồng hồ khác nhau dù thực tế chỉ cách
nhau khoảng 2 giây.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

Rate limit giới hạn tần suất request, còn cost guard giới hạn tổng tiền theo
user/tháng. Một request có prompt cực dài có thể qua rate limit nhưng bị cost
guard chặn vì ngân sách còn lại không đủ; ngược lại nhiều request nhỏ có thể
chưa vượt ngân sách nhưng bị rate limit chặn vì gửi quá nhanh.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

Nếu Redis mất kết nối, endpoint dùng chung sẽ trả lỗi cho cả liveness và
readiness. Load balancer coi cả ba container là không sống, lần lượt restart
chúng; sự cố Redis 30 giây biến thành restart dây chuyền và làm mất khả năng
phục vụ dù process vẫn chạy. Tách `/health` giúp container còn sống được giữ
lại, còn `/ready` trả 503 để ngừng nhận traffic.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Với Redis, các request đi qua ba container vẫn đọc cùng danh sách nên
`history_length` tăng 0, rồi 2, rồi 4... Nếu dùng dict trong từng process, mỗi
container có lịch sử riêng; khi load balancer chuyển instance, con số có thể
quay lại 0 hoặc tăng theo từng nhánh riêng, tạo cảm giác agent mất trí nhớ.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Khi tạo domain Railway lần đầu, domain có `Target port: -` nên không truy cập
được dù container đã chạy. Tôi xem `railway logs` và thấy Uvicorn chạy ở
`0.0.0.0:8080`; sau đó chạy `railway domain update
agent-production-f088.up.railway.app --port 8080`. Domain hoạt động và các
probe `/health`, `/ready` đều trả 200.
