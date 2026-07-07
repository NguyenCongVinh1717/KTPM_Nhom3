# Sportify — Premium Sportswear E-commerce

Website bán đồ thể thao, môn Kiểm thử phần mềm — Nhóm 3.

Repo gồm 2 phần độc lập trong cùng 1 thư mục gốc:
- **`backend/`** — Spring Boot 4 (Java 21), REST API, JWT auth, đăng nhập Google, thanh toán VNPay (sandbox), tư vấn AI (Spring AI + Gemini, RAG trên dữ liệu sản phẩm).
- **`frontend/`** — HTML/CSS/JS thuần (không framework), gọi API qua đường dẫn tương đối `/api/...`.

## 1. Yêu cầu môi trường trước khi bắt đầu

| Thành phần | Phiên bản | Ghi chú |
|---|---|---|
| Java (JDK) | 21 | Bắt buộc — kiểm tra bằng `java -version` |
| Git | Bất kỳ | Để clone repo |
| SQL Server | Bất kỳ bản còn hỗ trợ | Cài SQL Server + SSMS (SQL Server Management Studio) để tạo database |
| Trình duyệt | Chrome/Edge mới | Để mở frontend |
| IDE | IntelliJ IDEA (backend), VS Code (frontend) | Không bắt buộc dùng đúng 2 IDE này, chỉ là gợi ý |

Không cần cài Maven riêng (đã có `mvnw`/`mvnw.cmd` trong `backend/`), không cần cài Node/npm (frontend không qua bước build).

## 2. Clone dự án

```bash
git clone https://github.com/NguyenCongVinh1717/KTPM_Nhom3.git
cd KTPM_Nhom3
```

## 3. Cấu trúc thư mục

```
KTPM_Nhom3/
├── backend/
│   ├── src/main/java/SaleManagement/VinhNguyen/...
│   ├── src/main/resources/
│   │   ├── application.properties.example   ← bản mẫu, có sẵn trong repo
│   │   └── application.properties           ← BẠN TỰ TẠO (xem mục 4), không có sẵn trong repo
│   ├── pom.xml
│   └── mvnw / mvnw.cmd
├── frontend/
│   ├── products.html, product_detail.html, cart.html, login.html...
│   ├── admin_*.html (các trang quản trị)
│   └── api.js
├── .gitignore
└── README.md
```

## 4. Cấu hình Backend

### 4.1. Tạo file cấu hình thật từ file mẫu

```bash
cd backend/src/main/resources
cp application.properties.example application.properties        # Nếu máy là macOS/Linux/Git Bash
copy application.properties.example application.properties       # Nếu máy là Windows CMD
```

Mở `application.properties` vừa tạo, điền giá trị **thật của bạn** (những chỗ ghi `your-...` là chỗ cần điền):

```properties
# Database — trỏ về SQL Server của bạn (nếu cài local thì dùng localhost)
spring.datasource.url=jdbc:sqlserver://localhost:1433;databaseName=SaleManagement;encrypt=true;trustServerCertificate=true;
spring.datasource.username=sa
spring.datasource.password=<mat_khau_SQL_Server_cua_ban>

# JWT — tự đặt 1 chuỗi ngẫu nhiên bất kỳ, càng dài càng tốt (ví dụ dùng https://randomkeygen.com)
jwt.secret=<chuoi_bi_mat_tuy_y>
jwt.expiration=3600000

# Mail (gửi OTP quên mật khẩu) — dùng Gmail App Password, KHÔNG phải mật khẩu Gmail thường
# Tạo tại: Google Account → Security → 2-Step Verification → App passwords
spring.mail.username=<email_cua_ban>@gmail.com
spring.mail.password=<gmail_app_password_16_ky_tu>

# VNPay Sandbox — đăng ký merchant test miễn phí tại https://sandbox.vnpayment.vn
vnpay.tmnCode=<tmn_code_sandbox_cua_ban>
vnpay.hashSecret=<hash_secret_sandbox_cua_ban>
vnpay.baseUrl=https://sandbox.vnpayment.vn/paymentv2/vpcpay.html
vnpay.returnUrl=http://localhost:8081/order/vnpay-callback

# Google Gemini — TÙY CHỌN, chỉ cần nếu muốn dùng tính năng tư vấn AI (bỏ qua nếu không cần)
# Lấy key miễn phí tại https://aistudio.google.com/apikey
spring.ai.google.genai.api-key=<gemini_api_key>
spring.ai.google.genai.embedding.api-key=<gemini_api_key>
```

> ⚠️ File `application.properties` (bản thật) sẽ **không bao giờ xuất hiện** khi bạn `git status` hay `git add .` — đây là chủ đích (đã khai trong `.gitignore`), không phải lỗi. Không commit file này lên Git.

### 4.2. Tạo database

Mở SQL Server Management Studio, tạo 1 database rỗng tên đúng `SaleManagement` (khớp với `databaseName=SaleManagement` ở trên). **Không cần tạo bảng thủ công** — Spring Boot tự tạo toàn bộ bảng khi khởi động lần đầu nhờ:
```properties
spring.jpa.hibernate.ddl-auto=update
```

### 4.3. Chạy backend

**Cách 1 — IntelliJ IDEA:** File → Open → chọn thư mục `backend/` → chờ IntelliJ tải Maven dependency xong → chạy class `VinhNguyenApplication.java` (nút ▶ màu xanh).

**Cách 2 — dòng lệnh:**
```bash
cd backend
./mvnw spring-boot:run        # macOS/Linux/Git Bash
mvnw.cmd spring-boot:run      # Windows CMD/PowerShell
```

Backend chạy tại `http://localhost:8081`. Kiểm tra nhanh đã chạy đúng:
```bash
curl http://localhost:8081/products
```
Trả về JSON (rỗng `[]` cũng được, miễn không báo lỗi kết nối) là backend đã lên thành công.

## 5. Chạy Frontend

`frontend/api.js` gọi API qua đường dẫn **tương đối** `/api/...` (dòng đầu file: `const API_BASE = "/api";`) — vì môi trường triển khai thật dùng Nginx phục vụ frontend và proxy `/api/` sang backend trên **cùng 1 domain/port**. Khi mới clone về máy cá nhân (chưa có Nginx), có 2 cách chạy:

### Cách A — Test nhanh, không cần cài Nginx (khuyên dùng)

Mở `frontend/api.js`, đổi dòng đầu:
```js
const API_BASE = "/api";
```
thành:
```js
const API_BASE = "http://localhost:8081";
```
Sau đó mở `frontend/products.html` bằng extension **Live Server** trong VS Code (chuột phải file → "Open with Live Server"), hoặc mở trực tiếp bằng trình duyệt.

⚠️ **Nhớ đổi lại thành `/api`** trước khi commit — không push nhầm bản đã đổi `API_BASE` lên Git, sẽ làm hỏng bản chạy qua Nginx của người khác.

### Cách B — Giống hệt môi trường thật (cần cài Nginx)

Cài Nginx, cấu hình 1 server block trỏ `root` vào thư mục `frontend/` và `location /api/` proxy sang `http://localhost:8081` (bỏ tiền tố `/api` khi chuyển tiếp). Cách này phù hợp khi muốn test đúng y hệt bản deploy thật trước khi đưa lên server.

## 6. Tài khoản dùng thử

Nếu database mới tạo chưa có tài khoản admin, đăng ký tài khoản mới qua `register.html`, sau đó mở SSMS sửa cột `role` của user đó (bảng `User`) thành `ADMIN` để vào được các trang quản trị (`adminDashboard.html`, `admin_product.html`, `admin_brand.html`...).

**Đăng nhập bằng Google** yêu cầu domain bạn đang chạy (`localhost` hoặc domain thật sau này) phải được thêm vào mục **Authorized JavaScript origins** trong Google Cloud Console, ứng với `client_id` đang khai báo trong `login.html`. Nếu chưa có quyền truy cập Google Cloud Console của project, tính năng đăng nhập thường (email/mật khẩu) vẫn hoạt động bình thường không cần bước này.

## 7. Một số endpoint chính

| Endpoint | Chức năng |
|---|---|
| `GET /products` | Danh sách sản phẩm (phân trang, lọc theo giá/thương hiệu/màu/size) |
| `GET /products/{id}` | Chi tiết sản phẩm |
| `POST /auth/login`, `/auth/register`, `/auth/google` | Xác thực |
| `POST /cart/add`, `GET /cart` | Giỏ hàng |
| `POST /order` | Tạo đơn hàng |
| `POST /order/vnpay-callback` | Nhận kết quả thanh toán từ VNPay |
| `POST /ai/consult` | Tư vấn AI (cần cấu hình Gemini API key ở mục 4.1) |
| `GET/POST/PUT/DELETE /brands`, `/colors`, `/sizes` | Quản lý danh mục (yêu cầu quyền ADMIN cho POST/PUT/DELETE) |

## 8. Việc KHÔNG cần làm

- Không cần tạo `backend/target/` — tự sinh khi chạy `mvnw`, không commit lên Git.
- Không cần cài Maven riêng — dùng `mvnw`/`mvnw.cmd` có sẵn.
- Không cần cài Node/npm — frontend không qua build.
- Không cần tự tạo file `.p12`/`.crt`/`.key` — chỉ cần thiết nếu muốn test HTTPS cục bộ, không bắt buộc để chạy được chức năng chính.

## 9. Khắc phục sự cố thường gặp

| Lỗi | Nguyên nhân | Cách sửa |
|---|---|---|
| `Port 8081 was already in use` | Có tiến trình backend cũ chưa tắt | Windows: Task Manager tìm `java.exe` tắt đi. Linux/Mac: `lsof -i :8081` rồi `kill -9 <PID>` |
| Frontend gọi API lỗi 404 / Network Error / CORS | `API_BASE` trong `api.js` chưa đúng | Xem lại mục 5 — dùng Cách A khi chưa có Nginx |
| Backend không kết nối được database | Sai `spring.datasource.url`/`username`/`password`, hoặc SQL Server chưa bật TCP/IP | Kiểm tra lại `application.properties`; mở SQL Server Configuration Manager bật giao thức TCP/IP nếu cần |
| Ảnh sản phẩm không hiển thị | Thư mục lưu ảnh trống hoặc `upload.path` cấu hình sai | Kiểm tra dòng `upload.path=uploads/` trong `application.properties`, đảm bảo thư mục đó tồn tại |
| Lỗi khi gọi `/ai/consult`: `models/... is not found` | Sai tên model Gemini | Dùng đúng `gemini-embedding-001` (không phải `text-embedding-004` đã bị Google khai tử) |
| Không đăng nhập được bằng Google | Domain chưa được thêm vào Google Cloud Console | Xem mục 6, hoặc dùng đăng nhập email/mật khẩu thường thay thế |

