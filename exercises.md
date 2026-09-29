# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Hoàng Đức Minh  Mã học viên: .2A202602362

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu thiếu AGENT_API_KEY, app chết ngay khi khởi động. Điều này giúp phát hiện sớm lỗi cấu hình, tránh để service chạy với khóa mặc định và bị lạm dụng hoặc tính phí nhầm.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Với log JSON, ta có thể lọc, thống kê, cảnh báo và giám sát trên hệ thống production; print("đã trả lời xong") thì chỉ đọc bằng mắt.

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
| 1 stage (bản đầu) | 47.8 MB |
| Multi-stage | 63.9 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> 1-stage lớn hơn multi-stage vì chứa cả môi trường build và thư viện phụ trợ; multi-stage chỉ giữ runtime cần thiết. Multi-stage bỏ đi layer build, nên image nhỏ hơn rõ rệt.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Nếu sửa code trong app, chỉ layer sau COPY . . bị invalid; các layer trước đó như pip install có thể được cache lại. Nếu đặt COPY . . trước RUN pip install, mọi thay đổi code đều làm cache của pip install mất, dẫn đến cài lại dependency mỗi lần build

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Lỗ hổng Python → code thực thi với quyền root trong container → attacker có thể đọc/ghi file hệ thống hoặc escape ra host → tấn công máy chủ host.
USER cắt chuỗi đó bằng cách giảm quyền ngay khi chạy process, nên exploit không còn quyền root.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với rate limit 10/phút theo phút đồng hồ, trong 2 giây liên tiếp có thể gửi tối đa 20 request. Ví dụ: 10 request lúc :59 và 10 request lúc :01 vẫn được tính là 20, vì reset theo phút đồng hồ chứ không phải 60 giây trượt.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit: giới hạn số request, chủ yếu bảo vệ hệ thống.
Cost guard: giới hạn chi phí, chủ yếu bảo vệ ngân sách.
Ví dụ: rate limit cho qua nhưng cost guard chặn khi mỗi request quá đắt; ngược lại, request rẻ nhưng quá nhiều request thì rate limit chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp 2 endpoint thành 1 và nó kiểm tra Redis, khi Redis mất 30 giây: load balancer bắt đầu bỏ traffic khỏi node, các request đang chạy xử lý xong, rồi khi Redis quay lại endpoint mới thành OK và traffic tiếp tục.
Thứ tự: 1. Redis mất kết nối, 2. /health báo lỗi, 3. LB ngừng gửi traffic mới, 4. request cũ kết thúc, 5. khi Redis sống lại, service lại nhận traffic.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Nếu lịch sử nằm trong dict Python thay vì Redis, mỗi instance có bộ nhớ riêng nên history_length sẽ khác nhau giữa các container.
Khi scale lên 3 instance hoặc restart, lịch sử sẽ mất hoặc không đồng nhất giữa các request cùng X-User-Id.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Ví dụ lỗi phổ biến: /ready trả 503 hoặc REDIS_URL sai, dẫn đến service không kết nối Redis. Nguyên nhân: Redis chưa tạo hoặc URL trên cloud sai; cách sửa: set đúng REDIS_URL, tạo Redis instance, và đảm bảo app đọc đúng $PORT/health check.
