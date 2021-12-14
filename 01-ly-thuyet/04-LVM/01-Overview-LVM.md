<h1 align="center">Tổng quan về LVMI</h1>

# I. Giới thiệu

- **`LVM`** là một phương pháp cho phép ấn định không gian đĩa cứng thành những Logical Volume => khiến cho việc thay đổi kích thước trở nên dễ dàng (so với partition)

- Có thể thay đổi kích thước má không cần thay đổi partition table của OS. Hữu ích khi sử dụng hết bộ nhớ trống và muốn mở rộng.

> Chỉ cần thực hiện ấn định lại dung lượng mà không cần  phải phân vùng lại.

# II. Vai trò của LVM
- LVM hỗ trợ quản lý thay đổi kích thước lưu trữ ổ cứng

- **Mục tiêu**:
  - Không để hệ thống bị gián đoạn khi đang hoạt động
  - Không làm lỗi dịch vụ đang chạy
  - Có thể kết hợp Hot Swapping (thao tác thay thế nóng các thành phần bên trong máy tính)

# III. Một số thuật ngữ trong LVM

## 3.1 Physical volumes (PV):
- Đĩa cứng vật lý trong server
- Có thể kết hợp nhiều PV để tạo thành một Volume Groups với dung lượng bằng tổng dung lượng các PV
- PV chỉ là đại diện cho các ổ đĩa vật lý chứ không phải là bản thân ổ đĩa đó, vì vậy để cần phải tạo PV từ các dev đã mount.

## 3.2 Volume Groups (VG)
<h3 align="center"><img src="../../03-Images/document/55.png"></h3>

- Một tập hợp các PV, từ VG sẽ có thể phân chia thành các Logical Volumes và các Logical Volumes này có thể thay đổi kích thước dễ dàng.


## 3.3 Logical Volumes (LV)

<h3 align="center"><img src="../../03-Images/document/56.png"></h3>

- Đơn vị cuối cùng của hệ thống LVM, các LV tương đương với partition theo cách phân chia truyền thống
- LV có thể thay đổi kích thước dễ dàng, tất cả chỉ phụ thuộc vào kích thước của VG.


## 3.4 File Systems (FS)

- Tổ chức và kiểm soát các tập tin
- Được lưu trữ trên ổ đĩa cho phép truy cập nhanh chóng và an toàn
- Sắp xếp dữ liệu trên đĩa cứng máy tính
- Quản lý vị trí vật lý của mọi thành phần dữ liệu

## 3.5 Physical Drive

- Thiết bị lưu trữ dữ liệu, ví dụ như trong linux nó là /dev/sda

## 3.6 Partition
- Partitions là các phân vùng của Physical Drive, mỗi Physical Drive có 4 partition, trong đó partition bao gồm 2 loại chính là primary partition và extended partition.


## 3.7 Mô hình tổng quan
<h3 align="center"><img src="../../03-Images/document/57.png"></h3>


# IV. Ưu nhược điểm
## 1. Ưu điểm
- Tất cả disk coi như 1
- logical volumes có thể nằm trên nhiều disk
- Tạo các LV, linh động trong việc tăng, giảm
- Thay đổi kích thước LV khi mong muốn, không phụ thuộc vào vị trí LV.
- Resize/create/delete logical trực tiếp, physical volumes online
- Trực tiếp thay đổi trên LV đang sử dụng mà không phải restart
- Cung cấp tính năng snapshots
- Hỗ trợ nhiều loại device-mapper target,

## 2. Nhược điểm
- Các bước thiết lập phức tạp
- Không thể truy cập LVM từ windows (không hỗ trợ)