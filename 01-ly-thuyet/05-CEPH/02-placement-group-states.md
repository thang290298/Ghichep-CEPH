<h1 align="center">Các trạng thái của Placement group</h1>


- Kiểm tra trạng thái clsuter trong cụm CEPH bằng các câu lệnh `veph -w` hoặc `ceph -s`. Hệ thống sẽ thông báo tranng thái cảu `Placement group`. MOttj Placement group có 1 hoặc nhiều trạng thái, trong đó trạng thái tốt nhất là `active + clean`

  - `Creating`: CEPH đang được khởi tạo trong Placement group
  - `activating`: Placement group được khởi động nhưng chưa đạng trạng thái active
  - `active`: CEPH xử lý thành công Placement group
  - `clean`: Đảm bảo tất cả các object trong các PG có đủ số lượng replicate tại thời điểm đó
  - `down`: Bản replicate bị lỗi dẫn đến trạng thái Placement group là offline
  - `scrubbing`: CEPH check tính chất nhất quán Placement group metadata
  - `deep`: CEPH kiểm tra Placement group data dưới dạng mã checksum
  - `degraded`: Không đủ số lượng relicate object ở thời điểm hiện tại
  - `inconsistent`: CEPH phát hiện có sự không nhất quán 1 hoặc nhiều bản replicate của 1 object trong Placement group
  - `peering`: Placement group đang trong quá trình chuẩn bị thay đổi giữa các process
  - `repair`: CEPH kiểm tra trạng thái và fix lỗi các PG nếu tìm thấy
  - `recovering`: CEPH đang migrating/synchronizing object và thực hiện replicate chúng
  - `forced_recovery`: Phục hồi PG có độ ưu tiên cao được xử lý bởi người dùng
  - `recovery_wait`: PG đang đợi để thực hiện khôi phục
  - `recovvery_toofull`:Một tiến trình recovery đang trong hàng đợi do tỷ lệnh OSD đang vượt ngưỡng ratio
  - `recovery_unfound`: Dừng tiến trình recovery kho không tìm thấy object
  - `backfilling`: CEPH tìm và đồng bộ entire contents của 1 PG. Backfill là một trường hợp phục hồi đặc biệt
  - `forced_backfill`: Thực hiện san lâp đồng bộ với PG có độ ưu tiên cao
  - `backfill_wait`: PG đang đợi để backfill
  - `backfill_toofull`: Quá trình thực hiện san lấp giữa các OSD phải thực hiện đợi do OSD disk vượt ngưỡng ratio chp phép
  - `backfill_unfouund`: quá trinh sang lấp dừng lại do không tìm thấy object
  - `incomplete`: CEPH phát hiện 1 PG miss trong quá trình ghi hoặc không có bất kỳ bản sao chép nào. Nếu thấy trạng thái này đánh fails OSD, nếu case này diễn ra có thể thực hiện giảm min size để tiến hành recovery
  - ` statle`: PG không lấy được trạng thái
  - `remapped`: PG này phản anh đến PG khác với những gì mà crush map chỉ định đặc biệt
  - `undersized`: PG có ít hơn các bản coppy so với muuwcs replicate và cấu hình pool
  - `peered`: PG đã đựơc peer nhưng không thể phục vụ quá trình đọc ghi I/O do không đủ bản sao đẫ cấu hình tại pool để thực hiện phục hồi. Recovery có thể thực hiện trong thruowngf hợp này 
  - `snaptrim`: Đang thực hiện cắt snaps
  - ` snaptrim_wait`: đang đợi để thực hiện cắt snaps
  - ` snaptrim_error`: Dừng do lỗi thực hiện cắt snap
  