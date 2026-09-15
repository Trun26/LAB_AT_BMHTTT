# LAB 1 - BẮT GÓI TIN TELNET - SSH

## Thông tin sinh viên
- Họ và tên: Đào Quốc Trung
- MSSV: 1150080121
- Tên bài Lab: Bắt gói tin Telnet - SSH

## Nội dung thực hiện
- Thiết lập môi trường máy ảo bằng VMware.
- Sử dụng Windows 11 làm máy Client.
- Sử dụng Ubuntu Server làm máy Server.
- Cấu hình dịch vụ Telnet trên Ubuntu Server.
- Tạo tài khoản thử nghiệm để đăng nhập vào Server.
- Kết nối từ Windows 11 đến Ubuntu Server bằng giao thức Telnet.
- Sử dụng Wireshark để bắt gói tin.
- Sử dụng bộ lọc `tcp.port == 23` để phân tích lưu lượng Telnet.
- Thực hiện các lệnh trên Server và quan sát các gói TELNET trao đổi giữa Client và Server.

## Kết quả thực hiện
- Kết nối Telnet từ Windows 11 đến Ubuntu Server thành công.
- Đăng nhập thành công vào Ubuntu Server.
- Wireshark bắt được các gói tin sử dụng TCP port 23.
- Quan sát được các gói TELNET trao đổi giữa Client và Server.
- Qua quá trình thực hành nhận thấy Telnet không mã hóa nội dung truyền trên mạng, do đó có nguy cơ làm lộ dữ liệu khi lưu lượng bị bắt.

## Video demo
- YouTube: https://youtu.be/lOh6yQzEixY

## Ghi chú
Bài thực hành được thực hiện trên môi trường VMware với Windows 11, Ubuntu Server và Wireshark.
