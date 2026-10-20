# LAB 7 - Xây dựng chính sách Web Protection trên pfSense + Web Proxy

Môn: Thực hành An toàn hệ thống thông tin

| Thông tin | Nội dung |
|---|---|
| Họ và tên | Trần Văn Lâm |
| Lớp | 11CNPM2 |
| MSSV | ........................ |
| Báo cáo | `Lab7_pfSense_BaoCao.docx` |
| Video demo | [YouTube](https://www.youtube.com/@lamvan1020) |

## 1. Mục tiêu

Kế thừa môi trường LAB 6 và bổ sung một máy WEB-PROXY (Ubuntu Server 24.04) chạy E2Guardian, Squid và ClamAV. pfSense chỉ làm firewall/gateway và ép Client-A/B bắt buộc đi qua proxy; việc lọc nội dung web (L7) do WEB-PROXY đảm nhiệm. Trên nền đó xây dựng các chính sách: hard block, warning kèm temporary bypass, whitelist, giới hạn theo khung thời gian, quét virus khi tải file.

Luồng truy cập:

```
Client-A/B -> E2Guardian 10.6.0.30:8080 -> Squid 127.0.0.1:3128 -> pfSense -> Website
```

## 2. Mô hình mạng

| Thiết bị | Địa chỉ | Vai trò |
|---|---|---|
| pfSense-LAB6 | MGMT 192.168.56.254; LAN 10.6.0.1; DMZ 172.16.6.1 | Firewall/gateway, dùng lại từ LAB 6 |
| DC01 | 10.6.0.10/24 | AD DS + DNS, thêm zone `lab7.test` |
| Client-A | 10.6.0.21/24 | Máy kiểm thử, dùng proxy 10.6.0.30:8080 |
| Client-B | 10.6.0.22/24 | Máy kiểm thử, dùng proxy 10.6.0.30:8080 |
| DMZ-WEB | 172.16.6.10/24 | Nginx, 13 website mô phỏng `*.lab7.test` |
| WEB-PROXY (mới) | 10.6.0.30/24, GW 10.6.0.1, DNS 10.6.0.10 | E2Guardian :8080, Squid :3128, ClamAV |

## 3. Cấu hình máy ảo WEB-PROXY

| Mục | Giá trị |
|---|---|
| Hệ điều hành | Ubuntu Server 24.04 LTS |
| RAM / CPU / Disk | 2 GB / 2 vCPU / 20 GB |
| Card mạng | Internal Network `lab6-lan` |
| IP / Gateway / DNS | `10.6.0.30/24` / `10.6.0.1` / `10.6.0.10` |

Squid chỉ nghe `127.0.0.1:3128` để client bắt buộc đi vào E2Guardian trước.

## 4. Các bước thực hiện

1. **Snapshot:** chụp snapshot `LAB7_START` cho các máy trước khi ép proxy.
2. **DMZ-WEB:** chạy `sudo bash setup_lab7_dmz_web.sh` để tạo 13 website Nginx và các file `10MB.bin`, `100MB.bin`, `110MB.bin`, `eicar.com.txt`.
3. **DC01:** chạy `Add-LAB7-DNS.ps1` (PowerShell quyền Administrator) để tạo zone `lab7.test`, các bản ghi A trỏ về `172.16.6.10`.
4. **WEB-PROXY - mạng:** đặt IP tĩnh bằng netplan (`10.6.0.30/24`, gateway `10.6.0.1`, DNS `10.6.0.10`), ping thử `10.6.0.1` và `10.6.0.10`.
5. **WEB-PROXY - cài gói:** `sudo bash install_lab7_proxy.sh` (squid, e2guardian, clamav, clamav-daemon). Cài trước khi bật rule chặn trên pfSense.
6. **Squid:** sao lưu `squid.conf`, đặt `http_port 127.0.0.1:3128`; `http_access allow localhost` phải nằm trước `http_access deny all`.
7. **E2Guardian:**
   - `e2guardian.conf`: `filterip = 10.6.0.30`, `filterports = 8080`, `proxyip = 127.0.0.1`, `proxyport = 3128`, `loglevel = 2`, bật `contentscanner` (clamdscan), `enablessl = off`.
   - `e2guardianf1.conf`: `reportinglevel = 3`, `bypass = 300`, `contentscanexceptions = off`.
8. **Nạp danh sách:**
   ```bash
   cat hard_block_sites.txt warning_sites.txt | sudo tee -a /etc/e2guardian/lists/bannedsitelist
   cat allow_sites.txt | sudo tee -a /etc/e2guardian/lists/exceptionsitelist
   sudo systemctl restart squid e2guardian clamav-daemon
   ss -tlnp | grep -E "8080|3128"
   ```
   `hard_block_sites` phải nằm trong danh sách non-bypass (nếu package có), `warning_sites` thì không.
9. **Trình duyệt Client-A/B:** Manual Proxy `10.6.0.30`, port `8080`.
10. **pfSense - Alias:** `LAB7_CLIENTS` (10.6.0.21, 10.6.0.22), `LAB7_PROXY` (10.6.0.30), `WEB_PORTS` (80, 443), `MAIL_PORTS` (25, 465, 587, 110, 995, 143, 993).
11. **pfSense - Firewall > Rules > LAN**, đúng thứ tự:

    | # | Action | Source | Destination | Port | Ý nghĩa |
    |---|---|---|---|---|---|
    | 1 | Pass | LAN net | 10.6.0.10 | TCP/UDP 53 | DNS về DC01 |
    | 2 | Pass | LAB7_CLIENTS | LAB7_PROXY | TCP 8080 | Client dùng E2Guardian |
    | 3 | Pass | LAN net | Any | MAIL_PORTS | Cho email |
    | 4 | Block | LAB7_CLIENTS | Any | WEB_PORTS | Chặn web trực tiếp (chống bypass) |
    | 5 | Pass | LAB7_PROXY | Any | WEB_PORTS | Cho proxy đi HTTP/HTTPS |
    | 6 | Pass | LAB7_PROXY | 172.16.6.10 | TCP 80 | Cho proxy vào site test DMZ |
    | 7 | Block | LAN net | Any | Any | Explicit deny |

    Sau đó Apply Changes và Diagnostics > States > Reset States.
12. **Entertainment/Game:** tạo Schedule `LAB7_ENT_60M` trên pfSense, gắn vào rule cho phép truy cập nhóm entertainment/games trong khung 60 phút.

## 5. Kiểm tra theo kịch bản

| Kịch bản | Đích kiểm thử | Kết quả mong đợi |
|---|---|---|
| 1. Baseline | `normal.lab7.test` | Vào được qua proxy; tắt proxy thì pfSense chặn direct 80/443 |
| 2. Geo-Sim | `china`, `mideast` | Hard block, không có bypass |
| 3. Job Search | `jobs.lab7.test` | Hard block |
| 4. Warning | `weapons`, `politics`, `religion` | Trang cảnh báo, bypass tạm 5 phút |
| 5. Education | `education.lab7.test` | Vào bình thường, không cảnh báo |
| 6. Entertainment/Game | `entertainment`, `games` | Chỉ trong khung Schedule 60 phút |
| 7. Streaming | `streaming.lab7.test` | Cảnh báo, bypass tạm |
| 8. Specific game | `blockedgame.lab7.test` | Hard block |
| 9. Antivirus | `download.lab7.test/files/eicar.com.txt` | ClamAV chặn file EICAR |
| 10. Size (tùy chọn) | `100MB.bin`, `110MB.bin` | `110MB.bin` bị từ chối, chỉ tính khi bật `reply_body_max_size` |

Đối chiếu log sau mỗi kịch bản:

| Nguồn | Vị trí |
|---|---|
| pfSense | Status > System Logs > Firewall |
| E2Guardian | `/var/log/e2guardian/access.log` |
| Squid | `/var/log/squid/access.log` |
| ClamAV | `journalctl -u clamav-daemon` |

## 6. Cấu trúc thư mục

```
.
├── README.md
├── Lab7_pfSense_BaoCao.docx   # Báo cáo kèm ảnh minh chứng
└── config.xml                 # Cấu hình pfSense xuất từ Diagnostics > Backup & Restore
```

Không đưa private key CA (`ca.key`, `cert.key`) lên GitHub.

## 7. Lưu ý

- Không cài Squid/SquidGuard trực tiếp trên pfSense vì Netgate đã đánh dấu các package này là deprecated.
- Tên file danh sách của E2Guardian có thể khác nhẹ tùy bản package, kiểm tra bằng `ls /etc/e2guardian/lists/`.
- `clamav-daemon` có thể không chạy nếu chưa tải cơ sở dữ liệu virus, khi đó chạy `sudo freshclam` rồi restart dịch vụ.
- pfSense là stateful firewall, sau khi đổi rule cần Reset States hoặc tạo kết nối mới để rule có hiệu lực.
- `maxcontentfilecachescansize` của E2Guardian là giới hạn cache/quét, không phải hard download-size limit. Mục 100 MB chỉ đạt khi Squid hỗ trợ `reply_body_max_size` và `110MB.bin` bị từ chối khi kiểm thử.
- Schedule 60 phút là time-window policy, không phải daily category quota như Sophos.
- Mọi domain `lab7.test` cùng trỏ về `172.16.6.10` nên rule pfSense không phân biệt được tên miền; phần Schedule cần đích riêng (ví dụ thêm IP phụ cho entertainment/games) hoặc theo hướng dẫn của giảng viên.
- EICAR là chuỗi kiểm thử antivirus, không phải mã độc.
- HTTPS inspection (nếu làm) chỉ dùng trên máy ảo lab cô lập, không import CA vào máy cá nhân.

## 8. Tài liệu tham khảo

- Netgate, pfSense Documentation: Firewall Rules, Aliases, Time Based Rules, Captive Portal và thông báo ngừng hỗ trợ package Squid/SquidGuard/Lightsquid.
- Ubuntu Packages (24.04 LTS): Squid, E2Guardian, ClamAV.
- E2Guardian Project Documentation: filter groups, temporary bypass, AV scanner, SSL MITM.
- EICAR, Anti-Malware Testfile.
