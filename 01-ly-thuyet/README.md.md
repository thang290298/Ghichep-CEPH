<h1 align="center">Tổng quan về CEPH ( </h1>


## 1. Tổng quát

**Ceph** là 1 project mã nguồn mở, cung cấp giải pháp data storage. Ceph cung cấp hệ thống lưu trữ phân tán mạnh mẽ, tính mở rộng, hiệu năng cao, khả năng chịu lỗi cao. Xuất phát từ mục tiêu, Ceph được thiết kết với khả năng mở rộng cao, hỗ trợ lưu trữ tới mức exabyte cùng với tính tương thích cao với các phần cứng có sẵn.


<h3 align="center"><img src="../03-Images/document/1.png"></h3>

## 2. Lịch sử hình thành

- Argonaut – 03/07/2012 team CEPH đã phát triển và released bản Argonaut, bản phát hành "ổn định" lớn đầu tiên của Ceph. Bản phát hành này sẽ chỉ nhận được các bản sửa lỗi ổn định và cập nhật hiệu suất và các tính năng mới sẽ được lên lịch cho các bản phát hành trong tương lai.

- Bobtail (v0.56) – 01/01/2013 team CEPH đã phát triển và released bản Bobtail, bản phát hành "ổn định" lần thứ 2 của CEPH. Bản phát hành này tập trung chủ yếu vào sự ổn định, hiệu suất và khả năng nâng cấp từ loạt ổn định Argonaut trước đó (v0.48.x).

- Cuttlefish (v0.61) – 07/05/2013 team CEPH đã phát triển và released bản Cuttlefish, bản phát hành "ổn định" lần thứ 3 của CEPH. Bản phát hành này bao gồm một số cải tiến về tính năng và hiệu suất cũng như là bản phát hành ổn định đầu tiên có công cụ triển khai 'ceph-deploy' để thay đổi phương thức triển khai 'mkcephfs' trước đó.

- **Dumpling** (v0.67) – 14/08/2013, bản phát hành ổn định lần thứ 4. Bản này bao gồm global namespace, region support, a REST API cho việc monitoring

- **Emperor** (v0.72) – 09/01/2013, bản phát hành ổn định lần thứ 5. Bản này cung cấp tính năng mới multi-datacenter replication cho radosgw, cải thiện khả năng sử dụng và đạt được nhiều hiệu suất gia tăng và công việc tái cấu trúc nội bộ để hỗ trợ các tính năng sắp tới trong Firefly

- **Firefly** (v0.80) – 07/05/2014, bản phát hành ổn định lần thứ 6, bản phát hành này mang đến một số tính năng mới, bao gồm mã hóa, phân vùng bộ đệm (cache tiering), key/value OSD backend (thử nghiệm).

- **Giant** (v0.87) – 29/10/2014, bản phát hành ổn định lần thứ 7.

- **Hammer** (v0.94) – 07/04/2015, bản phát hành ổn định lần thứ 8.

- **Infernalis** (v9.2.0) – 06/01/2015 bản phát hành ổn định lần thứ 9.

- **Jewel** (v10.2.0) – 21/04/2016, nhóm phát triển Ceph đã phát hành Jewel, phiên bản Ceph đầu tiên trong đó CephFS được coi là ổn định. Các công cụ CephFS repair, disaster recovery tools đã được hoàn thành, một số chức năng bị tắt theo mặc định. Bản phát hành này bao gồm phụ trợ RADOS thử nghiệm mới có tên BlueStore, được lên kế hoạch làm phụ trợ lưu trữ mặc định trong các bản phát hành sắp tới.

- **Kraken** (v11.2.0) – 20/01/2017, nhóm phát triển Ceph đã phát hành Kraken. Định dạng lưu trữ BlueStore mới, được giới thiệu trong Jewel, hiện có định dạng trên disk ổn định và là một phần của bộ thử nghiệm. Mặc dù vẫn được đánh dấu là thử nghiệm, BlueStore đã sẵn sàng phát triển và nên được đánh dấu như vậy trong phiên bản tiếp theo Luminous.

- **Luminous** (v12.2.0) – 29/08/2017 nhóm phát triển Ceph đã phát hành Luminous. Trong số các tính năng khác, định dạng lưu trữ BlueStore (sử dụng raw disk thay vì hệ thống filesystem) hiện được coi là ổn định và được khuyến nghị sử dụng.

- **Mimic** (v13.2.0) – 01/06/2018, phát hành bản Mimic. Với việc phát hành Mimic, snapshots hiện ổn định khi được kết hợp với multiple MDS daemons và RESTful gateways frontend Beast được tuyên bố là ổn định và sẵn sàng để sử dụng.

- **Nautilus** (v14.2.0) – 19/03/2019 team CEPH đã phát triển và released bản **Nautilus**.
## 3. Giới thiệu

- Ceph là dự án mã nguồn mở, cung cấp giải pháp lưu trữ dữ liệu. Ceph cung cấp giải pháp lưu trữ phân tán mạnh mẽ, đáp ứng các yêu cầu về khả năng mở rộng, hiệu năng, khả năng chịu lỗi cao. Xuất phát từ mục tiêu ban đầu, Ceph được thiết kết với khả năng mở rộng không giới hạn, hỗ trợ lưu trữ tới mức exabyte, cùng với khả năng tương thích cao với các phần cứng có sẵn. Và từ đó, Ceph trở nên nổi bật trong ngành công nghiệp lưu trữ đang phát triển và mở rộng.
- Hiện nay, các nền tảng hạ tầng đám mây công cộng, riêng và lai dần trở nên phổ biến và to lớn. Bên cạnh đó, phần cứng – thành phần quyết định hạ tầng đám mây đang dần gặp các vấn đề về hiệu năng, khả năng mở rộng. Ceph đã và đang giải quyết được các vấn đề doanh nghiệp đang gặp phải, cung cấp hệ thống, hạ tầng lưu trữ mạnh mẽ, độ tin cậy cao.
- Nguyên tắc cơ bản của Ceph:
  - Khả năng mở rộng tất cả thành phần
  - Khả năng chịu lỗi cao
  - Giải pháp dựa trên phần mềm, hoàn toàn mở, tính thích nghi cao
  - Chạy tương thích với mọi phần cứng
- Ceph xây dựng kiến trúc mạnh mẽ, khả năng mở rộng không giới hạn, hiệu năng cao, cung cấp giải pháp thống nhất, nền tảng mạnh mẽ cho doanh nghiệp, giảm bớt sự phụ thuộc vào phần cứng đắt tiền. Hệ sinh thái Ceph cung cấp giải pháp lưu trữ dựa trên block, file, object, và cho phép tùy chỉnh theo ý muốn. Nền tảng Ceph xây dựng dựa trên các object, tổ chức trên các block. Tất cả các kiểu dữ liệu, block, file đều được lưu dưới dạng object, quản trị bởi Ceph cluster. Object storage hiện đã trở thành giải pháp cho hệ thống lưu trữ truyền thống, cho phép xây dựng kiến trúc hạ tầng độc lập với phần cứng.
- Ceph xây dựng giải pháp dựa trên Object, Các object được tổ chức, nhân bản trên toàn cluster, nâng cao tính bảo đảm dữ liệu. Tại Ceph, Object sẽ không tồn tại đường dẫn vật lý, toàn bộ Object được quản trị dưới dạng Key Object, tạo nền tảng mở với khả năng lưu trữ tới hàng petabyte-exabyte.
<h3 align="center"><img src="../03-Images/document/2.png"></h3>

## 4. Tính cần thiết của CEPH

Hiện nay, các nền tảng hạ tầng đám mây public, private, hybird cloud dần trở nên phổ biến và to lớn. Ceph trở thành giải pháp nổi bật cho các vấn đề đang gặp phải.

Các yêu cầu mong muốn của một hệ thống lưu trữ (storage).

- Sử dụng thay thế lưu trữ trên ổ đĩa server thông thường.
- Sử dụng để backup, lưu trữ an toàn.
- Sử dụng để thực hiện triển khai các dịch vụ High Avaibility khác như Load Balancing for Web Server, DataBase Replication…
- Xây dựng Storage giải quyết bài toán lưu trữ cho dịch vụ Cloud hoặc phát triển lên Cloud Storage (Data as a Service).

> **`CEPH`** có thể đáp ứng được các yêu cầu trên.

## 5. Ceph và giải pháp
### 1. Ceph – Giải Pháp Cloud Storage
- Thành phần quan trọng để phát triển Cloud chính là hạ tầng lưu trữ hay còn gọi là Storage. Cloud cần storage để phát triển, đáp ứng nhu cấp lưu trữ, tổ chức lượng dữ liệu cực lớn. Hiện nay, các giải pháp lưu trữ truyền thống đã dần tới giới hạn như gặp phải các vấn đề về chí phí, kiến trúc, tính mở rộng v.v. Ceph đã giải quyết được các vấn để đang gặp phải, đáp ứng nhu cầu lưu trữ của Cloud.
- Ceph hỗ trợ tốt các nền tảng Cloud nổi bật như OpenStack, CloudStack, OpenNebula. Đội ngũ phát triển và cộng tác của Ceph bao gồm các nhà cung cấp nền tảng lớn nhất (Canonical, Red Hat, SUSE), với nhiều năm kinh nghiệp cũng như nắm bắt được xu hướng phát triển, khiến Ceph luôn đi trước, bắt kịp thời đại, tương thích cao với Linux, thành một trong những hệ thống tốt nhất khi đánh giá xây dựng hạ tầng lưu trữ.
### 2. CEPH - Giải Pháp Software-Defined
- Để tiết kiệm chi phí, Ceph xây dựng giải pháp dựa trên phần mềm Software-defined Storage (SDS). Cung cấp giải pháp đáp ứng các khách hàng đã có sẵn hạ tầng lớn, không mong muốn đầu tư thêm nhiều chi phí. SDS hỗ trợ tốt trên nhiều phấn cứng từ bất kỳ nhà cung cấp. Mang đến các lợi thế về giá thành, tính bảo đảm, và khả năng mở rộng.

### 3.CEPH - Giải pháp lưu trữ thống nhất
- Ceph mang đến giải pháp lưu trữ trữ thống nhất bao gồm file-based và block-based access truy cập thống nhất qua nền tảng, giải pháp Ceph. Đáp ứng nhu cầu tăng trưởng dữ liệu hiện tại và cả trong tương lai. Ceph xây dựng giải pháp lưu trữ thống nhất (true unified storage solution) bao gồm object, block, file storage và đồng bộ qua nền tảng dựa trên phần mềm , hỗ trợ lưu trữ các luồng dữ liệu lớn, không có cấu trúc.
- Lợi dụng điểm mạnh của ceph, toàn bộ block hay file storage đều được lưu trữ dưới dạng đối tượng quản trị bởi Ceph Cluster. Ceph quản lý các object dưới kiến trúc riêng của mình. Object trong ceph được quản trị, tổ chức riêng biệt, và hỗ trợ mở rộng không giới hạn bằng cách lược bỏ các metadata, bỏ qua đường dẫn vật lý. Để làm được điều này, ceph sử dụng thuật toán động để tính toán, lưu trữ, tìm kiếm dữ liệu.
<h3 align="center"><img src="../03-Images/document/3.png"></h3>