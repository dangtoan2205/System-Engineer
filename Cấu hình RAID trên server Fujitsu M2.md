Hướng dẫn cấu hình RAID trên server Primergy RX2530 M2
-----

## 1. Khi bắt đầu khởi động server bấm phím
```
Ctrl + R
```
để vào menu RAID

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/cfe59425-96bb-4059-a75b-6ba8db003b0e" />

## 2. Tại cửa sổ `VD Mgmt` hiện tại tiến hành xóa cấu hình RAID có sẵn
Di chuyển xuống vị trí
```
ID: 0, OSG1, 931.00 GB
```

Tiếp theo
```
Nhấn phím F2
```

Tiếp tục, chọn
```
Delete VD
```
Để tiến hành xóa Raid 

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/3cc278a6-1257-4352-bcbd-4cff749cca40" />

Nhấn chọn `Yes` để xác nhận xóa

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/2d926497-2906-4362-81b5-4fa7f1eff6b6" />

## 3. Nhấn `Ctrl + N` để chuyển sang Tab `PD Mgt` để kiểm tra trạng thái Raid của ổ cứng

Nếu `State` -> là `UG` thì đã xóa RAID thành công

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/f5bdfc93-9960-4523-b8c3-3f78820838b7" />

## 4. Tiến hành tạo RAID mới

Nhấn 
```

Ctrl + N
```

Để trở về Tab `VD Mgmt`

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/a3526d72-fd7d-4361-840a-a289e45004de" />

Tạo RAID mới bằng cách chọn
```
PRAID CP400i(Bus 0x01, Dev 0x00)
```

Nhấn chọn phím
```
F2
```

Chọn tiếp
```
Create Virtual Drive
```
Để tạo RAID mới

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/34aa8935-1310-440e-8cc4-bb691b85fd64" />

## 5. Chọn RAID phù hợp

- Nếu chỉ có 2 ổ cứng thì chọn RAID 1
- Nếu có 3 ổ cứng thì chọn RAID 5

Trường hợp này chọn RAID 1 đồng thời tích chọn 2 ổ cứng đang có sẵn để tạo thành RAID 1

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/dde06e3b-3b2c-450e-ad92-7b8a34d94fa3" />

Nhấn chọn `OK` để xác nhận 

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/10def0da-9270-4aee-b6ce-c76a129c9293" />

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/b4604fe5-1ad4-4a93-8c78-b35b9c89e0b7" />

## 6. Nhấn ESC để thoát ra và truy cập boot cài hệ điều hành bình thường
