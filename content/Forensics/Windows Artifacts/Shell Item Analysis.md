# Shell Items Overview
![[Pasted image 20260721072057.png]]

- Windows Shell cho phép người dùng có các tệp shortcut ( hay còn gọi là link hoặc .lnk)
-  Mỗi tệp Shortcut là tệp chứa thông tin dùng để truy cập đến một tệp khác giống như một dạng con trỏ.
- Bên trong các tệp .lnk có rất nhiều thông tin hữu ích. Một số thông tin quan trọng gồm:
	- Loại ổ đĩa mà tệp/đối tượng đích nằm trên (ổ cứng cố định , ổ đi động,...).
	- Đường dẫn đầy đủ của tệp đích, bao gồm ký tự ổ đĩa, nhãn volume, và số Serial của các volume gắn cục bộ.
	- Với các volume từ xa (chia sẻ mạng), tệp .lnk lưu đường dẫn đầy đủ đến máy chủ chia sẻ và ký tự ổ đĩa (nếu chia sẻ được gắn kết).
- Ngoài ra, nếu tệp đích nằm trong một thư mục “đặc biệt” hoặc “được biết đến” (ví dụ: thư mục Printers), thông tin này cũng được lưu trong tệp .lnk.
- Bên cạnh những thông tin trên, thường sẽ có một hoặc nhiều bản sao metadata của tệp đích (dấu thời gian, kích thước, thuộc tính) bên trong tệp .lnk. Nếu tệp đích là một tệp thực thi, các tham số dòng lệnh cũng được lưu trong tệp .lnk.
![[Pasted image 20260721105129.png]]

| Section | Tên gọi khác | Dữ liệu chính được lưu | Ý nghĩa pháp y (Điều tra viên cần biết) |
|---------|--------------|------------------------|------------------------------------------|
| **1. Header** | Tiêu đề | - **LinkFlags**: Cờ cho biết các section nào có trong file LNK.<br>- **Timestamps** và **File Attributes** của file đích. | - Kiểm tra file LNK có hợp lệ hay không.<br>- Khôi phục thời gian của file gốc ngay cả khi file đích đã bị xóa. |
| **2. PIDL** | Target ID / Shell Items | - Danh sách các **Shell Item** đại diện cho từng cấp thư mục.<br>- Mỗi Shell Item có thể chứa **Tên**, **MFT Reference**, và **Timestamp** riêng. | - Ghép lại đường dẫn đầy đủ (ví dụ: `C:\Users\Teo\Documents\report.docx`).<br>- Khôi phục siêu dữ liệu của các thư mục cha, ngay cả khi chúng đã bị xóa. |
| **3. LinkInfo** | Thông tin ổ đĩa | - **VolumeID**: Loại ổ đĩa, Volume Serial Number, Volume Label (đối với ổ cục bộ).<br>- **Common Network Relative Link (CNRL)**: Server Name, Share Path (đối với tài nguyên mạng). | - Xác định USB hoặc ổ đĩa đã từng chứa file đích.<br>- Xác định máy chủ hoặc thư mục chia sẻ trên mạng mà shortcut từng trỏ tới.<br>- Thông tin vẫn tồn tại ngay cả khi USB đã bị rút hoặc kết nối mạng đã bị ngắt. |
| **4. StringData** | Chuỗi dữ liệu | - **Name String** (Tên shortcut).<br>- **Relative Path**.<br>- **Command-line Arguments**.<br>- **Working Directory**.<br>- **Icon Location**. | - Phát hiện tham số thực thi đáng ngờ của malware (ví dụ: `-hidden`, `-EncodedCommand`).<br>- Xác định shortcut có thể hoạt động khi di chuyển sang USB hoặc thư mục khác thông qua Relative Path.<br>- Khôi phục môi trường thực thi của chương trình. |
| **5. ExtraData** | Dữ liệu mở rộng | - **TrackerDataBlock**: Machine ID, Droid/DroidBirth (Object ID).<br>- **PropertyStoreDataBlock**: Thuộc tính mở rộng của Windows.<br>- Các ExtraData Block khác tùy phiên bản Windows. | - Liên kết cùng một file giữa nhiều máy tính (Host Correlation).<br>- Hỗ trợ xác định nguồn gốc của file sau khi sao chép hoặc di chuyển.<br>- Rất hữu ích trong điều tra lây nhiễm APT và theo dõi quá trình lan truyền của dữ liệu hoặc malware. |

![[Pasted image 20260721122857.png]]

- **4 bytes đầu tiên**: mô tả kích thước của header. Theo chuẩn Microsoft, giá trị này **luôn luôn là 0x4C**.
- **16 bytes tiếp theo**: là **CLSID (GUID)** dành riêng cho LNK files. Đây là cách Microsoft định danh loại đối tượng.
- **4 bytes kế tiếp**: là **flags** mô tả các section khác có tồn tại trong LNK file (ví dụ: LinkInfo, StringData, ExtraData…).

# Shortcut Files (.lnk)
![[Pasted image 20260721105356.png]]

- Shortcut file do Windows tạo ra 
	-  Người dùng có thể không tự tạo file LNK , nhưng **Windows chắc chắn sẽ tạo** khi bạn mở và sử dụng các file media hiển thị trong thư mục _Recent Documents/Files_.
	- Ví dụ: nếu mở và chỉnh sửa một file từ USB nhưng không copy vào máy, Windows vẫn tạo shortcut trong thư mục _Recent Items_ của tài khoản người dùng.
- ==Lưu ý: Windows Explorer sẽ tạo/cập nhật LNK ngay khi bạn **double-click** vào file. Dù file đích có thay đổi hay không, metadata trong LNK vẫn được cập nhật vì bạn đã truy cập nó. Ngoài ra, Windows cần khoảng **60 giây** trước khi một lần click mới được ghi nhận để cập nhật thời gian.==

Những thay đổi về LNK files từ Windows 10:
	- **Tạo LNK khi file được tạo mới**
		- Trước Windows 10: chỉ khi người dùng **mở file** thì mới sinh ra LNK trong thư mục _Recent_.
		- Từ Windows 10: ngay khi **tạo file mới**, hệ thống cũng sinh ra LNK trong _Recent_.
	- **Tên file trong LNK có thêm phần mở rộng (extension)**
		- Trước Windows 10: LNK chỉ lưu tên file, **không có extension**.
		    - Ví dụ: `NokiaStrategy.lnk` 
		- Windows 10: LNK lưu cả tên file **kèm extension**.
		    - Ví dụ: `NokiaStrategy.docx.lnk`
		- Với thư mục: không có extension → dễ nhận diện vì chúng là LNK không có phần mở rộng.

## LeCMD.exe - LNK File Analysis
![[Pasted image 20260721112920.png]]

![[Pasted image 20260721113146.png]]

- Phân tích chi tiết cấu trúc LNK file
	- Header 
		-  Chứa thông tin về file gốc 
			-  Timestamps (MAC) của file đích
			-  Kích thước file
			- Các flags xác định dữ liệu bổ sung
		- Flags trong header quyết định những **section khác** có tồn tại trong LNK.
		- Tùy theo flags, có thể có thêm thông tin như: _working directory_, _relative path_, v.v.
	-  Link Infomation 
		- Chỉ xuất hiện khi **HasLinkInfo flag** được bật
		- Bên trong LinkInfo cũng có các flags để xác định cấu trúc dữ liệu.
	- Target ID Infomation
		- Chỉ xuất hiện nếu **flag** tương ứng được bật.

## LNK File Analysis lp.exe  (có phí)
![[Pasted image 20260721114119.png]]

**lp.exe** là một tool phân tích **nhiều file LNK cùng lúc**   




# JumpLists

![[Pasted image 20260721141804.png]]

- **Thanh taskbar (Jumplist) của Windows 7–10** được thiết kế để giúp người dùng nhanh chóng truy cập các mục họ thường xuyên hoặc vừa mới sử dụng. Chức năng này không chỉ dành cho các file media gần đây, mà còn cho các **tác vụ** gần đây. 
	- Ví dụ: tạo một archive mới, ghi dữ liệu ra DVD, hoặc trò chuyện với bạn bè qua Skype.
## Destinations và Tasks
![[Pasted image 20260721145659.png]]
- Windows Jumplists bao gồm cả Destinations và Tasks . Những mục này thường được xác định bởi nhà phát triển ứng dụng
	- **Thiết lập mặc định**: hiển thị các file media được sử dụng gần đây hoặc thường xuyên, đồng thời cho phép các tác vụ mặc định như:
		- Ghim ứng dụng vào taskbar.
		- Khởi động ứng dụng
		- Đóng tất cả cửa sổ ứng dụng
### Destination
- Là các mục xuất hiện trong danh mục _Recent_, _Frequent_ hoặc _Custom_ (trong sơ đồ gọi là _Important_).
- Dựa trên thói quen sử dụng của người dùng.
- Có thể là file, thư mục, website hoặc nội dung khác, nhưng không nhất thiết phải có file gốc đi kèm.
- Người dùng có thể **ghim (pin)** hoặc **xóa (remove)** Destinations khỏi Jumplist.
### Task
- Là các hành động phổ biến trong ứng dụng, áp dụng cho tất cả người dùng, không phụ thuộc vào thói quen cá nhân.
- Không thể ghim hoặc xóa.
- Thực chất là các **LNK files** (có thể kèm tham số) trỏ tới các lệnh.

### Hai loại Jumplist

- **Automatic Jumplists**: được Windows tự động tạo cho mỗi ứng dụng.
- **Custom Jumplists**: được tạo bởi nhà phát triển ứng dụng với thông tin tùy chỉnh.
Cả hai loại này đều được tìm thấy trong thư mục _Recent_ mà trước đó chúng ta đã phân tích khi nói về LNK files.

👉 Tóm lại: **Destinations** phản ánh thói quen sử dụng của người dùng (file, thư mục, website…), còn **Tasks** là các hành động mặc định của ứng dụng. Việc phân tích Jumplists giúp điều tra viên tái dựng lịch sử sử dụng ứng dụng và file, kể cả khi file gốc đã bị xóa.