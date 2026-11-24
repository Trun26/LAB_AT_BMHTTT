# LAB 7 – Xây dựng chính sách Web Protection bằng pfSense, Squid, E2Guardian và ClamAV

- **Họ tên:** Đào Quốc Trung
- **MSSV:** 1150080121
- **Môi trường:** VMware Workstation

## Video thực hành

**Link YouTube:** [Xem video thực hành LAB 7](https://youtu.be/lbHo6bX0Rr8)

## 1. Mục tiêu

- Xây dựng hệ thống Web Proxy trên Ubuntu kết hợp với pfSense.
- Cấu hình Client truy cập web thông qua E2Guardian và Squid.
- Áp dụng chính sách chặn website và cảnh báo nội dung.
- Cho phép truy cập website thuộc danh sách ngoại lệ.
- Giới hạn truy cập theo khung giờ.
- Quét file thử nghiệm EICAR bằng ClamAV.
- Giới hạn dung lượng file tải xuống.
- Kiểm thử và ghi nhận các giới hạn của cấu hình thực tế.

## 2. Mô hình triển khai

Hệ thống được triển khai trên VMware, gồm:

| Thành phần | Vai trò |
|---|---|
| pfSense | Firewall, định tuyến và kiểm soát truy cập trực tiếp |
| Web Proxy Ubuntu | Chạy E2Guardian, Squid và tích hợp ClamAV |
| DMZ-WEB Ubuntu | Chạy nginx và các website mô phỏng |
| Windows Server | Cung cấp DNS cho các tên miền lab7.test |
| Windows 11 – Client A | Kiểm thử chính sách truy cập web |

### Luồng truy cập

    Client A (10.6.0.21)
        → E2Guardian (10.6.0.30:8080)
        → Squid (127.0.0.1:3128)
        → Website thử nghiệm

Client sử dụng proxy `10.6.0.30:8080`. E2Guardian chuyển tiếp yêu cầu đến Squid tại `127.0.0.1:3128`.

Các website thử nghiệm sử dụng tên miền thuộc `lab7.test` và được phục vụ bằng nginx.

## 3. Các website thử nghiệm

| Tên miền | Mục đích |
|---|---|
| normal.lab7.test | Truy cập web bình thường |
| china.lab7.test | Mô phỏng website thuộc nhóm bị chặn |
| mideast.lab7.test | Mô phỏng website thuộc nhóm bị chặn |
| jobs.lab7.test | Mô phỏng website tìm việc |
| weapons.lab7.test | Kiểm thử cảnh báo nội dung |
| politics.lab7.test | Kiểm thử cảnh báo nội dung |
| religion.lab7.test | Kiểm thử cảnh báo nội dung |
| education.lab7.test | Website giáo dục thuộc danh sách ngoại lệ |
| games.lab7.test | Kiểm thử giới hạn giờ truy cập |
| entertainment.lab7.test | Kiểm thử giới hạn giờ truy cập |
| streaming.lab7.test | Kiểm thử cảnh báo streaming |
| blockedgame.lab7.test | Kiểm thử chặn website game |
| download.lab7.test | Kiểm thử tải file và quét EICAR |

Các tên miền trên mô phỏng nội dung phục vụ thực hành, không phải hệ thống phân loại website thực tế.

## 4. Kết quả thực hiện 10 kịch bản

### Kịch bản 1 – Truy cập web bình thường

- Khi bật proxy, Client A truy cập được `normal.lab7.test`.
- Khi tắt proxy, trình duyệt không kết nối được.
- Bật lại proxy thì truy cập thành công.

**Kết quả:** Đạt trong phạm vi kiểm thử.

### Kịch bản 2 – Chặn nhóm website mô phỏng

- Đã thêm `china.lab7.test` và `mideast.lab7.test` vào danh sách chặn.
- Chưa có đủ ảnh xác nhận cả hai website đều không thể vượt chặn bằng Acknowledge.

**Kết quả:** Chưa đủ minh chứng để kết luận đạt đầy đủ.

### Kịch bản 3 – Chặn website tìm việc

- Đã đưa `jobs.lab7.test` vào danh sách chặn.
- Trang Access Denied xuất hiện.
- Khi bật bypass 300 giây, người dùng vẫn có thể nhấn Acknowledge để truy cập.
- Khi tắt bypass, website bị chặn.

**Kết quả:** Chưa đạt yêu cầu hard block đồng thời duy trì chức năng cảnh báo.

### Kịch bản 4 – Cảnh báo nội dung

- Đã thử các website `weapons`, `politics` và `religion`.
- Trang cảnh báo xuất hiện và có nút Acknowledge.
- Đã xác nhận mở tạm thành công với `weapons.lab7.test`.
- Chưa kiểm tra lại sau khi hết 300 giây.

**Kết quả:** Đã kiểm thử cảnh báo; chưa xác minh đầy đủ thời gian hết hạn bypass.

### Kịch bản 5 – Cho phép website giáo dục

- Đã thêm `education.lab7.test` vào danh sách ngoại lệ.
- Client A mở được trang Education Site.

**Kết quả:** Đạt trong phạm vi kiểm thử.

### Kịch bản 6 – Giới hạn giờ truy cập web giải trí

- Đã tạo ACL trong Squid cho `games.lab7.test` và `entertainment.lab7.test`.
- Khung giờ cho phép: 12:00–13:00 các ngày trong tuần.
- `games.lab7.test` bị chặn khi thử ngoài lịch.
- `entertainment.lab7.test` vẫn có lúc truy cập được qua E2Guardian.

**Kết quả:** Chưa đạt đầy đủ cho cả hai website.

### Kịch bản 7 – Cảnh báo website streaming

- Ban đầu truy cập được `streaming.lab7.test`.
- Sau khi thêm vào danh sách cảnh báo, E2Guardian hiển thị Access Denied và nút Acknowledge.
- Nhấn Acknowledge thì mở được website.
- Chưa kiểm tra lại khi hết thời gian bypass.

**Kết quả:** Đã kiểm thử cảnh báo và truy cập tạm; chưa xác minh hết hạn bypass.

### Kịch bản 8 – Chặn website game

- Đã cấu hình chặn `blockedgame.lab7.test`.
- Khi bypass bằng 300 giây, Acknowledge vẫn cho phép truy cập.
- Khi đổi bypass về 0, website bị chặn.
- Việc tắt bypass cũng làm mất chức năng Acknowledge ở các trang cảnh báo khác.

**Kết quả:** Chặn được khi tắt bypass, nhưng chưa tách biệt được hard block và cảnh báo.

### Kịch bản 9 – Quét file EICAR

- Truy cập `download.lab7.test/files/eicar.com.txt` qua proxy.
- E2Guardian hiển thị thông báo:

      Virus or bad content detected. Eicar-Signature

- File thử nghiệm bị chặn tải xuống.

**Kết quả:** Đạt.

### Kịch bản 10 – Giới hạn file tải xuống

- Đã cấu hình `reply_body_max_size` bằng 100 MB trong Squid.
- Khi tải trực tiếp qua cổng Squid `3128`:
  - File `10MB.bin` trả HTTP 200.
  - File `110MB.bin` trả HTTP 403.
- Đường truy cập qua E2Guardian `8080` trước đó trả nội dung HTML, nên chưa dùng kết quả này để kết luận.

**Kết quả:** Đạt khi kiểm thử trực tiếp qua Squid; chưa xác minh đầy đủ qua E2Guardian.

## 5. Bảng tổng hợp

| Kịch bản | Kết quả |
|---|---|
| 1. Truy cập web bình thường | Đạt trong phạm vi kiểm thử |
| 2. Chặn nhóm website | Chưa đủ minh chứng |
| 3. Chặn website tìm việc | Chưa đạt hard block khi vẫn bật bypass |
| 4. Cảnh báo nội dung | Chưa kiểm thử hết hạn bypass |
| 5. Cho phép website giáo dục | Đạt trong phạm vi kiểm thử |
| 6. Giới hạn giờ truy cập | Chưa đạt đầy đủ |
| 7. Cảnh báo streaming | Chưa kiểm thử hết hạn bypass |
| 8. Chặn website game | Chưa tách biệt hard block và cảnh báo |
| 9. Quét EICAR | Đạt |
| 10. Giới hạn dung lượng tải | Đạt trực tiếp qua Squid; chưa xác minh đầy đủ qua E2Guardian |

## 6. Trả lời câu hỏi

### 1. Vì sao không cài Squid/SquidGuard trực tiếp trên pfSense?

Trong mô hình bài lab, pfSense đảm nhiệm firewall và định tuyến. Đặt Squid, E2Guardian và ClamAV trên Ubuntu riêng giúp thuận tiện cấu hình, kiểm tra lỗi và giảm ảnh hưởng đến firewall.

### 2. pfSense và Web Proxy khác nhau như thế nào?

pfSense kiểm soát lưu lượng theo IP, cổng và kết nối mạng. Web Proxy xử lý yêu cầu web, tên miền, URL và nội dung tải xuống nên có thể áp dụng chính sách web chi tiết hơn.

### 3. Vì sao phải chặn truy cập trực tiếp cổng 80 và 443 từ Client?

Để người dùng không thể tắt proxy rồi truy cập web trực tiếp nhằm bỏ qua chính sách lọc.

### 4. Vì sao lọc web bằng IP không ổn định?

Một địa chỉ IP có thể phục vụ nhiều website và IP của website có thể thay đổi. Chặn theo IP có thể gây chặn nhầm hoặc bỏ sót.

### 5. Temporary bypass khác hard block như thế nào?

Temporary bypass cho phép người dùng xác nhận để truy cập tạm thời. Hard block chặn truy cập và không cho người dùng tự vượt qua bằng Acknowledge.

### 6. Vì sao không quét được nội dung EICAR qua HTTPS nếu không có SSL MITM?

Nội dung HTTPS được mã hóa. Nếu proxy không giải mã kết nối thì không đọc được nội dung bên trong để quét.

### 7. Lộ private key của CA dùng HTTPS inspection có nguy hiểm gì?

Người có private key có thể tạo chứng chỉ giả được các máy tin tưởng CA đó chấp nhận, dẫn đến nguy cơ giả mạo website và can thiệp kết nối.

### 8. maxcontentfilecachescansize có luôn là giới hạn tải file không?

Không. Tham số này liên quan đến kích thước nội dung phục vụ lưu đệm hoặc quét. Bài lab sử dụng `reply_body_max_size` của Squid để kiểm thử giới hạn tải xuống.

### 9. Vì sao lịch 60 phút không giống daily category quota của Sophos?

Schedule cho phép truy cập trong một khung giờ cố định. Daily quota tính tổng thời gian sử dụng trong ngày.

### 10. Trình duyệt báo lỗi nhưng E2Guardian không có log thì kiểm tra gì đầu tiên?

Kiểm tra Client đã bật đúng proxy `10.6.0.30:8080` chưa, sau đó kiểm tra kết nối mạng và trạng thái dịch vụ.

## 7. Nhận xét

Bài lab đã kiểm thử truy cập web qua proxy, lọc website theo danh sách, cảnh báo nội dung, quét EICAR và giới hạn dung lượng tải xuống.

Các phần cần hoàn thiện gồm:

- Tách biệt hard block và temporary bypass.
- Xác minh việc chặn trở lại sau khi hết thời gian bypass.
- Kiểm thử lịch truy cập đầy đủ cho cả website game và giải trí.
- Xác minh giới hạn tải xuống qua toàn bộ luồng E2Guardian → Squid.

Kết quả được ghi nhận theo các phép thử đã thực hiện. Những trường hợp chưa đủ minh chứng hoặc chưa kiểm thử đầy đủ không được xem là đạt hoàn toàn.
