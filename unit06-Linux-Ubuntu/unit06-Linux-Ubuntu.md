# 📘 Giáo án Unit 6: Linux Fundamentals (Ubuntu)

## 🎯 Mục tiêu

- Làm quen với môi trường **Ubuntu Server 20.04 LTS** (dòng lệnh).
- Quản lý **user/group**, quyền truy cập file.
- Cấu hình **mạng (static IP, DNS)** và truy cập từ xa qua **SSH**.
- Hiểu cơ bản về **dịch vụ (services), nhật ký (logs)** và **tường lửa (ufw)**.
- Tổng hợp thành **Linux Quick Reference Sheet** (tài liệu tra cứu nhanh).

---

## 🧱 Yêu cầu chuẩn bị

- VM: **Ubuntu Server 20.04 LTS**, RAM ≥ 2 GB, 2 NIC (Host-only + Bridged).
- Nếu SSH chưa có, cài đặt theo hướng dẫn tại mục 4.1.
- Có thể ping được Domain Controller `DC01.lab.local` (nếu cần dùng DNS lab).
- User Windows có thể SSH vào Ubuntu qua IP Host-only.

---

## 0) Kiến thức nền tảng

### 🔹 Linux Directory Structure

```
/
├── bin      → lệnh cơ bản (ls, cp, mv…)
├── etc      → file cấu hình hệ thống
├── home     → thư mục người dùng
├── var/log  → chứa log hệ thống
├── usr      → ứng dụng, thư viện
└── root     → thư mục chính của tài khoản root
```

### 🔹 Quyền truy cập cơ bản (Permissions)

| Ký hiệu | Quyền   | Giá trị    | Mô tả                   |
| ------- | ------- | ---------- | ----------------------- |
| r       | read    | 4          | đọc nội dung            |
| w       | write   | 2          | ghi/xóa sửa             |
| x       | execute | 1          | chạy file / vào thư mục |
| rwx     | 7       | full quyền |

Ví dụ:  
`chmod 755 file.sh` → **owner = rwx (7)**, **group = r-x (5)**, **others = r-x (5)**.

---

## 1) Cấu hình mạng & DNS

### 1.1. Xem thông tin mạng

```bash
ip a
ip route
cat /etc/resolv.conf
```

### 1.2. Đặt IP tĩnh (Netplan)

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Ví dụ:

```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: false
      addresses: [192.168.118.20/24]
      routes:
        - to: default
          via: 192.168.118.10
      nameservers:
        addresses: [192.168.118.10, 8.8.8.8]
```

Lưu lại rồi:

```bash
sudo netplan apply
```

> 💡 **Vì sao dùng IP tĩnh?**
>
> - Server cần địa chỉ cố định để SSH, monitoring, DNS mapping.
> - DHCP có thể đổi IP, gây lỗi kết nối hoặc mất cấu hình Zabbix/GLPI.
> - Trong doanh nghiệp, server (AD, File, Linux, DB) **luôn dùng IP tĩnh**, chỉ client mới DHCP.

### 1.3. Kiểm tra kết nối

```bash
ping dc01.lab.local
ping 8.8.8.8
```

---

## 2) Quản lý User và Group

### 2.1. Thêm user và đặt mật khẩu

```bash
sudo adduser thanh
sudo passwd thanh
```

### 2.2. Tạo group và thêm user

```bash
sudo groupadd it-support
sudo usermod -aG it-support thanh
```

### 2.3. Kiểm tra thành viên nhóm

```bash
groups thanh
id thanh
```

### 2.4. Xóa user hoặc group

```bash
sudo deluser username
sudo delgroup groupname
```

---

## 3) Quản lý File và Quyền

### 3.1. Kiểm tra quyền

```bash
ls -l
```

### 3.2. Đổi quyền và chủ sở hữu

```bash
sudo chmod 750 project/
sudo chown thanh:it-support project/
```

### 3.3. Tạo thư mục dùng chung

```bash
sudo mkdir /srv/shared
sudo chown root:it-support /srv/shared
sudo chmod 770 /srv/shared
```

---

## 4) Dịch vụ, Logs và SSH

### 4.1. Cài đặt SSH Server (nếu chưa có)

Kiểm tra SSH đã cài chưa:

```bash
sudo systemctl status ssh
```

Nếu báo _Unit ssh.service could not be found_, cài bằng:

```bash
sudo apt update
sudo apt install openssh-server -y
```

Sau đó bật dịch vụ:

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

Nếu bật tường lửa `ufw`, cho phép SSH:

```bash
sudo ufw allow ssh
sudo ufw status
```

SSH mặc định lắng nghe ở port 22.

### 4.2. Quản lý dịch vụ (systemd)

```bash
sudo systemctl restart ssh
sudo systemctl enable ssh
```

### 4.3. Truy cập SSH từ Windows

```bash
ssh thanh@192.168.118.20
```

_(Hoặc dùng PuTTY / Windows Terminal)_

### 4.4. Xem nhật ký hệ thống

```bash
sudo journalctl -xe
sudo tail -f /var/log/syslog
sudo cat /var/log/auth.log
```

---

## 5) Firewall cơ bản (UFW)

### 5.1. Bật UFW và cho phép SSH

```bash
sudo ufw enable
sudo ufw allow ssh
sudo ufw status verbose
```

### 5.2. Thêm rule khác

```bash
sudo ufw allow 80/tcp
sudo ufw deny 23/tcp
```

---

## 6) Một số lệnh quản trị nhanh

| Nhóm thao tác       | Lệnh cơ bản                                    | Ghi chú                    |
| ------------------- | ---------------------------------------------- | -------------------------- |
| Cập nhật hệ thống   | `sudo apt update && sudo apt upgrade -y`       | Cập nhật gói               |
| Quản lý tiến trình  | `ps aux`, `top`, `kill <PID>`                  | Kiểm tra & dừng tiến trình |
| Kiểm tra dung lượng | `df -h`, `du -sh *`                            | Theo dõi ổ đĩa             |
| Tìm kiếm            | `find / -name file.txt`, `grep "pattern" file` | Tìm file / nội dung        |
| Giám sát hệ thống   | `uptime`, `free -h`, `hostnamectl`             | Trạng thái cơ bản          |
| Đổi hostname        | `sudo hostnamectl set-hostname ubuntu01`       | Áp dụng ngay               |

---

## 7) Kiểm thử & Thực hành

- [ ] Cài và kiểm tra SSH Server.
- [ ] SSH vào Ubuntu từ Windows.
- [ ] Tạo 2 user (`alice`, `bob`) và 1 group (`devteam`).
- [ ] Chia sẻ thư mục `/srv/shared` cho `devteam` (chmod 770).
- [ ] Cấu hình IP tĩnh và DNS về DC01 (192.168.118.10).
- [ ] Dùng `ping`, `ssh`, `ufw status` để kiểm tra.
- [ ] Xem log `auth.log` khi SSH thành công.

---

## 📦 Deliverables

1. **Linux Quick Reference Sheet (cheat-sheet)**
   - Lệnh quản trị, network, user/group, permissions, logs, firewall, SSH.
2. **Screenshot Proofs:**
   - `ip a`, `netplan`, `ufw status`, `ls -l`, `groups`, `ssh login`.
3. **Checklist hoàn thành Unit 6:**
   - [ ] Cấu hình IP/DNS tĩnh
   - [ ] SSH hoạt động
   - [ ] Tạo user/group + phân quyền
   - [ ] Bật UFW + rule SSH
   - [ ] Tổng hợp tài liệu lệnh Linux cơ bản

---

## ✅ Kết quả kỳ vọng

- Ubuntu Server kết nối mạng ổn định, truy cập được qua SSH.
- Có user/group và phân quyền thư mục đúng chuẩn.
- Firewall hoạt động, log ghi nhận truy cập.
- Có file **Quick Reference Sheet** hoàn chỉnh.
