# LAB 5 – Thiết lập tường lửa pfSense

## Thông tin sinh viên

- **Họ và tên:** Đào Quốc Trung
- **MSSV:** 1150080121
- **Môn học:** An toàn hệ thống thông tin
- **Link video YouTube:** https://youtu.be/o4QR7e2Zg6E

## 1. Mục tiêu

- Xây dựng mô hình mạng có tường lửa pfSense bảo vệ LAN và vùng DMZ.
- Cấu hình WAN, LAN, DMZ, NAT và firewall rule.
- Dùng Windows Server làm máy trong LAN để kiểm thử lưu lượng.
- Thực hành nhiều tình huống firewall thay vì chỉ tạo một rule đơn giản.

## 2. Môi trường thực hành

Thực hành trên VMware Workstation với các máy ảo:

- pfSense: tường lửa quản lý WAN, LAN và DMZ.
- Windows 11: máy kiểm thử trong LAN.
- Ubuntu Server: máy LAN-Test.
- Metasploitable2: máy kiểm thử trong DMZ.

Trong bài thực hiện, Windows 11 được dùng thay máy Windows Server để kiểm thử lưu lượng; Metasploitable2 được dùng làm host DMZ.

### Bảng địa chỉ IP

| Thiết bị | Mạng VMware | Địa chỉ IPv4 | Gateway |
| --- | --- | --- | --- |
| pfSense WAN | NAT | 192.168.209.136/24, DHCP | Theo DHCP |
| pfSense LAN | VMnet1 | 10.0.0.1/8 | Không đặt |
| pfSense OPT1 – DMZ | VMnet2 | 172.16.0.1/16 | Không đặt |
| Windows 11 | VMnet1 | 10.0.0.2/8 | 10.0.0.1 |
| Ubuntu LAN-Test | VMnet1 | 10.0.0.3/8 | 10.0.0.1 |
| Metasploitable2 | VMnet2 | 172.16.0.2/16 | 172.16.0.1 |

Địa chỉ WAN có thể thay đổi theo DHCP. Mạng WAN không chồng lấn với LAN và DMZ. DNS dùng để kiểm thử Internet là 8.8.8.8.

## 3. Cấu hình nền tảng

- Cấu hình ba card mạng pfSense tương ứng WAN, LAN và OPT1.
- Đặt LAN là 10.0.0.1/8, OPT1 là 172.16.0.1/16.
- Đặt IP tĩnh và gateway cho các máy kiểm thử.
- Lưu IP tĩnh Metasploitable2 trong `/etc/network/interfaces`.
- Cấu hình IP Ubuntu bằng Netplan.
- Giữ Anti-Lockout Rule để truy cập giao diện quản trị.
- Vô hiệu hóa hai Default allow LAN khi thực hiện các tình huống.
- Apply Changes và Reset States trước khi kiểm thử các rule mới.

### Kết quả kiểm tra kết nối

- Metasploitable2 ping OPT1 172.16.0.1 thành công.
- Windows 11 ping Metasploitable2 172.16.0.2 thành công.
- Các host có IP và gateway đúng với mô hình.

## 4. Tình huống 1 – Chặn ICMP nhưng cho phép Web và DNS

### Yêu cầu

Chặn ping từ LAN nhưng vẫn cho phép phân giải DNS và truy cập HTTP/HTTPS.

### Cấu hình rule trên LAN

| Thứ tự | Action | Protocol | Source | Destination | Cổng đích |
| --- | --- | --- | --- | --- | --- |
| 1 | Block | ICMP IPv4 | LAN subnets | Any | Không áp dụng |
| 2 | Pass | TCP/UDP IPv4 | LAN subnets | Any | 53 |
| 3 | Pass | TCP IPv4 | LAN subnets | Any | 80, 443 |

Tạo alias `WEB_PORTS` chứa hai cổng 80 và 443. Tắt các rule Pass tổng quát; cấu hình DNS Windows là 8.8.8.8.

### Kiểm thử

Trên Windows 11:

    ping 8.8.8.8
    nslookup example.com 8.8.8.8
    curl.exe -4 --max-time 30 https://example.com

### Kết quả

- Ping 8.8.8.8: 100% mất gói.
- DNS: phân giải example.com thành công.
- HTTPS: trả về nội dung HTML của Example Domain.

### Giải thích

ICMP bị chặn bởi rule Block. DNS và Web vẫn hoạt động nhờ các rule Pass riêng theo giao thức và cổng.

## 5. Tình huống 2 – Chỉ cho một host ra Internet

### Yêu cầu

Cho phép host 10.0.0.2 ra Internet, chặn các host LAN còn lại.

### Cấu hình

Tắt các rule của tình huống 1. Hai Default allow LAN vẫn được vô hiệu hóa.

| Thứ tự | Action | Protocol | Source | Destination |
| --- | --- | --- | --- | --- |
| 1 | Pass | Any IPv4 | 10.0.0.2 | Any |
| 2 | Block | Any IPv4 | LAN subnets | Any |

Rule cho phép host 10.0.0.2 nằm trên rule chặn LAN.

### Kiểm thử

Trên Windows 11, IP 10.0.0.2:

    ping 8.8.8.8

Trên Ubuntu LAN-Test, IP 10.0.0.3:

    ping -c 4 -w 2 8.8.8.8

Tùy chọn `-w 2` giới hạn tổng thời gian thử hai giây.

### Kết quả

- Windows 11: nhận 4/4 phản hồi, 0% mất gói.
- Ubuntu LAN-Test: không nhận phản hồi, 100% mất gói.

### Giải thích

Host 10.0.0.2 khớp rule Pass phía trên nên được phép ra Internet. Các host LAN khác khớp rule Block phía dưới nên bị chặn.

## 6. Tình huống 3 – Cô lập DMZ khỏi LAN

### Yêu cầu

Chặn lưu lượng từ DMZ tới LAN nhưng vẫn cho phép DMZ truy cập Internet và DNS.

### Baseline trước khi chặn

Trên Windows 11, tạo rule cho phép ICMP Echo để tránh Windows Firewall ảnh hưởng phép thử:

    netsh advfirewall firewall add rule name="LAB-Allow-ICMPv4-Echo" protocol=icmpv4:8,any dir=in action=allow

Khi OPT1 chỉ có rule Pass nền tảng, từ Metasploitable2 chạy:

    ping -c 4 10.0.0.2

Kết quả: nhận 4/4 phản hồi, 0% mất gói.

### Cấu hình rule trên OPT1

| Thứ tự | Action | Protocol | Source | Destination |
| --- | --- | --- | --- | --- |
| 1 | Block | Any IPv4 | OPT1 subnets | LAN subnets |
| 2 | Pass | Any IPv4 | Any | Any |

Rule `Block DMZ to LAN` nằm trên rule nền tảng `Allow OPT1 to any`.

Apply Changes và Reset States trước khi kiểm thử lại.

### Kiểm thử sau khi chặn

Trên Metasploitable2:

    ping -c 4 -w 2 10.0.0.2
    ping -c 4 -w 2 8.8.8.8
    nslookup example.com 8.8.8.8

### Kết quả

- DMZ tới LAN 10.0.0.2: 100% mất gói.
- DMZ tới Internet 8.8.8.8: nhận phản hồi, 0% mất gói.
- DNS: phân giải example.com thành công.

### Giải thích

Trước khi thêm Block, DMZ truy cập LAN thành công. Sau khi thêm Block và Reset States, cùng phép thử bị chặn. Rule Pass phía dưới vẫn cho phép DMZ truy cập Internet và DNS.

## 7. Trả lời câu hỏi

### Câu 1. Phân biệt vai trò NAT và firewall rule

NAT chuyển đổi địa chỉ hoặc cổng, chẳng hạn chuyển địa chỉ nguồn của host LAN/DMZ sang địa chỉ WAN khi ra Internet. Firewall rule quyết định cho phép hoặc chặn lưu lượng. Có NAT không đồng nghĩa lưu lượng đã được cho phép.

### Câu 2. Vì sao đặt máy chủ Web/Mail/FTP trong DMZ?

DMZ tách các dịch vụ được truy cập từ bên ngoài khỏi LAN. Nếu máy chủ bị xâm nhập, rule cô lập giúp hạn chế truy cập vào máy và dữ liệu nội bộ.

### Câu 3. Nếu Block nằm dưới Pass tổng quát thì sao?

Với các interface rule thông thường, pfSense xét từ trên xuống và áp dụng rule khớp đầu tiên. Lưu lượng khớp Pass tổng quát phía trên sẽ được phép, nên rule Block phía dưới không chặn được lưu lượng đó.

### Câu 4. Chặn ping nhưng vẫn cho Web cần rule nào?

Tạo Block ICMP; Pass TCP/UDP cổng đích 53 cho DNS; Pass TCP cổng đích 80 và 443 cho Web. Tắt rule Pass tổng quát và Reset States trước khi kiểm thử.

### Câu 5. Logging giúp gì trong xử lý sự cố?

Firewall Log cung cấp thời điểm, interface, địa chỉ nguồn/đích, giao thức và cổng của lưu lượng được ghi nhận. Đối chiếu log với rule giúp xác định nguyên nhân chặn và phân biệt với lỗi DNS, routing hoặc dịch vụ.

### Câu 6. Nêu ít nhất ba biện pháp hardening

- Dùng mật khẩu quản trị mạnh và giới hạn máy được truy cập WebGUI/SSH.
- Cập nhật bản vá sau khi kiểm tra tương thích.
- Chỉ cho phép lưu lượng cần thiết, cô lập DMZ khỏi LAN.
- Tắt dịch vụ không sử dụng.
- Bật logging cho rule quan trọng và sao lưu cấu hình định kỳ.

## 8. Minh chứng

Báo cáo Word gồm ảnh cấu hình và kết quả kiểm thử:

- Cấu hình mạng và địa chỉ IP.
- Rule và kết quả của tình huống 1.
- Rule, phép thử Windows được phép và Ubuntu bị chặn của tình huống 2.
- Baseline trước Block, rule cô lập và kết quả sau Block của tình huống 3.

## 9. Kết luận

Đã thực hiện ba tình huống:

1. Chặn ICMP nhưng vẫn cho phép Web/DNS.
2. Chỉ cho host 10.0.0.2 ra Internet.
3. Cô lập DMZ khỏi LAN, giữ kết nối Internet và DNS.

Kết quả cho thấy pfSense có thể lọc lưu lượng theo giao thức/cổng, theo host nguồn và theo vùng mạng. Thứ tự rule, cấu hình IP/gateway/DNS và state cũ ảnh hưởng trực tiếp tới phép thử.
