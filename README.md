# Ceph Ansible deploy

> thực hiện trên node Ansible
## 1. Cài đặt cephadm trên các node ceph
### 1.1. Khởi tạo ssh key và coppy đến các node ceph
```sh
ssh-keygen
ssh-copy-id 192.168.70.61
ssh-copy-id 192.168.70.62
ssh-copy-id 192.168.70.63
```
### 1.2. Triển khai cài đặt Cephadm trên các node ceph
```sh
ansible-playbook -i inventory/ceph-lab-dbblock cephadm-setup.yml -e@extra_vars/customise_cephlab.yml
```
### 1.3. Khởi tạo các file deploy *.yml*
```sh
ansible-playbook -i inventory/ceph-lab-dbblock gen_deploy_yaml.yml -e@extra_vars/customise_cephlab.yml
```
> thực hiện trên node Ceph admin
## 2. Deploy Ceph với Cephadm
### 2.1. Tiển khai cài đặt môi trường bootstrap
```sh
cephadm bootstrap --mon-ip 192.168.70.61 --cluster-network 10.10.10.0/24 --initial-dashboard-user admin --initial-dashboard-password "passwd123" --config ceph.conf  --skip-monitoring-stack
```
- Trong đó: 
  - `--mon-ip`: là IP ext node admin quản trị
  - `--cluster-network`: thông tin dải ip ext
  - `--initial-dashboard-user`: thông tin user tài khoản truy cập  dashboard
  - `--initial-dashboard-password`: Thông tin mật khẩu truy cập 

### 2.2. Khởi tạo Key SSH và gửi key đến các node trong hệ thống
```sh
ssh-keygen
ssh-copy-id ceph01-quincy
ssh-copy-id ceph02-quincy
ssh-copy-id ceph03-quincy
ssh-copy-id -f -i /etc/ceph/ceph.pub root@ceph02-quincy
ssh-copy-id -f -i /etc/ceph/ceph.pub root@ceph03-quincy
```
### 2.3. Add host service và osd
```sh
ceph orch apply -i hosts.yaml
ceph orch apply -i mons.yaml
ceph orch apply -i mgrs.yaml
ceph orch apply -i osds.yaml
```


## 3. Quy hoạch tài nguyên cho Ceph cluster
### 3.1. Khởi tạo crush rule cho Ceph cluster:
```sh
ansible-playbook -i inventory/ceph-lab-dbblock add-crushrules.yml -e@extra_vars/customise_cephlab.yml
```
### 3.1.2. Khởi tạo các key cho Ceph cluster
```sh
ansible-playbook -i inventory/ceph-lab-dbblock add-key.yml -e@extra_vars/customise_cephlab.yml
```
### 3.1.3. Khởi tạo các pools cho Ceph cluster
```sh
ansible-playbook -i inventory/ceph-lab-dbblock add-pools.yml -e@extra_vars/customise_cephlab.yml
```
-------------
# Thông tin về Playbook
## 1. Thư mục `inventory`:
Trong thư mục chứa file khai báo thông tin các node ceph cluster và được phân nhóm theo sevices tương ứng đối với node ceph
```sh
  - admin: chứa host đóng vai trò quản trị ceph cluster, triển khai ceph sử dụng cephadm
  - mons: chứa các host triển khai mon service
  - mgrs: chứa các host triển khai mgr service
  - osds: chứa các host có osd sử dụng trong ceph cluster
```
Group thêm:
- `rgws`: chứa các host triển khai radosgw service (sử dụng cho cụm Ceph Object)

## 2 . Thư mục `extra_vars`:
Trong thư mục sẽ bao gồm các file thông tin của cluster, bao gốm các thông tin:
```sh
- create_pools : dánh sách pools sẽ được khởi tạo
- network_ceph: thông tin về dải network public và network replicate
- images: chứa thông tin cấu hình chi tiết về các pools sẽ được khởi tạo bao gồm các thông số cơ bản:
  - name
  - pg_num
  - pgp_num
  - size
```

## 3. Thư mục `host_vars`
Bao gồm các file chứa thông tin chi tiết của từng node trong cluster.
```sh
- Hostname
- IP của host
- Danh sách ổ cứng
- Loại ổ cứng SSD hay là HDD
```
