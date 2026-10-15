Cài đặt và cấu hình pfSense
-----

## Bước 1. Cài đặt hệ điều hành
1. Tải bộ cài pfSense CE 2.7.2 phù hợp với máy chủ.
2. Tạo USB boot hoặc sử dụng thiết bị quản trị từ xa để gắn bộ cài.
3. Khởi động server từ bộ cài.
4. Tiến hành cài đặt lên ổ đĩa đã chuẩn bị.
5. Hoàn tất cài đặt và khởi động lại.

## Bước 2. Gán Interface
Tại Console pfSense:

Chọn `1) Assign Interfaces`.

Nhập lần lượt:

```
Should VLANs be set up now? n

WAN interface: igb0
LAN interface: igb1
Optional interface: [Enter]

Do you want to proceed? y
```

Kết quả:

```
WAN -> igb0
LAN -> igb1
```

## Bước 3. Cấu hình WAN

Tại Console chọn `2) Set interface(s) IP address`.

Chọn `1 - WAN`.

```
Configure IPv4 via DHCP? y
Configure IPv6 via DHCP6? n
IPv6 Address: [Enter]
Revert to HTTP? n
```

> WAN sẽ nhận địa chỉ IP tự động từ DHCP Server phía upstream.

## Bước 4. Cấu hình LAN
Chọn `2) Set interface(s) IP address`, sau đó chọn `2 - LAN`.

```
Configure IPv4 via DHCP? n
LAN IPv4 Address: 192.168.1.254
Subnet bit count: 24
Upstream Gateway: [Enter]
Configure IPv6 via DHCP6? n
IPv6 Address: [Enter]
Enable DHCP Server on LAN? n
Revert to HTTP? n
```

Kết quả:
```
WAN (igb0) -> DHCP
LAN (igb1) -> 192.168.1.254/24
```

## Bước 5. Truy cập WebGUI
Cấu hình IP tĩnh trên laptop:
```
IP Address: 192.168.1.20
Subnet Mask: 255.255.255.0
Gateway: 192.168.1.254
```

Truy cập:
```
https://192.168.1.254
```

Tài khoản mặc định nếu chưa thay đổi:
```
Username: admin
Password: pfsense
```

> Đổi mật khẩu quản trị sau khi đăng nhập.

----
Cấu hình Firewall và NAT
----

## Bước 1. Cấu hình LAN Rule

Truy cập **Firewall → Rules → LAN**.

Nếu chưa có rule cho phép LAN, tạo rule:

| Thông số |	Giá trị |
| --- | --- |
| Action |	Pass |
| Interface	| LAN |
| Address Family |	IPv4 |
| Protocol	| Any |
|Source	LAN | subnets |
|Destination	| Any |
|Description	| Allow LAN to Any |

Chọn **Save → Apply Changes**.

> Rule này phù hợp với mạng lab tin cậy; khi triển khai production cần giới hạn theo nhu cầu thực tế.

## Bước 2. Cấu hình Outbound NAT

**Truy cập Firewall → NAT → Outbound**.

Chọn:
```
Automatic outbound NAT rule generation
```

Nhấn **Save → Apply Changes**.

> Chế độ này giúp pfSense tự động tạo quy tắc NAT cho mạng LAN đi ra WAN. Việc truy cập Internet còn phụ thuộc kết nối WAN, DNS và chính sách phía upstream.
