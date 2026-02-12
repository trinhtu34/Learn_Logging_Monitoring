# Nguyên tắc viết code cho Microservice 

Đây sẽ là 1 bản hướng dẫn từ việc viết code, lưu dữ liệu như thế nào trong Microservices có trường hợp thiết kế có 2 table ví dụ như sau: 

- 2 table KhachHang và SanPham. 1 KhachHang có thể có nhiều SanPham ở giỏ hàng, 1 SanPham có thể có trong giỏ hàng của 1 hoặc nhiều KhachHang. 
- 2 table LoaiSanPham và SanPham. 1 LoaiSanPham có thể có nhiều SanPham và 1 SanPham chỉ thuộc về 1 LoaiSanPham duy nhất.
- 2 table NhanVien và BangLuong. 1 NhanVien chỉ có thể có 1 BangLuong và 1 BangLuong chỉ thuộc về 1 NhanVien.

## Thiết kế Database

Để đảm bảo việc triển khai 1 dự án Microservice, ngay bước thiết kế database cần phải tách biệt nghiệp vụ và không liên quan tới các Service khác. 

### Mối quan hệ Many to Many 

1. Trong quan hệ Many to Many phải xác định đâu là table Owner

    Example: KhachHang <-> SanPham. Trong giỏ hàng của 1 khách hàng có thể có nhiều sản phẩm, 1 thông tin sản phẩm ( mã sản phẩm ) có thể có trong giỏ hàng của nhiều khách hàng. Thì ở đây khách hàng chính là Owner

2. Bảng trung gian phải tồn tại trong Database của Owner

    Example: Khi thiết kế Database cho trường hợp này, thì table KhachHang chính là Owner

### Mối quan hệ One to One, One to Many

Thiết kế như bình thường

## Client ( Web frontend - mobile app - ... )

Trường hợp giả định như sau: Hệ thống có 2 table là Todo chứa thông tin của 1 Todo trong Todo list của 1 user, 1 table là Tag chứa các thông tin về Tag của 1 user vừa tạo. Khi cần gắn 1 tag vào 1 Todo thì cần tạo bản ghi ở table TodoTag. API hiện có: 

- GET: /api/tag : lấy danh sách tag của 1 User
- GET: /api/tag/?ids=tag1,tag2 : lấy thông tin của 1 danh sách tag cụ thể 
- POST: /api/tag : thêm 1 tag mới
- DELETE: /api/tag/{tagID} : xóa 1 tag
- PUT: /api/tag/{tagID} : thay đổi dữ liệu của 1 tag

- GET: /api/todo : lấy danh sách todo của 1 User
- POST: /api/todo : thêm 1 todo mới
- DELETE: /api/todo/{todoID} : xóa 1 todo 
- PUT: /api/todo/{todoID} : thay đổi dữ liệu của 1 todo

- GET: /api/todo/{todoId}/tags/{tagID} : lấy thông tin tag của 1 todo ( table TodoTag )
- DELETE: /api/todos/{todoId}/tags/{tagID} : xóa 1 tag khỏi 1 todo ( table TodoTag )
- POST: /api/todos/{todoID}/tags : thêm 1 danh sách tag vào 1 todo ( table TodoTag )
```
# body
{
  "tagID": ["1", "3"]
}
```

Để đảm bảo các dự án dùng kiến trúc Microservices được nhất quán về cách truy xuất dữ liệu, cách gửi dữ liệu cho Backend thì cần đồng nhất 1 cơ chế về:

- Lấy và hiển thị dữ liệu 
- Đưa dữ liệu tới Backend 
- Thêm dữ liệu khi 2 table có quan hệ Many to Many

### Nguyên tắc về lấy và hiển thị dữ liệu 

- Trong mọi trường hợp Many to Many, One to Many và One to One, khi Client gọi tới API ví dụ `GET: /api/tag` thì sẽ nhận được 1 danh sách trả về bao gồm nhiều trường dữ liệu, trong đó sẽ có thuộc tính `TagID`. Client phải lưu `TagID` này lại cho các tác vụ sau này.

### Nguyên tắc về truyền dữ liệu tới Backend

- Khi call tới API `POST: /api/todos/{todoID}/tags` để thêm tags vào todo thì phải đưa tagID đã có vào payload

### Trường hợp cần kết hợp dữ liệu của cả 2 Services để tổng hợp và hiển thị cho Client

- Client call tới API `GET: /api/todo/{todoId}/tags/{tagID}` để xem todo đó có các tag nào, tiếp theo call API `GET: /api/tag/?ids=tag1,tag2` để lấy chi tiết dữ liệu để hiển thị lên giao diện. 

## Backend

### Nguyên tắc về thiết kế API 

### Nguyên tắc về viết Code cho trường hợp Database có table Many to Many ( như trong ví dụ KhachHang <-> SanPham )

- Code của API `GET: /api/todo/{todoId}/tags/{tagID}` sẽ được viết tại TodoService. Viết tại Owner là TodoService, còn TagService chỉ là `Source of Trusted`,