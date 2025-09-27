# 📘 Giáo án Unit 3: Join Domain & Account Standards

## 🎯 Mục tiêu
- Thêm máy Windows 10 vào domain `lab.local`
- Chuẩn hóa quy tắc đặt tên (user, group, OU structure)
- Xây dựng **checklist** để onboard Windows 10 vào domain
- Viết **tài liệu chuẩn đặt tên** (Naming Convention Document)

---

## 1) Join Windows 10 vào domain

### 🔹 Kiểm tra trước khi join
- Đảm bảo Windows 10 có **card mạng Host-only (VMnet1)** kết nối đến DC01
- DNS Server trên máy client phải trỏ về **IP của DC01 (192.168.118.10)**
- Có tài khoản Domain Admin để join (ví dụ: `lab\Administrator`)

### 🔹 Các bước join
1. Trên Windows 10 → **This PC → Properties → Advanced system settings**
2. Vào **Computer Name → Change…**
3. Chọn **Domain**, nhập: `lab.local`
4. Nhập tài khoản Domain Admin (ví dụ: `Administrator`)
5. Restart máy để áp dụng

### 🔹 Kiểm tra sau khi join
- Đăng nhập bằng tài khoản domain: `lab\user01`
- Kiểm tra trong **Active Directory Users and Computers (ADUC)** → máy tính xuất hiện trong OU tương ứng

---

## 2) Account & Naming Standards

### 🔹 Nguyên tắc đặt tên user
- Đơn giản, dễ nhớ, không dấu, thống nhất toàn hệ thống
- Ví dụ:
  - Sinh viên: `s<studentID>` → `s2023001`
  - Giảng viên: `g<lastname>` → `gtran`
  - Nhân viên kỹ thuật: `it.<firstname>` → `it.thanh`

### 🔹 Nguyên tắc đặt tên group
- Phản ánh chức năng/ban/role
- Ví dụ:
  - `grp.students`
  - `grp.teachers`
  - `grp.it-support`

### 🔹 Nguyên tắc OU structure
- OU gốc: `FIT`
  - `OU=Users`
    - `OU=Students`
    - `OU=Teachers`
    - `OU=IT`
  - `OU=Computers`
    - `OU=LabPCs`
    - `OU=StaffPCs`

---

## 3) Deliverables

### 📋 Checklist: Onboard Windows 10 into domain
- [ ] Máy client có DNS trỏ về DC01
- [ ] Ping được `dc01.lab.local`
- [ ] Join domain thành công
- [ ] Máy tính hiển thị trong OU `Computers`
- [ ] User domain đăng nhập thành công trên client

### 📄 Naming Convention Document
- **Users:** theo role + định danh (ví dụ: `s2023001`, `gtran`)
- **Groups:** tiền tố `grp.` + chức năng (`grp.it-support`)
- **OU structure:** phân tách theo user/computer/ban rõ ràng
