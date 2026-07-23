# About Registry
## Registry Structure

| Folder                    | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| HKEY_CURRENT_USER (HKCU)  | - Chứa thông tin cấu hình gốc cho người dùng hiện tại đang đăng nhập. Các thư mục , màu màn hình, Control Panel được lưu trữ ở đây. Thông tin này được liên kết với user proflie.<br>    <br>- Ví dụ user `Alice` mở Word, đổi theme, mở folder, search trong Explorer… các dấu vết thường nằm ở HKCU.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| HKEY_USERS (HKU)          | Chứa tất cả hồ sơ người dùng đang hoạt động trên máy. HKCU là một khóa con của HKU.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| HKEY_LOCAL_MACHINE (HKLM) | Chứa thông tin cấu hình dành riêng cho máy tính( cho bất kì User nào).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| HKEY_CLASSES_ROOT (HKCR)  | - Là một khóa con của HKLM\Software . Thông tin được lưu trữ ở đây đảm bảo chương trình đúng sẽ mở khi bạn mở trên Windows Explorer.<br>    <br>- Bắt đầu từ Windows 2000, thông tin này được lưu trứ trữ dưới 2 key là HKLM và HKCU.<br>    <br>    - Với HKLM chứa thiết lập mặt định có thể áp dụng cho tất cả người dùng trên máy tính cục bộ<br>        <br>    - Với HKCU nó ghi đè các cài đặt mặc định và chỉ áp dụng cho người dùng tương tác.<br>        <br>- HKCR chứa giao diện xem registry hợp nhất cho hai nguồn này .<br>    <br>- Thay đổi cấu hình:<br>    <br>    - Để thay đổi cho người dùng tương tác , phải sửa trên HKCU<br>        <br>    - Để thay đổi cho cấu hình mặc định phải sửa trên HKLM<br>        <br>    - Nếu chỉnh sửa trên HKCR thì cấu hình sẽ lưu trên HKLM, tuy nhiên nếu khóa đã tồn tại thì nó sẽ lưu trên HKCU |
| HKEY_CURRENT_CONFIG       | Chứa thông tin cấu hình phần cứng được sử dụng bởi máy tính cục bộ khi khởi động.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
## Registry hive
![[Pasted image 20260720144253.png]]
- Là một nhóm logic bao gồm các key( khóa) , subkey(khóa phụ), value(giá trị) trong Windows registry , được lưu trữ dưới dạng một tập tin vật lý riêng biệt trên ổ cứng
## Core
| Hive         | File offline                          | Registry path khi Windows chạy | Lưu cái gì?                                                                   | Ý nghĩa forensic chính                                                           |
| ------------ | ------------------------------------- | ------------------------------ | ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| **SYSTEM**   | `C:\Windows\System32\Config\SYSTEM`   | `HKLM\SYSTEM`                  | Cấu hình phần cứng, service, driver, USB, ổ đĩa, volume, timezone, ControlSet | Biết máy từng gắn thiết bị gì, service/driver nào tồn tại, timezone, boot config |
| **SOFTWARE** | `C:\Windows\System32\Config\SOFTWARE` | `HKLM\SOFTWARE`                | Phần mềm đã cài, Windows version, app config, uninstall info, một số autorun  | Biết máy cài app gì, app nào từng bị gỡ, cấu hình Windows/app                    |
| **SAM**      | `C:\Windows\System32\Config\SAM`      | `HKLM\SAM`                     | Local user accounts, local groups, RID, account flags                         | Xác định user local, nhóm Administrators, tài khoản lạ, RID/SID                  |
| **SECURITY** | `C:\Windows\System32\Config\SECURITY` | `HKLM\SECURITY`                | Security policy, password policy, audit policy, LSA-related data              | Phân tích chính sách bảo mật, chính sách mật khẩu, policy hệ thống               |
| **DEFAULT**  | `C:\Windows\System32\Config\DEFAULT`  | `HKEY_USERS\.DEFAULT`          | Registry profile mặc định cho LocalSystem/default user context                | Phân tích cấu hình mặc định/system context, ít dùng hơn nhưng vẫn quan trọng     |
## Backup hive
![[Pasted image 20260720144549.png]]
- Windows có cơ chế backup một số registry hive
- Đường dẫn : `%WinDir%\System32\Config\RegBack`
- Tác vụ scheduled task tên `RegIdleBackup` sẽ chạy định kỳ, khoảng mỗi 10 ngày, và copy các hive core vào thư mục `RegBack`
- Backup hive có thể chứa dữ liệu cũ đã bị xóa hoặc thay đổi trong hive hiện tại. nó chủ yếu backup các hive hệ thống, không backup hive user nằm trong thư mục user profile.
## User Registry hive
![[Pasted image 20260720144937.png]]

### NTUSER.DAT
- Chứa thông tin cấu hình và môi trường làm việc của từng user. Nó lưu rất nhiều dữ liệu liên quan đến hoạt động của user.
- `C:\Users\<username>\NTUSER.DAT`
### USRCLASS.DAT
![[Pasted image 20260720145144.png]]
- Mục đích chính của `UsrClass.dat` là hỗ trợ registry virtualization cho **User Account Control — UAC**.
- `C:\Users<username>\AppData\Local\Microsoft\Windows\UsrClass.dat`
- Hive này rất quan trọng vì chứa một số artifact liên quan đến:
```
program execution
folder opening
ShellBags
UAC virtualization
file extension associations
```

### Registry Key và value
![[Pasted image 20260720145755.png]]

![[Pasted image 20260720145804.png]]

### Registry Last write time

![[Pasted image 20260720145831.png]]

- Mỗi key trong registry thường có last write time -thời điểm key được cập nhật lần cuối cùng. Thường được lưu dưới dạng UTC.

### MRU list
![[Pasted image 20260720150015.png]]

- MRU list là một value dùng để theo dõi thứ tự các item được thêm vào một key

### Delete Registry key/value
![[Pasted image 20260720150114.png]]

- Registry hive có vùng allocated và unallocated giống system.
- Khi một registry key bị xóa, key đó thường chỉ bị đánh dấu unallocated, tương tự như file bị xóa trên ổ đĩa. Điều này có nghĩa là key đã xóa có thể phục hồi được bằng công cụ forensics.

# Analysis

| Nhóm phân tích                | Mục tiêu chính                         | Hive cần xem                       | Artifact / Key thường gặp                                                                                     | Có thể trả lời câu hỏi gì?                                                                                  | Ý nghĩa forensic                                   |
| ----------------------------- | -------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| **1. User / Group Profiling** | Xác định user và group trên máy        | `SAM`                              | Local Users, Local Groups, RID, Group Membership                                                              | Máy có user nào? User nào là admin? RID là gì? User từng đăng nhập chưa?                                    | Bước nền tảng để biết **ai có tài khoản trên máy** |
| **2. System Configuration**   | Xác định cấu hình hệ thống             | `SYSTEM`, `SOFTWARE`               | TimeZoneInformation, ComputerName, Services, Drivers, Network Interfaces, Windows Version, Installed Software | Máy tên gì? Timezone là gì? Service nào tồn tại? Driver nào được cài? Windows version gì? App nào được cài? | Giúp hiểu **máy được cấu hình như thế nào**        |
| **3. User Activity**          | Phân tích hành vi của từng user        | `NTUSER.DAT`, `USRCLASS.DAT`       | RecentDocs, RunMRU, UserAssist, TypedPaths, ComDlg32, ShellBags                                               | User đã mở file gì? Chạy chương trình gì? Gõ lệnh gì? Truy cập folder nào? Mở/lưu file ở đâu?               | Giúp dựng lại **user đã làm gì trên máy**          |
| **4. Analyze USB Devices**    | Phân tích USB/ổ ngoài từng gắn vào máy | `SYSTEM`, `SOFTWARE`, `NTUSER.DAT` | USBSTOR, USB, MountedDevices, MountPoints2, Volume GUID, Drive Letter                                         | USB nào từng cắm? Serial number là gì? Cắm lần đầu/cuối khi nào? Gán ổ nào? User nào có thể đã dùng USB?    | Rất quan trọng khi nghi **copy dữ liệu ra USB**    |

## Collecting User Infomation - Analyzing the SSAM

### SAM profiling user/groups

![[Pasted image 20260720151219.png]]

- Nhiều artifact Windows không ghi username trực tiếp mà ghi RID, SID nên cần dùng SAM để map RID/SID -> Username
-  **RID — Relative Identifier** là phần cuối của SID.
- SID là `S-1-5-21-111111111-222222222-333333333-1001` thì RID là `1001`

| RID   | Y                      |
| ----- | ---------------------- |
| 500   | Buitl-in Administrator |
| 501   | Guest                  |
| 1000+ | User được tạo sau này  |
-  Nếu User login bằng Microsoft Live ID , logon count có thể không tăng.
![[Pasted image 20260720151730.png]]


## Examiming System Configuration
![[Pasted image 20260720152022.png]]

###  Identify Microsoft OS Version
![[Pasted image 20260720152052.png]]
- Trong thực tế, investigator thường nhận một ổ cứng hoặc image mà không biết chính xác hệ điều hành là gì.
- Việc biết phiên bản Windows rất quan trọng vì:
```
đường dẫn artifact có thể khác
loại artifact có thể khác
cách parse artifact có thể khác
chương trình mặc định có thể khác
```

![[Pasted image 20260720152232.png]]

### Identify CurrentControlSet
![[Pasted image 20260720152318.png]]

- Các hive được sử dụng để điều khiển hệ thống khởi động gọi là Control Sets . Thông thường ta sẽ thấy 2 Control Sets , đó là `ControlSet001` và `ControlSet002`, trong tổ SYSTEM.
    - `ControlSet001`**:** Thường là cấu hình **hiện tại** mà Windows đang sử dụng hoặc vừa mới sử dụng để khởi động thành công.
    - `ControlSet002`**:** Thường là cấu hình **Last Known Good Configuration** (Cấu hình tốt gần nhất). Đây là bản "sao lưu" từ lần cuối cùng bạn đăng nhập vào Windows thành công mà không gặp lỗi hệ thống nghiêm trọng.

![[Pasted image 20260720152437.png]]

- Khi mở hive offline , chúng ta thường không thấy CurrentControlSet mà chỉ thấy ControlSet001/ControlSet002 nên để biết cái nào tương ứng với CurrentControlSet thì :
    - Ta xem Key : `SYSTEM\Select` , trong key này có value Current
        - Current =1 thì ControlSet001 = CurrentControlSet
        - Current =2 thì ControlSet002 = CurrentControlSet
    - Một value khác là LastKnowGood
        - LastKnowGood = 0x2 thì ControlSet002 là Last Know Good
### Computer Name
![[Pasted image 20260720152533.png]]

###  Time Zone Information
![[Pasted image 20260720152603.png]]
![[Pasted image 20260720152622.png]]

### Last Access Time
![[Pasted image 20260720152651.png]]

- Nếu Last Access timestamp bị tắt, ta sẽ không thể biết chính xác khi nào dữ liệu file được hệ thống truy cập lần cuối.
- Đây là thiết lập mặc định trên Windows 7 đến Windows 10, nhưng không phải trên mọi phiên bản Windows.
- Nếu không có Last Access Time, việc xác định khi nào user mở file sẽ khó hơn.
- ==Last Access Time có thể thay đổi khi file chỉ bị hệ thống “chạm” tới, không nhất thiết được user mở.==

### Network Interfaces
![[Pasted image 20260720152743.png]]

### Historical Networks
![[Pasted image 20260720152828.png]]
![[Pasted image 20260720152836.png]]

- Từ Windows 7 trở lên, Windows có cơ chế **Network Location Awareness**.
- NLA giúp Windows nhận biết máy đang kết nối loại network nào, từ đó áp dụng firewall profile phù hợp. NLA có thể cho biết máy từng kết nối những network nào.
- Tại thời điểm này, ta vẫn chưa biết chính xác lần đầu và lần cuối network này được kết nối là khi nào.Ta sẽ dùng:`ProfileGuid` để tìm thông tin thời gian ở một key khác trong `SOFTWARE` hive.

### Network Profiles Key 
![[Pasted image 20260720152928.png]]

### Share and Offline caching
![[Pasted image 20260720153039.png]]

### System Boot Autostart Programs
![[Pasted image 20260720153111.png]]

### Shutdown Infomation
![[Pasted image 20260720153135.png]]

## Analyzing User Activity
![[Pasted image 20260720153311.png]]

### Search History
![[Pasted image 20260720153336.png]]
![[Pasted image 20260720153344.png]]

### Type paths
![[Pasted image 20260720153416.png]]

- Khóa này sẽ hiển thị đường dẫn mình tự nhập thủ công vào menu Start hoặc thanh Explorer.
- Khóa này tự động sắp xếp theo thứ tự gần nhất mặc dù không dùng khóa MRU.

### Recent Docs
![[Pasted image 20260720153555.png]]

### Microsoft Recent Docs
![[Pasted image 20260720153638.png]]
- Không chỉ hệ điều hành lưu trữ các tài liệu cuối cùng mà người dùng đã lưu, mà các sản phẩm của microsoft office cũng làm điều này. Mỗi phiên bản Office sẽ giữ một danh sách tất cả các tệp được mở trong một chương trình cụ thể
- Vị trí này luôn nằm trong một khóa gọi là **FileMRU**.

### Explorer Common Dialog Box
![[Pasted image 20260720153733.png]]
- Các khóa **OpenSaveMRU** và **LastVisitedMRU** lưu lại lịch sử vị trí và tệp mà người dùng đã mở/lưu qua hộp thoại chuẩn của Windows:
	- OpenSaveMRU:
	![[Pasted image 20260720154023.png]]
	- LastVisitedMRU
	![[Pasted image 20260720154054.png]]
![[Pasted image 20260720154110.png]]

### Last Command Executed
![[Pasted image 20260720154141.png]]

- **RunMRU key** lưu lại lịch sử các lệnh mà người dùng đã chạy qua hộp thoại _Start → Run_.

### UserAssist Key

- Liệt kê tất cả các ứng dụng được khởi chạy từ giao diện GUI của người dùng. Hơn nữa , nó sẽ liệt kê số lần thực thi và thời gian chạy cuối cùng của ứng dụng
![[Pasted image 20260720154251.png]]
![[Pasted image 20260720154258.png]]
