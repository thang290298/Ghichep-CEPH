<h1 align="center">LVM Snapshot</h1>

# I. Giới thiệu

-  LVM Snapshot là tính năng cho phép backup phân vùng ổ cứng được chọn tức thời mà không làm gián đoạn các dịch vụ đang chạy.

- Những thay đổi được thực hiện sau khi Snapshort thì tính năng snapshort sẽ tạo ra 1 bản backup và cho phép khôi phục khi cần thiết

# II. Cách tạo Snapshot

- Kiểm tra cấu hỉnh

```sh
vgs
lvs
pvs
```
<h3 align="center"><img src="../../03-Images/document/65.png"></h3>

- Tạo Logical Volumes

```sh
root@node1-ctl:~# lvcreate -L 1G -n lv-lab LVM_volume1
  Logical volume "lv-lab" created.
root@node1-ctl:~#
```

- Tạo Snapshot cho Logical Volumes vừa tạo
```sh
root@node1-ctl:~# lvcreate -L 1GB -s -n lv-lab-snapshot /dev/LVM_volume1/lv-lab
  Logical volume "lv-lab-snapshot" created.
root@node1-ctl:~#
```
trong đó:
-  Cú pháp: lvcreate -L [size] -s -n [tên Snapshot] [Logical Volumes cần Snapshot]
- `-s`: Tạo Snapshot
- `-n`: Tên bản snapshot

Kiểm tra :
<h3 align="center"><img src="../../03-Images/document/66.png"></h3>

- Xóa Snapshot
```sh
root@node1-ctl:~# lvremove /dev/LVM_volume1/lv-lab-snapshot
Do you really want to remove and DISCARD active logical volume LVM_volume1/lv-lab-snapshot? [y/n]: y
  Logical volume "lv-lab-snapshot" successfully removed
root@node1-ctl:~#
```

Kiểm tra :
<h3 align="center"><img src="../../03-Images/document/67.png"></h3>


# III. Kiểm tra hoạt động Snapshot


- cấu hình định dạng và mount thư mục OS
```sh
mkfs -t ext4 /dev/LVM_volume1/lv-lab
mkdir /lvmlab
mount /dev/LVM_volume1/lv-lab /lvmlab
```
- thêm dữ liệu vào Logical Volumes bằng cách tạo file có dung lượng
```sh
dd if=/dev/mapper/LVM_volume1-lv--lab of=upload_test bs=2G count=1
```

- kiểm tra 

<h3 align="center"><img src="../../03-Images/document/68.png"></h3>


```sh
root@node1-ctl:/lvmlab# lvs
  LV              VG          Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv-lab          LVM_volume1 owi-aos---  1.00g
  lv-lab-snapshot LVM_volume1 swi-a-s---  1.00g      lv-lab 9.67
  lv-volume1      LVM_volume1 -wi-a----- 40.00g
root@node1-ctl:/lvmlab# pvs
```
> khi dữ liệu thay đổi , dung lượn file snapshot sẽ thay đổi theo

- Nếu LVM bị đầy, snapshot sẽ tự động xóa


# IV. Mở rộng Snapshot LVM

- Nếu cần mở rộng kích thước Snapshot thực hiện lệnh sau:
```sh
lvextend -L +1G /dev/LVM_volume1/lv-lab-snapshot
```

<h3 align="center"><img src="../../03-Images/document/69.png"></h3>

> Lưu ý: Kích thước Snaphort không thể lơn hơn kích thước logical volume chính. Tròn trường hợp này phải tăng kích thướng logical volume sau đó tiến hành tăng kích thước snapshot


- Khi mà Logical volume đang được gắn và kích hoạt cần bỏ kích hoạt sau đó tiến hành tăng kích thước bằng lệnh `lvchange` và tái kích hoạt sau khi nâng cấp thành công
- ví dụ:
```sh
umount /dev/LVM_volume1/lv-lab
lvchange -an /dev/LVM_volume1/lv-lab
lvextend -L +3G /dev/LVM_volume1/lv-lab
lvchange -ay /dev/LVM_volume1/lv-lab
mount /dev/LVM_volume1/lv-lab /lvmlab
resize2fs /dev/LVM_volume1/lv-lab
lvextend -L +1G /dev/LVM_volume1/lv-lab-snapshot
```
  - kết quả:
```sh
root@node1-ctl:/lvmlab# lvs
  LV              VG          Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv-lab          LVM_volume1 owi-aos--- 4.00g
  lv-lab-snapshot LVM_volume1 swi-a-s--- 2.00g      lv-lab 49.31
root@node1-ctl:/lvmlab#
```
```sh
root@node1-ctl:/lvmlab# lvdisplay /dev/LVM_volume1/lv-lab-snapshot
  --- Logical volume ---
  LV Path                /dev/LVM_volume1/lv-lab-snapshot
  LV Name                lv-lab-snapshot
  VG Name                LVM_volume1
  LV UUID                JPTrqK-Rema-VbQh-XbPF-7xYz-gohX-Get3MF
  LV Write Access        read/write
  LV Creation host, time node1-ctl, 2022-01-12 02:39:00 +0000
  LV snapshot status     active destination for lv-lab
  LV Status              available
  # open                 0
  LV Size                4.00 GiB
  Current LE             1024
  COW-table size         2.00 GiB
  COW-table LE           512
  Allocated to snapshot  49.31%
  Snapshot chunk size    4.00 KiB
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:3

root@node1-ctl:/lvmlab#
```

# V. Khôi phục Snapshot LVM

- Bỏ mount logical volume và thư mục
```sh
umount /dev/LVM_volume1/lv-lab
```
kiểm tra:
<h3 align="center"><img src="../../03-Images/document/70.png"></h3>


- Khôi phục snapshot
```sh
lvconvert --merge /dev/LVM_volume1/lv-lab-snapshot
```
```sh
root@node1-ctl:~# lvconvert --merge /dev/LVM_volume1/lv-lab-snapshot
  Merging of volume LVM_volume1/lv-lab-snapshot started.
  LVM_volume1/lv-lab: Merged: 51.72%
  LVM_volume1/lv-lab: Merged: 100.00%
root@node1-ctl:~#
```

- Sau khi thực hiện khôi phục file snapshot sẽ bị xóa 



# Tài liệu Tham khảo

- https://github.com/lacoski/khoa-luan/blob/master/LVM/lvm-snapshot.md
- https://www.tecmint.com/take-snapshot-of-logical-volume-and-restore-in-lvm/
- https://documentation.suse.com/sles/12-SP4/html/SLES-all/cha-lvm-snapshots.html