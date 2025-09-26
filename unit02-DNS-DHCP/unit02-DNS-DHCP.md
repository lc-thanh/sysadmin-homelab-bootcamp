# 📘 Giáo án Unit 2: DNS + DHCP (AD Integrated)

## 🎯 Mục tiêu

- Cấu hình **DNS forwarders** và **Reverse Lookup Zone**
- Cài đặt & authorize **DHCP Server** trong AD
- Tạo scope + options (003 gateway, 006 DNS)
- Kiểm thử client nhận IP động & phân giải tên miền trong domain

---

## 1) Cấu hình DNS Forwarders & Reverse Lookup Zone

### Forwarders

1. Mở **Server Manager → Tools → DNS**.
2. Chuột phải vào **DC01 → Properties → Forwarders**.
3. Thêm DNS public (ví dụ: `8.8.8.8`, `1.1.1.1`).
4. Kiểm tra: chạy trên DC
   ```cmd
   nslookup google.com
   ```

### Reverse Lookup Zone

1. Trong DNS Manager → **Reverse Lookup Zones → New Zone**.
2. Zone type: **Primary Zone**, lưu trong AD (AD Integrated).
3. Network ID: `192.168.118`.
4. Xác nhận PTR record sẽ tự động sinh khi client được cấp IP.

---

## 2) Cài & Authorize DHCP Server

1. **Server Manager → Add roles and features → DHCP Server → Install**.
2. Sau khi cài, chọn **Complete DHCP Configuration**.
3. Authorize server trong AD bằng tài khoản Domain Admin.
4. Mở **DHCP Console** → kiểm tra server `DC01.lab.local` có dấu mũi tên xanh (đã authorized).

---

## 3) Tạo DHCP Scope & Options

1. Trong DHCP Console → **IPv4 → New Scope**.
   - Tên scope: `LabScope`
   - Start IP: `192.168.118.100`
   - End IP: `192.168.118.200`
   - Subnet mask: `255.255.255.0`
   - Exclusion: `192.168.118.100–119` (dành cho server/manual IP)
   - Lease duration: mặc định (8 ngày)
2. Scope Options:
   - 003 Router (Default Gateway) (Để các máy khách có thể truy cập Internet, nhưng trong Lab hiện tại, các VMs sử dụng thêm 1 Adapter Bridged để truy cập mạng nên không cần): `192.168.118.1`
   - 006 DNS Servers: `192.168.118.10` (DC01)
   - 015 DNS Domain Name: `lab.local`

---

## 4) Kiểm tra trên Windows Client

1. Chỉnh card Host-only của Client về **Obtain IP automatically**.
2. Chạy lệnh:
   ```cmd
   ipconfig /release
   ipconfig /renew
   ipconfig /all
   ```
   → Kiểm tra IP nằm trong dải `192.168.118.100–200`, DNS = `192.168.118.10`.
3. Test ping & DNS:
   ```cmd
   ping dc01.lab.local
   nslookup dc01.lab.local
   nslookup google.com
   ```

---

## ✅ Kết quả cần có

- DC01 cấu hình forwarders và reverse lookup thành công.
- DHCP server được authorize trong AD.
- Windows Client nhận IP động từ DHCP (192.168.118.x).
- Client phân giải được cả tên miền nội bộ (`dc01.lab.local`) và internet (`google.com`).
