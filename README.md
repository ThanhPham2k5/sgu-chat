# 🎓 SGU-Chat
Dự án Chuyên đề / Doanh nghiệp: **Hệ thống Trực tuyến SGU Chat**  
Hệ thống giao tiếp nội bộ tích hợp nghiệp vụ đào tạo dành cho Sinh viên và Giảng viên Trường Đại học Sài Gòn (SGU).

---

## 🏗️ Kiến trúc Hệ thống (Architecture Overview)

Hệ thống được thiết kế theo mô hình **Headless Backend**:

1. **Frontend / Chat Engine:** Sử dụng **Mattermost Open-Source** (đã được cấu hình thương hiệu SGU Chat, bộ màu, kênh mặc định và tích hợp Bot/Webhooks). Đảm nhận toàn bộ giao diện Web, Desktop App và Mobile App. Tham khảo thêm tài liệu tại đây **https://docs.mattermost.com/**.
2. **Backend Services (Spring Boot):** Đảm nhận toàn bộ logic nghiệp vụ SGU (Import sinh viên, lọc từ ngữ thô tục, tra cứu lịch thi `/lichthi`, thông báo tiến độ học tập).
3. **Database (PostgreSQL 15):** Lưu trữ độc lập dữ liệu nghiệp vụ của Spring Boot (`sgu_chat_db`), không can thiệp trực tiếp vào CSDL của Mattermost.
4. **Đồng bộ Team:** Quản lý cấu hình giao diện & tính năng tập trung qua file `mattermost-config/config.json` mounted bằng Docker Volume.

---

## 🛠️ Yêu cầu Môi trường (Prerequisites)

Các thành viên trong nhóm dev cần cài đặt sẵn trên máy:
* [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Kèm Docker Compose)
* [JDK 17+](https://adoptium.net/) (Khuyên dùng Eclipse Temurin 17)
* [IntelliJ IDEA](https://www.jetbrains.com/idea/)
* [TablePlus](https://tableplus.com/) (Dùng để quản lý CSDL PostgreSQL - tùy chọn)

---

## 🚀 Hướng dẫn Khởi chạy Dự án cho Team (Quickstart)

### 1. Kéo mã nguồn về máy local
```bash
git clone https://github.com/ThanhPham2k5/sgu-chat.git
cd sgu-chat
```

### 2. Bật Hạ tầng Docker (Mattermost + PostgreSQL)
Mở Terminal tại thư mục gốc dự án và chạy duy nhất lệnh:
```bash
docker compose up -d
```
Kết quả:
* **Mattermost (Chat Engine)**: http://localhost:8065 (Tự động nạp giao diện SGU Chat từ file config.json).
* **PostgreSQL Database**: localhost:5432 (Username: sgu_user | Password: sgu_password | Database: sgu_chat_db).

## ⚙️ Cấu hình Spring Boot (application.properties)
Mở file src/main/resources/application.properties trong IntelliJ và dán các thông số kết nối sau:
```bash
spring.application.name=chat
server.port=8080

# --- PostgreSQL Configuration ---
spring.datasource.url=jdbc:postgresql://localhost:5432/sgu_chat_db
spring.datasource.username=sgu_user
spring.datasource.password=sgu_password
spring.datasource.driver-class-name=org.postgresql.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# --- Mattermost Integration Configuration ---
mattermost.server.url=http://localhost:8065
mattermost.bot.token=YOUR_BOT_ACCESS_TOKEN_HERE
```

## 🔄 Thiết lập Giao diện & Tích hợp Mattermost (System Console)

*(Lưu ý: Để đảm bảo sự ổn định của Docker Container, dự án không đồng bộ trực tiếp file `config.json`. Các thiết lập Mattermost sẽ được cấu hình trực tiếp qua Web UI).*

Khi khởi chạy hệ thống lần đầu (hoặc khi reset lại Docker), bạn cần thực hiện các bước sau:

### 1. Khởi tạo Admin & Bật Tích hợp (Integrations)
- Truy cập `http://localhost:8065` và tạo tài khoản Admin (VD: `admin@sgu.edu.vn` / `admin`).
- Vào góc trên bên trái chọn **System Console** $\rightarrow$ **Integrations** $\rightarrow$ **Integration Management**.
- Chuyển tất cả các mục sang **`True`** (Incoming/Outgoing Webhooks, Slash Commands, Bot Accounts) $\rightarrow$ Bấm **Save**.

### 2. Tùy chỉnh Thương hiệu SGU
- Tại System Console, vào **Site Configuration** $\rightarrow$ **Customization**.
- Đổi **Site Name** thành `SGU Chat` và thay đổi màu sắc/logo tùy ý.

### 3. Khởi tạo SGU Bot & Đồng bộ Backend
Để Spring Boot có thể gửi tin nhắn tự động vào Mattermost, hệ thống cần một Bot đại diện:
1. Quay lại màn hình Chat chính $\rightarrow$ Chọn Menu 9 dấu chấm (góc trên trái) $\rightarrow$ **Integrations** $\rightarrow$ **Bot Accounts**.
2. Chọn **Add Bot Account** $\rightarrow$ Đặt tên `sgu-bot` (Role: `System Admin`) $\rightarrow$ Bấm Create.
3. **Copy chuỗi Bot Access Token** và gửi cho các thành viên trong team.
4. Mọi người trong team dán Token này vào file `src/main/resources/application.properties` của Spring Boot:
   ```properties
   mattermost.bot.token=CHUỖI_TOKEN_VỪA_COPY

## 🗄️ Kết nối CSDL bằng TablePlus
* Host: localhost
* Port: 5432
* User: sgu_user
* Password: sgu_password
* Database: sgu_chat_db

## 🌿 Quy định Git & Branching:
* Branch chính: main (chỉ chứa code đã chạy ổn định).
* Mọi tính năng mới tiến hành tạo branch phụ: feature/ten-tinh-nang (ví dụ: feature/profanity-filter, feature/lichthi-command).
* Commit message theo chuẩn Conventional Commits:
```bash
git commit -m "type(scope): message details..." 
```
