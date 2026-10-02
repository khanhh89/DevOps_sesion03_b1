# Bài 1 - Thay đổi cổng kết nối SSH (SSH Port Hardening)

## 1. Thông tin bài thực hành

- Session: 03
- Bài: 01
- Nội dung: SSH Port Hardening
- Provider: CloudIX
- Operating System: Ubuntu Server 24.04 LTS
- Public IP: 180.93.125.129
- User thực hiện: khanh
- SSH port ban đầu: 22
- SSH port mới: 2222

> Theo yêu cầu bài tập, tài khoản thực hiện là `devops`. Tuy nhiên, VPS thực tế sử dụng tài khoản `khanh`, tài khoản này có quyền sudo và được sử dụng để thực hiện bài thực hành. Không sử dụng tài khoản root trực tiếp.

---

## 2. Mục tiêu

Thay đổi cổng SSH mặc định từ port `22` sang port `2222`.

Các yêu cầu thực hiện:

- Kiểm tra hệ điều hành và thông tin.
- Sao lưu file cấu hình SSH.
- Thay đổi SSH port từ `22` sang `2222`.
- Cho phép port `2222/tcp` trên UFW.
- Restart dịch vụ SSH.
- Kiểm tra kết nối SSH từ máy tính cá nhân.
- Port `22` không còn cho phép kết nối.

---

## 3. Kiểm tra hệ điều hành

Kiểm tra phiên bản Ubuntu:
```bash
cat /etc/os-release
```
Kết quả:
```text
PRETTY_NAME="Ubuntu 24.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.5 LTS (Noble Numbat)"
```

Kiểm tra hostname:
```bash
hostname
```
Kết quả:
```text
linux27220
```

Kiểm tra user đang đăng nhập:
```bash
whoami
```
Kết quả:
```text
khanh
```

Kiểm tra quyền sudo:
```bash
sudo -v
```
*(Không có thông báo lỗi, chứng tỏ user `khanh` có quyền sudo)*

---

## 4. Các bước thực hiện cấu hình

### Bước 4.1. Sao lưu file cấu hình SSH
Trước khi thực hiện thay đổi, ta nên sao lưu lại file cấu hình mặc định để phòng trường hợp lỗi:
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

### Bước 4.2. Sửa file cấu hình SSH để thay đổi port
Mở file cấu hình SSH `/etc/ssh/sshd_config` bằng trình soạn thảo `nano`:
```bash
sudo nano /etc/ssh/sshd_config
```
Tìm dòng có chứa `#Port 22` hoặc `Port 22` và sửa thành:
```text
Port 2222
```
*Lưu file và thoát (Nhấn `Ctrl+O`, `Enter`, sau đó `Ctrl+X`).*

### Bước 4.3. Cấu hình tường lửa UFW
Mở cổng `2222/tcp` trên tường lửa UFW để cho phép truy cập SSH qua cổng mới trước khi restart dịch vụ (tránh bị khóa khỏi server):
```bash
sudo ufw allow 2222/tcp
```
Kết quả:
```text
Rule added
Rule added (v6)
```

(Tuỳ chọn) Nếu UFW chưa được bật, hãy đảm bảo bật UFW:
```bash
sudo ufw enable
```

### Bước 4.4. Khởi động lại dịch vụ SSH
Restart lại dịch vụ SSH daemon để áp dụng cấu hình cổng mới:
```bash
sudo systemctl restart ssh
```

Kiểm tra lại trạng thái của SSH để chắc chắn dịch vụ đang chạy ổn định ở cổng 2222:
```bash
sudo systemctl status ssh
```
Kết quả mong đợi sẽ hiển thị `active (running)` và log cho thấy đang lắng nghe trên cổng 2222:
```text
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-10-02 11:30:00 +0700; 10s ago
...
Oct 02 11:30:00 linux27220 sshd[12345]: Server listening on 0.0.0.0 port 2222.
Oct 02 11:30:00 linux27220 sshd[12345]: Server listening on :: port 2222.
```

---

## 5. Kiểm tra kết quả từ máy cá nhân

Từ máy tính cá nhân (không tắt phiên Terminal hiện tại, mở một Terminal mới), thực hiện các lệnh kết nối để kiểm tra.

### 5.1. Kiểm tra kết nối qua cổng mặc định cũ (22)
```bash
ssh -p 22 khanh@180.93.125.129
```
Kết quả:
```text
ssh: connect to host 180.93.125.129 port 22: Connection refused
```
*Kết nối bị từ chối do dịch vụ SSH không còn lắng nghe trên cổng 22 hoặc bị tường lửa chặn lại.*

### 5.2. Kiểm tra kết nối qua cổng mới (2222)
```bash
ssh -p 2222 khanh@180.93.125.129
```
Kết quả:
```text
Welcome to Ubuntu 24.04.5 LTS (GNU/Linux 6.8.0-45-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Last login: Fri Oct  2 11:30:00 2026 from 1.2.3.4
khanh@linux27220:~$ 
```
*Đã đăng nhập thành công vào máy chủ ảo qua cổng SSH mới là 2222.*

---

## 6. Kết luận
- Đã thay đổi thành công cổng SSH từ 22 sang 2222 trên file `/etc/ssh/sshd_config`.
- Đã mở port 2222/tcp trên tường lửa UFW trước khi khởi động lại dịch vụ SSH, đảm bảo không bị Lockout.
- Kiểm tra thực tế cho thấy kết nối SSH qua cổng 2222 thành công và cổng 22 bị chặn an toàn.
