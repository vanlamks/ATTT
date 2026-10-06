README - TIẾN ĐỘ LAB pfSense (cập nhật 06/10/2026)
====================================================

1. FILE TRONG BỘ BÀI
--------------------
- Lab3_BaoCao       : báo cáo nộp bài (đã điền phần làm xong, ô nền vàng = cần bổ sung)
- README_TienDo_Lab_pfSense.txt   : file này (ghi tiến độ, thông số, việc cần làm)
Khi nộp: đổi tên báo cáo thành  [Mã lớp]-Lab3_MSSV-TênSV.docx

2. THÔNG SỐ ĐANG DÙNG
---------------------
pfSense CE 2.7.2-RELEASE (amd64), VirtualBox
  VM pfSense : FreeBSD 64-bit, 2 GB RAM, 2 CPU, đĩa 16 GB (VDI), không bật EFI
  Adapter 1  : Bridged Adapter (card Wi-Fi)         -> WAN  (em0)
  Adapter 2  : Host-only (10.0.0.100/8, DHCP tắt)   -> LAN  (em1)
  Adapter 3  : Internal Network, tên dmz-net        -> DMZ  (em2, gán OPT1 trong WebGUI)
  Adapter 4  : TẮT

  LAN pfSense      : 10.0.0.1/8 (không bật DHCP)
  Máy thật         : 10.0.0.100/8 (không Gateway, không DNS)
  Domain Controller: 10.0.0.2/8, GW 10.0.0.1, DNS 10.0.0.2
  DMZ pfSense      : 172.16.0.1/16
  DMZ-Web          : 172.16.0.2/16, GW 172.16.0.1 (DNS 8.8.8.8 đặt sau khi có rule Pass DMZ)
  LAN-Test         : 10.0.0.3/8, GW 10.0.0.1 (chỉ cho Tình huống 2)

  Đăng nhập WebGUI : https://10.0.0.1   (mặc định admin / pfsense, đổi trong Setup Wizard)
  SHA-256 của .iso.gz: 883fb7bc64fe548442ed007911341dd34e178449f8156ad65f7381a02b7cd9e4

3. ĐÃ XONG
----------
[x] Tải ISO, kiểm tra SHA-256, giải nén .iso
[x] Tạo VM pfSense, gắn ISO, cài đặt (Auto ZFS), tháo ISO
[x] Đặt Adapter 1 (Bridged)
[x] Host-only 10.0.0.100/8 (nhớ kiểm tra DHCP Server = Disabled)
[x] Console: LAN = 10.0.0.1/8
[x] Ping từ máy thật tới 10.0.0.1 thành công (4/4, TTL=64)

4. CÒN LẠI (theo thứ tự nên làm)
--------------------------------
[ ] Chụp bù: Adapter 2, Adapter 3, Get-FileHash, Network Manager (DHCP Disabled)
[ ] WebGUI https://10.0.0.1 -> Setup Wizard (DNS 8.8.8.8, múi giờ Asia/Ho_Chi_Minh,
    WAN DHCP và BỎ tick Block private/bogon networks, LAN 10.0.0.1/8, đổi mật khẩu) -> chụp Dashboard
[ ] Kiểm tra WAN đã có IP chưa (Dashboard). Nếu chưa: Release/Renew; vẫn không được thì dùng
    NAT Network 192.168.250.0/24 (KHÔNG dùng NAT mặc định 10.0.2.0/24)
[ ] Interfaces -> Assignments: thêm card 3 làm OPT1, đặt tên DMZ, Static 172.16.0.1/16
[ ] Firewall -> NAT -> Outbound: kiểm tra Automatic Rules có LAN và DMZ
[ ] Firewall -> Rules -> LAN: Disable 2 rule "Default allow LAN to any" (GIỮ Anti-Lockout),
    Apply, Reset States, rồi tạo rule Pass LAN net -> Any
[ ] Dựng VM Domain Controller (Windows Server 2019/2022), AD DS, DNS Forwarder 8.8.8.8
[ ] Kiểm tra rule nền tảng từ DC (ping 8.8.8.8, Resolve-DnsName, curl.exe -4)
[ ] Dựng VM DMZ-Web (Windows Server + IIS) cho Tình huống 3 và 4
[ ] Tình huống 1 đến 5 (mỗi tình huống: ảnh rule, ảnh kiểm thử, 1-2 câu giải thích)
[ ] Điền họ tên, MSSV, mã lớp trong báo cáo; xóa các ghi chú nền vàng

5. QUY TẮC QUAN TRỌNG KHI TEST FIREWALL
---------------------------------------
- Sau MỖI lần đổi rule: Apply Changes -> Diagnostics -> States -> Reset States -> test bằng lệnh MỚI.
- Rule xét từ TRÊN xuống, khớp rule đầu tiên là dừng.
- Diagnostics -> Ping của pfSense KHÔNG đi qua rule LAN; phải test từ DC hoặc LAN-Test.
- Tình huống 3: phải có ảnh baseline (ping thành công) TRƯỚC khi thêm Block, rồi ảnh thất bại SAU khi Block.
- Không bật tất cả VM cùng lúc (TH2: pfSense+DC+LAN-Test; TH3: pfSense+DC+DMZ-Web; TH4: pfSense+DMZ-Web).

6. LỖI ĐÃ GẶP VÀ CÁCH XỬ LÝ
---------------------------
- Máy boot lại vào trình cài đặt  -> Power Off, Settings -> Storage, gỡ ISO, Start lại.
- "Not enough disks selected"     -> ở bước chọn ổ, nhấn Space để hiện [*] ada0 rồi mới Enter.
- Get-FileHash báo không thấy file -> cd vào thư mục INSTALL_MEDIA trước (PowerShell mặc định ở System32).
- ISO bị lặp tên trong ô đường dẫn -> chọn lại file bằng Other..., không dán thừa tên file.


