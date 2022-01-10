<h1 align="center">Cache Volume</h1>

# Phân I. Giới thiệu
- Loại cache logical volume sử dụng LV nhỏ nằm trong phân vùng có tốc độ truy vấn nhanh (SSD), nâng cao hiệu năng LV nằm trên phân vùng lớn, chậm hơn (Hdd).
- Thực hiện phương pháp bằng cách lưu các block thường xuyên được sử dụng trên LV có tốc độ truy cập nhanh.

- Do yêu cầu từ dm-cache (kernel driver), LVM chia cache pool LV thành 2 phần:
  - Cache data LV
  - Cache metadata LV

- Ý tưởng:
  - Tạo 1 Cache LV, 1 Cache meta LV trên SSD.
  - Sử dụng 2 phân vùng trên tạo cache pool logical volume, thêm Cache pool vào LV có tốc độ truy vấn chậm

- Thuật ngữ
  - `Cache data logical volume` - logical volume chứa data blocks của cache pool logical volume
  - `Cache metadata logical volume` - logical volume chứa metadata của cache pool logical volume, giữ thông tin nơi mà các khối dữ liệu được lưu trữ (for example, on the origin logical volume or the cache data logical volume).
  - `Cache logical volume` - logical volume chứa origin logical volume và cache pool logical volume. Đây là thiết bị sử dụng đóng gói các cache volume component khác nhau.


# Phần II. Cấu hình
- Kiểm tra cấu hình hiện tại
```sh
lvs
vgs
pvs
```
```sh
[root@masterkvm7101 ~]# lvs
  LV   VG     Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  VPS1 VG1    -wi-ao----    1.09t
  root centos -wi-ao---- <214.00g
[root@masterkvm7101 ~]# vgs
  VG     #PV #LV #SN Attr   VSize    VFree
  VG1      3   1   0 wz--n-    1.09t    0
  centos   1   1   0 wz--n- <214.00g    0
[root@masterkvm7101 ~]# pvs
  PV         VG     Fmt  Attr PSize    PFree
  /dev/sda3  centos lvm2 a--  <214.00g    0
  /dev/sdb1  VG1    lvm2 a--  <223.00g    0
  /dev/sdc1  VG1    lvm2 a--   446.62g    0
  /dev/sdd1  VG1    lvm2 a--   446.62g    0
[root@masterkvm7101 ~]#
```