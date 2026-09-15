# ATTT
# LAB 1 - Bắt gói tin Telnet - SSH

## 1. Thông tin sinh viên
Họ và tên: Trần Văn Lâm
MSSV: 1150080102
Lớp: CNPM2

## 2. Tên bài Lab

**Examining SSH & Telnet in Wireshark**

## 3. Môi trường thực hiện

Bài Lab được thực hiện trên máy tính PC với môi trường mạng thực tế.

### Công cụ sử dụng

* Windows
* Wireshark
* PuTTY
* Telnet
* SSH

### Mô hình thực hiện

Mô hình gồm các thành phần:

* **Client:** Máy tính sử dụng PuTTY để kết nối đến Server.
* **Server:** Máy tính cung cấp dịch vụ Telnet và SSH.
* **Wireshark:** Được sử dụng để bắt và phân tích lưu lượng mạng.

## 4. Nội dung đã thực hiện

### 4.1. Thiết lập môi trường

* Kiểm tra kết nối mạng giữa Client và Server.
* Kiểm tra địa chỉ IP của các máy.
* Kiểm tra khả năng kết nối bằng lệnh `ping`.
* Cài đặt và cấu hình Wireshark.
* Cài đặt và sử dụng PuTTY để thực hiện kết nối từ Client đến Server.

### 4.2. Bắt gói tin Telnet

* Khởi động dịch vụ Telnet trên Server.
* Sử dụng PuTTY để kết nối đến Server thông qua Telnet.
* Sử dụng Wireshark để bắt lưu lượng Telnet.
* Lọc các gói tin sử dụng TCP port 23.
* Thực hiện đăng nhập bằng tài khoản thử nghiệm.
* Thực hiện một số thao tác trên Server.
* Phân tích các gói tin đã bắt được bằng Wireshark.
* Sử dụng chức năng **Follow TCP Stream** để quan sát phiên trao đổi dữ liệu.

### 4.3. Bắt gói tin SSH

* Khởi động và kiểm tra dịch vụ SSH trên Server.
* Sử dụng PuTTY để kết nối đến Server thông qua SSH.
* Sử dụng Wireshark để bắt lưu lượng SSH.
* Lọc các gói tin sử dụng TCP port 22.
* Thực hiện đăng nhập bằng tài khoản
