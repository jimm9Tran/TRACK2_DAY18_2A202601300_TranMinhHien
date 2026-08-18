# Reflection — Day 18 Lakehouse Lab

Anti-pattern  em dễ gặp nhất là coi external vector index như nguồn dữ liệu chính và quên đồng bộ vòng đời với lakehouse. NB7 cho thấy sau khi xóa tài liệu trong bảng Delta, truy vấn trong bảng trả về 0 kết quả, nhưng external index cũ vẫn trả về vector của tài liệu đã xóa. Điều này rủi ro: yêu cầu xóa có thể hoàn tất trên hệ thống nghiệp vụ nhưng nội dung vẫn bị truy xuất qua semantic search.

Nguyên nhân là index thường được cập nhật theo luồng ingest một chiều, trong khi delete, update và quyền truy cập không được truyền sang index. Nhóm cần coi lakehouse là system of record, còn vector database chỉ là derived index có thể tái tạo. Khi có thay đổi, chúng em sẽ dùng Change Data Feed/CDC để phát sự kiện upsert và delete, lưu checkpoint để xử lý đúng một lần, đồng thời đối soát định kỳ `doc_id` giữa bảng và index. Với yêu cầu xóa nhạy cảm, việc evict vector cần được xác nhận trước khi báo hoàn tất.
