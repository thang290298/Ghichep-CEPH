<h1 align="center">Tổng quan về SSD</h1>

## I. Giới thiệu

- **`SSD`** (Solid State Drive) là một loại phương tiện lưu trữ dữ liệu liên tục trên bộ nhớ flash trạng thái rắn. Hai thành phần chính tạo nên một ổ SSD: bộ điều khiển flash và chip nhớ flash NAND.

- Ổ cứng thể rắn hoặc Ổ lưu trữ bán dẫn là một thiết bị lưu trữ sử dụng bộ nhớ flash để lưu trữ dữ liệu trên máy tính một cách bền vững.
- Một ổ SSD đồng thời mô phỏng quá trình lưu trữ và truy cập dữ liệu giống như ổ đĩa cứng (HDD) thông thường và do đó dễ dàng được sử dụng cho nhiều mục đích khác nhau.
- Ổ SSD sử dụng SRAM hoặc DRAM hoặc bộ nhớ Flash để lưu dữ liệu, không nên nhầm lẫn với RAM Disk là một công nghệ mô phỏng và lưu dữ liệu trên RAM.

- Ổ cứng SSD không chỉ cải thiện về tốc độ đọc ghi dữ liệu so với đĩa cứng HDD. Ngoài ra SSD hỗ trợ người dùng cải thiện nhiệt độ, an toàn dữ liệu và điện năng tiêu thụ.


## II. Cấu tạo ổ cứng SSD

- SSD được xây dựng lên từ nhiều chip nhớ flash NOR và bộ nhớ NAND flash. SSD được làm hoàn toàn bằng linh kiện điện tử và không có bộ phận chuyển động vật lý như trong ổ đĩa cứng. Những con chip flash sẽ được lắp cố định trên một bo mạch chủ khoảng từ 10-60 NAND của hệ thống. Trên card PCI hoặc cũng có thể là lắp vào trong một chiếc hộp có hình dạng và kích thước giống như ổ cứng nhưng nhỏ hơn.

<h3 align="center"><img src="../../03-Images/document/32.png"></h3>

VD về ổ cứng samsung
<h3 align="center"><img src="../../03-Images/document/33.png"></h3>

Ổ cứng của samsung (Samsung MDX S4LN021X01-8030) trong hình có cấu tạo gồm:
- 8 chipset của PCI Express 2.0 kết nối bộ điều khiển với các thiết bị flash NAND
- 1 bộ điều khiển SSD. Bộ vi xử lý bên trong bộ điều khiển có tác dụng lấy dữ liệu đến và thao tác với nó. Bộ điều khiển loại bỏ bất kỳ lỗi nào, đảm bảo nó được ánh xạ chính xác. Đưa nó vào đèn flash hoặc lấy ra từ đèn flash
- 1 RAM module (DRAM DDR3): bộ nhớ ram sử dụng chuẩn giao tiếp DDR3.
- 64 MLC(Multi-Level Cell) các transistor có thể lưu trữ 2 bit. Module NAND flash trên + 32 kênh, mỗi mô-đun cung cấp 32 GB dung lượng lưu trữ (Micron 31C12NQ314 25nm). Một tế bào NAND flash đơn có thể lưu trữ một hoặc hai bit dữ liệu tương ứng MLC hoặc SLC. Các thiết bị MLC Flash có giá thành rẻ hơn dung lượng lưu trữ dữ liệu lớn hơn. Tổng bộ nhớ là 2048 GB, nhưng chỉ có 1,4 TB có sẵn sau khi qua trích lập dự phòng.