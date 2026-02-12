# Các Logging stacks phổ biến hiện nay

## Grafana - Loki

Logs -> Promtail -> Grafana Loki <- Grafana

- Các log file chứa các log của Application / System / platform / ... 
- Promtail là 1 Agent được cài đặt trên Server, dùng để thu thập log gửi tới Loki 
- Grafana Loki dùng để lưu trữ và lập Index, là hệ thống tổng hợp log từ Promtail, không lập index cho toàn bộ log mà chỉ lập index cho Label đi cùng Log. Giúp Loki có khả năng lưu trữ và chi phí với hiệu suất cao hơn. Ngôn ngữ truy vấn là logQL giống PrompQL của Prometheus.
- Grafana là công cụ Trực quan hóa, kết hợp với Loki để truy vấn, lọc và hiển thị kết quả dưới dạng bảng.

## EFK stack: ElasticSearch - Fluentbit - Kibana 

Application Server -> ElasticSearch <- Kibana

- Các Application sinh ra log thì các Agent trong từng Server sẽ thu thập log thông qua Fluentd hoặc Fluentbit sẽ thu thập và lưu trữ và lập index cho ElasticSearch
- Kibana có các Dashboard cụ thể, nó liên kết với ElasticSearch cho phép Query, Search, Filter log khác nhau để tìm kiếm, giám sát,...

## ELK stack: ElasticSearch - Logstash - Kibana 

Beats ( Agent ) -> Logstash -> ElasticSearch -> Kibana

Elastic Endpoint Security -> Kibana

- Metric Beats sẽ thu thập các metrics của hệ thống
- Logstash gom dữ liệu từ Metric Beats, xử lý dữ liệu log một cách linh hoạt như chuyển đổi định dạng dữ liệu, thêm dữ liệu, ... để cho dữ liệu log đa dạng hơn. Sau đó gửi dữ liệu tới ElasticSearch và đi tới Kibana

## VEK stack: Vector - ElasticSearch - Kibana

Vector (log-shipper) -> aggregator -> Logstash, Grafana Loki -> ElasticSearch -> Kibana, Grafana

- Vector gồm 3 phase: Sources -> Transforms -> Sinks. 
- Giải thích từng phase: Sources để nhận log tập trung. Transforms dùng để biến đổi, chuấn hóa log sau khi nhận được từ Sources. Sinks là bước lưu, gửi dữ liệu sau khi chuẩn hóa lên Grafana, lưu vào AWS S3,...  

# Buffer

Sau khi các Agent có log từ Application / System / Platform / ... Thì có thể dùng kafka/redis/RabbitMQ để làm buffer để phòng tránh trường hợp Logstash, Grafana Loki không hoạt động thì sẽ lưu lại để gửi sau. Tránh mất log, ngoài ra còn control được là sẽ đổ bao nhiêu dữ liệu vào Logstash

Example ELK Stack: beats -> buffer -> Logstash -> ElasticSearch <- Kibana 

# Tiêu chí lựa chọn công nghệ, áp dụng trên phạm vi của toàn bộ tất cả project

1. Đặc tính dữ liệu log

    - Log dạng dòng thời gian, metadata chủ yếu qua **labels (service/env/instance,...)**, payload giữ nguyên (schema-on-read) 
    - Format Log Labels: Promptail là Log agent, nó sẽ làm việc gán Log Labels 
```
env="prod"
service="payment-service"
namespace="ecommerce"
```

- Format Log Content
```
{
  "ts": "2026-01-21T16:40:12.345Z",
  "level": "error",
  "service": "payment-service",
  "message": "payment failed",
  "request_id": "req-abc-123",
  # tạm thời giờ chưa dùng traceid vì chưa có Tracing System
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "00f067aa0ba902b7",
  "order_id": "o789",
  "error_code": "PAY_01"
}
```
2. Mô hình truy vấn

    - LogQL: chọn theo labels, lọc dòng, hàm thời gian

3. Tốc độ ghi và lưu giữ

    - Ingest rất cao (TP-PB/tháng), retentioni dài ( tháng-năm), lưu trữ chính trên Object Storage ( S3/MinIO/Blob Storage), compaction bởi compactor 

4. Hệ sinh thái tích hợp 

     - Grafana + Prometheus/Tempo/OTel, correlation logs-metrics-traces, live tailing thuận tiện trong vận hành. 

5. Vận hành và chi phí

    - Kiến trúc stateless cho ingester/querier/gateway, dựa object storage (chi phí/GB thấp, mở rộng tuyến tính), cần kỷ luật labels để tránh Cardinality cao ( tránh các labels có số lượng giá trị khác nhau lớn bởi vì Loki index )

6. Kết luận lựa chọn 

Logs -> Promtail -> Grafana Loki -> Grafana

Metrics -> Prometheus -> Grafana 