Cấu hình RAID và boot cài hệ điều hành trên server Fujitsu M5
-----

## 1/ Khởi động nguồn server và nhấn phím F2 để vào màn hình Setup

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/27b100f2-c4bc-449c-899a-af68dcedb751" />


## 2/ Trên màn hình Setup -> chọn tab Advanced -> chọn tiếp -> SATA Configuration

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/40be380f-267f-48b5-86dc-9acf6e443389" />


## 3/ Tại màn hình SATA Configuration

SATA Mode hiện tại đang là [AHCI Mode] chuyển thành [RAID Mode] -> bấm Esc để thoát ra

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/dc4218e9-96e1-4590-bec6-857d653b5249" />

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/a1e1da99-2f64-4082-b0c6-da10544d1f5c" />

## 4/ Sau khi thoát ra tab Advanced -> chọn AVAGO MegaRAID <PRAID CP400i> Configuration Utility - 03.25.05.10 

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/4af378b0-6d2e-4d4d-a7f3-065d4ad4d2c7" />

## 5/ Chọn Main Menu (để vào màn hình chế độ RAID)

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/c8c50f40-2a38-4470-b562-81832b7371f8" />

## 6/ Tiến hành xóa RAID đang có 

Chọn Configuration Management -> chọn Clear Configuration 

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/e4751e82-531e-49a0-a711-f4e129b6d494" />

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/2efd5bd5-e265-42dc-bdc5-bee8259233ac" />

Tại màn hình này nhấn Enter để chuyển Confirm từ [Disabled] sang [Enabled] 

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/71535fa2-8db2-4d6f-8301-fb8334ddbb60" />

Sau khi chuyển sang [Enable] thì chọn Yes -> chọn OK lần nữa để xác nhận xóa các RAID hiện tại

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/85b8854d-ffda-4207-b578-32ceecf0dc69" />

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/b03c7a97-fa41-4dee-a845-2a6b89260d9a" />

## 7/ Sau đó chọn Virtual Drive Management (để kiểm tra đã xóa RAID thành công chưa)

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/d0f42ece-cab8-49d9-b6d8-d3140dfe9a82" />

Như hình ảnh là đã xóa RAID cũ thành công

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/4cfbb8f5-eccc-48f0-9431-7a614cf1e9e2" />

## 8/ Tiến hành tạo lại RAID mới. Chọn Configuration Management -> chọn Create Virtual Drive

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/b5a2a0fc-99c8-4c88-8ad7-1c9eba21d312" />

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/03b37218-4af3-4c46-adea-95eee96dd9a5" />

- Select RAID Level (Chọn RAID mong muốn) (Trong trường hợp này là cài 1 os ubuntu server mà trên server đang chỉ có 2 ổ cứng 300Gb -> thì chọn RAID 1)

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/7e381a36-af33-4fd9-be00-0c83741916a0" />

- Select Drives (Chọn ổ cứng mong muốn gộp thành RAID 1) (Nếu chọn hết thì chọn Check All) -> chọn Apply Changes -> chọn OK (để xác nhận)

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/c9c34115-1f75-459a-9efa-9da228b9619c" />

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/ef5207a1-af70-46e4-b00c-1258196a6d37" />

- Sau đó chọn Save Configuration -> Confirm (chuyển Disabled -> Enabled) -> chọn Yes -> chọn OK (để xác nhận tạo RAID).

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/885c112e-840c-4829-83bf-6f5d1cc28139" />

## 9/ Kiểm tra xem đã tạo RAID thành công hay chưa

 Chọn Virtual Drive Management (Như hình ảnh là đã được tạo RAID1 thành công)

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/68d478d8-8931-4286-b093-3f350d9e5ce4" />

## 10/ Chế độ SATA Mode hiện tại đang là [RAID Mode] cần chuyển thành [AHCI Mode] để có thể boot OS từ USB

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/36de41f4-fbe5-4da6-b346-e207456a8af9" />

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/d559d435-0b2c-4f03-aadd-09a4458710d2" />

 Bấm Esc để thoát về màn hình tab Advanced -> Chọn SATA Mode -> chuyển [RAID Mode] thành [AHCI Mode]

 ## 11/ Bấm Esc để thoát và chuyển sang tab Save & Exit -> chọn Save Changes and Exit (để lưu cấu hình và thoát khỏi màn hình Enter Setup)

 ## 12/ Khi server khởi động lại chọn phím F12 để vào Menu boot -> Chọn boot bằng USB -> Cài os như bình thường.
