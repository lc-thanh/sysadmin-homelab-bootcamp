# 📘 Giáo án Unit 7: System Monitoring (Zabbix)

## 🎯 Mục tiêu

- Cài đặt và cấu hình hệ thống **giám sát hạ tầng** với **Zabbix Server** trên Ubuntu.
- Cài đặt **Zabbix Agent** trên cả **Windows Server (DC01)** và **Ubuntu Client**.
- Theo dõi tài nguyên hệ thống (CPU, RAM, Disk, Network) và tạo **Dashboard giám sát**.
- Thiết lập **cảnh báo (Trigger)** và **Email Alert** cơ bản.
- Xuất báo cáo (screenshot) các host đã được giám sát.

---

## 🧱 Yêu cầu chuẩn bị

- Ubuntu Server 20.04/22.04 (RAM ≥ 2 GB, NIC Host-only + Bridged).
- Windows Server (DC01) & Windows 10 đã trong domain `lab.local`.
- Đảm bảo các máy có thể **ping lẫn nhau** (DC01 ↔ Ubuntu ↔ Client).
- Ubuntu đã cài SSH và có quyền sudo (theo Unit 6).
- Truy cập internet (Bridged NIC) để tải gói Zabbix.

---

## 0) Kiến thức nền tảng

### 🔹 Zabbix là gì?

**Zabbix** là hệ thống **giám sát mã nguồn mở** cho Server, VM, thiết bị mạng.
Nó thu thập và hiển thị dữ liệu về hiệu suất, trạng thái dịch vụ, dung lượng ổ đĩa, CPU, RAM, v.v.

> **Hiểu ngắn gọn:** Zabbix giúp người quản trị hệ thống phát hiện sự cố trước khi người dùng kịp phàn nàn.

### 🔹 Thành phần

| Thành phần         | Vai trò                             | Cài ở đâu                |
| ------------------ | ----------------------------------- | ------------------------ |
| **Zabbix Server**  | Trung tâm thu thập và xử lý dữ liệu | Ubuntu Server            |
| **Zabbix Agent**   | Gửi dữ liệu hệ thống về Server      | Cài trên Windows & Linux |
| **Frontend (Web)** | Giao diện quản trị                  | Truy cập qua trình duyệt |
| **Database**       | Lưu dữ liệu lịch sử                 | MariaDB/MySQL            |

### 🔹 Lợi ích trong doanh nghiệp

- Giám sát tập trung toàn bộ server, thiết bị mạng, dịch vụ.
- Phát hiện sớm khi CPU/RAM quá tải, dịch vụ ngừng hoạt động.
- Gửi cảnh báo Email/Telegram/SMS khi có sự cố.
- Thống kê, biểu đồ, giúp lập kế hoạch nâng cấp hạ tầng.
- Hoàn toàn miễn phí, không giới hạn số lượng thiết bị.

### 🔹 Ví dụ thực tế

Doanh nghiệp có 1 DC, 1 File Server, 1 Mail Server, 3 Switch, 1 Router.  
→ Zabbix Server theo dõi tất cả thiết bị: CPU, RAM, Disk, uptime, network traffic.  
Khi switch mất điện hoặc ổ D: của File Server đầy, Zabbix gửi email cho IT.

---

## 1) Cài đặt Zabbix Server trên Ubuntu

### 1.1. Cài đặt Database (MariaDB)

```bash
sudo apt update
sudo apt install mariadb-server -y
sudo mysql_secure_installation
```

Tạo database:

```bash
sudo mysql -u root -p
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'Zabbix@123';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 1.2. Cài đặt Zabbix

```bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu20.04_all.deb
sudo dpkg -i zabbix-release_7.0-1+ubuntu20.04_all.deb
sudo apt update
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

Import schema:

```bash
sudo zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

### 1.3. Cấu hình kết nối DB

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Chỉnh:

```
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=Zabbix@123
```

### 1.4. PHP Timezone

```bash
sudo nano /etc/zabbix/apache.conf
```

Thêm:

```
php_value date.timezone Asia/Ho_Chi_Minh
```

Khởi động dịch vụ:

```bash
sudo systemctl restart apache2 zabbix-server zabbix-agent
sudo systemctl enable apache2 zabbix-server zabbix-agent
```

Truy cập: `http://<UBUNTU_IP>/zabbix`  
→ Login mặc định: **Admin / zabbix**

---

## 2) Cài Zabbix Agent trên Windows Server & Client

### 2.1. Tải & cài

Tải từ [https://www.zabbix.com/download_agents](https://www.zabbix.com/download_agents)

Cài nhanh (PowerShell):

```powershell
msiexec /i zabbix_agent-7.0.0-windows-amd64-openssl.msi SERVER=192.168.118.20 HOSTNAME=DC01.lab.local /qn
```

Firewall rule:

```powershell
netsh advfirewall firewall add rule name="Zabbix Agent" dir=in action=allow protocol=TCP localport=10050
```

---

## 3) Cài Zabbix Agent trên Ubuntu (Nếu là Zabbix Server rồi thì không cần nữa)

```bash
sudo apt install zabbix-agent -y
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Sửa:

```
Server=192.168.118.20
ServerActive=192.168.118.20
Hostname=ubuntu01.lab.local
```

Khởi động:

```bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
sudo ufw allow 10050/tcp
```

---

## 4) Thêm Hosts vào Dashboard

- **Configuration → Hosts → Create Host**
- Hostname: `DC01.lab.local`
- Groups: `Windows Servers`
- Interface: `Agent → IP = 192.168.118.10`
- Template: `Template OS Windows by Zabbix agent`

Thêm tương tự cho Ubuntu → `Template OS Linux by Zabbix agent`.

---

## 5) Dashboard & Triggers

### 5.1. Dashboard

- Monitoring → Dashboard → Create Dashboard
- Thêm widget: CPU Load, Memory, Disk, Host Status.

### 5.2. Trigger cảnh báo CPU

```
{Template OS Linux by Zabbix agent:system.cpu.util.last(5m)}>85
```

---

## 6) Giám sát Switch/Router (SNMP)

### 6.1. Kích hoạt SNMP trên thiết bị

Ví dụ Cisco:

```bash
configure terminal
snmp-server community public RO
snmp-server location "Server Room"
snmp-server contact "it@company.local"
exit
```

### 6.2. Thêm vào Zabbix

- Configuration → Hosts → Create host
- Interface: SNMP, IP = 192.168.118.1
- Template: `Template Net Cisco SNMP` hoặc `Template Net Generic SNMPv2`

### 6.3. Theo dõi được

| Thông số    | Ví dụ               |
| ----------- | ------------------- |
| Port Status | Up/Down từng cổng   |
| Traffic     | In/Out Mbps         |
| CPU/Memory  | % sử dụng           |
| Temperature | Nhiệt độ thiết bị   |
| Uptime      | Thời gian hoạt động |

---

## 📦 Deliverables

1. Dashboard có ≥ 2 host (Windows, Ubuntu, Network device).
2. Trigger cảnh báo CPU hoạt động.
3. Screenshot minh chứng (Dashboard, Graph, Trigger).
4. Checklist hoàn thành Unit 7.

---

## ✅ Kết quả kỳ vọng

- Zabbix Server hoạt động ổn định.
- Tất cả host hiển thị dữ liệu realtime.
- Có cảnh báo và biểu đồ giám sát hệ thống & mạng.
