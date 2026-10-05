# Báo Cáo Thực Hành: Quản Lý Tiến Trình Nền Với nohup Và Tín Hiệu Kill

## 1. Mục tiêu
- Biết cách khởi chạy tiến trình chạy nền độc lập với phiên làm việc Terminal bằng lệnh `nohup` và ký tự `&`.
- Thành thạo công cụ giám sát tiến trình hệ thống bằng `ps`, `pgrep`, và kiểm tra log bằng `tail`.
- Hiểu rõ cơ chế và sử dụng thành thạo lệnh `kill` với các tín hiệu hệ thống (`SIGTERM - 15` và `SIGKILL - 9`) để quản lý tiến trình an toàn.

---

## 2. Kịch bản Shell Script (`loop-monitor.sh`)

Mã nguồn script ghi thời gian hệ thống vào file `/tmp/monitor.log` định kỳ mỗi 5 giây:

```bash
#!/bin/bash
while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
```

Cấp quyền thực thi cho file:
```bash
chmod +x loop-monitor.sh
```

---

## 3. Các bước thực hiện & Kết quả thực nghiệm

### Bước 1: Khởi chạy tiến trình chạy nền độc lập với nohup
Sử dụng công cụ `nohup` kết hợp toán tử `&` và chuyển hướng luồng I/O:
```bash
nohup ./loop-monitor.sh > /dev/null 2>&1 &
```

> **Giải thích câu lệnh:**
> - `nohup`: Bỏ qua tín hiệu `SIGHUP` (Signal Hang Up), giúp tiến trình tiếp tục chạy ngay cả khi đóng phiên Terminal hoặc ngắt kết nối SSH.
> - `> /dev/null 2>&1`: Chuyển hướng cả Standard Output (luồng 1) và Standard Error (luồng 2) vào "hố đen" `/dev/null` để tránh tạo ra file `nohup.out` không cần thiết.
> - `&`: Đưa tiến trình chạy dưới chế độ nền (background), trả lại dấu nhắc lệnh cho người dùng ngay lập tức.

---

### Bước 2: Giám sát và tìm số định danh tiến trình (PID)

Sử dụng lệnh `ps aux` hoặc `pgrep`:
```bash
ps aux | grep loop-monitor.sh
```

**Kết quả ghi nhận:**
```text
quangggg     606  0.0  0.0   4948  3612 ?        S    19:01   0:00 /bin/bash ./loop-monitor.sh
quangggg     617  0.0  0.0   4264  2420 pts/2    S+   19:01   0:00 grep loop-monitor.sh
```
*-> PID của tiến trình `loop-monitor.sh` được xác định là: **`606`**.*

---

### Bước 3: Kiểm tra tệp tin log ghi liên tục

Kiểm tra nội dung file `/tmp/monitor.log` để xác minh tiến trình đang ghi dữ liệu định kỳ mỗi 5 giây:
```bash
tail -n 10 /tmp/monitor.log
```

**Kết quả ghi nhận:**
```text
System time: Mon Oct  5 19:01:17 +07 2026
System time: Mon Oct  5 19:01:22 +07 2026
System time: Mon Oct  5 19:01:27 +07 2026
System time: Mon Oct  5 19:01:32 +07 2026
```
*-> Dữ liệu ghi nhận đúng chu kỳ 5 giây/lần.*

---

### Bước 4: Tắt tiến trình bằng tín hiệu Kill

Tiến hành gửi tín hiệu **SIGTERM (tín hiệu 15)** để yêu cầu tiến trình dừng an toàn (graceful termination):
```bash
kill -15 606
```

> **Lưu ý về các tín hiệu (Signals):**
> - **`SIGTERM` (15)**: Gửi thông điệp yêu cầu tiến trình tự dọn dẹp tài nguyên và kết thúc bình thường. Luôn ưu tiên dùng `kill -15` trước.
> - **`SIGKILL` (9)**: Buộc kernel hệ điều hành chấm dứt tiến trình ngay lập tức, tiến trình không thể bắt hay bỏ qua tín hiệu này. Chỉ sử dụng lệnh `kill -9 <PID>` khi tiến trình bị treo hoặc không phản hồi với `SIGTERM`.

---

### Bước 5: Kiểm tra xác nhận tiến trình đã dừng

Kiểm tra lại danh sách tiến trình bằng lệnh `ps aux` hoặc `pgrep`:
```bash
ps aux | grep loop-monitor.sh
```

**Kết quả ghi nhận:**
```text
quangggg     655  0.0  0.0   4264  2432 pts/2    S+   19:01   0:00 grep loop-monitor.sh
```
*-> Tiến trình `loop-monitor.sh` (PID `606`) đã biến mất khỏi danh sách tiến trình, xác nhận tiến trình đã được dừng thành công.*
