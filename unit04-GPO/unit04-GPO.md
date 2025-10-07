# 📘 Giáo án Unit 4: Group Policy (GPO)

## 🎯 Mục tiêu

- Hiểu rõ khái niệm **Group Policy Object (GPO)**: scope (phạm vi áp dụng), inheritance (kế thừa), GPMC (Group Policy Management Console).
- Thực hành tạo một số GPO cơ bản:
  - **Password Policy** (chính sách mật khẩu).
  - **Map Network Drive** (gắn ổ đĩa mạng cho user).
  - **Block USB Storage** (chặn thiết bị USB).
  - **Deploy Printers** (triển khai máy in cho phòng ban).
- Biết cách **deploy GPO**, kiểm tra và xác minh trên client.

---

## 0) Kiến thức nền tảng

### 🔹 Scope & Inheritance

- **Scope**: GPO áp dụng cho site, domain hoặc OU.
- **Inheritance**: OU con sẽ kế thừa GPO từ OU cha/domain, có thể chặn hoặc enforce.
- **GPMC**: Công cụ quản lý tập trung tất cả GPO.

👉 Mở **Group Policy Management Console (GPMC)** tại **Server Manager → Tools → Group Policy Management**.  
Trong cây quản lý, truy cập:

```
Forest: lab.local
 └── Domains
     └── lab.local
         ├── Default Domain Policy
         ├── Company
         ├── Domain Controllers
         ├── Group Policy Objects
         └── ...
```

### 🔹 Công cụ kiểm tra

- `gpresult /r` hoặc `gpresult /h report.html`.
- Event Viewer → **Applications and Services Logs → Microsoft → Windows → GroupPolicy → Operational**.

### 🔹 Lưu ý quan trọng

- **Computer Configuration**: áp dụng chính sách cho **computer objects** trong OU (cấu hình theo máy).
- **User Configuration**: áp dụng chính sách cho **user objects** trong OU (cấu hình theo người dùng).  
  👉 Vì vậy, khi thiết kế OU và link GPO, cần xác định rõ chính sách muốn áp dụng cho **máy tính** hay **người dùng** để đặt object vào OU phù hợp. (Ví dụ ở dưới Policy [Block USB Storage](#-block-usb-storage-áp-dụng-cho-ou-it)))

---

## 1) Tạo và áp dụng GPO

### 📝 Password Policy

1. Trong **Group Policy Management → Domains → lab.local**, chọn **Default Domain Policy**.
2. Chuột phải → Edit → `Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy`.
3. Thiết lập:
   - Minimum password length = 8.
   - Password must meet complexity requirements = Enabled.

### 📝 Map Network Drive (theo OU phòng ban)

👉 **Chuẩn bị thư mục share trên server trước khi cấu hình GPO**:

1. Trên DC01, tạo thư mục `C:\Shares\FinanceShare`.
2. Chuột phải → **Properties → Sharing → Advanced Sharing** → tick **Share this folder**.
3. Đặt tên share = `FinanceShare`.
4. Vào **Permissions** → thêm nhóm **Domain Users** (hoặc OU Finance) → cấp quyền Read/Change/Full Control theo nhu cầu.
5. Tab **Security** → chỉnh NTFS Permissions tương ứng.
6. Lúc này UNC path sẽ là: `\\DC01\FinanceShare`.

---

1. (Vẫn trong cửa sổ **Group Policy Management**) Mở OU `Company` → `Users`, chọn **Finance**.
2. Chuột phải → **Create a GPO in this domain, and Link it here** → đặt tên **MapDrive-Finance**.
3. Edit GPO: `User Configuration → Preferences → Windows Settings → Drive Maps`.
4. Tạo **Mapped Drive** trỏ đến `\\DC01\FinanceShare`.
5. Trong cửa sổ **New Drive Properties**, cấu hình chi tiết:
   - **Action**: `Update`
   - **Location**: `\\DC01\FinanceShare`
   - **Reconnect**: tick chọn để tự động gắn lại sau khi login
   - **Label as**: `Finance Shared Drive`
   - **Drive Letter**: chọn **Use → F:** (cố định ký tự ổ đĩa cho Finance)
   - **Connect as**: để trống (sử dụng user domain hiện tại)
   - **Hide/Show this drive**: `No change`
   - **Hide/Show all drives**: `No change`
6. Lặp lại với OU **HR**, **IT**, **Management** nếu cần (mỗi phòng ban có thể map đến một thư mục share khác nhau).

### 📝 Block USB Storage (áp dụng cho OU IT)

👉 **Lưu ý!** Nếu áp dụng Policy cho máy tính, thì phải vào mục `Computer Configuration` (chứ không phải `User Configuration` như ở dưới) (nhưng GPO này phải được link với OU Computers)

1. Trong OU **IT**, tạo GPO: **BlockUSB**.
2. Edit GPO: `User Configuration → Policies → Administrative Templates → System → Removable Storage Access`.
3. Chọn **All Removable Storage classes: Deny all access = Enabled**.

### 📝 Deploy Printers (theo phòng ban) (Chưa cần làm, sẽ làm ở Unit 5)

1. Trong OU **HR**, tạo GPO: **DeployPrinter-HR**.
2. Edit GPO: `User Configuration → Preferences → Control Panel Settings → Printers`.
3. Chuột phải → New → Shared Printer → `\\DC01\HP-LaserJet-HR`.
4. Có thể tạo thêm GPO cho **Finance**, **IT**, **Management** nếu mỗi phòng ban có máy in riêng.

---

## 2) Verify trên Client

1. Login vào máy Windows 10 client đã join domain.
2. Chạy `gpupdate /force`.
3. Dùng `gpresult /h gpo-report.html` để xuất báo cáo.
4. Kiểm tra:
   - Mật khẩu phải đủ 8 ký tự.
   - Ổ đĩa mạng xuất hiện trong File Explorer (theo phòng ban).
   - USB không thể truy cập (nếu thuộc OU IT).
   - Máy in xuất hiện trong Devices & Printers (theo phòng ban).

---

## 📦 Deliverables

- **Runbook**:
  - Các bước tạo và triển khai GPO theo OU.
  - Cách verify bằng `gpresult` và Event Viewer.
- **Screenshots**:
  - Trước & sau khi áp dụng GPO (ổ đĩa mạng, USB blocked, password prompt, printer list).
