CÀI NGINX OFFLINE TRÊN UBUNTU SERVER
----

## 1. Kiểm tra version Ubuntu Server hiện tại
```
cat /etc/os-release
```

```
dpkg --print-architecture
```

**Ví dụ**: Bộ cài trên dành cho Ubuntu 26.04, kiến trúc `amd64`.

## 2. Tải bộ cài
Trên máy có thể truy cập Internet, mở trình duyệt và truy cập kho chính thức:

[https://nginx.org/packages/ubuntu/pool/nginx/n/nginx/](https://nginx.org/packages/ubuntu/pool/nginx/n/nginx/?utm_source=chatgpt.com)

Link tải trực tiếp bộ cài đã sử dụng:

[nginx_1.30.6-1~resolute_amd64.deb](https://nginx.org/packages/ubuntu/pool/nginx/n/nginx/nginx_1.30.6-1~resolute_amd64.deb)

## 3. Chuyển bộ cài vào Ubuntu Server
Tạo thư mục trên Ubuntu Server:
```
mkdir -p /home/ubuntu/nginx-offline
```

Sử dụng **USB**, **WinSCP** hoặc **SFTP** để chuyển file `.deb` vào:
```
/home/ubuntu/nginx-offline
```

Kiểm tra file:
```
cd /home/ubuntu/nginx-offline
```

```
ls -lh
```

Trong thư mục cần có:
```
nginx_1.30.6-1~resolute_amd64.deb
```

File `.deb` là gói cài đặt, **không cần giải nén**.

## 4. Cài đặt Nginx offline
Chạy:
```
cd /home/ubuntu/nginx-offline
```

```
sudo dpkg -i ./nginx_*.deb
```

Kết quả cài đặt thành công trong lần triển khai này:
```
Unpacking nginx (1.30.6-1~resolute)…
Setting up nginx (1.30.6-1~resolute)…
Created symlink '/etc/systemd/system/multi-user.target.wants/nginx.service'
→ '/usr/lib/systemd/system/nginx.service'.
```

Không xuất hiện lỗi thiếu phụ thuộc trong log đã ghi nhận.

## 5. Khởi động Nginx
Khởi động dịch vụ và thiết lập tự khởi động cùng Ubuntu:
```
sudo systemctl enable --now nginx
```

Kiểm tra dịch vụ đang chạy:
```
sudo systemctl is-active nginx
```

Kết quả mong đợi:
```
active
```
Kiểm tra thiết lập tự khởi động:
```
sudo systemctl is-enabled nginx
```

Kết quả mong đợi:
```
enabled
```

Xem trạng thái chi tiết:
```
sudo systemctl status nginx --no-pager -l
```

Trạng thái cần có:
```
Active: active (running)
```

## 6. Cấp quyền cho Nginx không cần sudo
```
echo "alias nginx='sudo nginx'" >> ~/.bashrc
source ~/.bashrc
```

Sau đó dùng:
```
nginx -t
```

Tuy nhiên, máy vẫn có thể yêu cầu mật khẩu. **Nếu muốn kiểm tra cấu hình không cần nhập mật khẩu**, cấp quyền riêng cho thao tác này:

**1. Xác định đường dẫn Nginx**:
```
command -v nginx
```

Thông thường là `/usr/sbin/nginx`.

**2. Mở file phân quyền**:
```
sudo visudo -f /etc/sudoers.d/ubuntu-nginx
```

Thêm đúng dòng dưới đây, thay đường dẫn nếu kết quả bước 1 khác:
```
ubuntu ALL=(root) NOPASSWD: /usr/sbin/nginx -t
```

**3. Kiểm tra**:
```
nginx -t
```

Alias sẽ gọi sudo nginx -t, và quy tắc trên cho phép chạy mà không hỏi mật khẩu.

Không cần đổi quyền /var/log/nginx, /run/nginx.pid hay cho user sở hữu /etc/nginx. Dịch vụ Nginx đã tự chạy dưới tài khoản được cấu hình.

## 7. Kiểm tra phiên bản và cổng dịch vụ
Kiểm tra phiên bản:
```
nginx -v
```

Kết quả mong đợi:
```
nginx version: nginx/1.30.6
```

Kiểm tra cổng HTTP:
```
sudo ss -lntp 'sport = :80'
```
Khi Nginx hoạt động với cấu hình HTTP mặc định, kết quả cần có dòng LISTEN tại cổng 80 với tiến trình nginx.

## 8. Kiểm tra bằng trình duyệt
Từ máy tính có kết nối tới mạng lab, truy cập:
```
http://192.168.90.12
```
Nếu xuất hiện:
```
Welcome to nginx!
```

thì đã xác nhận truy cập HTTP tới Nginx thành công.

Nếu không truy cập được, kiểm tra lần lượt:

```
ip -br address

sudo systemctl status nginx --no-pager -l

sudo ss -lntp 'sport = :80'

sudo ufw status
```

Nếu UFW đang active và cần cho phép các máy trong mạng `192.168.90.0/24` truy cập HTTP:
```
sudo ufw allow from 192.168.90.0/24 to any port 80 proto tcp
```
Nếu có firewall giữa các mạng, cần cho phép máy truy cập kết nối đến `192.168.90.12`, `TCP port 80`.


## 9. Các đường dẫn quản trị
Với gói cài từ nginx.org đã sử dụng:

| Nội dung | Đường dẫn |
|---|---|
| Cấu hình chính | `/etc/nginx/nginx.conf` |
| Thư mục cấu hình website | `/etc/nginx/conf.d/` |
| Cấu hình website mặc định thường dùng | `/etc/nginx/conf.d/default.conf` |
| Nội dung website mặc định thường dùng | `/usr/share/nginx/html/` |
| Access log | `/var/log/nginx/access.log` |
| Error log | `/var/log/nginx/error.log` |

## 10. Các lệnh quản trị thường dùng
Kiểm tra cấu hình trước khi áp dụng thay đổi:
```
sudo nginx -t
```

Nạp lại cấu hình sau khi kiểm tra thành công:
```
sudo systemctl reload nginx
```

Khởi động lại dịch vụ:
```
sudo systemctl restart nginx
```

Dừng dịch vụ:
```
sudo systemctl stop nginx
```

Khởi động dịch vụ:
```
sudo systemctl start nginx
```

Xem 100 dòng error log gần nhất:
```
sudo tail -n 100 /var/log/nginx/error.log
```

Theo dõi access log trực tiếp:
```
sudo tail -f /var/log/nginx/access.log
```

## 11. Xử lý khi dịch vụ không khởi động
Kiểm tra cú pháp:
```
sudo nginx -t
```

Xem log dịch vụ:
```
sudo journalctl -u nginx -n 100 --no-pager
```

Nếu báo cổng 80 đã được sử dụng:
```
Address already in use
```

Kiểm tra tiến trình đang chiếm cổng:
```
sudo ss -lntp 'sport = :80'
```

Nếu sau này Roundcube chạy bằng Docker trên cùng máy, cấu hình cổng container khác cổng 80 của Nginx, ví dụ:
``
Nginx:              192.168.90.12:80
Roundcube Docker:   127.0.0.1:8080
```

