# Báo Cáo Tối Ưu Hóa Index Hệ Thống IoT SmartFactory

## 1. Vấn đề của Fat Covering Index
Trong hệ thống IoT ghi dữ liệu liên tục (hàng chục nghìn bản ghi/giây), việc tạo Composite Index trên 5 cột `(sensor_id, recorded_at, temperature, humidity, status)` khiến kích thước một nhánh B-Tree (Leaf Node) quá lớn. Mỗi thao tác `INSERT` phải phân bổ lại các trang dữ liệu (Page Splits) và cập nhật cây index nặng nề, gây ra hiện tượng Write Penalty nghiêm trọng làm rớt gói tin và hóa đơn lưu trữ tăng gấp 4 lần.

## 2. Giải pháp Lean Index và Đánh đổi (Trade-off)
Chúng tôi loại bỏ các cột dữ liệu đo lường liên tục (`temperature`, `humidity`, `status`) ra khỏi index và chỉ giữ lại 2 cột lọc chính: `(sensor_id, recorded_at)`.
- **Tốc độ Đọc (Read)**: Chấp nhận độ trễ tăng nhẹ ở mức mili-giây do MySQL phải thực hiện Bookmark Lookup (tìm bản ghi trên Clustered Index để lấy dữ liệu cảm biến).
- **Tốc độ Ghi (Write)**: Tốc độ `INSERT` tăng 4-5 lần do cây B-Tree tinh gọn, giải quyết triệt để lỗi timeout đường truyền pipeline.
- **Lưu trữ (Storage)**: Kích thước `index_length` giảm tới 60-70%, giúp tiết kiệm đáng kể chi phí hạ tầng Cloud SSD.
