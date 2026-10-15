Bảng so sánh Keepalived và HAProxy
----
**Keepalived** và **HAProxy** đều được sử dụng trong hệ thống High Availability (HA), nhưng có nhiệm vụ khác nhau:
- **Keepalived**: Đảm bảo tính sẵn sàng cao bằng cách quản lý IP ảo (VIP), tự động chuyển đổi giữa các server khi xảy ra sự cố (Failover).
- **HAProxy**: Cân bằng tải (Load Balancing), phân phối các kết nối đến nhiều server và kiểm tra trạng thái backend.

## 1. Bảng so sánh chi tiết
| Tiêu chí | Keepalived | HAProxy |
| --- | --- | --- |
| Chức năng chính | High Availability, Failover | Load Balancing, Reverse Proxy |
| Mục đích | Đảm bảo dịch vụ không bị gián đoạn khi một node lỗi | Phân phối lưu lượng và tăng khả năng phục vụ |
| Cơ chế hoạt động | VRRP, VIP, health-check | TCP/HTTP proxy, health-check |
| IP ảo (VIP) | Hỗ trợ trực tiếp | Không tự quản lý VIP failover |
| Chuyển đổi dự phòng | Có, tự động | Có thể chuyển backend lỗi, không tự chuyển VIP giữa hai máy |
| Cân bằng tải | Không phải chức năng chính; có thể kết hợp IPVS | Có, hỗ trợ nhiều thuật toán |
| Active–Passive | Có | Có, ở cấp backend |
| Active–Active | Không với cùng một VIP VRRP; có thể chia nhiều VIP | Có, phân phối đến nhiều backend |
| Health Check | Kiểm tra tiến trình, script, trạng thái node | Kiểm tra TCP, HTTP, SSL/TLS và backend |
| Giao thức | VRRP, sử dụng IPVS khi cấu hình LVS | TCP, HTTP, HTTPS và nhiều giao thức chạy trên TCP |
| SSL/TLS Termination | Không | Có |
| Reverse Proxy | Không | Có |
| Logging, thống kê | Cơ bản | Chi tiết, có trang thống kê |
| Cài trên Ubuntu | Có | Có |
| Chi phí | Miễn phí, mã nguồn mở	 | Bản Community miễn phí |
| Độ phức tạp | Thấp đến trung bình | Trung bình đến cao |
| Khả năng kết hợp | Có thể giám sát và bảo vệ HAProxy | Có thể chạy cùng Keepalived |

## 2. So sánh theo tình huống thực tế

| Tình huống |	Keepalived	| HAProxy |
| --- | --- | --- |
| Server 1 bị tắt nguồn |	VIP chuyển sang Server 2 |	Không tự chuyển VIP
| Dịch vụ trên Server 1 bị lỗi	| Có thể failover nếu cấu hình kiểm tra dịch vụ	| Có thể loại backend lỗi |
| Có 1000 request đồng thời	| Không tự chia request giữa hai node VRRP	| Phân phối request theo thuật toán |
| Cần một IP duy nhất để client truy cập |	Phù hợp	| Cần VIP hoặc giải pháp IP cố định bên ngoài |
| Muốn hai backend cùng xử lý request	| Không phải chức năng của VRRP	| Phù hợp |
| Muốn hệ thống vừa HA vừa Load Balancing	| Cần kết hợp thành phần cân bằng tải	| Cần kết hợp cơ chế HA cho chính HAProxy |

## 3. Mô hình 2 ESXI
<img width="614" height="610" alt="image" src="https://github.com/user-attachments/assets/2623c579-19db-48fb-b069-f4d296df978c" />

Với mô hình gồm 2 server vật lý chạy ESXi, mỗi server có một Ubuntu VM:
- Cài **Keepalived + HAProxy trên cả 2 Ubuntu VM**.
- Keepalived quản lý VIP, tự động chuyển từ VM1 sang VM2 khi VM1 hoặc dịch vụ quan trọng gặp sự cố.
- HAProxy tiếp nhận kết nối và chuyển tiếp đến backend phù hợp.
- Cấu hình health-check HAProxy trong Keepalived để tránh trường hợp VIP vẫn ở node có HAProxy bị lỗi.
**Lưu ý**: Hai Ubuntu VM cần có kết nối Layer 2 phù hợp cho VIP VRRP, hoặc phải thiết kế cơ chế failover định tuyến khác. Nếu sử dụng cả WAN và LAN, cần xác định VIP nằm trên mạng nào và đảm bảo đường đi hai chiều. Keepalived không tự đồng bộ trạng thái kết nối hay dữ liệu ứng dụng giữa hai VM.

## 4. Kết luận lựa chọn
| Nhu cầu triển khai	| Giải pháp | 
| --- | --- |
| Chỉ cần server dự phòng tự động	| Keepalived |
| Chỉ cần cân bằng tải giữa các backend	| HAProxy |
| Vừa dự phòng server vừa cân bằng tải	| Keepalived + HAProxy |
| Muốn hai Ubuntu VM hoạt động dự phòng cho dịch vụ proxy	| Keepalived + HAProxy |
