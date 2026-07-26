RAID (Redundant Array of Independent Disks) là kỹ thuật kết hợp nhiều ổ đĩa vật lý thành một hệ thống lưu trữ logic nhằm đạt được một hoặc nhiều mục tiêu sau:
---
- Tăng hiệu năng đọc/ghi.
- Tăng khả năng chịu lỗi khi ổ cứng bị hỏng.
- Tăng dung lượng lưu trữ.

Điều quan trọng cần nhớ là:

> *RAID không phải là Backup.*

Nếu xóa nhầm dữ liệu, bị ransomware hoặc file bị lỗi thì RAID vẫn đồng bộ lỗi đó sang các ổ khác.

## 1. RAID hoạt động như thế nào?

Giả sử bạn có 4 ổ HDD:

- Disk 1 - 1TB
- Disk 2 - 1TB
- Disk 3 - 1TB
- Disk 4 - 1TB

Thay vì hệ điều hành nhìn thấy 4 ổ riêng biệt, RAID Controller sẽ gộp chúng lại thành một Logical Drive.

<img width="264" height="215" alt="image" src="https://github.com/user-attachments/assets/c8483f1e-b1ae-4f10-a860-1a1db15adb2e" />

Controller sẽ quyết định:

- dữ liệu ghi xuống ổ nào
- ghi theo thứ tự nào
- có nhân bản dữ liệu không
- có tính toán Parity không

Đó chính là cơ chế hoạt động của RAID.

## 2. Các loại RAID

| RAID    | Tối thiểu ổ | Dung lượng khả dụng (ổ cùng kích thước) | Chịu lỗi                                          | Hiệu năng                    |
| ------- | ----------: | --------------------------------------: | ------------------------------------------------- | ---------------------------- |
| RAID 0  |           2 |                         Tổng dung lượng | 0 ổ                                               | Đọc/Ghi rất cao              |
| RAID 1  |           2 |                     50% tổng dung lượng | 1 ổ trong mỗi cặp                                 | Đọc cao, ghi trung bình      |
| RAID 5  |           3 |                    (N−1) × dung lượng ổ | 1 ổ                                               | Đọc cao, ghi khá             |
| RAID 6  |           4 |                    (N−2) × dung lượng ổ | 2 ổ                                               | Đọc cao, ghi thấp hơn RAID 5 |
| RAID 10 |           4 |                     50% tổng dung lượng | Có thể hỏng nhiều ổ nếu không cùng một cặp mirror | Đọc/Ghi rất cao              |

## 3. RAID nào phù hợp trong thực tế?

Đối với các môi trường doanh nghiệp, lựa chọn RAID thường phụ thuộc vào mục tiêu chính:

- Máy chủ ảo hóa (VMware ESXi, Hyper-V): RAID 10 thường được ưu tiên nhờ hiệu năng đọc/ghi cao và thời gian phục hồi nhanh.
- File Server/NAS: RAID 5 (ưu tiên dung lượng) hoặc RAID 6 (ưu tiên an toàn với ổ dung lượng lớn).
- Máy chủ cơ sở dữ liệu: RAID 10 để đảm bảo độ trễ thấp và hiệu năng ghi tốt.
- Lưu trữ dữ liệu ít thay đổi, cần dung lượng lớn: RAID 6 vì có thể chịu hỏng đồng thời hai ổ.
- Dữ liệu tạm thời hoặc không quan trọng: RAID 0 để tối đa hóa tốc độ.

----
## 3. Các bước thực hiện RAID

1/ Vào Bios -> Chọn F2 (Entering System Setup) 


2/ Tại cửa sổ System Setup Main Menu -> Chọn Device Settings


3/ Tại cửa sổ Device Settings -> Chọn Integrated RAID Controller 1: Dell PERC <PERC H730P Adapter> Configuration Utility


4/ Trước tiên phải kiểm tra tổng số lượng ổ cứng đang có trên server tại Main Menu

- Chọn Physical Disk Management (để kiểm tra tổng số lượng ổ cứng hiện có trên server)
  - Kiểm tra được loại ổ cứng (SSD hoặc HDD), kiểm tra được kiểu ổ cứng (SATA hoặc NVme), kiểm tra dung lượng ổ cứng, trạng thái

- Chọn Virtual Disk Management (để kiểm tra tình trạng ổ cứng đã được RAID chưa, RAID loại nào (RAID0, RAID1,...), đã gộp những ổ nào với nhau 

5/ Trường hợp đã có sẵn RAID trước đó thì tiến hành xóa ổ RAID hiện tại để format dữ liệu và tạo lại ổ RAID mới

-> Tại cửa sổ Main Menu -> Chọn Virtual Disk 0: RAID0, 893.250GB, Ready

6/ Tiến hành xóa ổ RAID 

-> Tại tùy chọn Operation -> Chọn Select operation -> Chọn Delete Virtual Disk


-> Chọn Go ngay bên dưới tùy chọn OperationManagerment


-> Nhấn phím Space để tích chọn Confirm -> Sau đó chọn Yes để tiến hành xác nhận xóa ổ

7/ Sau khi đã xóa các RAID có sẵn tiến hành gộp ổ RAID

Tại cửa sổ Main Menu -> Chọn Configuration Management -> Chọn Create Virtual Disk (để tạo RAID ổ cứng mới)

Tại Create Virtual Disk  -> Select RAID Level -> Chọn loại RAID (RAID0 hoặc RAID1)

Chọn Select Physical Disk (để chọn các loại ổ có sẵn gộp vào RAID)

Tích chọn ổ cứng mong muốn cho vào RAID đang tạo

Sau khi chọn xong thì chọn Apply Changes -> Chọn Create Virtual Disk để xác nhận -> Tích chọn Confirm -> Chọn Yes

8/ Kiểm tra xem đã tạo RAID thành công chưa ở tab Main Menu -> Virtual Disk Managerment

