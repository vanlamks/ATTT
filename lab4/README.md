# LAB 4 – Khảo sát và đánh giá bề mặt mạng bằng Nmap

| Thông tin | Nội dung |
| --- | --- |
| Họ và tên | Trần Văn Lâm |
| MSSV | 1150080102 |
| Lớp | 11CNPM2 |
| Môn | An toàn hệ thống thông tin |
| Ngày thực hiện | 29/09/2026 |
| Video | https://www.youtube.com/@lamvan1020 |

> **Cam kết:** Toàn bộ ảnh chụp lấy trực tiếp từ máy ảo của em, các lệnh do em tự gõ, chỉ quét các máy trong mạng VirtualBox Host-Only (`192.168.56.0/24`). Không đưa mật khẩu, token hay dữ liệu cá nhân lên repo.

---

## 1. Mục tiêu

- Dựng môi trường Kali Linux (máy quét) trong mạng VirtualBox Host-Only.
- Thực hiện host discovery, kiểm tra kết nối và TCP Connect scan bằng Nmap.
- Đọc và giải thích các trạng thái cổng `open` / `closed` / `filtered`.

## 2. Cấu hình môi trường

| Thành phần | Giá trị |
| --- | --- |
| Máy quét | Kali Linux VM |
| Card mạng | Host-Only (`eth0`) |
| IP Kali | `192.168.56.101/24` |
| Dải mạng | `192.168.56.0/24` |
| Phiên bản Nmap (Kali) | 7.95 |

Các host phát hiện trong mạng:

| IP | Ghi chú |
| --- | --- |
| `192.168.56.1` | Ping/scan **up** – adapter Host-Only của máy thật |
| `192.168.56.100` | Ping **up** – NIC ảo VirtualBox |
| `192.168.56.101` | Kali (máy quét) |

## 3. Các bước thực hiện

### 3.1. Kiểm tra IP của Kali

```bash
ip -br addr
```

![ip -br addr](images/4.2_ip-br-addr_kali.png)

**Kết quả:** `eth0` ở trạng thái `UP`, IP `192.168.56.101/24`. Kali đã nằm đúng dải Host-Only `192.168.56.0/24`.

### 3.2. Host discovery toàn dải /24

```bash
nmap -sn 192.168.56.0/24
```

![nmap -sn](images/5.1_nmap-sn.png)

**Kết quả:** quét 256 IP, phát hiện **3 host up** trong 35.52 giây.

| STT | IP | MAC / Vendor | Vai trò suy đoán | Bằng chứng |
| --- | --- | --- | --- | --- |
| 1 | 192.168.56.1 | `0A:00:27:00:00:12` (Unknown) | Adapter Host-Only của máy thật (Windows) | Tiền tố MAC `0A:00:27` do VirtualBox tạo; latency 0.00042s |
| 2 | 192.168.56.100 | `08:00:27:1E:19:09` (PCS Systemtechnik/Oracle VirtualBox virtual NIC) | NIC ảo VirtualBox (nhiều khả năng là DHCP server của Host-Only Network) | Vendor Oracle VirtualBox; ping TTL=255; toàn bộ cổng TCP đóng |
| 3 | 192.168.56.101 | Không hiển thị | Kali (chính máy quét) | Khớp kết quả `ip -br addr`; Nmap không hiện MAC vì quét chính máy mình |

### 3.3. Kiểm tra kết nối trước khi quét

```bash
ping -c 4 192.168.56.100
```

![ping](images/4.4_ping-192.168.56.100.png)

**Kết quả:** gửi 4, nhận 4, mất gói 0%, RTT min/avg/max = 0.255 / 0.377 / 0.611 ms, TTL = 255. Kết nối thông, đủ điều kiện để chuyển sang bước quét.

Máy `192.168.56.1` cũng phản hồi ping (up).

### 3.4. TCP Connect scan (`-sT`)

**Mục tiêu 192.168.56.100**

```bash
nmap -sT 192.168.56.100
```

![nmap -sT .100](images/6.1_nmap-sT_192.168.56.100.png)

- Host up (latency 0.0025s), quét trong 17.68 giây.
- 1000 cổng đều ở trạng thái **closed** (`conn-refused`), không có cổng open.

**Mục tiêu 192.168.56.1**

```bash
nmap -sT 192.168.56.1
```

![nmap -sT .1](images/6.1_nmap-sT_192.168.56.1.png)

- Host up (latency 0.00092s), quét trong 21.45 giây.
- 999 cổng **filtered** (`no-response`), 1 cổng **open**:

| Cổng | Trạng thái | Dịch vụ |
| --- | --- | --- |
| 3306/tcp | open | mysql |

## 4. Bảng tổng hợp

| Mục tiêu | Open | Closed | Filtered | Thời gian |
| --- | --- | --- | --- | --- |
| 192.168.56.100 | 0 | 1000 | 0 | 17.68 s |
| 192.168.56.1 | 1 (3306/mysql) | 0 | 999 | 21.45 s |

## 5. Nhận xét

- **`closed` vs `filtered`:** ở `.100`, máy đích trả về RST nên Nmap ghi `conn-refused` → cổng đóng, không có firewall chặn. Ở `.1`, các gói không nhận được phản hồi (`no-response`) → nhiều khả năng Windows Firewall đang bỏ gói (drop), chỉ để lọt cổng 3306.
- **Vì sao `-sT` không cần quyền root:** `-sT` dùng lời gọi hệ thống `connect()` để hoàn tất bắt tay 3 bước, không cần tạo gói thô (raw packet) như `-sS`. Đổi lại, kết nối bị ghi log trên máy đích và quét chậm hơn.
- **Rủi ro:** máy thật `192.168.56.1` đang mở cổng **3306 (MySQL)** ra mạng Host-Only. Nếu không cần thiết nên bind MySQL vào `127.0.0.1` hoặc chặn cổng bằng firewall.
- **Về `192.168.56.100`:** TTL=255 và toàn bộ cổng đóng cho thấy đây không phải một máy Linux đầy đủ dịch vụ mà nhiều khả năng là thành phần ảo của VirtualBox (DHCP). Đây là host "dư" so với số VM đang bật, đúng như tài liệu lab dự đoán (adapter máy thật hoặc DHCP service).

## 6. Việc còn thiếu (chưa có ảnh/kết quả)

- [ ] Máy đích Metasploitable 2 (chưa xuất hiện trong kết quả quét – cần bật VM, chỉ Host-Only, rồi quét lại `-sn`)
- [ ] SYN scan `-sS` (chạy với `sudo`) và bảng so sánh `-sT` / `-sS`
- [ ] Version detection `-sV`, OS detection `-O` / `-A`
- [ ] NSE script SMB (`smb-os-discovery`, `smb-vuln-ms17-010`)
- [ ] Xuất kết quả `-oN`, `-oX`, `-oG` và chuyển XML sang HTML
- [ ] Ảnh cấu hình VirtualBox Host-Only và snapshot `Before-LAB4`

## 7. Cấu trúc thư mục

```
LAB4/
├── README.md
└── images/
    ├── 4.2_ip-br-addr_kali.png
    ├── 4.4_ping-192.168.56.100.png
    ├── 5.1_nmap-sn.png
    ├── 6.1_nmap-sT_192.168.56.100.png
    └── 6.1_nmap-sT_192.168.56.1.png
```

> Bài lab chỉ phục vụ học tập và nghiên cứu trên máy ảo cá nhân trong mạng Host-Only.
