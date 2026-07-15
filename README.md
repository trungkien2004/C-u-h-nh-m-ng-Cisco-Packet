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

## Mở rộng: Phân đoạn mạng VLAN (VLAN Segmentation & Inter-VLAN Routing)

Để mô phỏng sát hơn với mô hình mạng doanh nghiệp thực tế – nơi các phòng ban cần được cách ly về mặt mạng nhưng vẫn có thể trao đổi dữ liệu khi cần – dự án được bổ sung thêm phần **VLAN** tại Site 1 (Switch1), sử dụng kỹ thuật **Router-on-a-Stick** để định tuyến liên VLAN.

### Mục tiêu
- Phân đoạn mạng LAN tại Site 1 thành các VLAN riêng theo phòng ban, tăng tính bảo mật và giảm broadcast domain.
- Cấu hình Trunk để nhiều VLAN cùng đi qua 1 đường truyền vật lý giữa Switch và Router.
- Triển khai định tuyến liên VLAN (Inter-VLAN Routing) bằng Router-on-a-Stick, cho phép các VLAN khác nhau vẫn giao tiếp được với nhau khi được phép.

### Bảng VLAN

| VLAN ID | Tên | Cổng Switch1 | Thiết bị | Dải IP | Gateway |
|---|---|---|---|---|---|
| 10 | ketoan | Fa0/2 | Server0 | 192.168.10.0/24 | 192.168.10.1 |
| 20 | kythuat | Fa0/1 | PC1 | 192.168.20.0/24 | 192.168.20.1 |
| - | Trunk | Gig0/1 | → Router0 (Gig0/0/0) | - | - |

### Sơ đồ định tuyến liên VLAN (Router-on-a-Stick)
```
        Server0 (VLAN 10)        PC1 (VLAN 20)
        192.168.10.10                192.168.20.2
              \                          /
               \  Fa0/2          Fa0/1 /
                \________________/
                     Switch1
                  (Trunk: Gig0/1)
                        |
                   Router0 (Gig0/0/0)
                 ├─ Sub-if .10 → 192.168.10.1
                 └─ Sub-if .20 → 192.168.20.1
```

### Các bước triển khai chi tiết

**1. Tạo VLAN trên Switch1**
Tạo VLAN 10 (đặt tên `ketoan`) và VLAN 20 (đặt tên `kythuat`) bằng lệnh `vlan` / `name` trong chế độ Global Configuration.

**2. Cấu hình Trunk trên cổng nối lên Router0**
Đưa cổng `GigabitEthernet0/1` (cổng nối Switch1 lên Router0) sang chế độ Trunk bằng `switchport mode trunk`, cho phép nhiều VLAN cùng đi qua 1 đường truyền vật lý.

**3. Xác nhận Trunk hoạt động**
Dùng lệnh `show interfaces trunk` để kiểm tra: cổng Gig0/1 đang ở chế độ trunk, encapsulation 802.1Q, và VLAN 1, 10, 20 đều đang active/forwarding trên đường trunk này.

**4. Cấu hình Router-on-a-Stick trên Router0**
Bật cổng vật lý `GigabitEthernet0/0/0` (`no shutdown`), sau đó tạo 2 sub-interface:
- `GigabitEthernet0/0/0.10`: encapsulation dot1Q 10, IP `192.168.10.1/24` (Gateway cho VLAN 10)
- `GigabitEthernet0/0/0.20`: encapsulation dot1Q 20, IP `192.168.20.1/24` (Gateway cho VLAN 20)

**5. Kiểm tra toàn bộ interface trên Router0**
Dùng `show ip interface brief` để xác nhận cả 2 sub-interface đều ở trạng thái **up/up**, sẵn sàng định tuyến liên VLAN.

**6. Cập nhật địa chỉ IP cho Server0 (VLAN 10)**
- Global Settings: Default Gateway `192.168.10.1`, DNS Server `192.168.2.10`
- FastEthernet0: IP tĩnh `192.168.10.10/24`

**7. Cập nhật địa chỉ IP cho PC1 (VLAN 20)**
- Global Settings: Default Gateway `192.168.20.1`, DNS Server `192.168.2.10`
- FastEthernet0: IP tĩnh `192.168.20.2/24`

**8. Gán cổng vào đúng VLAN và xác nhận lần cuối**
Gán `FastEthernet0/1` vào VLAN 20 (PC1) và `FastEthernet0/2` vào VLAN 10 (Server0) bằng `switchport access vlan`. Kiểm tra lại bằng `show vlan brief`, xác nhận đúng: VLAN 10 → Fa0/2, VLAN 20 → Fa0/1.

**9. Kiểm thử kết nối**
Từ PC1, thực hiện `ping 192.168.20.1` (Gateway cùng VLAN) và `ping 192.168.10.10` (Server0 khác VLAN) bằng Command Prompt để xác nhận Inter-VLAN Routing hoạt động — kết quả: `ping 192.168.20.1` đạt 4/4 Reply (0% loss); `ping 192.168.10.10` đạt 3/4 Reply (25% loss, gói đầu timeout do phân giải ARP lần đầu giữa 2 VLAN – hiện tượng bình thường, không phải lỗi).

### Hình ảnh minh chứng – VLAN

![Switch1 - Tạo VLAN](img_project_cisco/img30.png)
*Tạo VLAN 10 (ketoan) và VLAN 20 (kythuat) trên Switch1 bằng lệnh `vlan` / `name`*

![Switch1 - Cấu hình Trunk](img_project_cisco/img31.png)
*Cấu hình cổng GigabitEthernet0/1 sang chế độ Trunk (`switchport mode trunk`) để kết nối lên Router0*

![Switch1 - Xác nhận Trunk](img_project_cisco/img32.png)
*Kết quả `show interfaces trunk`: Gig0/1 hoạt động ở chế độ trunk, encapsulation 802.1Q, VLAN 1, 10, 20 đang active*

![Switch1 - Xác nhận Trunk (tiếp)](img_project_cisco/img33.png)
*Xác nhận lại thông tin Trunk trước khi chuyển sang cấu hình Router0*

![Router0 - Tạo Sub-interface VLAN 10](img_project_cisco/img34.png)
*Bật cổng Gig0/0/0, tạo sub-interface Gig0/0/0.10 với encapsulation dot1Q 10 và IP Gateway 192.168.10.1*

![Router0 - Tạo Sub-interface VLAN 20](img_project_cisco/img35.png)
*Hoàn tất sub-interface VLAN 10, tiếp tục tạo sub-interface Gig0/0/0.20 cho VLAN 20*

![Router0 - Kiểm tra Interface](img_project_cisco/img36.png)
*Hoàn tất sub-interface VLAN 20 (IP Gateway 192.168.20.1) và kiểm tra `show ip interface brief`: cả 2 sub-interface đều up/up*

![Server0 - Cấu hình Gateway/DNS](img_project_cisco/img37.png)
*Cập nhật Global Settings trên Server0: Default Gateway 192.168.10.1, DNS Server 192.168.2.10*

![Server0 - Địa chỉ IP VLAN 10](img_project_cisco/img38.png)
*Cập nhật địa chỉ IP tĩnh trên Server0: 192.168.10.10/24 (thuộc VLAN 10)*

![PC1 - Cấu hình Gateway/DNS](img_project_cisco/img39.png)
*Cập nhật Global Settings trên PC1: Default Gateway 192.168.20.1, DNS Server 192.168.2.10*

![PC1 - Địa chỉ IP VLAN 20](img_project_cisco/img41.png)
*Cập nhật địa chỉ IP tĩnh trên PC1: 192.168.20.2/24 (thuộc VLAN 20)*

![Switch1 - Gán cổng và xác nhận VLAN](img_project_cisco/img42.png)
*Gán chính xác Fa0/1 → VLAN 20 (PC1), Fa0/2 → VLAN 10 (Server0), xác nhận bằng `show vlan brief`*

![PC1 - Kiểm thử kết nối liên VLAN](img_project_cisco/img43.png)
*Kết quả `ping 192.168.20.1` (Gateway VLAN 20): 4/4 Reply, 0% loss. Kết quả `ping 192.168.10.10` (Server0 – VLAN 10): 3/4 Reply, 25% loss – gói đầu timeout do quá trình phân giải ARP lần đầu giữa 2 VLAN, các gói sau đều Reply thành công. Xác nhận Inter-VLAN Routing hoạt động chính xác.*

### Kỹ năng thể hiện qua phần mở rộng này
- Triển khai và quản lý VLAN trên Switch Cisco (tạo VLAN, gán cổng, đặt tên).
- Cấu hình đường Trunk (802.1Q) giữa Switch và Router, kiểm tra bằng `show interfaces trunk`.
- Triển khai định tuyến liên VLAN bằng kỹ thuật Router-on-a-Stick (sub-interface, encapsulation dot1Q).
- Kỹ năng debug thực tế: phát hiện và khắc phục lỗi cú pháp CLI (gõ nhầm lệnh, dấu nhắc bị dính chữ), đối chiếu `show vlan brief` / `show interfaces trunk` / `show ip interface brief` trước và sau khi sửa để xác nhận cấu hình đúng.


