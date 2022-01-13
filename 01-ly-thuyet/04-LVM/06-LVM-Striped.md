<h1 align="center">Striped logical volumes</h1>

# Phần I. Giới thiệu Striped logical volumes

- `Striped`: là 1 tính năng của LVM quy định cách ghi dữ liệu lên các Physical Volume để tạo nên logical volume. Striped dùng để thay thế cho cách ghi dữ liệu mắc định lần lượt lên các Physical Volume.

- Ví dụ:
- KHi có 1 logical Volume được hình thành từ 3 ổ đĩa là `/dev/sdb`, `/dev/sdc` và `/dev/sdd`. Khi thực hiện ghi dữ liệu thì logical Volume sẽ ghi dữ liệu theo trình tự `/dev/sdb` đầy sau đó chuyển sang ghi dữ liệu vào `/dev/sdc` và tiếp tục ghi dữ liệu vào  `/dev/sdd` nếu `/dev/sdc` đầy dữ liệu. 
- KHi sử dụng LVM Stripe dữ liệu được ghi vào logical volume sẽ được ghi đều lên các Physical Volume


- Vai trò của Striped:
  - Tăng hiệu năng ghi dữ liệu lên ổ cứng
  - Tiết kiệm không gian lưu trữ dữ liệu


# Phần II. Khởi tạo Logical Volume với LVM Striped

- Cấu hình cơ bản: 
```sh
root@node1-ctl:~# pvs
  PV         VG Fmt  Attr PSize   PFree
  /dev/sdb1     lvm2 ---  <30.00g <30.00g
  /dev/sdc1     lvm2 ---  <30.00g <30.00g
  /dev/sdd1     lvm2 ---  <30.00g <30.00g
root@node1-ctl:~#
```

- Khởi tạo Volume Group trên các Physical Logical
```sh
vgcreate -s 32M VG_Pool /dev/sdb1 /dev/sdc1 /dev/sdd1
```
Trong đó:
  - `-s 32M`: Dùng để khai báo kích thước của `PE Size` là 32Mb.
  - `VG_Pool`: Tên của Volume Group
  - `/dev/sdb /dev/sdc /dev/sdd`: là các Physical Logical được sử dụng

- Kiểm tra:
```SH
root@node1-ctl:~# vgcreate -s 32M VG_Pool /dev/sdb1 /dev/sdc1 /dev/sdd1
  Physical volume "/dev/sdb1" successfully created.
  Physical volume "/dev/sdc1" successfully created.
  Physical volume "/dev/sdd1" successfully created.
  Volume group "VG_Pool" successfully created
root@node1-ctl:~# vgs
  VG      #PV #LV #SN Attr   VSize   VFree
  VG_Pool   3   0   0 wz--n- <89.91g <89.91g
root@node1-ctl:~#
```
- KHởi tạo Thin Pool trên 3 Physical Logical
```sh
lvcreate -L 35G --thinpool data_pool1 -i 3 VG_Pool
```

- Khởi tạo Logical Volume từ thin Pool
```sh
lvcreate -V 5G --thin -n data1 VG_Pool/data_pool1
```

- Khởi tạo Logical Volume từ `VG_Pool` có tên là `lv_data` có mức dung lượng `15G`
```sh
root@node1-ctl:~# lvcreate -L 500 -n lv_data -i 3 VG_Pool
  Using default stripesize 64.00 KiB.
  Rounding up size to full physical extent 512.00 MiB
  Rounding size 512.00 MiB (16 extents) up to stripe boundary size 576.00 MiB(18 extents).
  Logical volume "lv_data" created.
root@node1-ctl:~#
```

- Kết quả: 
```sh
root@node1-ctl:~# lvdisplay VG_Pool/lv_data -m
  --- Logical volume ---
  LV Path                /dev/VG_Pool/lv_data
  LV Name                lv_data
  VG Name                VG_Pool
  LV UUID                yRVoiz-dxAK-DZwZ-DIPj-I21Q-Bub1-GC2Df7
  LV Write Access        read/write
  LV Creation host, time node1-ctl, 2022-01-13 08:56:03 +0000
  LV Status              available
  # open                 0
  LV Size                576.00 MiB
  Current LE             18
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     768
  Block device           253:3

  --- Segments ---
  Logical extents 0 to 17:
    Type                striped
    Stripes             3
    Stripe size         64.00 KiB
    Stripe 0:
      Physical volume   /dev/sdb1
      Physical extents  376 to 381
    Stripe 1:
      Physical volume   /dev/sdc1
      Physical extents  374 to 379
    Stripe 2:
      Physical volume   /dev/sdd1
      Physical extents  376 to 381


root@node1-ctl:~#
```

> Stripes 3: thể hiện số lượng ổ cứng để sử dụng để tạo logical volume là 3


# Phần III. Kiểm tra kết quả


- Kiểm tra vấn đề sử dụng ổ cứng để tạo Logical Volume sử dụng stripes như với câu lệnh:

```sh
root@node1-ctl:~# pvs
  PV         VG      Fmt  Attr PSize   PFree
  /dev/sdb1  VG_Pool lvm2 a--  <29.97g 18.03g
  /dev/sdc1  VG_Pool lvm2 a--  <29.97g 18.09g
  /dev/sdd1  VG_Pool lvm2 a--  <29.97g 18.03g
root@node1-ctl:~#
```

- Ta thấy , mỗi  Physical Volume đã sử dụng 11.94GB để tạo ra các Group Volume ở chế độ Striped. Sau đây sẽ tạo volume group khong sử dụng Striped để kiểm tra:

  - tạo volume group dung lượn 5GB
```sh
lvcreate -L 5G -n lv_data1 VG_Pool
```
  - kiểm tra 
```sh
root@node1-ctl:~# pvs
  PV         VG      Fmt  Attr PSize   PFree
  /dev/sdb1  VG_Pool lvm2 a--  <29.97g 13.03g
  /dev/sdc1  VG_Pool lvm2 a--  <29.97g 18.09g
  /dev/sdd1  VG_Pool lvm2 a--  <29.97g 18.03g
root@node1-ctl:~#
```
  - Ta thấy dung lượng trống Physical Volume /dev/sdb1 đã 5GB để tạo ra logical volume còn các Physical Volume /dev/sdc1 /dev/sdd1 


  