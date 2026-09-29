# LAB4 - KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

## 1. Thông tin sinh viên

- **Họ và tên:** Đào Quốc Trung
- **MSSV:** 1150080121
- **Tên lab:** LAB4 - Khảo sát và đánh giá bề mặt mạng bằng Nmap
- **Video thực hành:** https://youtu.be/BM6C1ZFoNrY

## 2. Môi trường thực hành

- **Phần mềm ảo hóa:** VMware Workstation
- **Kiểu mạng:** Host-Only
- **Máy khảo sát:** Kali Linux
- **Máy đích:** Metasploitable 2
- **Máy kiểm tra bổ sung:** Windows 11
- **Công cụ chính:** Nmap

### Địa chỉ IP trong môi trường LAB

| Máy | Vai trò | Địa chỉ IP |
|---|---|---|
| Kali Linux | Máy thực hiện khảo sát bằng Nmap | 172.16.16.129/24 |
| Metasploitable 2 | Máy đích thực hành | 172.16.16.130 |
| Windows 11 | Máy kiểm tra kết nối | 172.16.16.128 |

> Toàn bộ thao tác trong LAB4 được thực hiện trên các máy ảo của sinh viên trong mạng Host-Only, không quét hệ thống bên ngoài.

## 3. Cách dựng môi trường

1. Khởi động các máy ảo Kali Linux, Metasploitable 2 và Windows 11 bằng VMware Workstation.
2. Cấu hình card mạng của các máy ở chế độ **Host-Only**.
3. Kiểm tra địa chỉ IP thực tế trên từng máy.
4. Kiểm tra khả năng liên lạc giữa các máy bằng `ping`.
5. Sử dụng Kali Linux làm máy khảo sát và Metasploitable 2 làm máy đích.
6. Tạo thư mục `~/LAB4` trên Kali để lưu các output/log làm bằng chứng.

## 4. Các tình huống đã thực hiện

### 4.1. Kiểm tra kết nối và Host Discovery

- Kiểm tra kết nối giữa Kali Linux, Windows 11 và Metasploitable 2.
- Thực hiện Host Discovery bằng Nmap để xác định các host đang hoạt động trong mạng Host-Only.

**Kết quả: PASS**

### 4.2. Khảo sát cổng TCP

Thực hiện các kỹ thuật quét TCP trong bài LAB để xác định và so sánh trạng thái các cổng.

Metasploitable 2 phát hiện nhiều dịch vụ đang hoạt động như:

- FTP
- SSH
- Telnet
- SMTP
- HTTP
- SMB
- MySQL
- PostgreSQL
- VNC
- IRC
- Apache Tomcat

**Kết quả: PASS**

### 4.3. Quét UDP

Thực hiện UDP Scan có kiểm soát trên máy Metasploitable 2 và quan sát các trạng thái cổng do Nmap trả về.

**Kết quả: PASS**

### 4.4. Nhận diện dịch vụ và phiên bản

Sử dụng tùy chọn `-sV` để nhận diện dịch vụ và phiên bản phần mềm đang chạy trên các cổng mở.

Một số dịch vụ phát hiện được gồm:

- Apache HTTP Server
- Samba
- MySQL
- PostgreSQL
- VNC
- Apache Tomcat

**Kết quả: PASS**

### 4.5. Nhận diện hệ điều hành

Sử dụng OS Detection của Nmap để thu thập fingerprint và xác định hệ điều hành của máy đích.

Kết quả cho thấy máy đích thuộc họ **Linux/Unix**, phù hợp với máy Metasploitable 2.

**Kết quả: PASS**

### 4.6. NSE - Kiểm tra SMB

Thực hiện NSE Script để kiểm tra dịch vụ SMB:

- `smb-os-discovery`
- `smb-vuln-ms17-010`

`smb-os-discovery` thu được thông tin về máy Metasploitable và dịch vụ Samba.

Đối với `smb-vuln-ms17-010`, output thực hành không hiển thị kết luận `VULNERABLE`. Vì vậy không kết luận máy có hoặc không có lỗ hổng chỉ dựa trên việc không xuất hiện dòng cảnh báo.

**Kết quả: PASS**

### 4.7. Xuất kết quả và lưu bằng chứng

Kết quả Nmap được lưu dưới nhiều định dạng:

- Normal Text (`.txt`, `.nmap`)
- XML (`.xml`)
- Grepable (`.gnmap`)
- HTML chuyển đổi từ XML

Các output/log được lưu trong thư mục `~/LAB4`.

**Kết quả: PASS**

### 4.8. Before / After Hardening

Thực hiện quét trước và sau khi thay đổi cấu hình phòng thủ, sau đó so sánh kết quả.

Các tệp bằng chứng:

- `before_hardening.txt`
- `after_hardening.txt`

Kết quả so sánh cho thấy một số cổng/dịch vụ thay đổi sau hardening, trong đó có các cổng liên quan:

- `8009/tcp`
- `8180/tcp`

Sau hardening, các dịch vụ trên không còn xuất hiện như trước, cho thấy bề mặt dịch vụ có thể truy cập đã được thu hẹp.

**Kết quả: PASS**

## 5. Output/Log của bài thực hành

Các tệp được tạo trong quá trình thực hành:

```text
after_hardening.txt
before_hardening.txt
metasploitable.html
metasploitable.txt
metasploitable.xml
nmap_metasploitable.gnmap
nmap_metasploitable.nmap
nmap_metasploitable.xml
smb.txt
