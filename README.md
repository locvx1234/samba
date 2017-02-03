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
		; path of files to share
		path = /samba/admin/data
		; users admitted to use the file sharing service
		valid users = admin
		invalid users = user2 user3
		; no guest user is admitted
		guest ok = no
		; make the share writable as Samba make it as readonly by default
		writable = yes
		; make the share visible as shared folder
		browsable = yes

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
#### Kiểm tra cấu hình 

	# testparm
	
#### Tạo user 

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
	
Để login từ Windows vào, tại Windows Explorer ta nhập theo cú pháp `\\ip-samba-server`        

Để logout, ta sử dụng lệnh `net use * /delete`  trong command line.


	
	
	