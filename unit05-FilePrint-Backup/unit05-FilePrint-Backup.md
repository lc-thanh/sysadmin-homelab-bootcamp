# 📘 Giáo án Unit 5: File/Print Server + Backup

## 🎯 Mục tiêu

- Triển khai **File Server** với NTFS & Share permissions đúng chuẩn doanh nghiệp (mô hình **AGDLP**).
- Bật **Access‑Based Enumeration (ABE)**, **Previous Versions (VSS)** cho người dùng tự khôi phục.
- Cấu hình **Print Server** và **deploy printer qua GPO** theo phòng ban.
- Thiết lập **Windows Server Backup** theo lịch, biết **khôi phục** file/folder.
- Xuất bộ **SOP/KB/Checklist** cho doanh nghiệp.

---

## 🧱 Yêu cầu chuẩn bị

- DC01 (Windows Server) đã ở domain `lab.local`, đang nắm vai **File/Print/Backup**.
- OU & Group theo Unit 3 (ví dụ: `grp.finance`, `grp.hr`, `grp.it-support`, `grp.management`).
- Ổ dữ liệu riêng (khuyên dùng **D:** cho Shares/Backup), dung lượng trống ≥ 30 GB.
- Máy Client Windows 10/11 đã join domain, 2 NIC: Host‑only (lab) + Bridged (Internet).

---

## 0) Kiến thức nền tảng (tóm tắt)

- **Share Permission** (áp dụng khi truy cập qua UNC `\\Server\Share`): thường đặt **Everyone = Read** hoặc **Authenticated Users = Change** rồi **khóa chặt bằng NTFS**.
- **NTFS Permission** (áp dụng ở mức file/folder trên volume): là **quyền quyết định** (hiệu lực cuối cùng là **Intersection** giữa Share và NTFS).
- **Nguyên tắc**: _Least privilege_, **không phân quyền trực tiếp cho User**, mà **User ∈ Group → Group có quyền** (**AGDLP**: Accounts → Global Groups → Domain Local Groups → Permissions).
- **Thứ tự ưu tiên**: **Deny** > Allow; quyền **kế thừa** có thể bị chặn.

> **Công thức hiệu lực**: `Effective = Share ∩ NTFS` (giao của 2 tập quyền).

---

## 1) Triển khai File Server (Folders, NTFS, Share)

### 1.1. Tạo cấu trúc thư mục

```
D:\Shares\
├─ Dept\
│  ├─ Finance\
│  ├─ HR\
│  ├─ IT\
│  └─ Management\
└─ Common\
   ├─ Public\
   └─ Transfer\
```

### 1.2. Gợi ý ma trận phân quyền (NTFS)

| Thư mục                     | Nhóm được cấp    | Quyền NTFS (Typical)                                    |
| --------------------------- | ---------------- | ------------------------------------------------------- |
| `D:\Shares\Dept\Finance`    | `grp.finance`    | **Modify** (This folder, subfolders, files)             |
|                             | `grp.management` | **Read**                                                |
|                             | `grp.it-support` | **Modify** (hỗ trợ kỹ thuật)                            |
| `D:\Shares\Dept\HR`         | `grp.hr`         | **Modify**                                              |
| `D:\Shares\Dept\IT`         | `grp.it-support` | **Modify**                                              |
| `D:\Shares\Dept\Management` | `grp.management` | **Modify**                                              |
| `D:\Shares\Common\Public`   | `Domain Users`   | **Read** hoặc **Modify** (nếu chia sẻ tự do)            |
| `D:\Shares\Common\Transfer` | `Domain Users`   | **Read + Create** (Không được Write file của User khác) |

Khung chuẩn thêm:

- **SYSTEM**, **Administrators** = **Full Control**.
- **CREATOR OWNER** = **Full Control** trên **Subfolders and files only** (giúp chủ sở hữu kiểm soát file của họ).

### 1.3. Tạo Share + bật ABE

- Với mỗi thư mục Dept, share ở mức tên ngắn, ví dụ: `\\DC01\Dept-Finance`, `\\DC01\Dept-HR`, …
- **Share Permissions**: để **Authenticated Users = Change** (hoặc Everyone = Read/Change tùy chính sách), **khóa bằng NTFS** như ma trận.

**LƯU Ý!** Ở đây **Quyền thật = Share ∩ NTFS** (Giao giữa 2 tập quyền, có nghĩa là khi cả 2 đều có quyền đó thì User mới có quyền). Vì vậy, để dễ cho Lab, thì ta share Full Control cho Everyone luôn.

- Bật **Access-Based Enumeration (ABE)** để **user chỉ thấy thư mục họ có quyền**:
  - Mở **Share Properties → Settings** hoặc trong **Server Manager → File and Storage Services → Shares → Properties → Settings → Enable access-based enumeration**.

### 1.4. Home Drive (H:) tự tạo theo user (khuyến nghị)

1. Tạo `D:\Shares\Homes` → NTFS:
   - `Users` = **List folder / Read** (**This folder only**)
   - `CREATOR OWNER` = **Full Control (Subfolders and files only)**
   - `Administrators`, `SYSTEM` = **Full Control**
2. Tạo Share: `\\DC01\Homes` (Enable ABE).
3. Cách gán:
   - **Cách A (ADUC)**: Mở **User → Profile → Home folder → Connect H: to `\\DC01\Homes\%username%`**.
   - **Cách B (GPP)**: `User Configuration → Preferences → Windows Settings → Drive Maps` (Nhưng cách này Server không tự tạo folder cho User).

**Lưu ý**: User đăng nhập lần đầu có thể sẽ không thấy Mapped Drive (H:) ngay, mà phải đợi khoảng mấy giây rồi Sign out -> đăng nhập lại mới có.

> Lợi ích: User có ổ H: cá nhân, tách biệt, tự động tạo đúng quyền.

---

## 2) Shadow Copies (VSS) – “Previous Versions”

### 2.1. Bật trên volume dữ liệu (D:)

1. **This PC → chuột phải D: → Properties → Shadow Copies** → **Enable**.
2. **Settings**: dung lượng (ví dụ 10–20% dung lượng ổ), lịch chụp (ví dụ **09:00, 12:00, 15:00, 18:00**).
3. Kiểm thử snapshot: **Create Now**.

### 2.2. SOP: Người dùng tự khôi phục

1. Trên Client, mở thư mục mạng (ví dụ `\\DC01\Dept-Finance`).
2. Chuột phải vào **folder/file → Properties → Previous Versions**.
3. Chọn bản **Date/Time** phù hợp → **Open** để xem → **Restore** hoặc **Copy** đến vị trí mong muốn.

> Ghi chú: VSS bảo vệ lỗi “xóa/ghi đè nhầm”. Không thay thế **backup** định kỳ (mất cả volume thì VSS cũng mất).

---

## 3) Windows Server Backup (WSB)

### 3.1. Cài đặt & thiết kế

- Cài **Windows Server Backup**: _Server Manager → Add Roles and Features → Features → Windows Server Backup_.
- **Đích backup**: **đĩa khác** hoặc ổ gắn ngoài; **không** nên đặt cùng volume dữ liệu.
- Chiến lược khuyến nghị (lab):
  - **Daily** lúc **23:00**: Backup **Custom** các thư mục `D:\Shares` + **System State**.
  - Giữ lại ≥ 7 phiên bản gần nhất (tùy dung lượng).

### 3.2. Lập lịch (GUI)

1. Mở **Windows Server Backup → Backup Schedule**.
2. Chọn **Custom** → Add Items: `D:\Shares` + **System State**.
3. Thêm **exclusions** nếu có (ví dụ thư mục tạm).
4. Chọn **Once a day at 23:00** → đích lưu trữ (khuyến nghị: **dedicated backup disk**).

### 3.3. Dùng lệnh (wbadmin) (Để test)

```powershell
# Backup dữ liệu (file/folder/volume) thủ công (ad-hoc)
wbadmin start backup -backupTarget:E: -include:D:\Shares -quiet

# Khởi động System State Backup
wbadmin start systemstatebackup -backupTarget:E: -quiet
```

> Trong lab có thể dùng **Task Scheduler** để chạy hai job: dữ liệu và System State.

### 3.4. Khôi phục (Restore)

- **File/Folder**: **Windows Server Backup → Recover → This server → Files and Folders** → duyệt thời điểm → restore về **Original** hoặc **Alternate location**.
- **System State**: dùng **Recover** với **System State** (cần downtime và theo sát wizard).

---

## 4) Print Server & Deploy Printer qua GPO

### 4.1. Cài vai trò & thêm máy in

1. **Server Manager → Add Roles and Features → Print and Document Services → Print Server**.
2. Mở **Print Management**.
3. **Nếu máy in thật qua LAN**: `Print Management → Printers → Add Printer → Add a new TCP/IP or Web Services Printer → Hostname/IP` (ví dụ `192.168.1.120`) → chọn **driver** tương ứng (x64).
4. **Nếu chia sẻ máy in cắm USB vào DC01**: Add Local Printer → cổng **USB** tương ứng.
5. Đặt tên & **Share**: ví dụ **HP‑LaserJet‑HR**, UNC: `\\DC01\HP-LaserJet-HR`.
6. (Tùy chọn) **List in the directory** để client có thể tìm qua AD.

### 4.2. Deploy qua GPO theo phòng ban

1. Tạo GPO **DeployPrinter-HR** link với `OU=HR` (User scope).
2. `User Configuration → Preferences → Control Panel Settings → Printers` → **New → Shared Printer** → `\\DC01\HP-LaserJet-HR` → **Action = Update**.
3. Trên Client (user HR): `gpupdate /force` → kiểm tra **Devices & Printers** / **Printers & Scanners**.

### 4.3. Point and Print (an toàn)

- `Computer Configuration → Policies → Administrative Templates → Printers → Point and Print Restrictions`: **Enabled**
  - **Users can only point and print to these servers**: `DC01.lab.local`
  - **When installing drivers for a new connection**: **Show warning and elevation prompt** (an toàn)
- Tránh tắt cảnh báo hoàn toàn trong môi trường thật.

---

## 5) Kiểm thử & Chứng minh

### 5.1. Quyền truy cập

- Đăng nhập 2 user thuộc **Finance** và **HR** trên 2 máy Client.
- Vào `\\DC01\Dept-Finance` và `\\DC01\Dept-HR` → **User Finance** phải có **Modify** ở Finance và **Read/No access** ở HR (theo ma trận).
- Dùng **Effective Access** tab (Properties → Security → Advanced → Effective Access) để kiểm tra.

### 5.2. VSS Previous Versions

- Tạo file `report.docx`, sửa nội dung → lưu.
- Khôi phục lại phiên bản trước qua **Previous Versions** như SOP 2.2.

### 5.3. Printer

- User HR đăng nhập → máy in **HP-LaserJet-HR** tự xuất hiện.
- In test page; nếu lỗi driver, kiểm tra **Point and Print** & driver x64.

---

## 6) Troubleshooting nhanh

- **Không thấy share**: kiểm tra **Firewall (File and Printer Sharing)**, ABE, DNS (`\\DC01\share`).
- **Sai quyền**: nhớ rằng **Deny** chặn mọi thứ; soát **kế thừa** và **nhóm chồng chéo**.
- **VSS không có bản**: kiểm tra dung lượng, lịch snapshot.
- **GPO không áp**: `gpresult /h report.html`, Event Viewer → GroupPolicy/Operational.
- **Printer lỗi driver**: cập nhật driver x64, chỉnh **Point and Print** theo 4.3.

---

## 📦 Deliverables

1. **SOP: Restore file bị xóa/ghi đè (Previous Versions + Backup)**

   - B1: Thử **Previous Versions** trên client (folder/file → Properties → Previous Versions → Restore/Copy).
   - B2: Nếu không có bản phù hợp → **IT khôi phục từ Windows Server Backup** (Recover → Files and Folders).
   - B3: Ghi log: _User, file, thời điểm, nguồn restore (VSS/WSB)._

2. **KB cho end‑user: Kết nối & in thử máy in phòng ban**

   - Vào **Settings → Bluetooth & devices → Printers & scanners → Add device** (hoặc **Control Panel → Devices and Printers → Add a printer**).
   - Chọn **Select a shared printer by name** → nhập `\\DC01\HP-LaserJet-HR` → **Next**.
   - In **Test page** và báo IT nếu lỗi.

3. **Checklist hoàn thành Unit 5**
   - [ ] Tạo Shares + ABE + ma trận NTFS.
   - [ ] Tạo **Homes (H:)** + tự tạo thư mục theo user.
   - [ ] Bật **VSS** trên D:, khung lịch & dung lượng hợp lý.
   - [ ] Cài **Windows Server Backup** + lịch hằng ngày 23:00 (bao gồm System State).
   - [ ] Cài **Print Server**, thêm máy in, deploy qua **GPO** theo OU.
   - [ ] Thực hiện **bài test** quyền, VSS, in test page.
   - [ ] Chụp **screenshots**: Shares, NTFS Advanced, ABE, VSS snapshots, WSB schedule, Printer in AD/GPO, `gpresult`.

---

## 🔧 Phụ lục: Lệnh nhanh (tham khảo)

```powershell
# Tạo share và bật ABE (Windows Server 2019+)
New-SmbShare -Name "Dept-Finance" -Path "D:\Shares\Dept\Finance" -FullAccess "Administrators","SYSTEM" -ChangeAccess "LAB\grp.finance","LAB\grp.it-support" -FolderEnumerationMode AccessBased

# Phân quyền NTFS (ví dụ Finance)
icacls "D:\Shares\Dept\Finance" /inheritance:d
icacls "D:\Shares\Dept\Finance" /grant "LAB\grp.finance:(OI)(CI)M" "LAB\grp.it-support:(OI)(CI)M" "Administrators:(F)" "SYSTEM:(F)"
icacls "D:\Shares\Dept\Finance" /grant "CREATOR OWNER:(OI)(CI)(IO)F"

# Bật Shadow Copies tức thời
vssadmin list volumes
# (Bật/điều chỉnh qua GUI khuyến nghị; dòng lệnh VSS phức tạp hơn)

# Cài Windows Server Backup
Install-WindowsFeature Windows-Server-Backup

# Backup ad-hoc dữ liệu Shares sang ổ E:
wbadmin start backup -backupTarget:E: -include:D:\Shares -quiet
```

---

## ✅ Kết quả kỳ vọng

- Người dùng thấy đúng **thư mục phòng ban** (nhờ ABE), có/không có quyền theo ma trận.
- Có **Previous Versions** để tự khôi phục.
- **Backup** chạy theo lịch và có recovery thử nghiệm.
- Máy in phòng ban **tự xuất hiện** nhờ GPO và in test page thành công.
