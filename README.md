![image](https://github.com/user-attachments/assets/6a97263b-65fc-4b99-80ad-c615dadf0b77)

## 1. Luồng hoạt động chính

### 1.1 Luồng hoạt động của Giáo viên:

#### • Tạo bài tập và thiết lập tiêu chí:

- Giáo viên đăng nhập và sử dụng **Lesson Builder & Scripting Assignment Service**.
- Tạo bài tập (ví dụ: _Listening_), thiết lập tiêu chí chấm điểm riêng.
- Bài tập được lưu vào cơ sở dữ liệu.

#### • Xử lý bài tập Listening (tệp âm thanh):

- Giáo viên tải lên tệp ghi âm.
- Hệ thống lưu trữ trên **CDN/S3 Storage** (đảm bảo độ trễ thấp).

#### • Huấn luyện LoRA Adapters (Background Jobs):

- Hệ thống tự động khởi chạy **Background Jobs** sau khi tạo bài tập.
- **Mục tiêu**: Huấn luyện _LoRA Adapter_ riêng cho từng giáo viên (**Multi-LoRA Serving**).
- **Lợi ích**:
  - Cá nhân hóa mô hình chấm điểm AI cho từng giáo viên.
  - Tối ưu hiệu quả và chi phí so với các phương pháp khác (_Prompt Engineering, Fine-tuning..._).
- Sau huấn luyện, hệ thống thông báo khi **LoRA Adapter** sẵn sàng sử dụng.

![image](https://github.com/user-attachments/assets/8287d254-0d75-4871-a517-703b5cb5120d)

#### • Soạn tài liệu học tập cá nhân hóa (Composer Agent):

- Yêu cầu soạn tài liệu cá nhân hóa cho học viên cụ thể hoặc nhóm học viên.
- **Composer Agent** truy xuất dữ liệu qua **Query Service**:
  - **Query Service** cung cấp dữ liệu kết hợp:
    - **Bài học IELTS liên quan**: Truy xuất từ **Vector Store** (_embedding IELTS Lessons_).
    - **Tiến độ học tập học viên**: Lấy từ **Database Dashboard**.
  - **Composer Agent** sử dụng dữ liệu tổng hợp để sinh tài liệu dự thảo: - Dựa trên bài học IELTS liên quan và tiến độ học tập học viên. - Sinh tài liệu cá nhân hóa: _Bài tập bổ sung, giải thích, luyện tập,..._ - Tài liệu dự thảo được cung cấp để giáo viên xem xét, chỉnh sửa, tùy chỉnh.
![image](https://github.com/user-attachments/assets/2b83047a-5aa4-4dfa-85ef-0a96670d2e08)


### 1.2 Luồng hoạt động của học sinh

#### • Nộp bài và tiếp nhận xử lý:

- Học sinh hoàn thành và nộp bài.
- Dữ liệu bài nộp được gửi vào **Kafka** (_message queue_).
- **Consumers** (_backend service_) đăng ký và xử lý dữ liệu song song.

#### • Chấm điểm bài tập AI (tùy loại bài):

- Dựa trên loại bài tập (_Speaking, Reading, Listening, Writing_) và tiêu chí chấm điểm.
- **Dynamic Adapter Loader** kích hoạt, tải **LoRA Adapter** phù hợp.
- **Grading Agents** (tương ứng loại bài tập) sử dụng **LoRA Adapter** để đánh giá bài làm.

#### • Phản hồi cá nhân hóa (Feedback Agent):

- **Feedback Agent** tổng hợp kết quả từ **Grading Agents**.
- Cung cấp phản hồi cá nhân hóa cho học sinh.
- Sử dụng **Query Service** (thay vì truy vấn trực tiếp **DB**) để:
  - **An toàn**: Tránh truy vấn không hợp lệ từ mô hình **LLM**.
  - **Thu thập dữ liệu**: Lấy lịch sử bài làm trước đó của học sinh.
- **Phân tích xu hướng lỗi sai và tiến bộ**:
  - **Gợi ý cải thiện**: Tài liệu, bài tập phù hợp (nếu lỗi lặp lại).
  - **Khen ngợi**: Tạo động lực học tập (nếu có tiến bộ).
![image](https://github.com/user-attachments/assets/798c43b4-02eb-4837-9c5f-14493e3b057c)


## 2. Các tính năng nổi bật:

### 2.1 Tính năng Ranking (Bảng xếp hạng):

- **Mục tiêu**: Tạo bảng xếp hạng giúp học sinh cạnh tranh, tăng động lực học tập và cải thiện kết quả.
- **Công nghệ cốt lõi**:
  - **Thuật toán Elo Ranking**: Đảm bảo độ chính xác cao trong xếp hạng.
  - **Redis Sorted Sets**: Lưu trữ và truy vấn bảng xếp hạng thời gian thực, hiệu suất cao.
  - **Server-Sent Events (SSE)**: Cập nhật bảng xếp hạng liên tục đến giao diện người dùng (_real-time_).
![image](https://github.com/user-attachments/assets/64283e3a-03ec-4fbe-83cb-d681a3f0df41)


    ### 2.2 Tính năng AI-Supported Group Study (Học nhóm có hỗ trợ từ AI):
- **Mục tiêu**: Tạo môi trường học tập nhóm, kết nối học viên để luyện tập và trao đổi.
- **Công nghệ cốt lõi**:
  - **Redis Pub/Sub**: Mở rộng khả năng scale **WebSocket**, đồng bộ kết nối giữa các **WebSocket servers**, đảm bảo hiệu suất khi số lượng lớn học viên tham gia.
  - **Conductor Agent**: Giám sát và hỗ trợ nội dung cuộc gọi giữa học sinh.
- **Chức năng của Conductor Agent**:
  - **Lấy thông tin học sinh**: Từ **Query Service** (lịch sử học tập, chủ đề, kỹ năng).
  - **Gợi ý nội dung giao tiếp**: Đảm bảo cuộc trò chuyện có định hướng và hiệu quả.
  - **Giám sát nội dung**: Phát hiện và cảnh báo nội dung không phù hợp, đảm bảo môi trường học tập an toàn.
![image](https://github.com/user-attachments/assets/7ee0d85a-da67-40bf-9d39-aa1043448b73)


