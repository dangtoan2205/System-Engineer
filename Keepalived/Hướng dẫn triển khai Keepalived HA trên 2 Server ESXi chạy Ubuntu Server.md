Hướng dẫn triển khai Keepalived HA trên 2 Server ESXi chạy Ubuntu Server
----
Triển khai Keepalived Active–Standby sử dụng VRRP và Virtual IP (VIP) trên 2 máy ảo Ubuntu Server, mỗi máy nằm trên một ESXi vật lý riêng biệt.
Mục tiêu là khi một ESXi vật lý bị tắt, lỗi phần cứng hoặc Ubuntu Server gặp sự cố, máy Ubuntu còn lại sẽ tự động tiếp nhận VIP để duy trì dịch vụ.
> Lưu ý quan trọng: Keepalived không tự đồng bộ dữ liệu, không chuyển máy ảo từ ESXi này sang ESXi khác và không tự khôi phục toàn bộ dịch vụ. Nó chủ yếu quản lý việc chuyển VIP và có thể kiểm tra tình trạng dịch vụ để kích hoạt chuyển đổi.

## 1. Sơ đồ mô hình kết nối

<img width="612" height="649" alt="image" src="https://github.com/user-attachments/assets/8fbb166b-226f-4e01-9fc2-7fb1c1b8fc6b" />

**Bảng quy hoạch địa chỉ IP ví dụ**

| Thành phần	| IP / Cấu hình |
| --- | --- |
| ESXi 1	| Máy chủ vật lý số 1 |
| ESXi 2	| Máy chủ vật lý số 2 |
| Ubuntu 1 – MASTER	| 192.168.10.11/24 |
| Ubuntu 2 – BACKUP	| 192.168.10.12/24 |
| Keepalived VIP	| 192.168.10.100/24 |
| Gateway hiện tại	| 192.168.10.1 |
| LAN Port Group	| PG-LAN, VLAN 10 |
| VRRP Virtual Router ID	| 51 |

> Các địa chỉ IP này chỉ là ví dụ, bạn có thể thay bằng dải IP thực tế.

## 2. Cấu hình Network trên 2 ESXi
Trên mỗi ESXi, cần tạo hai mạng riêng biệt.

| Thành phần | vSwitch WAN |	vSwitch LAN |
| --- | --- | --- |
| Physical adapter |	vmnic0 |	vmnic1 |
| Virtual Switch |	vSwitch-WAN |	vSwitch-LAN |
| Port Group | PG-WAN	 | PG-LAN |
| Kết nối |	Switch WAN |	Switch LAN |
| VM Ubuntu |	Card WAN (nếu cần) |	Card LAN |

Thực hiện trên cả hai ESXi:
1. Vào **Host → Configure → Networking → Virtual switches**.
2. Tạo `vSwitch-WAN`, gán uplink `vmnic0`.
3. Tạo `vSwitch-LAN`, gán uplink `vmnic1`.
4. Tạo Port Group `PG-LAN` trên `vSwitch-LAN`, đặt VLAN ID là `10` nếu switch vật lý cấu hình VLAN 10.
5. Kết nối card LAN của mỗi Ubuntu vào `PG-LAN`.

Nếu cổng switch vật lý đang là access VLAN 10 thì đặt VLAN ID của Port Group bằng `0` (không gắn thẻ VLAN trên ESXi).

**Điều kiện bắt buộc**: LAN của hai Ubuntu phải thông nhau ở Layer 2, cùng broadcast domain để VIP có thể chuyển từ máy này sang máy kia. Hai cổng `vmnic1` không nên chỉ nối tới hai mạng LAN tách biệt.
Ở mô hình này, VM Ubuntu chỉ cần một card LAN để chạy Keepalived. Chỉ thêm card WAN nếu Ubuntu thực sự đảm nhận vai trò router, firewall, proxy hoặc dịch vụ cần kết nối WAN.

## 3. Kiểm tra Ubuntu Server trước khi cài đặt
Thực hiện trên cả hai Ubuntu Server.

**Bước 1: Xác định tên card mạng**
```
ip -br addr
ip route
```

Ví dụ Ubuntu hiển thị:
```
ens160    UP    192.168.10.11/24
ens192    UP    10.10.10.11/24
```

Giả sử:
- `ens160`: Card LAN, dùng cho Keepalived.
- `ens192`: Card WAN, nếu VM được cấp mạng WAN.

> Tên card mạng có thể khác nhau giữa các VM. Trong các cấu hình bên dưới, tôi sử dụng ens160 làm ví dụ.

**Bước 2: Kiểm tra kết nối giữa hai Ubuntu**
Từ Ubuntu 1:
```
ping -c 4 192.168.10.12
```

Từ Ubuntu 2:
```
ping -c 4 192.168.10.11
```

> Nếu hai máy chưa ping được nhau, cần kiểm tra lại vSwitch, Port Group, VLAN, switch vật lý và firewall trước khi tiếp tục.

## 4. Cài đặt Keepalived trên cả hai Ubuntu
Áp dụng cho Ubuntu Server 22.04/24.04 hoặc phiên bản tương thích.
```
sudo apt update
sudo apt install keepalived -y
```

Kiểm tra phiên bản:
```
keepalived --version
```

Sao lưu cấu hình cũ nếu có:
```
sudo cp /etc/keepalived/keepalived.conf \
/etc/keepalived/keepalived.conf.bak
```

> Nếu file chưa tồn tại thì có thể bỏ qua bước sao lưu.

## 5. Cấu hình Keepalived trên Ubuntu 1 – MASTER
Mở file:

```
sudo nano /etc/keepalived/keepalived.conf
```

Nhập nội dung:
```
global_defs {
    router_id UBUNTU_MASTER
}

vrrp_instance VI_1 {
    state MASTER
    interface ens160
    virtual_router_id 51
    priority 150
    advert_int 1

    unicast_src_ip 192.168.10.11
    unicast_peer {
        192.168.10.12
    }

    virtual_ipaddress {
        192.168.10.100/24 dev ens160
    }
}
```

Giải thích:

| Tham số |	Chức năng |
| --- | --- |
| `state MASTER`	| Trạng thái khởi tạo |
| `interface ens160`	| Card LAN sử dụng VRRP |
| `virtual_router_id 51`	| ID của nhóm VRRP |
| `priority 150`	| Độ ưu tiên cao hơn Backup |
| `advert_int 1`	| Gửi bản tin VRRP mỗi giây |
| `unicast_src_ip`	| IP thực của Ubuntu 1 |
| `unicast_peer`	| IP thực của Ubuntu 2 |
| `virtual_ipaddress`	| VIP dùng chung |

## 6. Cấu hình Keepalived trên Ubuntu 2 – BACKUP
Mở file:
```
sudo nano /etc/keepalived/keepalived.conf
```

Nhập nội dung:
```
global_defs {
    router_id UBUNTU_BACKUP
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens160
    virtual_router_id 51
    priority 100
    advert_int 1

    unicast_src_ip 192.168.10.12
    unicast_peer {
        192.168.10.11
    }

    virtual_ipaddress {
        192.168.10.100/24 dev ens160
    }
}
```

Các thông số `virtual_router_id`, `advert_int` và VIP phải giống nhau giữa hai node. Priority khác nhau để xác định máy ưu tiên làm MASTER.
Ở đây sử dụng VRRP Unicast để hai Ubuntu gửi thông điệp trực tiếp tới nhau, không phụ thuộc vào việc switch hỗ trợ VRRP multicast. Cấu hình này được Keepalived hỗ trợ chính thức. 


## 7. Cấu hình Firewall và khởi động Keepalived
VRRP sử dụng **IP Protocol 112**, không phải TCP/UDP port 112.

Nếu Ubuntu sử dụng UFW, trên Ubuntu 1 cho phép VRRP từ Ubuntu 2:
```
sudo ufw allow from 192.168.10.12 \
to 192.168.10.11 proto 112
```

Trên Ubuntu 2:
```
sudo ufw allow from 192.168.10.11 \
to 192.168.10.12 proto 112
```

Nếu phiên bản UFW không hỗ trợ cú pháp này, cần tạo rule qua `before.rules` hoặc firewall đang sử dụng. Không cần bật UFW chỉ để chạy Keepalived.

Kiểm tra cú pháp cấu hình trên cả hai máy:
```
sudo keepalived -t \
-f /etc/keepalived/keepalived.conf
```

Nếu hợp lệ, khởi động:
```
sudo systemctl enable --now keepalived
sudo systemctl status keepalived
```

## 8. Kiểm tra hoạt động HA

**Kiểm tra VIP trên Ubuntu 1**
```
ip -br addr show ens160
```

Kết quả mong đợi:
```
ens160 UP 192.168.10.11/24 192.168.10.100/24
```

**Kiểm tra VIP trên Ubuntu 2**
```
ip -br addr show ens160
```

Kết quả mong đợi:
```
ens160 UP 192.168.10.12/24
```

<img width="786" height="312" alt="image" src="https://github.com/user-attachments/assets/c0429f84-374c-4ede-bc62-36bc93704bcb" />

**Thử nghiệm Failover thực tế**

Trên Ubuntu 1:
```
sudo systemctl stop keepalived
```

Sau đó kiểm tra trên Ubuntu 2:
```
ip addr show ens160
```

Ubuntu 2 phải tiếp nhận VIP `192.168.10.100`.

Khởi động lại Keepalived trên Ubuntu 1:
```
sudo systemctl start keepalived
```

Theo cấu hình mặc định, Ubuntu 1 có priority cao hơn nên có thể giành lại MASTER khi hoạt động bình thường.

Để kiểm tra lỗi ở cấp ESXi vật lý, có thể thực hiện trong cửa sổ bảo trì: tắt có kiểm soát ESXi 1 và quan sát Ubuntu 2 tiếp nhận VIP. Không nên thử tắt đột ngột máy chủ production khi chưa có phương án khôi phục.

**Theo dõi log**
```
sudo journalctl -u keepalived -f
```

Kiểm tra bản tin VRRP:
```
sudo tcpdump -ni ens160 'ip proto 112'
```

> Nếu cả hai Ubuntu cùng có VIP, đó là dấu hiệu bất thường, thường liên quan đến VRRP không truyền được giữa hai node, sai VLAN hoặc firewall chặn gói tin.

## 9. Cấu hình nâng cao: Tự động chuyển VIP khi dịch vụ lỗi
Cấu hình ở trên xử lý tình huống ESXi, Ubuntu hoặc Keepalived ngừng hoạt động. Tuy nhiên, nếu Ubuntu vẫn chạy nhưng dịch vụ chính bị lỗi, Keepalived có thể vẫn giữ VIP.

Ví dụ hai Ubuntu chạy Nginx, HAProxy hoặc OpenVPN, bạn nên bổ sung kiểm tra dịch vụ.

Trên cả hai Ubuntu, tạo script:
```
sudo nano /usr/local/bin/check_service.sh
```

Nội dung ví dụ dành cho Nginx:
```
#!/bin/bash

systemctl is-active --quiet nginx
exit $?
```

Phân quyền:
```
sudo chmod 755 /usr/local/bin/check_service.sh
```

Bổ sung vào file `keepalived.conf` trên cả hai node, ở ngoài khối `vrrp_instance`:
```
vrrp_script chk_service {
    script "/usr/local/bin/check_service.sh"
    interval 2
    timeout 2
    fall 2
    rise 2
    user root
}
```

Trong khối `vrrp_instance VI_1`, thêm:
```
track_script {
    chk_service
}
```

> Với cấu hình này, nếu kiểm tra dịch vụ thất bại hai lần liên tiếp, node sẽ chuyển sang trạng thái FAULT và nhường VIP. Đây là cơ chế track_script của Keepalived. 

> Để tránh chuyển VIP khi dịch vụ chưa khởi động, cần bảo đảm dịch vụ Nginx, HAProxy hoặc OpenVPN được cài đặt và cấu hình hoạt động trên cả hai Ubuntu.

