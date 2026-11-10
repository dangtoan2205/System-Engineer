CÀI DOCKER VÀ DOCKER COMPOSE OFFLINE TRÊN UBUNTU SERVER 22.04
----

## 1. Tải file bằng trình duyệt
Mở đường dẫn này trên máy tải được Internet:

[Link tải bộ cài Docker cho Ubuntu 22.04 amd64](https://download.docker.com/linux/ubuntu/dists/jammy/pool/stable/amd64/)

Tải **5 file `.deb`** sau:

| Tên file bắt đầu bằng | Chức năng |
|---|---|
| `containerd.io_` | Dịch vụ chạy container |
| `docker-ce-cli_` | Lệnh Docker |t
| `docker-ce_` | Docker Engine |
| `docker-buildx-plugin_` | Plugin build image |
| `docker-compose-plugin_` | Docker Compose |


Link cài:

[containerd.io_](https://download.docker.com/linux/ubuntu/dists/jammy/pool/stable/amd64/containerd.io_2.4.2-3~ubuntu.22.04~jammy_amd64.deb)

[docker-buildx-plugin_](https://download.docker.com/linux/ubuntu/dists/jammy/pool/stable/amd64/docker-buildx-plugin_0.38.1-1~ubuntu.22.04~jammy_amd64.deb)

[docker-ce-cli_](https://download.docker.com/linux/ubuntu/dists/jammy/pool/stable/amd64/docker-ce-cli_29.9.3-1~ubuntu.22.04~jammy_amd64.deb)

[docker-ce_`](https://download.docker.com/linux/ubuntu/dists/jammy/pool/stable/amd64/docker-ce_29.9.3-1~ubuntu.22.04~jammy_amd64.deb)

[docker-compose-plugin_](https://download.docker.com/linux/ubuntu/dists/jammy/pool/stable/amd64/docker-compose-plugin_5.7.0-1~ubuntu.22.04~jammy_amd64.deb)

## 2. Cài docker offline

```
sudo dpkg -i \
  ./containerd.io_*.deb \
  ./docker-ce-cli_*.deb \
  ./docker-ce_*.deb \
  ./docker-buildx-plugin_*.deb \
  ./docker-compose-plugin_*.deb
```

## 3. Sau khi cài thành công, bật Docker và cấp quyền
```
sudo systemctl enable --now docker
sudo usermod -aG docker "$USER"
newgrp docker
```

Kiểm tra:
```
docker version
docker compose version
docker ps
```
