<h1 align="center">Tổng quan về filesystem</h1>

# Phần I. Giới thiệu
- Trong máy tình, file system hoặc filesystem được sử dụng để kiểm soát dữ liệu, lưu trữ và lấy lại. Quản lý vị trí ghi file, giúp xác định vị trí điểm bắt đầu và kêt thúc. File system được bắt nguồn từ hệ thống lưu trữ trên giấy, nhóm các dữ liệu được gọi là “file”. Cấu trúc và luật logic được sử dụng để quản lý các nhóm thống tin và chúng được gọi là `file system`.

- Có nhiều loại file system, đối với mỗi loài system sẽ có đắc điểm khách nhau về tính chất về tốc độ, tính linh hoạt, bảo mật, size...1 số file system được thiết kể để sử dụng cho 1 số ứng dụng đặc biệt
  - ví dụ:
    - ISO 9660 file system được thiết đặc biệt cho đĩa quang
- File system có thể sử dụng nhiều trên loại thiết bị lưu trữ và các loại phương tiên truyền thống khác nhau, nổi bật và thông thường nhất là ổ đĩa cứng. Một số file system được sử dụng cho local data storage devices, bên cạnh đó cung cấp truy chế truy cập file thông qua giao thức mạng (NFS, SMB, 9P). Một số file system là ảo, có nghĩa cung cấp “file” ảo sử dụng cho những yêu cấu tính toán hoặc ánh xạ vào nhưng file system khác. File system quản lý truy cập trên cả nội dung của file và metadata của nhưng file này. Nó chịu trách nhiệp sắp xếp không gian lưu trữ đảm bảo, tin cậy, rõ ràng, có hệ thống.

# Phần II. Kiến trúc

- File system gồm 2 hoặc 3 lớp tùy trường hợp được phân chia rõ ràng, có lúc lại kết hợp với nhau
  - Lớp thứ nhất – “logical file system”: “logical file system” chịu trách nhiệm tương tác với ứng dụng người dùng. Nó cung cấp API cho các hoạt động cơ bản – Mở, đóng, đọc, .. và truyền các yêu cầu xuống lớp dưới cho việc xử lý. “Logical file system” quản lý hoạt động mở đối tượng “file table” và “per-process file descriptors”. Lớp này cung cấp “file access, directory operations, và security and protection”
  - Lớp thứ 2 (không bắt buộc) - virtual file system. Là lớp giao diện cho phép hỗ trợ đồng thời nhiều loại file system vật lý, còn được gọi là thực thi file system
  - Lớp thứ 3 - physical file system. Đây là lớp liên quan đến hoạt động vật lý của thiết bị lưu trữ (disk). Nó xử lý các khối vật lý cho việc đọc hoặc ghi. Nó xử lý các buffer, memory management và chịu trách nhiệm bố trí các khối vật lý trong những ví trí được chỉ định. Physical file system tương tác với device drivers hoặc các kênh tới thiết bị lưu trữ.
