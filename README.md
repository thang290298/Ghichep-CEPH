# Ceph Manage - SmartCloud

# Sử dụng Playbook
## 1. Cài đặt cephadm trên các node ceph
```
ansible-playbook -i inventory/cephobj-ntl4 cephadm-setup.yml -e@extra_vars/customise_obj_ntl4.yml
```

## 2. Sinh các file yaml phục vụ khởi tạo dịch vụ
```
ansible-playbook -i inventory/cephobj-ntl4 gen_deploy_yaml.yml -e@extra_vars/customise_obj_ntl4.yml
```

## 3. Quy hoạch tài nguyên cho Ceph cluster
1. Thiết lập crush rule cho Ceph cluster:
    ```
    ansible-playbook -i inventory/cephobj-ntl4 add-crushrules.yml -e@extra_vars/customise_obj_ntl4.yml
    ```

2. Khởi tạo các pool cho Ceph cluster
    ```
    ansible-playbook -i inventory/cephobj-ntl4 add-pools.yml -e@extra_vars/customise_obj_ntl4.yml
    ```
-------------
## Playbook
### 1. Thư mục `extra_vars`:
Trong thư mục sẽ bao gồm các file thông tin của cluster được phân tách theo từng cụm:
- Ceph RBD NTL3 Colocation
- Ceph RBD NTL4 ĐHSXKD
- Ceph RBD TT3 Colocation
- Ceph RBD TT4 ĐHSXKD
- Ceph Object NTL4

Các file trong thư mục `extra_vars` sẽ chứa các thông tin về:
- Service triển khai trên từng cụm Ceph
- Danh sách các pool cần khởi tạo
- Thông tin chi tiết các pool: tên, số pg, rule, ...

### 2. Thư mục `host_vars`
Bao gồm các file chứa thông tin chi tiết của từng node trong cluster.
- Hostname
- IP của host
- Danh sách ổ cứng

### 3. Thư mục `inventory`
Chứa các file inventory theo từng cụm.
Trong mỗi file inventory sẽ chia group theo chức năng. Các group này sẽ được sử dụng để tạo các file template phục vụ sinh file yaml dùng để deploy hệ thống.

Các group bắt buộc:
- `admin`: chứa host đóng vai trò quản trị ceph cluster, triển khai ceph sử dụng cephadm
- `mons`: chứa các host triển khai mon service
- `mgrs`: chứa các host triển khai mgr service
- `osds`: chứa các host có osd sử dụng trong ceph cluster

Group thêm:
- `rgws`: chứa các host triển khai radosgw service (sử dụng cho cụm Ceph Object)

Ví dụ:
```
[admin]
cephobj00 ansible_host=192.168.60.71

[mons]
cephobj00 ansible_host=192.168.60.71
cephobj01 ansible_host=192.168.70.71

[mgrs]
cephobj02 ansible_host=192.168.70.72
cephobj03 ansible_host=192.168.70.73

[rgws]
cephobj00 ansible_host=192.168.60.71

[osds]
cephobj00 ansible_host=192.168.60.71
```