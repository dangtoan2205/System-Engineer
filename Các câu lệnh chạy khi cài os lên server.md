Các câu lệnh cần thiết khi cài đặt và cập nhật Ubuntu Server
----

1/ Update các gói tin hệ thống (recomend)

```
sudo apt update && sudo apt full-upgrade -y
```

2/ update + full-upgrade + dọn các gói không cânf thiết

```
sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y && sudo apt autoclean
```


Tắt nguồn Ubuntu Server ngay lập tức
----

```
sudo poweroff
```

hoặc
```
sudo shutdown -h now
```

Các câu lệnh cài Docker + Compose
----

```
sudo apt update
sudo apt install -y docker.io docker-compose-v2
sudo systemctl enable --now docker
```

Kiểm tra

```
docker --version
docker compose version
sudo systemctl status docker
```
