# Reflection

Trong Top 5 Lakehouse Anti-Patterns, rủi ro lớn nhất với team mình là coi
retention như một thao tác dọn dẹp duy nhất. NB6 cho thấy `VACUUM` chỉ thu hồi
file đã bị tombstone; orphan do job lỗi chưa từng commit vẫn nằm trên storage.
Tương tự, Iceberg `expire_snapshots` có thể giảm số snapshot nhưng không tự
xóa mọi manifest/file bị bỏ lại. Nếu chỉ nhìn số snapshot, team dễ kết luận
sai rằng chi phí S3 đã giảm.

Mình sẽ biến retention thành một workflow có số đo: (1) ghi nhận version và
orphan candidates trước khi xóa, (2) chạy expiry/vacuum theo retention policy,
(3) quét orphan với grace period, (4) kiểm tra lại bytes và row counts, rồi mới
phát cảnh báo chi phí. Time travel và checkpoint giúp rollback khi job dọn dẹp
gặp lỗi; catalog là nơi lưu trạng thái cần audit. Cách này chậm hơn một lệnh
cleanup đơn lẻ, nhưng tránh mất dữ liệu hợp lệ và làm cho tuyên bố “đã tiết
kiệm storage” có bằng chứng kiểm tra được.
