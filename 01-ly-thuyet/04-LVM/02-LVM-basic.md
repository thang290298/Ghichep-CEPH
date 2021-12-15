<h1 align="center">Cấu hình LVM cơ bản</h1>

# Phần I. Chuẩn bị
- thiết lập chuẩn bị 1 VM có thông số cấu hình như sau:
  - OS: Ubuntu 20.04
  - RAM: 3GB
  - CPU: 3 Core
  - Disk: 
    - OS: 25 GB
    - Disk 1: 30GB
    - Disk 2: 30GB
    - Disk 3: 30GB

 
<h3 align="center"><img src="../../03-Images/document/58.png"></h3>

# Phần II: Tạo Logical Volume trên LVM

## 1. Tạo Partition

```
fdisk /dev/sdb
```

- Nhập `n` để tạo phân vùng mới sẽ nhắc bạn chỉ định cho một phân vùng chính hoặc phân vùng mở rộng. Nhập `p` cho phân vùng chính hoặc `e` cho phân vùng mở rộng.

```
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
```
- Sau đó, bạn sẽ được nhắc nhập số của phân vùng sẽ được tạo. Bạn có thể nhấn Enter để chấp nhận mặc định.
```
Partition number (1-4, default 1): 1
```

- Sau đó, bạn sẽ được nhắc nhập kích thước của phân vùng sẽ được tạo ví dụ dưới chúng ta sẽ tạo đĩa với size 5GB. Bạn có thể nhấn `Enter` để sử dụng tất cả không gian có sẵn.

```
First sector (2048-33554431, default 2048): ` gõ Enter`
Last sector, +sectors or +size{K,M,G} (2048-33554431, default 33554431): ` 10002400 ` // giá trị disk 
Partition 1 of type Linux and of size 4 MiB is set
```
- Lựa chọn định dạng LVM cho disk
  - chọn `t` để thay đổi định dạng phân vùng disk vừa khởi tạo

  ```
  Command (m for help): t
  selected Partition 1
  ```
  sau đó bấm `L` và gõ `8e` để chọn định dạng `Linux LVM`

- Chạy lệnh  `w` để lưu các thay đổi.
```
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

Hình ảnh minh họa:

<h3 align="center"><img src="../../03-Images/document/flisk.png"></h3>

## 2. Tạo Physical Volume
