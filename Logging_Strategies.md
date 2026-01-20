# Quy trình truyền tải Log từ Kubernetes Nodes tới Centralize Logging System

1. Xây dựng cơ chế Write Log cho Kubernetes Pods để có thể stdout ra Agent / Collector 
2. Cài đặt Agent trên từng Kubernetes Node, điều này được viết trong file config của Kubernetes khi triển khai là file `.yaml`
3. 
