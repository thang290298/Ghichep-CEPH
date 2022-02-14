<h1 align="center">Kết nối Client với Ceph Cluster</h1>

# Phần I. Chuẩn bị

## 1. Mô hình triển khai

<h3 align="center"><img src="..\03-Images\Lab\22.png"></h3>

Trong đó:
  - Ba node CEPH Luminous: CentOS 7 - 64bit.
  - 1 Node Client: CentOS 7 - 64bit.
  - NICs:
    - `eth0`: SSH và cài đặt.
    - `eth1`: Kết nối thông tin giữa các node CEPH, đường cho clients kết nối.
    - `eth2`: Đồng bộ dữ liệu giữa các node CEPH.


## 2. IP Planning

| Hostname | hardware | Interface |
|--------------|-------|------|
| ceph-luminous1-admin | <ul><li>2 CPU - 2GB RAM</li><li>20GB Disk OS</li><li>40GB Disk DATA</li></ul>| <ul><li>eth0: 192.168.33.18  (MNGT)</li><li>eth1: 192.168.34.18 (CEPH_COM)</li><li>eth2: 192.168.35.18 (CEPH_REP)</li></ul>|
| ceph-luminous2 | <ul><li>2 CPU - 2GB RAM</li><li>20GB Disk OS</li><li>40GB Disk DATA</li></ul>| <ul><li>eth0: 192.168.33.19  (MNGT)</li><li>eth1: 192.168.34.19 (CEPH_COM)</li><li>eth2: 192.168.35.19 (CEPH_REP)</li></ul>|
| ceph-luminous3 |  <ul><li>2 CPU - 2GB RAM</li><li>20GB Disk OS</li><li>40GB Disk DATA</li></ul>| <ul><li>eth0: 192.168.33.20  (MNGT)</li><li>eth1: 192.168.34.20 (CEPH_COM)</li><li>eth2: 192.168.35.20 (CEPH_REP)</li></ul>|
| ceph-client |  <ul><li>2 CPU - 2GB RAM</li><li>20GB Disk OS</li></ul>| <ul><li>eth0: 192.168.33.21  (MNGT)</li><li>eth1: 192.168.34.21 (CEPH_COM)</li></ul>|


# Phần II. Triển khai
## 1. Tính toán chỉ số replicate, PG tạo một pool image

**Đứng trên node CEPH để thực hiện**
- Truy cập trang tính toán tự động số PG dựa trên thông tin hệ thống đã có.
```sh
https://linuxkidd.com/ceph/pgcalc.html
```

<h3 align="center"><img src="..\03-Images\Lab\23.png"></h3>

- Chọn `Generate Commands` để download file vừa tạo

<h3 align="center"><img src="..\03-Images\Lab\24.png"></h3>

- Mở file vừa tạo ra để lấy các lệnh chạy trên node CEPH:

<h3 align="center"><img src="..\03-Images\Lab\25.png"></h3>

- Kết quả:
```sh
[root@ceph-luminous1-admin ceph-deploy]# ceph osd pool create newPool 128
pool 'newPool' created
[root@ceph-luminous1-admin ceph-deploy]# ceph osd pool set newPool size 2
set pool 1 size to 2
[root@ceph-luminous1-admin ceph-deploy]# while [ $(ceph -s | grep creating -c) -gt 0 ]; do echo -n .;sleep 1; done
........[root@ceph-luminous1-admin ceph-deploy]#
```
<h3 align="center"><img src="..\03-Images\Lab\26.png"></h3>
<h3 align="center"><img src="..\03-Images\Lab\27.png"></h3>

## 2. Tạo Image VM (Dùng cho vm) từ CEPH server

- Cú pháp:
```sh
rbd create {pool-name}/{images} --size {size}

rbd info {pool-name}/{images}
```

- Ví dụ: Thực tạo 1 volume có dung lượng 20GB trên pool 'newPool' vừa khởi tạo
```sh
[root@ceph-luminous1-admin ceph-deploy]# rbd create newPool/vol_client --size 20G
[root@ceph-luminous1-admin ceph-deploy]# rbd info newPool/vol_client
rbd image 'vol_client':
        size 20GiB in 5120 objects
        order 22 (4MiB objects)
        block_name_prefix: rbd_data.37246b8b4567
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        flags: 
        create_timestamp: Sun Feb 13 22:14:44 2022
[root@ceph-luminous1-admin ceph-deploy]# 
```

## 3. Cài đặt CEPH client trên client linux
- Trên client cài đặt ceph client
```sh
yum install ceph-common -y
```
- Trên client download `ceph.conf` và `key` về `/etc/ceph/`

```
scp root@192.168.33.18:/etc/ceph/ceph.conf /etc/ceph/
scp root@192.168.33.18:/etc/ceph/{key-name}.keyring /etc/ceph/
```

`{key-name}`: Check ở ceph server
<h3 align="center"><img src="..\03-Images\Lab\28.png"></h3>

Kết quả:

<h3 align="center"><img src="..\03-Images\Lab\29.png"></h3>

- Add `config` vào `rbdmap` trên `ceph client`

**Lưu ý:** Trường hợp trong `/etc/ceph/` đã có sẵn file cấu hình `rbdmap` thì cần move sang `rbdmap.bak` trước khi copy cấu hình vào.

```
mv /etc/ceph/rbdmap /etc/ceph/rbdmap.bak
```

```
echo "{pool-name}/{images}            id=admin,keyring=/etc/ceph/ceph.client.admin.keyring" >> /etc/ceph/rbdmap
```

```
echo "newPool/vol_client        id=admin,keyring=/etc/ceph/ceph.client.admin.keyring" >> /etc/ceph/rbdmap
```


- Kiểm tra lại cấu hình:

```
sudo modprobe rbd
rbd feature disable {pool-name}/{images}  exclusive-lock object-map fast-diff deep-flatten
systemctl start rbdmap && systemctl enable rbdmap
```

```
sudo modprobe rbd
rbd feature disable newPool/vol_client exclusive-lock object-map fast-diff deep-flatten
systemctl start rbdmap && systemctl enable rbdmap
```

- Trên client xuất hiện phân cùng `rbd0` phân phối tự ceph server xuống có dung lượng 20GB.

<h3 align="center"><img src="..\03-Images\Lab\30.png"></h3>


## 4. Mount, resize và kiểm tra phân vùng
### Chia phân vùng disk

```sh
fdisk /dev/rbd0
```
<h3 align="center"><img src="..\03-Images\Lab\31.png"></h3>

### ĐỊnh dạng và mount phân vùng disk

```sh
mkfs.ext4 /dev/rbd0p1
mkdir /data
mount /dev/rbd0p1 /data/
```

<h3 align="center"><img src="..\03-Images\Lab\32.png"></h3>

- Sửa trong `fstab` để khi khởi động lại VM thì phân vùng sẽ tự động khởi động theo:
```sh
[root@localhost ceph]# blkid /dev/rbd0p1
/dev/rbd0p1: UUID="7f6e24f9-ad4f-423d-bfd4-b9d37bfdd592" TYPE="ext4" 
[root@localhost ceph]# echo UUID="7f6e24f9-ad4f-423d-bfd4-b9d37bfdd592 /data ext4 noauto 0 0" >> /etc/fstab
[root@localhost ceph]#
```
### Extend volume

**Thực hiện trên node CEPH**

```
rbd resize --size 2048 {pool-name}/{images} (to decrease)
rbd resize --size 2048 {pool-name}/{images} --allow-shrink (to increase)
```

Ví dụ resize `newPool/vol_client` lên 30GB

```
rbd resize --size 30G newPool/vol_client --allow-shrink
```

Kết quả trên client:

<h3 align="center"><img src="..\03-Images\Lab\33.png"></h3>

- Phân vùng đã nhận đủ dung lượng khi tăng lên, nhưng dung lượng thực tế được sử dụng vẫn chưa đủ. Ta phải extend phía client lên.


#### Resize lại phân vùng trên Client


- Cài đặt công cụ `parted`:

```
yum install parted -y
```

- Hiển hị thông tin phần vùng:

<h3 align="center"><img src="..\03-Images\Lab\34.png"></h3>


- Sử dụng các lệnh sau để resize lại phân vùng:

```
yum -y install cloud-utils-growpart
growpart /dev/rbd0 1
resize2fs /dev/rbd0p1
```

- Kiểm tra dữ liệu bên trong phân vùng, thực hiện tạo 1 file có dung lượng 26GB

<h3 align="center"><img src="..\03-Images\Lab\35.png"></h3>


# Tài liệu tham khảo
- https://github.com/quanganh1996111/ceph/blob/main/ceph/thuc-hanh/docs/3-client-connect-ceph.md
- https://github.com/domanhduy/ghichep/blob/master/DuyDM/CEPH/thuc-hanh/docs/3.client-linux-connect-ceph.md