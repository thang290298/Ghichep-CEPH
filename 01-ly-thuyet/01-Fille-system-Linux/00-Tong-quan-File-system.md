<h1 align="center">Tổng quan về filesystem trên Linux</h1>

# Phần I. File system
- FIle system được sử dụng với mục đích để quản lý cách đọc, ghi và lưu trữ dữ liệu trên thiết bị, hỗ trợ truy cập nhanh chóng khi cần thiết

<h3 align="center"><img src="../../03-Images/document/4.png"></h3>

- Sự khác biệt giữa giữa 1 ổ đĩa hoặc phân vùng và hệ thống tập tin được lưu trữu trên đó là rất quan trọng. Một số chương trình(bao gồm chương trình tạo ra hệ thống tập tin) hoạt động trực tiếp trên các Sector thô của 1 ổ đĩa hay phân vùng., Nếu có 1 hệ thống tập tin tồn tại trên đó thì nó sẽ bị phá hủy hoặc hỏng hóc
- Để một phân vùng hoặc ổ đĩa có thể sử dụng như một hệ thống tập tin thì nó cần được khởi tạo và các cấu trúc dữu liệu của kiểu tập tin đó sẽ được lưu vào ổ đĩa. Đây được gọi là ` quá trình tạo hệ thống tập tin`

- Hầu hết các loại hệ thống tập tin UNIX đều có cấu trúc chung tương tự nhau, mặc dù các chi tiết cụ thể khác nhau khá nhiều. Các khái niệm chủ chốt là superblock, inode, data block, directory block và indirection block.
  - Superblock: chưa các thông tin về hệ thống tập tin một cách tổng thể, chẳng hạn như kích thước của nó (thông tin chính xác ở đây phụ thuộc vào hệ thống tập tin)
  - Inode: chứa tất cả các thông tin về một tập tin, ngoại trừ tên của nó. Tên được lưu trữ trong thư mục, cùng với số lượng lớn các inode. Mục nhập thư mục bao gồm tên tập tin và các số lượng inode đại diện cho tập tin đó. Inode chứa khối lượng lớn các khối dữ liệu, được sử dụng để lưu trữ dữ liệu trong tập tin.
  - Data block: đây là nơi dữ liệu được lưu trữ
# Phần II. Các Loại filesystem phổ biến trên Linux

Các loại filesystem được linux hỗ trợ
- Cơ bản: Minix,Ext,EXT2, EXT3, EXT4, XFS, Btrfs, JFS, NTFS,ReiserFS,Swap,..
- Filesystem dùng cho dạng lưu trữ  Flash: thẻ nhớ, usb,...
- Filesystem dành cho hệ cơ sở dữ liệu
- Filesystem mục đích đặc biệt: procfs, sysfs, tmpfs, squashfs, debugfs,…

# Phần III. Phân vùng và filesystem

Một phân vùng chưa trong đó 1 filesystem, trong một số trường hợp thì filesystem có thể mở rộng phân vùng nếu filesystem sử dụng các liên kết

Filesystem là phương pháp lưu trữ hoặc tìm kiếm tập tin trên một đĩa cứng ( trong một phân vùng)


So sánh giữa filesystem trên hệ điều hành Windows và hệ điều hành Linux:

|  | Windows | Linux |
|--------------|-------|------|
| Phân vùng | Disk1| /dev/sda1|
| Loại Filesystem | NTFS/VFAT| Minix,Ext,EXT2, EXT3, EXT4, XFS, Btrfs, JFS, NTFS,ReiserFS,Swap,..|
| Mounting Parameters | DrivelLetter| MountPoint|
| Hệ điều hành lưu trữ | C:/| /|

# Phần IV. Filesystem Hierarchy Standard (FHS)
Filesystem của linux được tổ chức theo tiêu chuẩn cấp hệ thống tập tin `Filesystem Hierarchy Standard ( FHS )`, tiêu chuẩn này định nghĩa mục đích sử dụng của từng thư mục
<h3 align="center"><img src="../../03-Images/document/5.png"></h3>

nghĩa mục đích sử dụng của từng thư mục
<h3 align="center"><img src="../../03-Images/document/6.png"></h3>

Các thư mục được mô tả như sau:

<dev width="100%">
<dev align="center">

| Windows | Linux |


</dev></dev>

























## 2. Đặc hiển một số loại Filesystem phổ biến
   - `Minix`: là hệ thống lâu đời nhất và được cho là đáng tin cậy nhất, nhưng nó khá hạn chế về các tính năng (một số nhãn thời gian – time stamp bị thiếu, tối đa 30 kí tự / tệp tin)
   - `Ext – Extended file system`: là định dạng file hệ thống đầu tiên được thiết kế dành riêng cho Linux. Có tổng cộng 4 phiên bản và mỗi phiên bản lại có một tính năng nổi bật. Phiên bản Ext đầu tiên là phần nâng cấp từ file hệ thống Minix được sử dụng tại thời điểm đó nhưng lại không đáp ứng được nhiều tính năng phổ biến ngày nay. Và tại thời điểm này, chúng ta không nên sử dụng Ext vì có nhiều hạn chế, không còn được hỗ trợ nhiều distribution.
   - `Ext2`: thực chất không phải là file hệ thống journaling, được phát triển để kế thừa các thuộc tính của file hệ thống cũ, đồng thời hỗ trợ dung lượng ổ cứng lên 2TB. Ext2 không sử dụng journal cho nên sẽ có ít dữ liệu được ghi vào ổ đĩa hơn. Do lượng yêu cầu viết và xóa dữ liệu khá thấp nên nó phù hợp với các thiết bị lưu trữ gắn ngoài như thẻ nhớ, USB,…
   - `Ext3`: về căn bản đây chỉ là Ext2 và đi kèm với journaling. Mục đích chính của Ext3 là tương tích ngược với Ext2, và do vậy, những ổ đĩa, phân vùng có thể dễ dàng được chuyển đổi giữa 2 chế độ mà không cần phỉa format như trước kia. Tuy nhiên, vấn đề về giới hạn của Ext2 vẫn còn nguyên trên Ext3. Ext3 không thực sự phù hợp để làm file hệ thống dành cho máy chủ bởi vì không hỗ trợ tính năng tạo disk snapshot và file được khôi phục sẽ rất khó để xóa bỏ sau này.
   - `Ext4`: cũng giống như Ext3, lưu giữ được những ưu điểm và tính tương thích ngược với phiên bản trước đó. Như vậy, chúng ta có thể dễ dàng kết hợp các phân vùng định dạng Ext2, Ext3 và Ext4 trên cùng 1 ổ đĩa để tăng hiệu suất hoạt động.