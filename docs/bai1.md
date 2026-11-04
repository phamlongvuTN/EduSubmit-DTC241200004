
# Bài 1 – Tổng quan hệ thống và MEP
- **Họ và tên:** Phạm Long Vũ
- **MSSV:** DTC241200004
- **Lớp:** Phát triển ứng dụng phân tán
- **Repository:** https://github.com/phamlongvuTN/EduSubmit-DTC241200004
- **File Word chi tiết:** [docs/hello.docx](hello.docx)

---

## 1. Cài đặt
Đã cài đặt thành công .NET 10 SDK, Git, Visual Studio Code và các extension cần thiết (C# Dev Kit, REST Client).
Kiểm tra môi trường:
- `dotnet --version`: 10.0.x
- `git --version`: 2.x
- Chạy thử dự án `HelloPtudpt`: In ra màn hình dòng chữ `Hello, World!` thành công.

---

## 2. MepDemo (Bước 7)
1. Ở MEP 3 (Queue-based):Mỗi bài nộp (`sub-x`) chỉ do duy nhất 1 worker tiếp nhận và xử lý (các worker chia nhau công việc từ hàng đợi chung để tránh xử lý trùng lặp và phân tán tải trọng).
2. Ở MEP 4 (Publish-Subscribe): Một sự kiện bài nộp phát ra được gửi đồng thời tới cả 3 dịch vụ (Email Notification, Audit Log, Dashboard Service).
3. Mô tả 1 câu cho mỗi MEP:
   - MEP 1 (Request-Response): Client gửi yêu cầu và đồng bộ chờ Server xử lý rồi nhận lời đáp.
   - MEP 2 (One-Way / Fire-and-Forget): Client bắn thông điệp đi và tiếp tục ngay công việc mà không cần chờ Server phản hồi.
   - MEP 3 (Queue / Competing Consumers): Nhiều worker cùng lắng nghe chung một hàng đợi, mỗi thông điệp chỉ do một worker duy nhất xử lý.
   - MEP 4 (Publish-Subscribe): Một sự kiện được xuất bản (publish) sẽ gửi tới tất cả các subscriber đã đăng ký quan tâm sự kiện đó.
   - MEP 5 (Streaming): Dữ liệu được truyền liên tục theo từng mẩu nhỏ (chunk) theo thời gian thực thay vì đợi gom cả gói lớn.

---

## 3. BasicWebApi (Bước 8)
- Đã chạy thử nghiệm API bằng REST Client với file `requests.http` đạt kết quả:
  - `GET /` $\rightarrow$ 200 OK.
  - `POST /greetings` (hợp lệ) $\rightarrow$ 201 Created (có header `Location`).
  - `POST /greetings` (tên rỗng) $\rightarrow$ 400 Bad Request.
- Trả lời câu hỏi mục 8.4:
  1. Dòng `builder.Services.AddSingleton...` phải đứng trước `var app = builder.Build()` vì mọi cấu hình và dịch vụ cần được đăng ký vào DI Container trước khi ứng dụng được build và đóng băng container.
  2. Trong `/hello/{name}?language=en`, tham số `name` lấy từ Route parameter, còn `language` lấy từ Query string.
  3. POST hợp lệ trả về `Results.Created (201)` theo đúng chuẩn RESTful biểu thị một tài nguyên mới vừa được khởi tạo thành công trên hệ thống.

---

## 4. MiniDistributedSystem (Bước 9)
- Hai chương trình độc lập `Submission.Api` (chạy trên cổng 5000) và `Submission.Client` (gọi tới API) chạy song song trên 2 terminal:
  - Client gửi `GET /api/assignments` $\rightarrow$ Nhận danh sách bài tập (200 OK).
  - Client gửi `POST /api/submissions` $\rightarrow$ Nhận mã lượt nộp Guid và trạng thái `received` (201 Created).
- Hai tiến trình độc lập, không tham chiếu code của nhau mà giao tiếp lỏng qua giao thức HTTP và định dạng JSON.
- Địa chỉ API truyền qua tham số dòng lệnh `args[0]` giúp Client linh hoạt chạy trên nhiều môi trường khác nhau mà không cần sửa code.

---

## 5. Thí nghiệm lỗi (Bước 10)

| # | Thao tác | Dự đoán (ghi TRƯỚC khi chạy) | Kết quả thực tế | Đúng/Sai | Giải thích |
|---|----------|------------------------------|-----------------|----------|------------|
| a | Thêm bài tập thứ 3 vào mảng `assignments` của API | Client vẫn nhận và in được bài tập 3 bình thường mà không cần sửa code | Terminal 2 in đủ 3 bài tập | Đúng | Client dùng `List<Assignment>` nhận mảng động nên tự co giãn theo dữ liệu JSON trả về |
| b | Tắt API ở Terminal 1 rồi mới chạy Client | Client sẽ báo lỗi kết nối vì API không chạy | Báo lỗi `HttpRequestException: No connection could be made...` | Đúng | Hai tiến trình độc lập qua mạng; gọi mạng có thể lỗi nếu server ngừng hoạt động |
| c | Sửa `StudentCode: ""` ở Client rồi chạy lại | API sẽ trả mã lỗi 400 Bad Request và Client bị dừng lại | Báo ngoại lệ mã trạng thái `400 (Bad Request)` | Đúng | API có code kiểm tra `string.IsNullOrWhiteSpace`, `EnsureSuccessStatusCode()` bắt lỗi dừng ngay |
| d | Chạy API cổng 5050, Client gọi cổng 5000 | Client gọi 5000 bị lỗi; chỉ cần gõ lệnh truyền URL 5050 là chạy được | Truyền URL `http://localhost:5050` vào lệnh thì Client chạy thành công | Đúng | Địa chỉ API được truyền qua tham số dòng lệnh `args[0]`, không bị viết cứng trong code |

---

## 6. Dự án EduSubmit tự làm (Bước 11 & 12)
- Dự án `EduSubmit-DTC241200004` được tự khởi tạo từ đầu gồm 2 project độc lập:
  - `Submission.Api`: Xây dựng bằng ASP.NET Core Minimal API.
  - `Submission.Client`: Console Application giao tiếp qua `HttpClient`.
- Đã thực hiện Bài mở rộng (Bước 12):
  - API lưu trữ danh sách các bài nộp vào `submissions` (in-memory).
  - Thêm endpoint `GET /api/submissions/{id:guid}` trả về `200 OK` nếu tìm thấy, `404 Not Found` nếu không tồn tại.
  - Client sau khi nộp bài sẽ tự động gọi lại endpoint trên để kiểm tra và in xác nhận từ server.
- Luồng thông điệp giữa hai chương trình:
  ```text
  Client --( HTTP GET /api/assignments )---------> Api  : Request-Response (lấy danh sách bài tập)
  Client <--( 200 OK [JSON Assignments] )---------- Api

  Client --( HTTP POST /api/submissions )--------> Api  : Request-Response (gửi bài nộp)
  Client <--( 201 Created [JSON SubmissionResult] ) Api

  Client --( HTTP GET /api/submissions/{id} )-----> Api  : Request-Response (đọc lại chi tiết lượt nộp)
  Client <--( 200 OK [JSON SubmissionResult] )----- Api
  ```

---

## 7. Khó khăn gặp phải và cách giải quyết
- Lỗi cổng mạng bị chiếm dụng (`Address already in use`): Do tiến trình API cũ chưa tắt hẳn, giải quyết bằng cách nhấn `Ctrl + C` ở terminal cũ hoặc truyền tham số `--urls` sang cổng khác.
- Lỗi gọi API khi chưa bật server (`actively refused it`): Nhận diện được tính chất phụ thuộc mạng của hệ thống phân tán; luôn đảm bảo Server khởi động sẵn sàng trước khi Client thực hiện request.

