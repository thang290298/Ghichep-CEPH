<h1 align="center">Cài đặt CEPH phiên bản Luminous</h1>


# Phần I. Chuẩn bị

## 1. Mô hình triển khai

<h3 align="center"><img src="..\03-Images\Lab\12.png"></h3>

Trong đó:
  - Ba node CEPH: CentOS 7 - 64bit.
  - Disk: Mỗi node CEPH có 2 Disk. 1 Disk cài OS, 1 Disk lưu trữ dữ liệu cho Client.
  - NICs:
    - `eth0`: SSH và cài đặt.
    - `eth1`: Kết nối thông tin giữa các node CEPH, đường cho clients kết nối.
    - `eth2`: Đồng bộ dữ liệu giữa các node CEPH.
  - Phiên bản cài đặt: CEPH Luminous.


## 2. IP Planning

| Hostname | hardware | Interface |
|--------------|-------|------|
| ceph-luminous1-admin | <ul><li>2 CPU - 2GB RAM</li><li>20GB Disk OS</li><li>40GB Disk DATA</li></ul>| <ul><li>eth0: 192.168.33.18  (MNGT)</li><li>eth1: 192.168.34.18 (CEPH_COM)</li><li>eth2: 192.168.35.18 (CEPH_REP)</li></ul>|
| ceph-luminous2 | <ul><li>2 CPU - 2GB RAM</li><li>20GB Disk OS</li><li>40GB Disk DATA</li></ul>| <ul><li>eth0: 192.168.33.19  (MNGT)</li><li>eth1: 192.168.34.19 (CEPH_COM)</li><li>eth2: 192.168.35.19 (CEPH_REP)</li></ul>|
| ceph-luminous3 |  <ul><li>2 CPU - 2GB RAM</li><li>20GB Disk OS</li><li>40GB Disk DATA</li></ul>| <ul><li>eth0: 192.168.33.20  (MNGT)</li><li>eth1: 192.168.34.20 (CEPH_COM)</li><li>eth2: 192.168.35.20 (CEPH_REP)</li></ul>|

# Phần II. Cài đặt
## 1. Thiết lập ban đầu
### Bước 1: Tắt firewall, Selinux
```sh
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

### Bước 2: Cài đặt Epel repository và Update các gói cài đặt
```sh
yum install epel-release -y
yum update -y
```
### Bước 3: Cài đặt NTP
```sh
yum install chrony -y 

systemctl start chronyd 
systemctl enable chronyd
systemctl restart chronyd 

chronyc sources -v

sudo date -s "$(wget -qSO- --max-redirect=0 google.com 2>&1 | grep Date: | cut -d' ' -f5-8)Z"
ln -f -s /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime

```

### Bước 4: Set hostname
```sh
cat << EOF >> /etc/hosts
192.168.34.18 ceph-luminous1-admin
192.168.34.19 ceph-luminous2
192.168.34.20 ceph-luminous3
EOF
```

### Bước 5: Cài đặt CMDlog
```sh
curl -Lso- https://raw.githubusercontent.com/thang290298/CMD-Log/main/cmdlog.sh | bash
```
### Bước 6: Khởi động lại máy để load được cấu hình selinux
```sh
init 6
```

## 2. Cài đặt CEPH
### Cài đặt ceph-deploy
```sh
yum install -y wget
wget https://download.ceph.com/rpm-nautilus/el7/noarch/ceph-deploy-2.0.1-0.noarch.rpm --no-check-certificate
rpm -ivh ceph-deploy-2.0.1-0.noarch.rpm
```

### Cài đặt python-setuptools để ceph-deploy có thể hoạt động ổn định.
```sh
curl https://bootstrap.pypa.io/ez_setup.py | python
```

### Kiểm tra cài đặt
```sh
ceph-deploy --version
```
- Kết quả trả về:
```sh
[root@ceph-luminous1-admin ~]# ceph-deploy --version
2.0.1
[root@ceph-luminous1-admin ~]#
```
### Tạo key SSH
```sh
ssh-keygen
```
- Kết quả: 
```sh
[root@ceph-luminous1-admin ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Ka+QFfhW3d3c11iSNcuT3ZiQMnru2MGMRB/oGTTdP78 root@ceph-luminous1-admin
The key's randomart image is:
+---[RSA 2048]----+
|       .oo ....+o|
|     .  +o=.oooOO|
|    . .o.=.+..==O|
|     . o=.o   o o|
|      =.S*     o |
|     + o. =     .|
|    o   .+ .    .|
|     . .. o    E |
|      .          |
+----[SHA256]-----+
[root@ceph-luminous1-admin ~]#
```
- thực hiện `coppy` key sang các node khác.
```sh
ssh-copy-id root@ceph-luminous1-admin
ssh-copy-id root@ceph-luminous2
ssh-copy-id root@ceph-luminous3
```

### Tạo các thư mục ceph-deploy để thao tác cài đặt vận hành cluster
```sh
mkdir /ceph-deploy && cd /ceph-deploy
```
### Khởi tại file cấu hình cho cụm với node quản lý là `ceph-luminous1-admin`
```sh
ceph-deploy new ceph-luminous1-admin
```
- Kiểm tra thông tin folder `ceph-deploy`
```sh
[root@ceph-luminous1-admin ceph-deploy]# ll -alh
total 20K
drwxr-xr-x   2 root root 4.0K Feb 10 09:08 .
dr-xr-xr-x. 19 root root 4.0K Feb 10 09:07 ..
-rw-r--r--   1 root root  211 Feb 10 09:08 ceph.conf
-rw-r--r--   1 root root 3.1K Feb 10 09:08 ceph-deploy-ceph.log
-rw-------   1 root root   73 Feb 10 09:08 ceph.mon.keyring
[root@ceph-luminous1-admin ceph-deploy]#
```
- trong thư mục bao gồm các file:
  - **`ceph.conf`**: File config được khởi tạo tự động
  - **`ceph-deploy-ceph.log`**: File log ghi chép toàn bộ các thao tác sử dụng ceph-deploy
  - **`ceph.mon.keyring`**: đây là file sinh ra tự động được sử dụng ddeer khởi tạo Cluster

- Thực hiện bổ sung cấu hình network vào bên trong file: **`ceph.conf`**, bao gồm:
  - `Public network`: Đường trao đổi thông tin giữa các node Ceph và cũng là đường client kết nối vào.
  - `Cluster network`: Đường đồng bộ dữ liệu.
  - nội dung như sau:
```sh
cat << EOF >> /ceph-deploy/ceph.conf
osd pool default size = 2
osd pool default min size = 1ini
osd crush chooseleaf type = 0
osd pool default pg num = 128
osd pool default pgp num = 128

public network = 192.168.34.0/24
cluster network = 192.168.35.0/24
EOF
```
### Cài đặt ceph trên toàn bộ các node ceph
> Lưu ý: Nên sử dụng byobu, tmux, screen để cài đặt tránh hiện tượng mất kết nối khi đang cài đặt CEPH.

```sh
yum -y install byobu
```
 - tiến hành cài đặt trên các node
```sh
ceph-deploy install --release luminous  ceph-luminous1-admin ceph-luminous2 ceph-luminous3
```

### kiểm tra phiên bản sau khi cài đặt
```sh
root@ceph-luminous1-admin ceph-deploy]# ceph -v
ceph version 12.2.13 (584a20eb0237c657dc0567da126be145106aa47e) luminous (stable)
[root@ceph-luminous1-admin ceph-deploy]#
```

### Khởi tạo cluster với các node mon (Monitor-quản lý) dựa trên file ceph.conf

```sh
ceph-deploy mon create-initial
```

Sau khi thực hiện lệnh phía trên sẽ sinh thêm ra 05 file trong thư mục ceph-deploy:
  - ceph.bootstrap-mds.keyring
  - ceph.bootstrap-mgr.keyring
  - ceph.bootstrap-osd.keyring
  - ceph.bootstrap-rgw.keyring
  - ceph.client.admin.keyring
```sh
root@ceph-luminous1-admin ceph-deploy]# ll
total 312
-rw------- 1 root root     71 Feb 10 11:33 ceph.bootstrap-mds.keyring
-rw------- 1 root root     71 Feb 10 11:33 ceph.bootstrap-mgr.keyring
-rw------- 1 root root     71 Feb 10 11:33 ceph.bootstrap-osd.keyring
-rw------- 1 root root     71 Feb 10 11:33 ceph.bootstrap-rgw.keyring
-rw------- 1 root root     63 Feb 10 11:33 ceph.client.admin.keyring
-rw-r--r-- 1 root root    278 Feb 10 11:28 ceph.conf
-rw-r--r-- 1 root root 284596 Feb 10 11:33 ceph-deploy-ceph.log
-rw------- 1 root root     73 Feb 10 11:21 ceph.mon.keyring
[root@ceph-luminous1-admin ceph-deploy]#
```
 - Để Node ceph-luminous1-admin có thể tương tác được với Cluster chúng ta cần node ceph-luminous1-adminvới quyền admin bằng cách bố sung `admin.keying` cho node

 ```sh
 ceph-deploy admin ceph-luminous1-admin
 ```
<h3 align="center"><img src="..\03-Images\Lab\13.png"></h3>

### Cài đặt ceph-mgr trên ceph-luminous1-admin

- `ceph-mgr` là thành phần cài đặt cần được khởi tạo từ bản luminous, có thể đặt trên nhiều node và hoạt động theo cơ chế Active-Passive.
- Tiến hành cài đặt ceph-mgr
```sh
ceph-deploy mgr create ceph-luminous1-admin
```

<h3 align="center"><img src="..\03-Images\Lab\14.png"></h3>

- Kiểm tra:
```sh
ceph -s
```

<h3 align="center"><img src="..\03-Images\Lab\15.png"></h3>

## 4. Khởi tạo OSD
### Tạo OSD thông qua ceph-deploy tại host ceph-luminous1-admin

- Tại node CEPH01, dùng `ceph-deploy`, để partition ổ cứng OSD, thay ceph01 bằng hostname của host chứa OSD.
```sh
ceph-deploy disk zap ceph-luminous1-admin /dev/vdb
```
<h3 align="center"><img src="..\03-Images\Lab\16.png"></h3>

- Tạo OSD với `ceph-deploy`

```sh
ceph-deploy osd create --data /dev/vdb ceph-luminous1-admin
```

- Kiểm tra osd vừa khởi tạo
```sh
ceph osd tree
```

<h3 align="center"><img src="..\03-Images\Lab\17.png"></h3>


### thực hiện khởi tạo OSD tương tự đối các node còn lại
- kết quả sau khi khơi tạo osd trên 3 node ( không thực hiện với disk OS)
```sh
ceph -s
```

<h3 align="center"><img src="..\03-Images\Lab\18.png"></h3>
<h3 align="center"><img src="..\03-Images\Lab\19.png"></h3>
<h3 align="center"><img src="..\03-Images\Lab\20.png"></h3>


### khởi tạo dashboard 
- Ceph-mgr hỗ trợ dashboard để quan sát trạng thái của cluster, Enable mgr dashboard trên host ceph-luminous1-admin

```sh
ceph mgr module enable dashboard
ceph mgr services
```
- kiểm tra:
  - http://ceph-luminous1-admin:7000/
<h3 align="center"><img src="..\03-Images\Lab\21.png"></h3>


