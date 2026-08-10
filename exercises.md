# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay phần giữ chỗ dưới mỗi câu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Hoàng Long  Mã học viên: 2A202601134

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên Render, nếu tôi quên đặt `API_TOKEN`, fail-fast làm container
> dừng ngay và log báo lỗi cấu hình. Nhờ vậy URL chưa được mở với một token mặc
> định mà ai cũng đoán được. Nếu dùng `"changeme"`, service vẫn chạy và người
> ngoài có thể gọi `/chat` trước khi tôi phát hiện cấu hình sai.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một log tôi quan sát được là:
> `{"event":"chat_completed","severity":"INFO","ts":"2026-08-10T09:42:23.524925+00:00","client_id":"sv-test","prompt_tokens":4,"completion_tokens":36,"usd_cost":2.22e-05}`.
> Từ các trường này tôi có thể lọc/tổng hợp chi phí theo `client_id`, đồng thời
> đếm lỗi hoặc đo lượng token theo thời gian. Dòng `print("đã trả lời xong")`
> không chứa dữ liệu để làm hai việc đó.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t chat:single .
docker build -t chat:multi .
docker images | grep chat
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | khoảng 1.8 GB theo baseline của Dockerfile ban đầu |
| Multi-stage | 296 MB (đo bằng `docker images`) |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Image multi-stage thực tế của tôi là 296 MB. Tôi đã thử build lại bản một
> stage để đo nhưng mạng tải base `python:3.11` quá chậm nên chưa hoàn tất; đề
> cung cấp baseline khoảng 1.8 GB. Phần chênh lệch chủ yếu là base image Python
> đầy đủ, công cụ build/compiler, cache và các file nguồn không cần cho runtime;
> stage cuối bản mới chỉ dùng `python:3.11-slim` và copy dependency đã cài.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi tôi sửa `app/main.py` rồi build lại, các layer base image, `WORKDIR`,
> `COPY requirements.txt` và `pip install` đều hiện `CACHED`. Docker chỉ chạy
> lại `COPY app`, các layer sau nó và bước export image. Nếu đặt `COPY . .`
> trước `pip install`, mọi thay đổi code sẽ làm mất cache của layer copy, kéo
> theo việc cài lại toàn bộ dependency dù `requirements.txt` không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu ứng dụng Python có lỗ hổng thực thi lệnh, kẻ tấn công trước hết chạy lệnh
> với UID của process trong container. Nếu process là root, một lỗi cấu hình
> mount/capability hoặc lỗ hổng container runtime có thể giúp hắn tác động tới
> host với quyền cao. `USER appuser` đổi process sang UID 10001 không đặc quyền,
> nên quyền của mã bị khai thác bị giới hạn ngay từ bước thực thi trong container.

---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

> Header `WWW-Authenticate: Bearer` cho client biết cơ chế xác thực mà server
> yêu cầu theo chuẩn HTTP. Tôi trả cùng thông báo `invalid or missing bearer
> token` cho thiếu header, sai scheme và sai token để không tiết lộ bước nào đã
> đúng; nếu trả quá chi tiết, người dò token có thêm thông tin để thu hẹp tấn công.

---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

> Client chỉ gửi được 10 request liên tiếp rồi request thứ 11 nhận 429, vì xô
> không bao giờ chứa quá `capacity=10`. Nếu bỏ `min(capacity, ...)`, sau 10 phút
> nó tích thêm 100 token ở tốc độ 10 token/phút; tùy trạng thái trước lúc im
> lặng, xô có thể đạt khoảng 110 token và cho phép một burst rất lớn thay vì 10.

---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

> Với hạn mức 30 USD/tháng, sự cố từ 2 giờ sáng có thể tiêu gần 30 USD trước khi
> bị chặn và chỉ tự hồi phục khi sang tháng mới. Với hạn mức 1 USD/ngày, thiệt
> hại của client bị giới hạn quanh 1 USD (soft quota có thể vượt bởi đúng request
> cuối), và service tự có ngân sách mới vào 00:00 UTC ngày hôm sau.

---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu endpoint chung kiểm tra Redis, Redis mất kết nối làm probe của cả ba
> container trả 503. Orchestrator coi cả ba instance không khỏe, rút chúng khỏi
> load balancer rồi có thể restart đồng loạt. Các request đang xử lý bị gián
> đoạn, trong khi restart không sửa được Redis và cụm tiếp tục lặp lỗi. Tách ra
> giúp `/healthz` chỉ phản ánh process, còn `/readyz` tạm ngừng traffic tới
> instance chưa dùng được Redis.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Sau khi Render báo deploy thành công, pytest CP5 có lúc nhận 404 cho
> `/healthz` và `/readyz`, dù trình duyệt lại thấy JSON 200. Tôi mở Application
> Logs và thấy Uvicorn đã chạy ở cổng 10000, các health check nội bộ đều trả 200;
> gọi lặp lại cũng cho thấy Render edge lúc 404 lúc 200 ngay sau deploy. Tôi chờ
> edge ổn định rồi chạy lại pytest; các public deployment test sau đó pass. Lỗi
> 404 ở `/` là bình thường vì ứng dụng không khai báo route gốc.
