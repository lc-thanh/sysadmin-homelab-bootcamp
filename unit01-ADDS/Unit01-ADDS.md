# 📘 Giáo án Tuần 1–2: Cài đặt Active Directory Domain Services (AD DS) + Setup Mạng (Host‑only + Bridged)

## 🎯 Mục tiêu

- Cấu hình **Windows Server** thành Domain Controller (DC)
- Tạo domain `lab.local`
- Tạo OU + user test
- Join máy Windows Client vào domain
- **Mạng cho VM:** Adapter 1 = **Host‑only (VMnet1)**, Adapter 2 = **Bridged** (ra mạng thật)
- Kiểm tra kết nối mạng (ping, firewall)

---

## 0) Setup mạng cho các VM (VMware Workstation Pro)

### 🔹 Mô hình 2 card / máy

**Vào VM -> Settings -> Add... để add thêm một card mạng cho VM**

- **Adapter 1 = VMnet1 (Host‑only)** → mạng lab nội bộ (Domain/DNS/DHCP)
- **Adapter 2 = Bridged** → nhận IP từ router thật (internet/update) (Để truy cập Internet khi cần)

### 🔹 Virtual Network Editor

**Vào Edit -> Virtual Network Editor để tắt DHCP**

- `VMnet1` (Host‑only): Subnet **192.168.118.0/24**
- **Tắt DHCP** của VMware cho VMnet1 (để sau này DC cấp DHCP)

### 🔹 IP planning

**Domain Controller (DC01)**

- Host‑only: `192.168.118.10/24` (Static), **DNS = 192.168.118.10**
- Bridged: DHCP từ router thật (vd `192.168.1.50`) – chỉ để đi internet

**Windows Client**

- Host‑only: `192.168.118.20/24` (Static _hoặc_ sau này lấy DHCP từ DC)
- **DNS = 192.168.118.10** (IP DC)
- Bridged: DHCP từ router thật (vd `192.168.1.60`)

**Ubuntu**

- Host‑only: `192.168.118.30/24` (Static)
- **DNS = 192.168.118.10**
- Bridged: DHCP từ router thật (vd `192.168.1.70`)

> Hệ điều hành sẽ tự chọn route: traffic nội bộ (192.168.118.0/24) đi qua Host‑only; internet (0.0.0.0/0) đi qua Bridged.

---

## 1) Đặt hostname & IP tĩnh cho Windows Server (DC)

```powershell
Rename-Computer -NewName "DC01" -Restart
```

- Host‑only IP: `192.168.118.10`
- Subnet: `255.255.255.0`
- Gateway: bỏ trống (lab nội bộ) hoặc dùng `192.168.118.1` nếu cần
- **DNS:** `192.168.118.10`

---

## 2) Cài AD DS

### GUI (Server Manager)

1. **Server Manager → Add roles and features → AD DS → Install**
2. **Promote** → **Add a new forest** → `lab.local`
3. Đặt **DSRM password** → Install → reboot

### PowerShell

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "lab.local" -DomainNetbiosName "LAB" -InstallDNS
```

---

## 3) Tạo OU & User test

```powershell
# OU
New-ADOrganizationalUnit -Name "HaUI Users" -Path "DC=lab,DC=local"

# User
New-ADUser -Name "Test User" -SamAccountName "testuser" -UserPrincipalName "testuser@lab.local" `
  -Path "OU=HaUI Users,DC=lab,DC=local" -Enabled $true

# Đặt mật khẩu
Set-ADAccountPassword -Identity "testuser" -Reset
```

---

## 4.1) Kiểm tra kết nối & Firewall

- Từ Client:

```cmd
ping 192.168.118.10
```

- Nếu không ping được, trên DC mở ICMP Inbound:

```powershell
netsh advfirewall firewall add rule name="ICMPv4" dir=in action=allow protocol=icmpv4:8,any
```

- Kiểm tra AD:

```powershell
dcdiag /v
Get-ADDomain
```

---

## 4.2) Join Windows Client vào domain

1. **Card Host‑only** của Client: DNS = `192.168.118.10`
2. **System → Rename this PC (advanced) → Change → Domain →** `lab.local`
3. Nhập domain admin → Restart
4. Đăng nhập: `lab\testuser`

---

## ✅ Kết quả cần có

- 01 Domain Controller (`lab.local`)
- 01 OU (“HaUI Users”)
- 01 User test
- 01 Client join domain bằng `lab\testuser`
- Ping Client ↔ DC ok (ICMP mở)
- VM dùng **2 card**: Host‑only (lab) + Bridged (internet), DNS của Host‑only trỏ về DC
