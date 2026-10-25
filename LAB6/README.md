# LAB 6 – pfSense tích hợp Active Directory và chính sách truy cập

- **Họ tên:** Đào Quốc Trung
- **MSSV:** 1150080121
- **Môi trường:** VMware Workstation
- **Phạm vi:** Thực hiện 5 kịch bản chính, không thực hiện kịch bản mở rộng 6–7.

## Video thực hành

**Link YouTube:** https://youtu.be/npX6mLesMR4
## 1. Mục tiêu

- Xây dựng pfSense riêng với bốn interface: WAN, MGMT, LAN_DOMAIN và DMZ.
- Tích hợp pfSense với Active Directory và kiểm tra xác thực người dùng.
- Cấu hình Captive Portal trên LAN_DOMAIN sử dụng tài khoản AD.
- Tạo Alias cho máy trạm, nhóm máy trạm và các website mô phỏng.
- Áp dụng chính sách theo IP, cổng dịch vụ và thời gian.
- Kiểm thử kết quả trước và sau khi áp dụng firewall rule.

## 2. Mô hình mạng

| Thành phần | Mạng VMware | Địa chỉ |
|---|---|---|
| pfSense WAN | VMnet8 – NAT | DHCP, ghi nhận 192.168.209.137/24 |
| pfSense MGMT | VMnet3 | 192.168.56.254/24 |
| Máy thật quản trị | VMnet3 | 192.168.56.1/24 |
| pfSense LAN_DOMAIN | VMnet4 | 10.6.0.1/24 |
| DC01 – AD DS/DNS | VMnet4 | 10.6.0.10/24 |
| Client-A | VMnet4 | 10.6.0.21/24 |
| Client-B | VMnet4 | 10.6.0.22/24 |
| pfSense DMZ | VMnet5 | 172.16.6.1/24 |
| Ubuntu – Nginx | VMnet5 | 172.16.6.10, .20, .30, .40/24 |

- Gateway của hai client: `10.6.0.1`.
- DNS của hai client: `10.6.0.10`.
- Gateway của Ubuntu: `172.16.6.1`.
- Hai client là hai máy ảo riêng.
- Ubuntu sử dụng bốn địa chỉ IP để mô phỏng bốn website.
- Máy thật truy cập WebGUI qua mạng MGMT.

## 3. Active Directory và LDAPS

### Thông tin domain

- **Domain:** `lab6.test`
- **NetBIOS:** `LAB6`
- **Domain Controller:** `DC01.lab6.test`
- **IP:** `10.6.0.10`
- **Tài khoản:** `sv_a`, `sv_b`, `svc_ldap`

### Cấu hình Authentication Server

| Thuộc tính | Giá trị |
|---|---|
| Name | AD_LAB6 |
| Type | LDAP |
| Hostname | dc01.lab6.test |
| Port | 636 |
| Transport | SSL/TLS Encrypted |
| Peer Certificate Authority | LAB6-RootCA |
| Protocol version | 3 |
| Search scope | Entire Subtree |
| Base DN | DC=lab6,DC=test |
| Authentication containers | CN=Users,DC=lab6,DC=test |
| Bind DN | CN=svc_ldap,CN=Users,DC=lab6,DC=test |
| User naming attribute | sAMAccountName |

LDAP simple bind trên cổng 389 bị DC yêu cầu phương thức xác thực an toàn hơn. Bài thực hành chuyển sang LDAPS, cấp chứng chỉ cho DC01 và import LAB6-RootCA vào pfSense.

**Kết quả:** Diagnostics → Authentication xác thực thành công cả `sv_a` và `sv_b`.

## 4. Captive Portal

- **Zone:** `lab6_portal`
- **Interface:** `LAN_DOMAIN`
- **Authentication Server:** `AD_LAB6`

| Máy trạm | IP | Tài khoản đăng nhập |
|---|---|---|
| Client-A | 10.6.0.21 | sv_a |
| Client-B | 10.6.0.22 | sv_b |

**Kết quả:** Status → Captive Portal hiển thị đồng thời hai phiên đăng nhập.

Captive Portal xác thực người dùng. Firewall base vẫn áp dụng chính sách theo IP, protocol, port và thời gian; không sử dụng trực tiếp username LDAP làm điều kiện của rule.

## 5. Alias và website mô phỏng

| Alias | Giá trị |
|---|---|
| CLIENT_A | 10.6.0.21 |
| CLIENT_B | 10.6.0.22 |
| POLICY_CLIENTS | 10.6.0.21 và 10.6.0.22 |
| GENERAL_SITE | 172.16.6.10 |
| JOB_SITE | 172.16.6.20 |
| SOCIAL_SITE | 172.16.6.30 |
| RESTRICTED_SITE | 172.16.6.40 |
| WEB_PORTS | 80 và 443 |

Nginx trả về nội dung riêng cho từng IP:

- `GENERAL_SITE - LAB6`
- `JOB_SITE - LAB6`
- `SOCIAL_SITE - LAB6`
- `RESTRICTED_SITE - LAB6`

Đã kiểm tra cả bốn website hoạt động trên Ubuntu trước khi kiểm thử firewall.

## 6. Các khung giờ

| Schedule | Thời gian |
|---|---|
| LUNCH_WINDOW | Thứ Hai–Thứ Sáu, 11:30–13:00 |
| WORK_HOURS | Thứ Hai–Thứ Sáu, 08:00–17:00 |
| TEST_NOW | Lịch tạm để kiểm thử khi quay video |

Múi giờ pfSense: `Asia/Saigon`.

`TEST_NOW` được dùng để kiểm thử trạng thái cho phép ngoài giờ thực tế, sau đó chuyển rule về lịch yêu cầu.

## 7. Năm kịch bản thực hành

### Kịch bản 1 – Chặn site theo máy trạm

**Chính sách:** Chặn Client-A truy cập JOB_SITE qua TCP/80.

**Kết quả:**

- Client-A truy cập GENERAL_SITE thành công.
- Client-A truy cập JOB_SITE bị timeout khi bật rule Block.
- Khi tắt rule Block, Client-A truy cập JOB_SITE thành công.
- Client-B vẫn truy cập JOB_SITE thành công.

### Kịch bản 2 – Chặn site cho nhóm máy

**Chính sách:** Chặn POLICY_CLIENTS truy cập RESTRICTED_SITE qua TCP/80.

**Kết quả:**

- Cả Client-A và Client-B bị chặn tới RESTRICTED_SITE.
- GENERAL_SITE vẫn truy cập được.

Alias POLICY_CLIENTS đại diện cho nhóm IP máy trạm, không phải nhóm người dùng AD.

### Kịch bản 3 – Cho phép SOCIAL_SITE theo giờ nghỉ trưa

**Chính sách:**

1. Pass CLIENT_B → SOCIAL_SITE TCP/80, gắn LUNCH_WINDOW.
2. Block CLIENT_B → SOCIAL_SITE TCP/80 đặt ngay bên dưới.

**Kết quả:**

- Khi dùng TEST_NOW đang hoạt động: trả về `SOCIAL_SITE - LAB6`.
- Khi chuyển về LUNCH_WINDOW ngoài giờ: kết nối mới bị timeout.
- Đã sửa thứ tự rule, Save và Apply Changes để chính sách có hiệu lực đúng.

### Kịch bản 4 – Giới hạn thời gian và dịch vụ

**Chính sách:**

- Cho phép POLICY_CLIENTS sử dụng WEB_PORTS theo WORK_HOURS.
- Chặn các lưu lượng khác của nhóm client bằng rule bên dưới.

**Kết quả kiểm thử:**

- HTTP tới GENERAL_SITE thành công khi lịch thử đang hoạt động.
- ICMP tới cùng máy chủ bị chặn.
- HTTP bị timeout ngoài lịch web.
- Sau khi thêm ngoại lệ GENERAL_SITE TCP/80 phía trên, HTTP tới site này được phép lại.

**Phạm vi xác nhận:** Đã kiểm thử HTTP nội bộ và ICMP. Chưa có minh chứng kiểm thử đầy đủ HTTPS/443 và Internet.

Ngoại lệ GENERAL_SITE trong cấu hình cuối cho phép truy cập site này độc lập với lịch web chung.

### Kịch bản 5 – Cách ly DMZ với LAN

**Chính sách:** Block DMZ subnets → LAN_DOMAIN subnets, IPv4, mọi protocol.

**Kiểm thử:**

- Bật ICMP inbound phù hợp trên DC01.
- Trước khi áp dụng Block: Ubuntu ping DC01 nhận 4/4 gói.
- Sau khi áp dụng Block phía trên rule đối chứng: mất 100% gói.
- Chiều LAN → GENERAL_SITE vẫn được phép theo rule tương ứng.

**Kết luận:** Rule ngăn DMZ khởi tạo kết nối vào LAN. Lưu lượng trả lời cho kết nối do LAN khởi tạo được xử lý theo state.

## 8. Lệnh kiểm thử

Trên Windows:

    ipconfig
    curl.exe --noproxy "*" --max-time 5 http://172.16.6.10
    curl.exe --noproxy "*" --max-time 5 http://172.16.6.20
    curl.exe --noproxy "*" --max-time 5 http://172.16.6.30
    curl.exe --noproxy "*" --max-time 5 http://172.16.6.40
    ping -n 4 172.16.6.10

Trên Ubuntu:

    ip -br addr
    ip route
    sudo nginx -t
    curl http://172.16.6.10
    curl http://172.16.6.20
    curl http://172.16.6.30
    curl http://172.16.6.40
    ping -c 4 10.6.0.10

## 9. Nhận xét

- Rule được xét từ trên xuống; rule cụ thể cần đặt trước rule cho phép rộng.
- Phải Save và Apply Changes sau khi thay đổi cấu hình hoặc thứ tự.
- State cũ có thể ảnh hưởng phép thử; cần tạo kết nối mới và xóa state liên quan khi cần.
- Website mô phỏng theo IP giúp kiểm chứng policy nhưng không thay thế hệ thống phân loại URL/nội dung thực tế.
- Rule tạm phục vụ kiểm thử cần được rà soát khi hoàn thiện cấu hình.
- Ảnh cấu hình và kết quả thực hành được trình bày trong báo cáo Word.

## 10. Kết luận

Đã triển khai mô hình pfSense gồm bốn interface, tích hợp AD qua LDAPS và xác thực hai client bằng Captive Portal.

Đã kiểm thử năm kịch bản về chặn site theo máy, nhóm IP, lịch truy cập, giới hạn dịch vụ và cách ly DMZ với LAN. Phạm vi kiểm thử của từng kịch bản được ghi rõ trong báo cáo; không thực hiện phần mở rộng 6–7.
