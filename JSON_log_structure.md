# Microservices + Event Driven Architecture -> Async Communication 

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

# Microservices -> Sync Communication 