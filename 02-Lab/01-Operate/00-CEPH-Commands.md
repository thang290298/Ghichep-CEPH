<h1 align="center">Một số câu lệnh sử dụng trong vận hành CEPH</h1>

## 1. Ceph-deploy

**`Cài đặt Ceph trên client`**
```sh
ceph-deploy install {Client}
```
  - ví dụ:
```sh
ceph-deploy install --release luminous  ceph-luminous1-admin ceph-luminous2 ceph-luminous3
```

**`Khởi tạo cụm`**

```sh
ceph-deloy mon create-initial
```

**`Coppy key admin và config`**
```sh
ceph-deploy --overwrite-conf admin {host}
```

  - ví dụ:

```sh
ceph-deploy --overwrite-conf admin ceph-luminous2
```

## 2. Tạo mới OSD

**`Zapdisk`**
```sh
ceph-deploy disk zap  {host} /dev/{disk}
```
  - ví dụ:

```sh
ceph-deploy osd create --data /dev/vdb ceph-luminous1-admin
```

**`Tạo mới node Mon`**

```sh
ceph-deploy mon create hostname1 hostname2 hostname3...
```
  - Ví dụ:
```sh
ceph-deploy mon create  ceph-luminous2 
```

**`Xóa Node Mon`**

```sh
ceph-deploy mon destroy hostname1 hostname2 hostname3...
```
  - Ví dụ: 
```sh
ceph-deploy mon destroy  ceph-luminous2 
```

**`Tạo mới node MGR`**

```sh
ceph-deploy mgr create hostname1
```
  - Ví dụ:
```sh
ceph-deploy mgr create ceph-luminous2
```

**`Đẩy config đến các node client`**
```sh
ceph-deploy --overwrite-conf config push client1
```

**`Tạo mới node Rados Gateway`**

```sh
ceph-deploy rgw create host1
```


## 2. Kiểm tra Service CEPH
### 1. MON

- Tiến trình giám sát ( Ceph Monitor - CEPH MON)

Đây là thành phần tập trung và trạng thái toàn bộ cluster, giám sát trạng thái OSD, MON, PG, Crush map. Các cluster sẽ thực hiện giám sát, chia sẻ thông tin về những thay đổi và không lưu trữ dũ liệu

```sh
systemctl start ceph-mon@$(hostname)
systemctl stop ceph-mon@$(hostname)
systemctl restart ceph-mon@$(hostname)
systemctl status ceph-mon@$(hostname)
```
  -  Ví dụ:
```sh
[root@ceph-luminous1-admin ~]# systemctl status ceph-mon@ceph-luminous1-admin
● ceph-mon@ceph-luminous1-admin.service - Ceph cluster monitor daemon
   Loaded: loaded (/usr/lib/systemd/system/ceph-mon@.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-02-14 12:05:45 +07; 3 days ago
 Main PID: 979 (ceph-mon)
   CGroup: /system.slice/system-ceph\x2dmon.slice/ceph-mon@ceph-luminous1-admin.service
           └─979 /usr/bin/ceph-mon -f --cluster ceph --id ceph-luminous1-admin --setuser ceph --setgroup ceph

Feb 14 12:05:45 ceph-luminous1-admin systemd[1]: Started Ceph cluster monitor daemon.
Feb 15 03:39:01 ceph-luminous1-admin ceph-mon[979]: 2022-02-15 03:39:01.725956 7f3346928700 -1 received  signal: Hangup from  PID: 2133 task name: killall -q -1 ceph-mon ceph-mgr ceph-mds ceph-osd ceph-fuse radosgw  UID: 0
Feb 15 03:39:01 ceph-luminous1-admin ceph-mon[979]: 2022-02-15 03:39:01.734415 7f3346928700 -1 received  signal: Hangup from  PID: 2134 task name: killall -q -1 ceph-mon ceph-mgr ceph-mds ceph-osd ceph-fuse radosgw rbd-mirror  UID: 0
Feb 16 03:25:01 ceph-luminous1-admin ceph-mon[979]: 2022-02-16 03:25:01.840985 7f3346928700 -1 received  signal: Hangup from  PID: 5078 task name: killall -q -1 ceph-mon ceph-mgr ceph-mds ceph-osd ceph-fuse radosgw  UID: 0
Feb 16 03:25:01 ceph-luminous1-admin ceph-mon[979]: 2022-02-16 03:25:01.850131 7f3346928700 -1 received  signal: Hangup from  PID: 5079 task name: killall -q -1 ceph-mon ceph-mgr ceph-mds ceph-osd ceph-fuse radosgw rbd-mirror  UID: 0
Feb 17 03:36:02 ceph-luminous1-admin ceph-mon[979]: 2022-02-17 03:36:02.083027 7f3346928700 -1 received  signal: Hangup from  PID: 5771 task name: killall -q -1 ceph-mon ceph-mgr ceph-mds ceph-osd ceph-fuse radosgw  UID: 0
Feb 17 03:36:02 ceph-luminous1-admin ceph-mon[979]: 2022-02-17 03:36:02.089839 7f3346928700 -1 received  signal: Hangup from  PID: 5772 task name: killall -q -1 ceph-mon ceph-mgr ceph-mds ceph-osd ceph-fuse radosgw rbd-mirror  UID: 0
[root@ceph-luminous1-admin ~]#
```

### 2. OSD

- Đa phần CEPH cluster đượch thực hiện bởi tiến trình Ceph OSD. CEPH OSD lưu tất cả dữ liệu người dùng ở dạng object. CEPH Cluster bao gồm nhiều OSD. CEPH có cơ chế phan phôi object vào các OSD khác nhua đảm bảo tính toàn vẹn và sẵn sàn của dữ liệu
- OSD trên node nào thì đứng trên node đó để kiểm tra trạng thái của OSD đó

**`Xác định ID của OSD`** 

```sh
ceph osd tree
```
<h3 align="center"><img src="..\..\03-Images\Lab\36.png"></h3>

```sh
systemctl stop ceph-osd@{osd-id}
systemctl start ceph-osd@{osd-id}
systemctl restart ceph-osd@{osd-id}
systemctl status ceph-osd@{osd-id}
```

<h3 align="center"><img src="..\..\03-Images\Lab\37.png"></h3>

### 3. MSD

- CEPH MDS tập tring vào Metadata Server và yêu cầu riêng cho CEPHFS, một số storage methods block và object-based storage không yêu cầu MDS services.

```sh
systemctl restart ceph-mds@$(hostname)
```
  - ví dụ:
```sh
[root@ceph-luminous1-admin ~]# systemctl status ceph-mds@ceph-luminous1-admin
● ceph-mds@ceph-luminous1-admin.service - Ceph metadata server daemon
   Loaded: loaded (/usr/lib/systemd/system/ceph-mds@.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
[root@ceph-luminous1-admin ~]#
```

### 4. Rados Gateway (RGW)
- Ceph phân phối và cung cấp object storage interface thông qua Ceph object gateway hay còn gọi là RADOS

```sh
systemctl status ceph-radosgw@rgw.$(hostname).service
```

  - ví dụ
```sh
[root@ceph-luminous1-admin ~]# systemctl status ceph-radosgw@rgw.ceph-luminous1-admin.service
● ceph-radosgw@rgw.ceph-luminous1-admin.service - Ceph rados gateway
   Loaded: loaded (/usr/lib/systemd/system/ceph-radosgw@.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
[root@ceph-luminous1-admin ~]#
```

### 5. MGR 
- MGR chạy song song với monitor daemons để thực hiện cung cấp và giao diện bổ sung cho hệ thống giám sát và quản lý từ bên ngoài

```sh
systemctl start ceph-mgr@$(hostname)
systemctl restart ceph-mgr@$(hostname)
systemctl status ceph-mgr@$(hostname)
```

ví dụ:
```sh
[root@ceph-luminous1-admin ~]# systemctl status ceph-mgr@ceph-luminous1-admin
● ceph-mgr@ceph-luminous1-admin.service - Ceph cluster manager daemon
   Loaded: loaded (/usr/lib/systemd/system/ceph-mgr@.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2022-02-15 15:40:59 +07; 2 days ago
 Main PID: 4303 (ceph-mgr)
   CGroup: /system.slice/system-ceph\x2dmgr.slice/ceph-mgr@ceph-luminous1-admin.service
           └─4303 /usr/bin/ceph-mgr -f --cluster ceph --id ceph-luminous1-admin --setuser ceph --setgroup ceph

Feb 17 17:26:35 ceph-luminous1-admin ceph-mgr[4303]: ::ffff:192.168.33.133 - - [17/Feb/2022:17:26:35] "GET /toplevel_data HTTP/1.1" 200 163 "http://ceph-luminous1-admin:7000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64...dg/98.0.1108.50"Feb 17 17:26:41 ceph-luminous1-admin ceph-mgr[4303]: ::ffff:192.168.33.133 - - [17/Feb/2022:17:26:41] "GET /toplevel_data HTTP/1.1" 200 163 "http://ceph-luminous1-admin:7000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64...dg/98.0.1108.50"Feb 17 17:26:47 ceph-luminous1-admin ceph-mgr[4303]: ::ffff:192.168.33.133 - - [17/Feb/2022:17:26:47] "GET /toplevel_data HTTP/1.1" 200 163 "http://ceph-luminous1-admin:7000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64...dg/98.0.1108.50"Feb 17 17:26:53 ceph-luminous1-admin ceph-mgr[4303]: ::ffff:192.168.33.133 - - [17/Feb/2022:17:26:53] "GET /toplevel_data HTTP/1.1" 200 163 "http://ceph-luminous1-admin:7000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64...dg/98.0.1108.50"Feb 17 17:26:59 ceph-luminous1-admin ceph-mgr[4303]: ::ffff:192.168.33.133 - - [17/Feb/2022:17:26:59] "GET /toplevel_data HTTP/1.1" 200 163 "http://ceph-luminous1-admin:7000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64...dg/98.0.1108.50"Feb 17 17:27:05 ceph-luminous1-admin ceph-mgr[4303]: ::ffff:192.168.33.133 - - [17/Feb/2022:17:27:05] "GET /toplevel_data HTTP/1.1" 200 163 "http://ceph-luminous1-admin:7000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64...dg/98.0.1108.50"Feb 17 17:27:11 ceph-luminous1-admin ceph-mgr[4303]: ::ffff:192.168.33.133 - - [17/Feb/2022:17:27:11] "GET /toplevel_data HTTP/1.1" 200 163 "http://ceph-luminous1-admin:7000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64...dg/98.0.1108.50"Feb 17 17:27:17 ceph-luminous1-admin ceph-mgr[4303]: ::ffff:192.168.33.133 - - [17/Feb/2022:17:27:17] "GET /toplevel_data HTTP/1.1" 200 163 "http://ceph-luminous1-admin:7000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64...dg/98.0.1108.50"Feb 17 17:27:23 ceph-luminous1-admin ceph-mgr[4303]: ::ffff:192.168.33.133 - - [17/Feb/2022:17:27:23] "GET /toplevel_data HTTP/1.1" 200 163 "http://ceph-luminous1-admin:7000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64...dg/98.0.1108.50"Feb 17 17:27:29 ceph-luminous1-admin ceph-mgr[4303]: ::ffff:192.168.33.133 - - [17/Feb/2022:17:27:29] "GET /toplevel_data HTTP/1.1" 200 163 "http://ceph-luminous1-admin:7000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64...dg/98.0.1108.50"Hint: Some lines were ellipsized, use -l to show in full.
[root@ceph-luminous1-admin ~]#
```


## 3. Kiểm tra trạng thái CEPH

- Hiện thị trạng thái Cụm CEPH
```sh
ceph -s
ceph health
```

<h3 align="center"><img src="..\..\03-Images\Lab\38.png"></h3>

- Hiển thị trạng thái cum CEPH theo thời gian thực 
```sh
ceph -w
```
- Kiểm tra trạng thái sử dụng disk của mỗi pool
```sh
ceph df
```

- Kiểm tra trạng thái sử dụng disk của mỗi pool theo Object
```sh
rados df
```

## 3. Lệnh MGR service

- Kiểm tra thông tin các module của MGR
```sh
ceph mgr dump
```


<h3 align="center"><img src="..\..\03-Images\Lab\39.png"></h3>

- Enable cấc module MGR
```sh
ceph mgr module enable {module}
ceph mgr module enable zabbix
```

<h3 align="center"><img src="..\..\03-Images\Lab\40.png"></h3>


## 4. Lệnh thao tác với OSD
- Kiểm tra OSD được tạo từ phân vùng ổ LVM nào trên host
```sh
ceph-volume lvm list
```

- OSD nào thì kiểm tra trên node đó.

<h3 align="center"><img src="..\..\03-Images\Lab\41.png"></h3>

- Hiển trị trạn thái các OSD trong cụm.
```sh
ceph osd stat
```
<h3 align="center"><img src="..\..\03-Images\Lab\42.png"></h3>

- Hiển thị tình trạng used, r/w, state của các OSD

<h3 align="center"><img src="..\..\03-Images\Lab\43.png"></h3>

- Hiển thị Crushmap OSD

```sh
ceph osd tree
ceph osd crush tree
ceph osd crush tree --show-shadow
```
<h3 align="center"><img src="..\..\03-Images\Lab\45.png"></h3>

- Kiểm tra chi tiết location của 1 OSD
```sh
ceph osd find {osd-id}
```

ví dụ:
```sh
ceph osd find 1
```

<h3 align="center"><img src="..\..\03-Images\Lab\46.png"></h3>

- Kiểm tra chi tiết metadata của 1 OSD
```sh
ceph osd metadata {osd-id}
```
ví dụ:
```sh
ceph osd metadata 0
```

<h3 align="center"><img src="..\..\03-Images\Lab\47.png"></h3>


- Benchmark OSD
```sh
ceph tell osd.{osd-id} bench
```

vi dụ:
```sh
ceph tell osd.1 bench
```

<h3 align="center"><img src="..\..\03-Images\Lab\48.png"></h3>


- Hiển thị trạng thái sử dụng của các OSD

```sh
ceph osd df 
ceph osd df tree
```

<h3 align="center"><img src="..\..\03-Images\Lab\49.png"></h3>


- Hiển thị latency Aplly, Commit data trên các OSD

```sh
ceph osd perf
```
<h3 align="center"><img src="..\..\03-Images\Lab\50.png"></h3>

- Xóa 1 osd ra khỏi cụm Ceph (Thực hiện trên node của OSD đó)
```sh
i={osd-id}
ceph osd out osd.$i
ceph osd down osd.$i
systemctl stop ceph-osd@$i
ceph osd crush rm osd.$i
ceph osd rm osd.$i
ceph auth del osd.$i
```


## 5. Các lệnh thao tác trên pool
- Tạo pool (phải tính toán đúng tham số về replicate, PG)
```sh
ceph osd pool create {pool-name} {pg-num} [{pgp-num}] [replicated] \
     [crush-ruleset-name] [expected-num-objects]
```
- Enable aplication pool
```sh
osd pool application enable {pool-name} {application}
osd pool application enable newPool rbd
```

- Hiển thị toàn bộ tham số của các pool
```sh
ceph osd pool ls detail
```
<h3 align="center"><img src="..\..\03-Images\Lab\51.png"></h3>

- Hiện thị tham số của 1 pool

```sh
ceph osd pool get {pool-name} all
```
Ví dụ:
<h3 align="center"><img src="..\..\03-Images\Lab\52.png"></h3>


- Điều chỉnh lại giá trị của pool
```sh
ceph osd pool set {pool-name} {key} {value}
```

- Xóa pool
```sh
ceph osd pool delete {pool-name}
```
## 6. Thao tác đối với RBD
- Hiển thị các images trong pool
```sh

rbd ls {pool-name}
```

<h3 align="center"><img src="..\..\03-Images\Lab\53.png"></h3>

- Create 1 images

```sh
rbd create {pool-name}/{images} --size {size}
```
ví dụ:
```sh
rbd create newPool/vol_client1 --size 15G
```
<h3 align="center"><img src="..\..\03-Images\Lab\54.png"></h3>

- Hiển thị chi tiết Images
```sh
rbd info {pool-name}/{images}
```
<h3 align="center"><img src="..\..\03-Images\Lab\55.png"></h3>


- Hiển thị dung lượng thực tế của images

```sh
rbd diff {pool-name}/{images} | awk '{SUM += $2} END {print SUM/1024/1024 " MB"}'
```
ví dụ:
```sh
[root@ceph-luminous1-admin ~]# rbd diff newPool/vol_client  | awk '{SUM += $2} END {print SUM/1024/1024 " MB"}'
26917.7 MB
[root@ceph-luminous1-admin ~]#
```

- Hiển thị images đang được mapped (trên Client)
```sh
rbd showmapped
```

Ví dụ:
```sh
[root@localhost ~]# rbd showmapped
id pool    image      snap device    
0  newPool vol_client -    /dev/rbd0 
[root@localhost ~]# 
```
- Xóa Images
```sh
rbd rm {pool-name}/{images}
```
Ví dụ:
```sh
[root@ceph-luminous1-admin ~]# rbd rm newPool/vol_client1
Removing image: 100% complete...done.
[root@ceph-luminous1-admin ~]#
```
- Create snapshot
```sh
rbd snap create {pool-name}/{images}@{snap-name}
```
Ví dụ: 
```sh
rbd snap create newPool/vol_client@vol_client_snap1
```

- Kiểm tra tất cả các bản snapshot của 1 volume

```sh
rbd snap ls {pool-name}/{images}
```
Ví dụ:
```sh
[root@ceph-luminous1-admin ~]# rbd snap ls newPool/vol_client
SNAPID NAME              SIZE TIMESTAMP                
     4 vol_client_snap1 30GiB Thu Feb 24 11:04:03 2022 
[root@ceph-luminous1-admin ~]#
```
- Protect bản snapshot
```sh
rbd snap protect {pool-name}/{images}@{snap-name}
```
Ví dụ:
```sh
rbd snap protect newPool/vol_client@vol_client_snap1
```

- Roolback snapshot (Cho images trở về tại thời điểm snapshot)
```sh
rbd snap rollback {pool-name}/{images}@{snap-name}
```

- Clone snapshot thành 1 images mới
```sh
rbd clone {pool-name}/{images}@{snap-name} {pool-name}/{images-clone}
```

Ví dụ: 
```sh
rbd clone newPool/vol_client@vol_client_snap1 newPool/client-clone
```

- Kiểm tra các images được clone từ snapshot
```sh
rbd children {pool-name}/{images}@{snap-name}
```
Ví dụ:
```sh
[root@ceph-luminous1-admin ~]# rbd children newPool/vol_client@vol_client_snap1
newPool/client-clone
[root@ceph-luminous1-admin ~]#
```

- Tách hẳn images mới ra khỏi images parent
```sh
rbd flatten {pool-name}/{images-clone}
```
Ví dụ: 
```sh
[root@ceph-luminous1-admin ~]# rbd flatten newPool/client-clone
Image flatten: 48% complete...2022-02-24 11:27:33.355706 7ff2967fc700 -1 librbd::ImageWatcher: 0x7ff28c01eae0 image watch failed: 140679707755184, (107) Transport endpoint is not connected
2022-02-24 11:27:33.360870 7ff2967fc700 -1 librbd::Watcher: 0x7ff28c01eae0 handle_error: handle=140679707755184: (107) Transport endpoint is not connected
Image flatten: 100% complete...done.
[root@ceph-luminous1-admin ~]#
```
> lưu ý: trong quá trình tách hệ thống sẽ chuyển từ trạng thái `HEALTH_OK` sang `HEALTH_WARN` cảu OSD và PG
<h3 align="center"><img src="..\..\03-Images\Lab\56.png"></h3>


- Unprotect bản snapshot

```sh
rbd snap unprotect {pool-name}/{images}@{snap-name}
```
- Xóa 1 bản snapshot


```sh
rbd snap rm {pool-name}/{images}@{snap-name}
```
ví dụ:
```sh
[root@ceph-luminous1-admin ~]# rbd snap rm newPool/vol_client@vol_client_snap1
Removing snap: 100% complete...done.
[root@ceph-luminous1-admin ~]#
```

- Xóa toàn bộ snapshot của 1 volume
```sh
rbd snap purge {pool-name}/{images}
```

- Export volume
```sh
rbd export --rbd-concurrent-management-ops 20 --pool={pool-name} {images} {images}.img
```
## 7. Các lệnh thao tác đối với Object


- Show toàn bộ pool name

```sh
rados lspools
```

- Show toàn bộ Object trên cụm

```sh
rados -p {pool-name} ls
```

- Upload Object lên cụm Ceph

```sh
rados -p {pool-name} put {object-file}
```

- Download Object từ cụm Ceph

```sh
rados -p {pool-name} get {object-file}
```

- Xóa 1 Object cụm Ceph

```sh
rados -p {pool-name} rm {object-file}
```

- Kiểm tra các client kết nối đến Object
sh
```
rados -p {pool-name} listwatchers {object-file}
```

- Benchmark Object bằng rados bench

```sh
rados -p {pool-name} listwatchers {object-file}
```

## 8. Các lệnh thao tác xác thực trên Ceph

- Hiển thị toàn bộ các key authen của cụm Ceph

```sh
ceph auth list
```

- Create hoặc get key

```sh
ceph auth get-or-create {key-name} mon {permission} osd {permission} mds {permission} > {key-name}.keyring
```

- Cập nhật permission key đã có

```sh
ceph auth caps {key-name} mon {permission} osd {permission} mds {permission}
```

- Xóa key

```sh
ceph auth delete {key-name}
```


# Tài liệu Tham Khảo

- https://github.com/quanganh1996111/ceph/blob/main/ceph/thuc-hanh/docs/4-ceph-commands.md