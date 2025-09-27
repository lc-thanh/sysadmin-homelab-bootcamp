# 📘 Giáo án Unit 3: Join Domain & Account Standards

## 🎯 Mục tiêu
- Chuẩn hóa quy tắc đặt tên (user, group, OU structure)
- Xây dựng **checklist** để onboard Windows 10 vào domain (kiểm tra sau khi join)
- Viết **tài liệu chuẩn đặt tên** (Naming Convention Document)
- Quản lý OU và Group theo mô hình doanh nghiệp

---

## 1) Checklist: Onboard Windows 10 vào domain

### 🔹 Kiểm tra sau khi join
*(Phần join domain chi tiết đã thực hiện ở Unit 1, ở đây chỉ cần xác nhận lại)*  

- [ ] Máy client có DNS trỏ về DC01  
- [ ] Ping được `dc01.lab.local`  
- [ ] Máy tính hiển thị trong OU `Computers` hoặc OU phù hợp  
- [ ] User domain đăng nhập thành công trên client (`lab\\username`)  

---

## 2) Account & Naming Standards

### 🔹 Nguyên tắc đặt tên user
- Đơn giản, dễ nhớ, không dấu, thống nhất toàn hệ thống
- Ví dụ:
  - Nhân viên: `emp.<firstname><lastname>` → `emp.lcthanh`
  - Quản lý: `mgr.<lastname>` → `mgr.nguyen`
  - IT Support: `it.<firstname>` → `it.thanh`

### 🔹 Nguyên tắc đặt tên group
- Phản ánh chức năng/phòng ban
- Ví dụ:
  - `grp.finance`
  - `grp.hr`
  - `grp.it-support`
  - `grp.management`

### 🔹 Nguyên tắc OU structure
- OU gốc: `Company`
  - `OU=Users`
    - `OU=Finance`
    - `OU=HR`
    - `OU=IT`
    - `OU=Management`
  - `OU=Computers`
    - `OU=Workstations`
    - `OU=Servers`

---

## 3) Quản lý máy tính trong OU (Enterprise)

Mặc định khi join một máy tính vào domain, nó sẽ rơi vào container **Computers** (không phải là OU do bạn tạo). Container này không cho phép bạn gán Group Policy trực tiếp. Vì vậy trong doanh nghiệp cần **di chuyển máy tính sang OU tùy chỉnh** để quản lý chính sách tập trung.

### 🔹 Cách 1: Dùng Active Directory Users and Computers (ADUC) – thủ công
1. Mở **ADUC** trên Domain Controller.  
2. Mặc định máy tính mới join sẽ nằm trong **Computers** container.  
3. Chuột phải vào máy tính đó → **Move…**.  
4. Chọn OU đích (ví dụ: `Workstations`) → **OK**.  

### 🔹 Cách 2: Dùng PowerShell
```powershell
# Di chuyển máy tính DESKTOP01 từ container mặc định sang OU=Workstations
Move-ADObject -Identity "CN=DESKTOP01,CN=Computers,DC=lab,DC=local" `
-TargetPath "OU=Workstations,OU=Computers,DC=lab,DC=local"
```

### 🔹 Cách 3: Thiết lập OU mặc định cho máy tính mới join
Nếu muốn tất cả máy mới join domain **tự động** rơi vào OU `Workstations`, chạy lệnh:

```cmd
redircmp "OU=Workstations,OU=Computers,DC=lab,DC=local"
```

---

## 4) Quan hệ OU – Group – User

Trong môi trường doanh nghiệp:  
- **OU**: Là container tổ chức → dùng để phân loại theo phòng ban, gán Group Policy.  
- **Group**: Là tập hợp user để phân quyền tài nguyên (folder, printer, app…).  
- **User**: Tài khoản cá nhân, được thêm vào Group.  

### 🔹 Sơ đồ minh họa (Markdown Tree)
```
Company
│
├── Users
│   ├── Finance (OU)
│   │   ├── grp.finance (Group)
│   │   │   ├── emp.nguyen (User)
│   │   │   └── emp.tran (User)
│   │
│   ├── HR (OU)
│   │   ├── grp.hr (Group)
│   │   │   ├── emp.le (User)
│   │   │   └── emp.bui (User)
│   │
│   ├── IT (OU)
│   │   ├── grp.it-support (Group)
│   │   │   ├── it.thanh (User)
│   │   │   └── it.pham (User)
│   │
│   └── Management (OU)
│       ├── grp.management (Group)
│       │   ├── mgr.nguyen (User)
│       │   └── mgr.le (User)
│
└── Computers
    ├── Workstations (OU)
    └── Servers (OU)
```

---

## 5) Deliverables

### 📋 Checklist: Onboard Windows 10 into domain
- [ ] Client cấu hình DNS đúng
- [ ] Ping được domain controller
- [ ] Máy tính xuất hiện trong OU mong muốn
- [ ] User domain login thành công

### 📄 Naming Convention Document
- **Users:** theo role + định danh (ví dụ: `emp.lcthanh`, `mgr.nguyen`)  
- **Groups:** tiền tố `grp.` + phòng ban (`grp.finance`, `grp.it-support`)  
- **OU structure:** phân tách rõ Users/Computers theo phòng ban/doanh nghiệp  
