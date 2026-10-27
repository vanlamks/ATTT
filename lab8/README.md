# LAB 8 – Local DNS Attack (DNS Spoofing & DNS Cache Poisoning)

Bài thực hành An toàn Hệ thống Thông tin – mô phỏng tấn công DNS cục bộ trên mạng ảo cô lập **DNSLAB (10.9.0.0/24)**, kế thừa hạ tầng pfSense từ các LAB trước.

> ⚠️ **Phạm vi an toàn bắt buộc**: toàn bộ thao tác chỉ thực hiện trong mạng ảo DNSLAB, giữa các máy ảo cá nhân của sinh viên. Không sử dụng script hay kỹ thuật trong repo này trên mạng thật, Wi-Fi trường, Internet hay hệ thống của người khác. DNSLAB không có đường ra WAN trong pha tấn công. Script tấn công đã được hard-code theo đúng IP/domain của bài lab để không thể dùng ngoài phạm vi này.

## 1. Thông tin bài nộp

| Mục | Giá trị |
|---|---|
| Họ và tên | Trần Văn Lâm |
| Lớp | 11CNPM2 |
| Bài | LAB 8 – Local DNS Attack |
| Báo cáo | `LAB8_Local_DNS_Attack_BaoCao.docx` |

## 2. Mô hình mạng

LAB 8 kế thừa pfSense WAN/DMZ/MGMT và LAN-DOMAIN (10.6.0.0/24) từ các LAB trước, bổ sung interface **OPT2 = DNSLAB (10.9.0.1/24)**. LAN-DOMAIN không tham gia kịch bản tấn công.

| Thiết bị | IP | Vai trò |
|---|---|---|
| pfSense | DNSLAB OPT2: 10.9.0.1/24 | Firewall, cô lập DNSLAB |
| DNS-SERVER | 10.9.0.10 (Ubuntu Server 24.04) | BIND9 – recursive/cache resolver |
| VICTIM | 10.9.0.100 (Ubuntu 24.04) | Máy nạn nhân |
| ATTACKER | 10.9.0.200 (Kali Linux) | Chạy script spoof + fake web |
| APP-TEST / AUTH-DNS | 172.16.6.20 (kế thừa) | Authoritative DNS mô phỏng, delay 1.5s |
| DMZ-WEB | 172.16.6.10 (kế thừa) | Website hợp lệ cần bảo vệ |

## 3. Cấu trúc thư mục tài nguyên

```
LAB9_Resources/
├── 01_DNS_SERVER/        # named.conf.options, named.conf.local, zone lab9.test (+ nghịch)
├── 02_VICTIM/             # netplan mẫu 10.9.0.100, DNS = 10.9.0.10
├── 03_ATTACKER/           # dns_spoof_lab9.py, fake_index.html
├── 04_APP_TEST_AUTH_DNS/  # auth_dns_delayed.py (authoritative mô phỏng, delay 1.5s)
├── 05_PFSENSE/            # firewall_policy_matrix.csv
├── 06_WIRESHARK/          # bộ display filter
├── 07_REPORT/             # mẫu báo cáo Word
├── SHA256SUMS.txt
└── README_FIRST.txt
```

## 4. Các bước thực hiện (tóm tắt)

1. **Dựng mạng DNSLAB**: snapshot pfSense/Kali → thêm Adapter 5 (Internal Network `dns-lab`) → gán OPT2 = DNSLAB (10.9.0.1/24, không gateway) → áp firewall_policy_matrix.csv để cô lập DNSLAB khỏi LAN-DOMAIN/MGMT/WAN.
2. **Cấu hình DNS-SERVER (BIND9)**: IP tĩnh 10.9.0.10 → cài `bind9` → chép 2 file `named.conf.*` và 2 zone file → `named-checkconf`/`named-checkzone` → `systemctl restart bind9`.
3. **Cấu hình VICTIM**: netplan tĩnh 10.9.0.100, DNS duy nhất 10.9.0.10.
4. **Cấu hình ATTACKER (Kali)**: IP 10.9.0.200, cài Scapy/Wireshark/tcpdump, Promiscuous Mode = Allow VMs trên card `dns-lab`, chạy fake web (`fake_index.html`) trên cổng 80.
5. **Baseline**: chạy `auth_dns_delayed.py` trên APP-TEST, xác nhận `www.external.lab9.test` → 172.16.6.10 trước khi tấn công.
6. **Task 1 – Hosts file redirection**: sửa `/etc/hosts` trên VICTIM, so sánh với kết quả `dig` trực tiếp.
7. **Task 2 – Direct DNS Spoofing**: `dns_spoof_lab9.py --mode victim` trên Kali, giả mạo phản hồi gửi thẳng tới VICTIM.
8. **Task 3 – DNS Cache Poisoning**: `dns_spoof_lab9.py --mode resolver` trên Kali, giả mạo phản hồi gửi tới DNS-SERVER để đầu độc cache dùng chung.
9. **Khôi phục & Hardening**: flush cache, khôi phục `/etc/hosts`, trả Promiscuous Mode về Deny, bật lại DNS COOKIE và lặp Task 3 để so sánh hiệu quả phòng thủ.

Chi tiết lệnh, cấu hình và kết quả kỳ vọng từng bước được trình bày đầy đủ trong `LAB8_Local_DNS_Attack_BaoCao.docx`.

## 5. Kiểm tra toàn vẹn gói tài nguyên

```bash
sha256sum -c SHA256SUMS.txt
```

## 6. Tài liệu tham khảo

- SEED Labs, *Local DNS Attack Lab*
- Ubuntu Server Documentation – [Domain Name Service (DNS)](https://ubuntu.com/server/docs/how-to/networking/install-dns/)
- Netgate pfSense – [Interface Configuration](https://docs.netgate.com/pfsense/en/latest/interfaces/configure.html)
- Oracle VirtualBox 7.2 – [Virtual Networking / Promiscuous Mode](https://docs.oracle.com/en/virtualization/virtualbox/7.2/user/networkingdetails.html)
- ISC BIND 9 – [Configuration Reference](https://bind9.readthedocs.io/en/stable/reference.html)
- RFC 2606 – [Reserved Top Level DNS Names](https://datatracker.ietf.org/doc/html/rfc2606)
