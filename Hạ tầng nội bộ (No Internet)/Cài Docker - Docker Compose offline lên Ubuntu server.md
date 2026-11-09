CÀI DOCKER VÀ DOCKER COMPOSE OFFLINE TRÊN UBUNTU SERVER
----

## 1. Kiểm tra phiên bản Ubuntu Server hiện tại
```
cat /etc/os-release
```
> Ví dụ: Phiên bản đang cài cho Ubuntu (Ubuntu Server 26.04)

## 2. Nguồn tải bộ cài
Trên máy có Internet, truy cập trang hướng dẫn:

[Link bộ cài docker offline](https://docs.cyberwatch.com/help/en/deploy/offline/swarm/install_docker/)

Tại phần **On Debian-based distributions**, chọn liên kết Ubuntu tương ứng với phiên bản hệ điều hành của máy cần cài.

[Tài liệu chính thức của Docker](https://docs.docker.com/engine/install/ubuntu/#install-from-a-package)

## 3. Chuẩn bị các gói cài đặt
Các gói sử dụng
| Gói | Phiên bản trong lần cài | Chức năng |
|---|---|---|
| `containerd.io` | `2.4.2-3` | Runtime container |
| `docker-ce-cli` | `29.9.3-1` | Công cụ dòng lệnh Docker |
| `docker-ce` | `29.9.3-1` | Docker Engine |
| `docker-buildx-plugin` | `0.38.1-1` | Hỗ trợ build image |
| `docker-compose-plugin` | `5.7.0-1` | Quản lý ứng dụng bằng Compose |

**Lưu ý**: Phải tải đủ các gói cài gồm: `containerd.io` , `docker-ce-cli` , `docker-ce` , `docker-buildx-plugin` , `docker-compose-plugin`. Và chọn đúng version.

**File `.deb` là gói cài đặt, không cần giải nén.**

## 4. Chuyển bộ cài vào Ubuntu Server
Tạo thư mục:
```
mkdir -p /home/ubuntu/docker-offline
```

Dùng **USB**, **SCP** hoặc **SFTP** để chuyển năm file `.deb` vào thư mục:
```
/home/ubuntu/docker-offline
```

Truy cập thư mục và kiểm tra:
```
cd /home/ubuntu/docker-offline
ls -lh
```

Chỉ để các gói Docker cần cài trong thư mục này. Mỗi gói chỉ nên có một phiên bản.

## 5. Cài Docker bằng dpkg
Chạy:
```
cd /home/ubuntu/docker-offline
```

Sau đó chạy nguyên khối lệnh:
```
sudo dpkg -i \
  ./containerd.io_*.deb \
  ./docker-ce-cli_*.deb \
  ./docker-ce_*.deb \
  ./docker-buildx-plugin_*.deb \
  ./docker-compose-plugin_*.deb
```

**Lưu ý**: dấu `\` phải nằm cuối dòng, không thêm khoảng trắng hoặc ký tự phía sau.

Dấu hiệu cài thành công là xuất hiện các dòng:
```
Setting up containerd.io ...
Setting up docker-ce-cli ...
Setting up docker-ce ...
Setting up docker-buildx-plugin ...
Setting up docker-compose-plugin ...
```
Không có thông báo lỗi phụ thuộc hoặc lỗi cấu hình ở cuối.

## 6. Khởi động Docker và bật tự chạy
Chạy:
```
sudo systemctl enable --now docker
```

Kiểm tra trạng thái:
```
sudo systemctl is-active docker
```

Kết quả mong đợi:
```
active
```

Kiểm tra tự khởi động cùng Ubuntu:
```
sudo systemctl is-enabled docker
```

Kết quả mong đợi:
```
enabled
```

Xem trạng thái chi tiết:
```
sudo systemctl status docker --no-pager -l
```

## 7. Kiểm tra Docker Engine và các plugin

Kiểm tra Docker Engine:
```
sudo docker version
```
Kiểm tra Docker Compose:
```
docker compose version
```

Kiểm tra danh sách container:
```
sudo docker ps
```

Kiểm tra Buildx:
```
docker buildx version
```

**Lưu ý**: lệnh sau chỉ hiển thị phiên bản Docker CLI, chưa đủ xác nhận Docker Engine đang chạy:
```
docker -v
```

## 10. Cấu hình quyền cho user ubuntu
Mặc định, có thể dùng Docker với sudo:
```
sudo docker ps
```

Nếu muốn tài khoản ubuntu sử dụng Docker không cần sudo, chạy:
```
sudo usermod -aG docker ubuntu
```

Sau đó **đăng xuất SSH và đăng nhập lại**.

Kiểm tra nhóm:
```
id
```

Kiểm tra Docker:
```
docker ps
```

Nhóm `docker` có quyền quản trị tương đương root trên máy, vì vậy chỉ thêm tài khoản quản trị được tin cậy.

