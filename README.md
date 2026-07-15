# Mô Phỏng Hạ Tầng Mạng Doanh Nghiệp Đa Chi Nhánh – Cisco Packet Tracer

## Giới thiệu
Dự án mô phỏng hạ tầng mạng cho một doanh nghiệp có **3 chi nhánh/site** được kết nối với nhau thông qua đường truyền WAN (Serial link), sử dụng **Cisco Packet Tracer**. Hệ thống được thiết kế với sơ đồ địa chỉ IP rõ ràng, triển khai dịch vụ **DHCP phân tán tại từng site** và **DNS tập trung** phục vụ toàn bộ hệ thống, mô phỏng đúng mô hình mạng WAN thực tế mà doanh nghiệp có nhiều văn phòng/chi nhánh thường sử dụng.

## Sơ đồ mạng
```
      [Site 1: 192.168.1.0/24]                [Site 2: 192.168.2.0/24]              [Site 3: 192.168.3.0/24]
       Server0 (DHCP) + PC1                     Server2 (DNS trung tâm)               Server1 (DHCP) + PC0
              |                                          |                                     |
          Router0 ──────Se 10.0.0.0/8────────────── Router1 ──────Se 20.0.0.0/8────────── Router2
        (Gi0/0/0: .1.1)                    (Gi0/0/0: .2.1)                        (Gi0/0/0: .3.1)
```

## Bảng địa chỉ IP (IP Addressing Table)

| Thiết bị | Interface | Địa chỉ IP | Subnet Mask | Vai trò |
|---|---|---|---|---|
| Router0 | GigabitEthernet0/0/0 | 192.168.1.1 | 255.255.255.0 | Gateway – Site 1 |
| Router0 | Serial0/1/0 | 10.0.0.1 | 255.0.0.0 | WAN link tới Router1 |
| Router1 | GigabitEthernet0/0/0 | 192.168.2.1 | 255.255.255.0 | Gateway – Site 2 |
| Router1 | Serial0/1/0 | 10.0.0.2 | 255.0.0.0 | WAN link tới Router0 |
| Router1 | Serial0/1/1 | 20.0.0.1 | 255.0.0.0 | WAN link tới Router2 |
| Router2 | GigabitEthernet0/0/0 | 192.168.3.1 | 255.255.255.0 | Gateway – Site 3 |
| Router2 | Serial0/1/0 | 20.0.0.2 | 255.0.0.0 | WAN link tới Router1 |
| Server0 | FastEthernet0 | 192.168.1.10 | 255.255.255.0 | DHCP Server – Site 1 |
| Server2 | FastEthernet0 | 192.168.2.10 | 255.255.255.0 | DNS Server trung tâm |
| Server1 | FastEthernet0 | 192.168.3.10 | 255.255.255.0 | DHCP Server – Site 3 |
| PC1 | FastEthernet0 | DHCP (192.168.1.2) | 255.255.255.0 | Client – Site 1 |
| PC0 | FastEthernet0 | DHCP | 255.255.255.0 | Client – Site 3 |

## Công nghệ và kỹ thuật triển khai
- **Định tuyến WAN (Serial Link)** giữa 3 Router, sử dụng dải địa chỉ classful riêng cho từng liên kết (10.0.0.0/8, 20.0.0.0/8) để dễ phân biệt và quản lý.
- **DHCP Server phân tán**: mỗi site tự vận hành DHCP Server riêng (Server0 tại Site 1, Server1 tại Site 3), cấp phát IP động cho các máy trạm trong mạng LAN nội bộ.
- **DNS tập trung**: toàn bộ hệ thống (cả 3 site) sử dụng chung một DNS Server đặt tại Site 2 (Server2 – 192.168.2.10), khai báo A Record cho `yahoo.com` và `gmail.com` trỏ đúng tới Mail Server tương ứng.
- **Dịch vụ Email nội bộ (SMTP/POP3)**: triển khai 2 Mail Server mô phỏng 2 domain khác nhau –  Server0 phục vụ domain `yahoo.com` (Site 1), Server1 phục vụ domain `gmail.com` (Site 3) – cho phép người dùng ở 2 site khác nhau gửi/nhận email qua lại với nhau thông qua hệ thống DNS trung tâm.
- **Cấu hình Gateway/DNS trên client và server** đảm bảo mọi thiết bị định tuyến đúng ra ngoài site và phân giải tên miền thông qua DNS trung tâm.

## Điểm nổi bật trong thiết kế
- **Tách biệt vai trò dịch vụ theo site**: DHCP và Mail Server đặt gần người dùng (giảm độ trễ, dễ khoanh vùng sự cố), trong khi DNS đặt tập trung (dễ quản lý, đồng bộ dữ liệu tên miền toàn hệ thống) – đúng nguyên tắc thiết kế hạ tầng mạng doanh nghiệp nhiều chi nhánh.
- **Tích hợp đầy đủ chuỗi dịch vụ mạng**: DHCP (cấp IP) → DNS (phân giải tên miền) → SMTP/POP3 (gửi/nhận email) – mô phỏng đúng luồng hoạt động thực tế của một hệ thống mạng doanh nghiệp hoàn chỉnh, không chỉ dừng ở cấu hình định tuyến cơ bản.
- **Đặt IP WAN theo dải riêng biệt** (10.x, 20.x) giúp tránh xung đột với các dải LAN nội bộ (192.168.x), thuận tiện cho việc đọc bảng định tuyến và troubleshooting.
- **Khả năng mở rộng**: mô hình cho phép dễ dàng thêm site mới (Router3, Router4...) bằng cách nối thêm Serial link mà không ảnh hưởng tới cấu trúc hiện có.
- **Đã kiểm thử thực tế**: xác nhận DNS phân giải đúng qua lệnh `ping` tên miền, và xác nhận luồng email hoạt động đầu-cuối (end-to-end) giữa 2 client ở 2 site khác nhau.

## Các bước triển khai chính
1. Thiết kế sơ đồ mạng WAN 3 site, xác định vai trò từng Router và Server.
2. Gán địa chỉ IP tĩnh cho các cổng LAN (GigabitEthernet) và WAN (Serial) trên từng Router.
3. Cấu hình Clock Rate trên các cổng Serial (DCE) để thiết lập đường truyền WAN.
4. Cài đặt và cấu hình dịch vụ DHCP tại Server0 (Site 1) và Server1 (Site 3), khai báo Default Gateway và DNS Server tương ứng.
5. Cấu hình DNS Server tại Server2 (Site 2), khai báo A Record cho `yahoo.com` và `gmail.com` trỏ tới đúng Mail Server tương ứng.
6. Bật dịch vụ SMTP/POP3 trên Server0 (domain `yahoo.com`) và Server1 (domain `gmail.com`), tạo tài khoản người dùng cho từng domain.
7. Gán DHCP client cho PC0, PC1 và kiểm tra khả năng nhận IP tự động.
8. Cấu hình Mail Client trên PC1 (`duy@yahoo.com`) và PC0 (`duc@gmail.com`), thực hiện gửi/nhận email thử nghiệm.
9. Kiểm tra kết nối liên site bằng lệnh `ping` tên miền (`ping yahoo.com`, `ping gmail.com`) để xác nhận DNS phân giải chính xác.

## Kết quả đạt được
- Các máy trạm (PC0, PC1) nhận địa chỉ IP tự động chính xác từ DHCP Server tương ứng tại site của mình.
- Toàn bộ hệ thống 3 site kết nối và định tuyến thông suốt qua các liên kết WAN.
- DNS Server trung tâm phục vụ phân giải tên miền cho cả 3 site, đảm bảo tính nhất quán dữ liệu (xác nhận qua lệnh `ping` tên miền thành công, TTL và RTT ổn định).
- Hệ thống Email nội bộ hoạt động end-to-end: PC1 (Site 1, domain yahoo.com) gửi và nhận thành công email với PC0 (Site 3, domain gmail.com) thông qua DNS trung tâm và 2 Mail Server SMTP/POP3.
- Mô hình phản ánh đúng thực tế vận hành mạng của doanh nghiệp có nhiều chi nhánh/văn phòng, với đầy đủ chuỗi dịch vụ DHCP – DNS – Email.

## Kỹ năng thể hiện qua dự án
- Thiết kế và triển khai hạ tầng mạng WAN nhiều site.
- Xây dựng bảng địa chỉ IP (IP Addressing Table) khoa học, dễ quản lý.
- Cấu hình và vận hành dịch vụ DHCP, DNS, Email (SMTP/POP3) trên Cisco Packet Tracer.
- Hiểu và triển khai được luồng hoạt động liên kết giữa các dịch vụ mạng (DHCP → DNS → Email) trong một hệ thống thực tế.
- Kỹ năng kiểm tra, chẩn đoán kết nối mạng liên site (troubleshooting) bằng công cụ dòng lệnh.

## Hình ảnh minh chứng

### Router0 – Site 1 Gateway
![Router0 - GigabitEthernet0/0/0](img_project_cisco/img1.png)
*Cấu hình cổng GigabitEthernet0/0/0: 192.168.1.1/24 – Gateway cho Site 1*

![Router0 - Serial0/1/0](img_project_cisco/img2.png)
*Cấu hình cổng Serial0/1/0: 10.0.0.1/8 – Liên kết WAN tới Router1*

### Router1 – Site 2 Gateway (kết nối trung tâm)
![Router1 - GigabitEthernet0/0/0](img_project_cisco/img3.png)
*Cấu hình cổng GigabitEthernet0/0/0: 192.168.2.1/24 – Gateway cho Site 2*

![Router1 - Serial0/1/0](img_project_cisco/img4.png)
*Cấu hình cổng Serial0/1/0: 10.0.0.2/8 – Liên kết WAN tới Router0*

![Router1 - Serial0/1/1](img_project_cisco/img5.png)
*Cấu hình cổng Serial0/1/1: 20.0.0.1/8 – Liên kết WAN tới Router2*

### Router2 – Site 3 Gateway
![Router2 - GigabitEthernet0/0/0](img_project_cisco/img6.png)
*Cấu hình cổng GigabitEthernet0/0/0: 192.168.3.1/24 – Gateway cho Site 3*

![Router2 - Serial0/1/0](img_project_cisco/img7.png)
*Cấu hình cổng Serial0/1/0: 20.0.0.2/8 – Liên kết WAN tới Router1*

![Router2 - Serial0/1/1](img_project_cisco/img8.png)
*Cấu hình cổng Serial0/1/1: 30.0.0.2/8 – Liên kết WAN mở rộng*

### Server0 – DHCP Server tại Site 1
![Server0 - Global Settings](img_project_cisco/img9.png)
*Thiết lập Gateway/DNS: Default Gateway 192.168.1.1, DNS Server 192.168.2.10*

![Server0 - FastEthernet0](img_project_cisco/img10.png)
*Địa chỉ IP tĩnh: 192.168.1.10/24*

![Server0 - Dịch vụ DHCP](img_project_cisco/img11.png)
*Cấu hình DHCP Pool "serverPool": cấp phát IP từ 192.168.1.x, tối đa 10 người dùng*

### PC1 – Client tại Site 1
![PC1 - Global Settings](img_project_cisco/img12.png)
*Cấu hình nhận IP tự động qua DHCP*

![PC1 - FastEthernet0](img_project_cisco/img13.png)
*Kết quả: PC1 nhận địa chỉ IP 192.168.1.2/24 từ DHCP Server*

### Server2 – DNS Server trung tâm (Site 2)
![Server2 - Global Settings](img_project_cisco/img14.png)
*Thiết lập Gateway: 192.168.2.1, tự phân giải DNS qua chính nó (192.168.2.10)*

![Server2 - FastEthernet0](img_project_cisco/img15.png)
*Địa chỉ IP tĩnh: 192.168.2.10/24 – đóng vai trò DNS Server chung cho toàn hệ thống*

### Server1 – DHCP Server tại Site 3
![Server1 - Global Settings](img_project_cisco/img16.png)
*Thiết lập Gateway/DNS: Default Gateway 192.168.3.1, DNS Server 192.168.2.10*

![Server1 - FastEthernet0](img_project_cisco/img17.png)
*Địa chỉ IP tĩnh: 192.168.3.10/24*

![Server1 - Dịch vụ DHCP](img_project_cisco/img19.png)
*Cấu hình DHCP Pool "serverPool": cấp phát IP từ 192.168.3.x, tối đa 100 người dùng*

### PC0 – Client tại Site 3
![PC0 - Global Settings](img_project_cisco/img20.png)
*Cấu hình nhận IP tự động qua DHCP, Gateway 192.168.3.1, DNS 192.168.2.10*

![PC0 - Global Settings (chi tiết)](img_project_cisco/img21.png)
*Xác nhận cấu hình DHCP: Default Gateway 192.168.3.1, DNS Server 192.168.2.10*

![PC0 - FastEthernet0](img_project_cisco/img22.png)
*Kết quả: PC0 nhận địa chỉ IP 192.168.3.100/24 từ DHCP Server1*

### Kiểm thử hệ thống DNS
![PC0 - Ping test DNS](img_project_cisco/img23.png)
*Lệnh `ping yahoo.com` và `ping 192.168.1.2` thực hiện từ PC0: DNS phân giải chính xác tên miền sang IP, kết nối thông suốt liên site (0% loss)*

![Server2 - Cấu hình DNS Records](img_project_cisco/img24.png)
*Khai báo A Record trên DNS Server trung tâm: `gmail.com → 192.168.3.10`, `yahoo.com → 192.168.1.10`*

![Ping test DNS bổ sung](img_project_cisco/img25.png)
*Kiểm tra phân giải DNS cho cả 2 domain `yahoo.com` và `gmail.com` từ client, xác nhận độ trễ (RTT) và tỷ lệ mất gói 0%*

### Dịch vụ Email (SMTP/POP3) liên site
![Server0 & Server1 - Cấu hình Email](img_project_cisco/img26.png)
*Bật dịch vụ SMTP/POP3 trên Server0 (domain `yahoo.com`, user Duy) và Server1 (domain `gmail.com`, user Duc)*

![PC1 & PC0 - Cấu hình Mail Client](img_project_cisco/img27.png)
*Thiết lập tài khoản email trên client: `duy@yahoo.com` (PC1 – Site 1) và `duc@gmail.com` (PC0 – Site 3)*

![PC1 - Nhận thư qua POP3](img_project_cisco/img28.png)
*PC1 thực hiện Receive Mail: hệ thống tự động phân giải DNS (`yahoo.com` → 192.168.1.10) trước khi kết nối POP3 Server*

![PC1 - Nội dung thư đã nhận](img_project_cisco/img29.png)
*Kết quả: PC1 nhận thành công email phản hồi từ `duc@gmail.com` – xác nhận luồng Email hoạt động end-to-end giữa 2 site khác nhau*




