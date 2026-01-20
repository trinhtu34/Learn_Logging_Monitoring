# Tư duy thiết kế hệ thống log 

1. Log có cấu trúc 

Nên thiết kế log thành các cấu trúc cụ thể ví dụ như dạng Key-Value

2. Lưu ý khi thiết kế hệ thống Logging cho hiệu quả

Khi làm 1 hệ thống log thì nên có 1 trace_id, mục đích của điều này làm cho việc theo dõi log đã đi qua services nào từ đó giúp cho việc theo dõi đường đi của lỗi dễ hơn

3. Log level

DEBUG - INFO - WARN - ERROR - CRITICAL/FATAL 

4. Không log các dữ liệu nhảy cảm 

Password, token, secret, PII, file upload, ... 

5. Log phải dễ truy vấn và lưu trữ lâu dài

- Tùy theo doanh nghiệp, bài toán cụ thể để biết nên lưu log nào trong thời gian bao lâu
- Chia log thành 2 loại là: log sẽ được truy vấn nhiều ( Hot Storage ) và log không được truy vấn nhiều ( Cold Storage ). 
- Tùy vào loại log mà tìm nơi lưu trữ cho phù hợp ví dụ như Hot Storage lưu ở local hoặc đâu đó trong X ngày, sau đó đưa vào Cold Storage ví dụ như `S3 Glacier`



