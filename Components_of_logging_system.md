# Các thành phần trong hệ thống log tập trung ( Centralize logging )

**Pipline cho Centralize System Logging**

Log Producer ( nguồn sinh log )

Log Collector / Agent 

Log Transport / Forwarder

Log Processor ( Xử lý / Chuẩn hóa log )

Log Storage / Index

Log Query & Visualization

Alerting & Integration 

## Log Producer

Là nơi log được sinh ra ví dụ như Application, Container, Hệ điều hành, Cloud Services, ...

## Log Collector / Agent 

Thu thập log tại máy chủ, application, ... để gửi về hệ thống để quản lý Centralize. 

Thành phần này phải nhẹ, ít tiêu tốn tài nguyên, có khả năng tail file, có khả năng đọc từ stdout, stderr, systemd, buffer log khi mất kết nối với hệ thống Centralize log. 

## Log Transport / Forwarder

Truyền tải log từ Collector đến Storage, có thể làm đồng thời từ Buffering và quering về tránh mất log

Example: 
- Fluentd nhận log từ Fluent Bit và đẩy về ElasticSearch 
- Vector gửi log vào Kafka để tách biệt pipline ingest và pipline index

## Log processer ( Xử lý / chuẩn hóa log )

Chuyển đổi log về dạng có cấu trúc, enrich thêm metadata. Chuyển log thành cấu trúc json ( dạng key-value ), mã hóa dữ liệu nhạy cảm, thêm lable cần thiết, ...  

## Log Storage / Index

Lưu trữ log lâu dài, có khả năng truy vấn

Example: ElasticSearch, Loki ( Grafana, log dạng "index + object storage")

## Log Query and Visualization ( Truy vấn và Hiển thị)

Giúp dễ dàng tìm kiếm và phân tích log 

Example:
- Truy vết theo requestID, traceID
- Dashboard thống kê error rate, request per second, top services lỗi nhiều

## Alerting and Integration ( Cảnh báo và Tích hợp )

Chủ động thông báo khi có bất thường 

Example: 
- Log có X lỗi 5XX trong N phút => gửi cảnh báo vào Platform nhận thông báo như Slack / Email
- Log chứa từ khóa "OutOfMemoryError" => Trigger incident 