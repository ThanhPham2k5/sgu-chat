# 🎓 SGU-Chat
Dự án Chuyên đề / Doanh nghiệp: **Hệ thống Trực tuyến SGU Chat**  
Hệ thống giao tiếp nội bộ tích hợp nghiệp vụ đào tạo dành cho Sinh viên và Giảng viên Trường Đại học Sài Gòn (SGU).

---

## 🏗️ Kiến trúc Hệ thống (Architecture Overview)

Hệ thống được thiết kế theo mô hình **Headless Backend**:

1. **Frontend / Chat Engine:** Sử dụng **Mattermost Open-Source** (đã được cấu hình thương hiệu SGU Chat, bộ màu, kênh mặc định và tích hợp Bot/Webhooks). Đảm nhận toàn bộ giao diện Web, Desktop App và Mobile App.
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
git clone [https://github.com/ThanhPham2k5/sgu-chat.git](https://github.com/ThanhPham2k5/sgu-chat.git)
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

## 🔄 Quy trình Cập nhật & Đồng bộ Giao diện Mattermost (config.json)
Khi bất kỳ thành viên nào vào System Console trên Mattermost (http://localhost:8065) để chỉnh sửa giao diện, logo, bộ màu hay cài đặt Webhook/Bot:

### 1. Trích xuất cấu hình mới nhất ra ngoài Repo:
```bash
docker cp sgu-chat-mattermost:/mm/mattermost/config/config.json ./mattermost-config/config.json
```

### 2. Commit và Push lên GitHub:
```bash
git add mattermost-config/config.json
git commit -m "chore: update mattermost system console settings"
git push origin main
```

### 3. Các thành viên còn lại cập nhật:
```bash
git pull origin main
docker compose restart sgu-chat-mattermost
```

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