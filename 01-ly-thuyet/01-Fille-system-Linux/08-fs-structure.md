<h1 align="center">Cấu trúc hệ thống tập tin</h1>

## I. Tổng quan

- Disk được sử dụng nhiều cho hoạt động lưu trữ và duy trì FS. 2 điểm quan trọng của Disk khiến nó được sử dụng:
  - Chúng có thể được viết lại bằng cách thay thế; có thể đọc một khối từ đĩa, sửa một khối và viết nó ngược trở lại đĩa trong cùng vị trí.
  - Chúng có thể được truy xuất trực tiếp bất cứ khối thông tin nào trên đĩa
- Để cải thiện hiệu suất IO, quá trình truyền giữa memory và disk thực hiện trên đơn vị blocks. Mỗi block có 1 hoặc nhiều sector. Dựa trên disk drive, sector size từ 32 bytes tới 4096 byte. Thông thường 512 byte.

## II. Cấu trúc File system

Filesystem cung cấp tính thuận tiện và hiệu quá khi truy cập disk. Cho phép data lưu, định vị, truy xuất dễ dàng.

Filesystem có 2 vấn đề cơ bản:
  - Cách Filesystem thể hiển từ phía user.
  - Giải thuật, cấu trúc dữ liệu ánh xạ logical filesystem vào thiết bị lưu trữ

Filesystem được tạo thành từ nhiều cấp, các cấp được thiết kế để tận sử dụng các tính năng của các cấp khác. Mô hình cơ bản

<h3 align="center"><img src="../../03-Images/document/18.png"></h3>

### 2.1 I/O control - Điều khiển nhập/xuất

- Là cấp thấp nhất chứa các device drivers và các bộ quản lý ngắt để chuyển thông tin giữa main memory và disk system.
- Device driver có thể coi như bộ dịch. Input high level cmd, output chỉ thị phân cứng (hardware-specific instructions) sử dụng bởi Hardware controller (giao diện I/O device toàn hệ thống).
- Device driver thường viết các mẫu bit xác định tới các vị trí trong I/O controller’s memory để báo với bộ điều khiển vị trí trên thiết bị nào và hoạt động gì xảy ra.

### 2.2 Basic file system - Hệ thống tập tin cơ bản
- Phát ra các lệnh thông thường tới các  device driver  tương ứng để đọc và viết các khối vật lý trên đĩa. Mỗi khối vật lý được xác định bởi địa chỉ đĩa (ví dụ, đĩa 1, cyclinder 73, track 2, sector 10).
- Lớp này còn quản lý  **memory buffers, caches**  lưu trữ các  file system, directory, data blocks khác nhau .
- Block trong cluster được cấp phát trước khi tiền trình chuyển tới disk block xảy ra.
- Khi full,  buffer manager  sẽ tìm thêm hoặc giải thoát  buffer memory  để chấp nhận  requested I/O . Cache thường sử dụng để lưu file-system metadata thương sử dụng, năng cao hiệu năng.

### 2.3 File-organization module - Module tổ chức tập tin
- Nhận biết file, their logical blocks, physical blocks.
- Bằng hiểu biết về cấp phát file, vị trí của file, FOM module có thể chuyển dịch logical block addresses thành physical block addresses cho phép fs truyển tải.
- Mỗi  file’s logical blocks  được đánh số 01 tới N. Vì physical blocks chứa data thường không trung logical numbers, thao tác chuyển dịch cần thiết để xác định mỗi block.
- FOM bao gồm  free-space manager , theo dõi khối không không được gán, cung cấp block đó cho FOM khi được yêu cầu.

### 2.4 Logical file system - Hệ thống tập tin luận lý
- Quản lý thông tin metadata. Metadata chứa tất cả cấu trúc hệ thống tập tin, ngoại trừ dữ liệu thật sự (hay nội dung của các tập tin).
- Logical file system quản lý cấu trúc thư mục để cung cấp module tổ chức tập tin những thông tin yêu cầu sau đó, được cho tên tập tin ký hiệu.
- Nó duy trì cấu trúc tập tin bằng khối điều khiển tập tin. Một khối điều khiển tập tin (file control block-FCB)(inode in UNIX file systems) chứa thông tin về tập tin, gồm người sở hữu, quyền và vị trí của nội dung tập tin.