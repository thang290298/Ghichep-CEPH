<h1 align="center">Kiến trúc và các thành phần CEPH</h1>

# Phần I. Kiến trúc CEPH

<h3 align="center"><img src="../../03-Images/document/79.png"></h3>

- CEPH được xây dựng giải pháp dựa trên phần mềm Software-defined Storage (SDS), cung cấp giải pháp đáp ứng các khách hàng có hạ tâng lớn nhưng không muốn đầu tư thêm chi phí. SDS hỗ trợ tốt trên nhiều nên tảng phần cứng từ mọi nhà cung cấp, đem đến các lợi thế về giá thành, tính đảm bảo và khả năng mở rộng

- Ceph storage cluster được xây dụng từ các tiến trình . Mỗi tiến trình đều có một vai trò và giá trị sử dụng khác nhau. Đây là yếu tố để so sánh Ceph đối với các hệ thống tương tự



## 1. Lưu Trữ Phân Phối Dữ Liệu Đáng Tin Cây (RADOS)

- RADOS là yếu tố nền tảng tạo nên Ceph storage cluster. Ceph data được lưu trên object, RADOS có trách nhiệm tổ chức, lưu trữ các object. RADOS layer chắc chắn dũ liệu luôn chính xác, đảm bảo

- RADOS cung cập tính nhất quán, luôn có bản sao dữ liệu, phát hiện lỗi và khôi phục trên mọi node trong cluster. KHi ứng dụng lưu trữ tới ceph cluster, dữu liệu sẽ được lưu trữ tại Ceph Object Storage Device (OSD) dưới dạng object. Đây là thành phần duy nhất mà Ceph Cluster sử dụng để lưu trữ và truy vấn dữ liệu.
- Thông thường tổng số ổ cứng vật lý sử dụng trong Ceph cluster sẽ bằng với số lượng tiến trình OSD sử dụng để lưu trữ dữ liệu

## 2. Tiến Trình Giám Sát (Ceph Monitor – Ceph MON)

- Ceph Monitor tập trung vào trạng thái toàn cluster, giám sát trạng thái OSD,MON,PG,Crush map.Các Cluster node sẽ giám sát và chia sẻ thông tin về các thay đổi. Quá trình giám sát sẽ không lưu trữ dữ liệu. librados lib hỗ trợ truy cập RADOS thông qua PHP, Ruby, Java, Py, C/++.
- Cung cấp giao diện thân thiện với Ceph storage cluster, RADOS và các service RBD, RGW, POSIX interface trong Cephfs

> librados API hỗ trợ truy cập trực tiếp tới RADOS, cho phép nhà phát triển tạo mới giao thức tương tác với Ceph storage cluster.

## 3. Ceph Block Device Hay RADOS Block Device (RBD)


- Đây là thành phần cung cấp block storage, có thể mapped, formmatted, mounted giống như bất kỳ các disk thông thường
- Ceph block storage hỗ trợ cacs tính năng provisioning và snapshots


## 4. Ceph Object Gateway Hay RADOS Gateway (RGW)

- Thành phần cung cấp giao diện RESTful API, tương thích với Amazone S3 (Simple Storage Service) và Openstack Object Storage API (Swift)
- RGW cũng hỗ trợ Openstack keystone authencation services.


## 5. Ceph Metatdata Server ( MDS)
- Thành phần tập trung phân cấp files và lưu trữu metadata dành riêng cho Cephfs. Ceph block device và RADOS gateway không yêu cầu metadata vì chúng không cần MDS deamon. MDS không trực tiếp hỗ trợ khách hàng, vì thế loại bỏ tính lỗi đơn cho hệ thống

## 6. Ceph File system (CephFS)

- Cung cấp POSIX-compliant, phân phối filesystem
- CephFS dựa trên Ceph MDS để thể hiện tính phân cấp file, metadata.

# Phần I. Các thành phần chính
