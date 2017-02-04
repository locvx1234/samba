# samba
Ghi chép về Samba

## 1. Samba là gì ?

*Samba* là một phần mềm mã nguồn mở cung cấp dịch vụ chia sẻ file giữa các nền tảng (Window, Linux, UNIX và các OS khác) sử dụng giao thức [SMB/CIFS](https://en.wikipedia.org/wiki/Server_Message_Block).

Samba bao gồm các dịch vụ `smb`, `nmb`, `winbind`

`smbd` daemon quản lý chia sẻ file và in của các client. Bên cạnh đó nó còn có trách nhiệm chứng thực người dùng, quản lý tài nguyên và chia sẻ dữ liệu qua giao thức SMB. Cổng mặc định mà server lắng nghe cho các traffic SMB là 139 và 445.

`nmbd` daemon cung cấp tên của NetBIOS cho các request, sử dụng cổng mặc định 137

`winbindd` xử lý các thông tin và người dùng và nhóm người dùng nhận được từ Windows server. Điều này làm cho các nền tảng Linux/UNIX hiểu thông tin về người dùng và nhóm người dùng của Windows. 

## 2. Cài đặt và cấu hình 

#### Cài đặt 

	# apt-get install samba
	
#### Cấu hình 

File cấu hình samba ở đường dẫn `/etc/samba/smb.conf`

	# cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
	# vi /etc/samba/smb.conf
	
Trong file có 2 loại chú thích `#` và `;`. Trong đó `#` là chú thích thực, `;` là xác định các thuộc tính nếu xóa ký tự đó.  

File gồm 2 phần `Global Settings` và `Share Definitions`

	#======================= Global Settings =======================
	
	workgroup = WORKGROUP  #  workgroup của Windows mà máy Linux đăng nhập
	server string = %h server (Samba, Ubuntu) # Mô tả về Samba server
	interfaces = 127.0.0.0/8 eth0 # Các giao tiếp mạng mà samba lắng nghe
	log file = /var/log/samba/log.%m # Vị trí lưu log cho từng máy truy cập với `m` là tên máy
	max log size = 1000 # Dung lượng tối đa của file log (KB)
	
	#======================= Share Definitions =======================
	[share1]
		comment = Private Documents
		path = /samba/admin/data # Đường dẫn tới file share 
		valid users = admin  # user hợp lệ 
		# valid users = @group_name
		invalid users = user2 user3 # user không hợp lệ 
		guest ok = no # guest không được truy cập 
		writable = yes # có quyền ghi, mặc định là readonly
		browsable = yes # hiển thị thư mục share 

	[share2]
		comment = Public Documents
		path = /samba/user2/data
		valid users = user2 admin
		guest ok = no
		writable = yes
		browsable = yes

	[share3]
		comment = Public Documents
		path = /samba/user3/data
		valid users = user3 admin
		guest ok = no
		writable = yes
		browsable = yes

		
`share1` chỉ được truy cập bởi `admin`, `share2` được truy cập bởi `admin` và `user2` nhưng `user3` thì không. `share3` thì được truy cập bởi `admin` và ``user3`.

`anonymous` được truy cập bởi tất cả mọi người 


Cấu hình cho anonymous
	
	[anonymous]
		comment = Public Documents
		path = /samba/anonymous
		guest ok = yes
		writable = yes
		browsable = yes

#### Kiểm tra cấu hình 

	# testparm
	
#### Bảo mật cho Samba server

Samba sử dụng một database để lưu trữ pass của user. Tương tự như file `/etc/passwd`, trên Ubuntu được đặt trong `/var/lib/samba/private/`

Tạo user và đặt pass trong Samba user database:

	# smbpasswd -a admin
	New SMB password:
	Retype new SMB password:
	#
	
Tương tự với user2, user3. Đảm bảo rằng cả 3 user này đã được tạo là user thông thường nếu không sẽ bị `Failed to add entry for the user`. 

Kiểm tra các user đã tạo :

	# pdbedit -L
	admin:1002:
	user2:1004:
	user3:1005:
	
Lưu ý với folder share của user nào thì ta cấp quyền thích hợp cho user đó.

Với anonymous : 
	
	# cd /samba
	# chmod -R 0755 anonymous/
	# chown -R nobody:nogroup anonymous/
		
Để login từ Windows vào, tại Windows Explorer ta nhập theo cú pháp `\\ip-samba-server`        

Để logout, ta sử dụng lệnh `net use * /delete`  trong command line.

## 3. File permission Windows và các thuộc tính 

Các permission trong Linux khác Windows. Ở Linux có 3 chế độ read, write và execute tương ứng với 3 lớp user là owner (u), group (g) và other (o). Windows thì sử dụng 4 thuộc tính chính là : readonly, system, hidden và archive.

* Read-only : Nội dung của file có thể đọc bởi user nhưng không thể ghi và xóa.
* System : Thuộc tính này để đánh dấu các file hệ thống. Các file này không nên chỉnh sửa hay xóa.
* Hidden : File được đánh dấu ẩn đi, trừ khi hệ điều hành thiết hiển thị file ẩn.
* Archive : Bất cứ khi nào file được chỉnh sửa, bit này sẽ được đánh dấu, nó cho biết thời gian sửa đổi cuối cùng 

Bên cạnh đó còn các thuộc tính khác : Volume Label, Directory.

Không có bit nào xác định một file được thực thi, mà nó xác định cách thực thi dựa vào phần mở rộng của file. Của file Windows được lưu trên Linux Samba cần giữ được các thuộc tính của file. Samba bảo toàn các bit này bằng cách tái sử dụng cho các bit permission của file Linux. 

Cấu hình :
	
	[share]
		...
		store dos attributes = yes
		map archive = yes ;default is yes
		map system = yes  ;default is no
		map hidden = yes  ;default is no
		create mask = 0744           ;default is 0744
		directory mask = 0755        ;default is 0755
		
		
*Tham khảo*

https://github.com/kalise/Linux-Tutorial/blob/master/content/samba_server.md


https://help.ubuntu.com/12.04/serverguide/samba-fileserver.html