# LAB 3: Các Mối Đe Dọa An Toàn Thông Tin & Phân Tích Tài Nguyên

## 1. Thông tin sinh viên
* **Họ và tên:** Trần Văn Lâm
* **MSSV:** [Điền MSSV của bạn]
* **Tên Lab:** LAB 3 - Threats & Assets / Các mối đe dọa an toàn thông tin
* **Phiên bản môi trường:** Ubuntu 22.04 LTS / Python 3.10 / Docker Engine 24.0.5

## 2. Cách dựng môi trường
1. Khởi tạo máy ảo Ubuntu hoặc môi trường Linux cục bộ.
2. Cài đặt Python 3 và các thư viện phụ thuộc bằng lệnh:
   ```bash
   sudo apt update && sudo apt install -y python3-pip python3-venv
   ```
3. Thiết lập thư mục làm việc cho LAB3 và giải nén các tài nguyên mẫu an toàn (`ddos_sample.csv`, `mailbomb_sample.csv`, `phishing_email.txt`).
4. Cấu hình kiểm thử cục bộ bằng kịch bản `local_load_test.py` nhắm mục tiêu độc quyền vào `127.0.0.1:8080`.

## 3. Các tình huống đã thực hiện
* **Tình huống 1:** Phân tích mẫu dữ liệu tấn công Từ chối dịch vụ (DDoS) thông qua tệp `ddos_sample.csv` đã được làm sạch log.
* **Tình huống 2:** Kiểm tra mẫu email rác/bom thư (Mail Bomb) qua tệp `mailbomb_sample.csv`.
* **Tình huống 3:** Đánh giá mã độc/kịch bản kiểm thử tải cục bộ `local_load_test.py` trên giao diện loopback (`127.0.0.1:8080`).
* **Tình huống 4:** Kiểm tra mẫu lừa đảo trực tuyến (Phishing) từ `phishing_email.txt` và phân tích các dấu hiệu Social Engineering.

## 4. Kết quả thực hiện
* **Tình huống 1 (DDoS Log Analysis):** PASS
* **Tình huống 2 (Mail Bomb Sample):** PASS
* **Tình huống 3 (Local Load Test):** PASS
* **Tình huống 4 (Phishing/Social Engineering):** PASS

## 5. Lỗi gặp phải và cách khắc phục
* **Lỗi 1:** Xung đột cổng kết nối `8080` khi chạy kịch bản kiểm thử tải.
  * *Cách khắc phục:* Kiểm tra các tiến trình đang chạy bằng lệnh `ss -lntp` và tắt tiến trình chiếm cổng trước khi thực thi `local_load_test.py`.
* **Lỗi 2:** Thiếu quyền thực thi trên các tệp kịch bản python.
  * *Cách khắc phục:* Sử dụng lệnh `chmod +x` cho các tệp script hoặc chạy trực tiếp thông qua lệnh thông dịch `python3`.