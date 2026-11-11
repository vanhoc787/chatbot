TRỢ LÝ ẢO NỘI BỘ AGRIBANK BẮC LONG AN
=====================================

1. Chức năng
------------
- Đăng nhập và xác thực người dùng bằng JWT.
- Trò chuyện với trợ lý AI qua OpenAI; hỏi đáp có thể sử dụng nội dung tài liệu
  đã được nạp và tìm kiếm theo vector (RAG).
- Lưu lịch sử hội thoại và tin nhắn vào PostgreSQL.
- Quản lý người dùng, vai trò, trạng thái tài khoản và đổi mật khẩu.
- Nạp nhiều tài liệu Word .docx, trích xuất nội dung, chia đoạn và tạo embeddings.
- Xem và xóa tài liệu đã nạp.

2. Công nghệ và yêu cầu
----------------------
- Backend: Node.js, Express, TypeORM.
- Frontend: React, Vite.
- Cơ sở dữ liệu: PostgreSQL; chức năng tìm kiếm vector cần extension pgvector.
- Node.js phiên bản 20.19+ hoặc 22.12+.
- Cần OpenAI API key để tạo câu trả lời và embeddings.

3. Chuẩn bị cấu hình
--------------------
Clone repository, sau đó tạo file BE/.env (không commit file này) với các biến:

  PORT=3000
  DB_HOST=localhost
  DB_PORT=5432
  DB_USERNAME=postgres
  DB_PASSWORD=mat_khau_postgres
  DB_DATABASE=chatbot
  JWT_SECRET=thay_bang_chuoi_bi_mat_dai_va_ngau_nhien
  JWT_EXPIRES_IN=1d
  OPENAI_API_KEY=your_openai_api_key
  OPENAI_API_BASE=https://api.openai.com
  OPENAI_EMBED_MODEL=text-embedding-3-small
  CHATGPT_MODEL=gpt-4o-mini

Tạo database PostgreSQL chatbot và đảm bảo server PostgreSQL đã cài đặt pgvector.
Backend kết nối đến PostgreSQL theo các biến DB_*; không dùng DATABASE_URL.
Frontend hiện gọi API tại http://localhost:3000 (FE/src/config/api.js), nên
backend mặc định cần chạy ở cổng 3000.

4. Chạy ứng dụng trên máy mới
-----------------------------
Mở hai terminal tại thư mục repository.

Terminal 1 - backend:

  cd BE
  npm install
  node src/server.js

Terminal 2 - frontend:

  cd FE
  npm install
  npm run dev

Mở địa chỉ Vite hiện trong terminal (mặc định http://localhost:5173).
Có thể dùng npm start trong BE để chạy backend không tự restart khi sửa mã;
trong FE, npm run build để tạo bản build production.

5. Cơ sở dữ liệu và tài khoản
-----------------------------
Backend chỉ khởi động sau khi kết nối được PostgreSQL. Các migration được
TypeORM nạp từ BE/src/migrations và hiện tại là migration cập nhật schema
(bao gồm cột header_content và kiểu vector); chúng không tạo đầy đủ schema cơ
sở cho một database trống. Vì vậy, trước khi npm run migration:run, cần
chuẩn bị database có schema nền phù hợp với các entity trong BE/src/models.

Khi database đã có schema nền:

  cd BE
  npm run migration:run

Lệnh tạo migration chỉ dùng khi thay đổi entity và đã có database để đối chiếu:

  cd BE
  npm run migration:generate -- src/migrations/TenMigration
  npm run migration:run

Lần đầu backend kết nối thành công, ứng dụng tự tạo tài khoản quản trị:
username: admin
mật khẩu mặc định: Admin@6421

Đổi mật khẩu mặc định ngay sau lần đăng nhập đầu tiên; không sử dụng tài khoản
mặc định trên môi trường thật.

6. Ghi chú
----------
- Nạp tài liệu chấp nhận .docx; giao diện giới hạn 10 file và 10 MB mỗi file.
  Backend giới hạn kích thước từng file 50 MB.
- Backend lưu file nạp trong BE/uploads; đây là dữ liệu runtime, không đưa lên Git.
- Không đưa BE/.env, API key, mật khẩu database hoặc JWT secret lên GitHub.



Chạy Migrations: npx typeorm migration:generate -d src/config/dataSource.js src/migrations/InitMigration
                 npx typeorm migration:run -d src/config/dataSource.js







chay SQL tạo 1 cột header_content: 
- ALTER TABLE "Documents" ADD COLUMN header_content TEXT;
- UPDATE "Documents" SET header_content = LEFT(full_text, 250)
