
# Các loại log 

## Application logs

Ghi lại hành vi trạng thái của Application khi thực hiện các tác vụ, thường tới từ logic của backend

## System logs 

Log của hệ điều hành và nền tảng, nguồn có thể đến từ Kernel, Systemd, init script, windows event logs, syslog

## Infrastructure / Platform logs 

Theo dõi trạng thái nhưng ở tầng Platform, như Kubernetes Cluster, Load Balancer, Cloud Service, ...

## Access logs

Theo dõi các request tới hệ thống, thường là cho các tác vụ Audit hoặc phân tích lưu lượng truy cập, như Web server, API gateway, reserve proxy, firewall, nginx,...

## Security / Audit logs

Các hoạt động liên quan tới bảo mật, sẽ biết ai truy cập vào Resouce nào, thay đổi gì như DB audit log, PCI, IDS, WAS, Cloud IAM Audit service,... 

## Event logs

Ghi nhận các event của hệ thống ở mức High-Level

## Trace logs

Theo dõi đường đi của 1 request qua nhiều services 