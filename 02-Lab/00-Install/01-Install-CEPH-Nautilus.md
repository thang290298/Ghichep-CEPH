<h1 align="center">Cài đặt CEPH phiên bản nautilus</h1>

# Phần I. Chuẩn bị

## 1. Mô hình triển khai

<h3 align="center"><img src="..\..\03-Images\Lab\1.png"></h3>

Trong đó:
  - Ba node CEPH: CentOS 7 - 64bit.
  - Disk: Mỗi node CEPH có 4 Disk. 1 Disk cài OS, 3 Disk lưu trữ dữ liệu cho Client.
  - NICs:
    - `eth0`: SSH và cài đặt.
    - `eth1`: Kết nối thông tin giữa các node CEPH, đường cho clients kết nối.
    - `eth2`: Đồng bộ dữ liệu giữa các node CEPH.
  - Phiên bản cài đặt: CEPH Nautilus.

## 2. IP Planning

<h3 align="center"><img src="..\..\03-Images\Lab\2.png"></h3>


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
echo "192.168.34.15 CEPH01" >> /etc/hosts
echo "192.168.34.16 CEPH02" >> /etc/hosts
echo "192.168.34.17 CEPH03" >> /etc/hosts
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
[root@CEPH01 ~]# ceph-deploy --version
2.0.1
[root@CEPH01 ~]#
```
### Tạo key SSH
```sh
ssh-keygen
```
- Kết quả: 
```sh
[root@CEPH01 ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:ZKvszQ3vIJQLH2380vzrgrBFo/jahAl4VeI1dgZDnyM root@CEPH01
The key's randomart image is:
+---[RSA 2048]----+
|    ..\..O.o        |
|   . = * .       |
|    o E *        |
| . .   Boo       |
|. o ..\..+oS.       |
| . ..\..Bo=.+       |
|    o.B++.+      |
|     +o+.*..\..     |
|    ..\..o o.+o+.   |
+----[SHA256]-----+
[root@CEPH01 ~]#
```

- thực hiện `coppy` key sang các node khác.
```sh
ssh-copy-id root@CEPH01
ssh-copy-id root@CEPH02
ssh-copy-id root@CEPH03
```

### Tạo các thư mục ceph-deploy để thao tác cài đặt vận hành cluster
```sh
mkdir /ceph-deploy && cd /ceph-deploy
```

### Khởi tại file cấu hình cho cụm với node quản lý là ceph01
```sh
ceph-deploy new ceph01
```
- Kiểm tra thông tin folder `ceph-deploy`
```sh
[root@CEPH01 ceph-deploy]# ll -alh
total 20K
drwxr-xr-x   2 root root 4.0K Dec 15 15:12 .
dr-xr-xr-x. 19 root root 4.0K Dec 15 15:12 ..\..
-rw-r--r--   1 root root  197 Dec 15 15:12 ceph.conf
-rw-r--r--   1 root root 2.9K Dec 15 15:12 ceph-deploy-ceph.log
-rw-------   1 root root   73 Dec 15 15:12 ceph.mon.keyring
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
osd pool default min size = 1
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
ceph-deploy install --release nautilus CEPH01 CEPH02 CEPH03 
```

> Quá trình cài đặt diễn tra trong khoảng 30 đến 60 phút để hoàn tất cài đặt trên 3 node CEPH

### kiểm tra phiên bản sau khi cài đặt
```sh
[root@CEPH01 ceph-deploy]# ceph -v
ceph version 14.2.22 (ca74598065096e6fcbd8433c8779a2be0c889351) nautilus (stable)
[root@CEPH01 ceph-deploy]#
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
[root@CEPH01 ceph-deploy]# ll
total 292
-rw------- 1 root root    113 Dec 15 15:36 ceph.bootstrap-mds.keyring
-rw------- 1 root root    113 Dec 15 15:36 ceph.bootstrap-mgr.keyring
-rw------- 1 root root    113 Dec 15 15:36 ceph.bootstrap-osd.keyring
-rw------- 1 root root    113 Dec 15 15:36 ceph.bootstrap-rgw.keyring
-rw------- 1 root root    151 Dec 15 15:36 ceph.client.admin.keyring
-rw-r--r-- 1 root root    197 Dec 15 15:12 ceph.conf
-rw-r--r-- 1 root root 264732 Dec 15 15:36 ceph-deploy-ceph.log
-rw------- 1 root root     73 Dec 15 15:12 ceph.mon.keyring
[root@CEPH01 ceph-deploy]#
```

## 3. Khởi tạo MGR

- **`Ceph-mgr`** là thành phần cài đặt cần khởi tạo từ bản nautilus, có thể cài đặt trên nhiều node hoạt động theo cơ chế `Active-Passive`.

### Cài đặt ceph-mgr trên CEPH01

- Copy thư file `/ceph-deploy/ceph.client.admin.keyring` sang thư mục `/etc/ceph/`:
```sh
cp /ceph-deploy/ceph.client.admin.keyring /etc/ceph/
```
- Tiến hành cài đặt ceph-mgr
```sh
ceph-deploy mgr create CEPH01
```

<h3 align="center"><img src="..\..\03-Images\Lab\3.png"></h3>

- Kiểm tra:
```sh
ceph -s
```

<h3 align="center"><img src="..\..\03-Images\Lab\4.png"></h3>

## 4. Khởi tạo OSD
### Tạo OSD thông qua ceph-deploy tại host CEPH01

- Tại node CEPH01, dùng `ceph-deploy`, để partition ổ cứng OSD, thay ceph01 bằng hostname của host chứa OSD.
```sh
ceph-deploy disk zap CEPH01 /dev/vdb
```
<h3 align="center"><img src="..\..\03-Images\Lab\5.png"></h3>

- Tạo OSD với `ceph-deploy`

```sh
ceph-deploy osd create --data /dev/vdb CEPH01
```

- Kiểm tra osd vừa khởi tạo
```sh
ceph osd tree
```
<h3 align="center"><img src="..\..\03-Images\Lab\6.png"></h3>

<h3 align="center"><img src="..\..\03-Images\Lab\7.png"></h3>




### thực hiện khởi tạo OSD tương tự đối các node còn lại
- kết quả sau khi khơi tạo osd trên 3 node ( không thực hiện với disk OS)
```sh
ceph -s
```

<h3 align="center"><img src="..\..\03-Images\Lab\8.png"></h3>
<h3 align="center"><img src="..\..\03-Images\Lab\9.png"></h3>
<h3 align="center"><img src="..\..\03-Images\Lab\10.png"></h3>

### khởi tạo dashboard 
- Ceph-mgr hỗ trợ dashboard để quan sát trạng thái của cluster, Enable mgr dashboard trên host ceph01

- Tạo file mật khẩu : `/ceph-deploy/passwd.txt`chứa thông tin mật khẩu mật khẩu có nội dung: Abc@123abc
- thực hiện các lệnh:
```sh
yum install ceph-mgr-dashboard -y
ceph mgr module enable dashboard
ceph dashboard create-self-signed-cert
ceph dashboard set-login-credentials admin -i /ceph-deploy/passwd.txt
ceph mgr services
```
trong đó:
  - admin: tên user
  - /ceph-deploy/passwd.txt : file password đã khởi tạo trước đó

>Lưu ý cài đặt yum install ceph-mgr-dashboard -y trên cả ceph01 và ceph02
- Truy cập vào mgr dashboard với username và password vừa đặt ở phía trên để kiểm tra
```sh
https://CEPH01:8443/
```
- Kết quả: 

<h3 align="center"><img src="..\..\03-Images\Lab\11.png"></h3>

# Tài liệu Tham khảo

- https://github.com/uncelvel/tutorial-ceph/blob/master/docs/setup/ceph-nautilus.md

- https://github.com/quanganh1996111/ceph/blob/main/ceph/thuc-hanh/docs/1-install-ceph-nautilus.md