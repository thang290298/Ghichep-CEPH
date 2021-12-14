<h1 align="center">Cài đặt CEPH phiên bản nautilus</h1>

# Phần I. Chuẩn bị

## 1. Mô hình triển khai

<h3 align="center"><img src="..\03-Images\Lab\1.png"></h3>

Trong đó:
  - Ba node CEPH: CentOS 7 - 64bit.
  - Disk: Mỗi node CEPH có 4 Disk. 1 Disk cài OS, 3 Disk lưu trữ dữ liệu cho Client.
  - NICs:
    - `eth0`: SSH và cài đặt.
    - `eth1`: Kết nối thông tin giữa các node CEPH, đường cho clients kết nối.
    - `eth2`: Đồng bộ dữ liệu giữa các node CEPH.
  - Phiên bản cài đặt: CEPH Nautilus.

## 2. IP Planning

<h3 align="center"><img src="..\03-Images\Lab\2.png"></h3>