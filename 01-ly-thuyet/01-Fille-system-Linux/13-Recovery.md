<h1 align="center">Khôi phục</h1>

## I. Tổng quan

- File và directory được lưu trên main memory và disk => khi xảy ra lỗi cần khả năng khôi phục.

- Lỗi hệ thống có thể từ xảy ra từ xung đột trên disk file system data (directory structures, free-block pointers, free FCB pointers).

**Vấn đề**
- Khi tạo 1 file, sẽ thay đổi 1 phần filesystem trên disk.
- Directory structure được chỉnh sửa, FCBs được cấp phát, data block cấp phát, bộ đếm trên block tăng hoặc giảm.
- Đây là 1 chuỗi các hoạt động => có thể dẫn đến lỗi nếu quá trình xử lý bị ảnh hưởng.
- 1 lỗi phổ biến là tính năng caching của OS để nâng cao hiệu năng IO. 1 số thay đổi tới disk, trong khi 1 số được cache. Nếu cache chưa được đưa xuống disk mà hệ thống lỗi => xảy ra lỗi về data đang xử lý.
- 1 số vấn đề bên cạnh như thực thi FS, disk controller, user app cũng có thể gây lỗi FS.

## II. Các phương pháp khôi phục
**Consistency Checking**
- Khi xảy ra lỗi, Filesystem cần tìm kiếm lỗi và sửa. Để phát hiện,sử dụng phương pháp scan toàn bộ metadata trên mỗi file và kiếm tra tính nhất quán hệ thống. Quá trình diễn ra rất lâu và xảy ra khi hệ thống khởi động lại.
- Phương pháp bên cạnh, Filesystem có thể lưu trạng thái trong Filesystem metadata. Khi metadata thay đổi, các trạng thái metadata sẽ đánh dấu.
- **Consistency checker** – chương trình hệ thống như fsck (Unix) sẽ đối chiếu data trong Directory structure với data block trên disk, cố gắng sửa các mâu thuẫn tìm thấy.
- **allocation và free-spacemanagement algorithms** sẽ xác định loại lỗi mà check tìm thấy, sử dụng phương pháp tương ứng để chỉnh sửa. Đối với HDD có thể gây lỗi cao Unix sẽ sử dụng cơ chế đồng bộ, tránh việc lỗi ko thể sửa.

**Log-Structured File Systems**
- Giải quyết 1 số vấn đề consistency checking đang gặp phải, áp dụng log-based transaction-oriented (or journaling) file systems.
- Giải pháp tập trung vào kỹ thuật log-based recovery khi filesysytem metadata update.

Tổng quát, tất cả metadata thay đổi được ghi tuần tự tới log. Mỗi tập hoạt động được thực hiện 1 nhiệm vụ xác đinh = transaction. Khi thay đổi được ghi vào log, chúng sẽ được cân nhắc để commited, system call có thể trả lại user process, cho phép nó tiếp tục thực hiện => Log entry sẽ lưu lại hoạt động thực sự trên filesystem structure.

Khi thực hiện thay đổi, con trỏ sẽ update hành động đã hoàn thành, hoặc chưa hoàn thành. Khi hoạt động transaction hoàn thành => quá trình ghi log sẽ được xóa khỏi log file (1 bộ đệm xử lý). circular buffer ghi tới vị trí đánh dấu lưu trữ, ghi đè vào giá trị cũ nếu có. Log có thể chia thành các phần trong FS hoặc trên đĩa.