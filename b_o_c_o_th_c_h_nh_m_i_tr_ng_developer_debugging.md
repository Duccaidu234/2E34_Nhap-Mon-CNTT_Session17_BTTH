# BÁO CÁO THỰC HÀNH MÔ HÌNH BROWSER PROFILE, EXTENSIONS VÀ DEBUGGING DEVTOOLS

**Vị trí:** Developer Intern  
**Nhiệm vụ:** Thiết lập môi trường làm việc & Chẩn đoán lỗi trang `test.internal-tool.com`  
**Ngày thực hiện:** 02/11/2026  

---

## PHẦN A – THIẾT LẬP PROFILE VÀ BOOKMARK CHUẨN DEV

### 1. Hiện tượng xung đột khi dùng chung 1 Profile
Nếu tiếp tục dùng chung 1 Profile duy nhất cho cả GitHub cá nhân và GitHub công ty, **Xung đột phiên làm việc (Session Conflict)** sẽ xảy ra. 
* **Nguyên nhân:** Trình duyệt lưu trữ Cookie xác thực phiên đăng nhập (như `user_session`) vào chung một thư mục dữ liệu (`User Data/Default`). 
* **Hậu quả:** Khi đăng nhập tài khoản GitHub công ty, Cookie này sẽ **ghi đè (overwrite)** lên Cookie tài khoản GitHub cá nhân trên cùng domain `github.com`. Điều này khiến bạn bị văng đăng xuất (Logout) ở tài khoản cá nhân, hoặc gây nhầm lẫn Identity (Commit code cá nhân bằng Email công ty và ngược lại), vi phạm quy định an toàn thông tin.

---

### 2. Mô hình 3 Profile chuẩn Developer

| STT | Tên đặt (Profile Name) | Mục đích sử dụng |
| :---: | :--- | :--- |
| **1** | **`Personal`** | Phục vụ các hoạt động cá nhân (mạng xã hội, email riêng, GitHub cá nhân, giải trí). Tách biệt hoàn toàn với tài nguyên công ty. |
| **2** | **`Company_Work`** | Phục vụ công việc dự án chính thức (GitHub công ty, Jira, Slack, tài liệu nội bộ `test.internal-tool.com`). Đăng nhập tài khoản doanh nghiệp. |
| **3** | **`Sandbox/Test`** | Phục vụ thử nghiệm ứng dụng, chạy thử nghiệm tài khoản đa vai trò (Admin/User), cài đặt các Extension phụ trợ mà không làm chậm Profile chính. |

---

### 3. Cấu trúc cây thư mục Bookmark chuẩn mực (Ctrl + Shift + O)

* 📁 **`01_Internal_System`** *(Hệ thống nội bộ & Quản lý công việc)*
  1. `Company GitHub Repository` (`https://github.com/company-org`)
  2. `Internal Testing Tool` (`https://test.internal-tool.com`)
* 📁 **`02_Dev_Documentation`** *(Tài liệu kỹ thuật & API Spec)*
  1. `MDN Web Docs` (`https://developer.mozilla.org`)
  2. `Project API Specification / Swagger` (`https://api-docs.internal-tool.com`)
* 📁 **`03_Dev_Tools`** *(Công cụ hỗ trợ lập trình & Kiểm thử)*
  1. `JSON Formatter / Validator` (`https://jsonlint.com`)
  2. `RegEx101 - Regular Expression Tester` (`https://regex101.com`)

---

### 4. Quy trình xử lý khi đồng nghiệp mượn máy tính 5 phút
* **Giải pháp:** Sử dụng **`Guest Profile` (Chế độ Khách)**.
* **Giải thích:** 
  * `Guest Profile` tạo ra một môi trường hoàn toàn sạch: không truy cập được thanh Bookmark, không tự động điền (Autofill) mật khẩu, không xem được Lịch sử duyệt web (History) hay Session Cookie của Profile `Personal`/`Company_Work`.
  * Khác với `Incognito Mode` (vẫn truy cập được Bookmark và Extension của Profile hiện tại), `Guest Profile` đảm bảo riêng tư 100% và tự động xóa sạch dữ liệu duyệt web ngay khi đóng cửa sổ.

---

## PHẦN B – CHỌN ĐÚNG EXTENSION HỖ TRỢ

### 1. Phân loại Extension cho chuỗi JSON thô
* **Nhóm lựa chọn:** **Dev Suite** (Tiện ích hỗ trợ phát triển phần mềm).
* **Tên tiện ích cụ thể:** **JSON Formatter** (hoặc *JSON Viewer / JSON Editor*).
* **Giải thích lý do 2 nhóm còn lại không phù hợp:**
  * **Productivity Suite** (Tiện ích tăng năng suất như *Notion Web Clipper, Todoist*): Chỉ hỗ trợ quản lý công việc/ghi chú, không có chức năng parse hoặc định dạng trực quan dữ liệu cấu trúc JSON thô.
  * **Security Suite** (Tiện ích an ninh như *uBlock Origin, LastPass*): Chỉ tập trung vào chặn quảng cáo, bảo mật mật khẩu hoặc chống theo dõi, không có tính năng can thiệp hiển thị dữ liệu API.

### 2. Cấu hình quyền Site Access
* **Cấu hình chuẩn:** **On click** (Hoặc *On specific sites* - chỉ áp dụng trên domain cần dùng).
* **Giải thích theo quy tắc quản trị Extensions:** Theo nguyên tắc tối thiểu quyền hạn và an toàn thông tin (Least Privilege Principle), đặt **On click** ngăn tiện ích tự động đọc hoặc can thiệp dữ liệu (DOM/Cookies) trên toàn bộ các trang web khác (*On all sites*), giúp phòng tránh nguy cơ lộ lọt dữ liệu nhạy cảm hoặc bị dính tấn công Supply Chain qua Extension.

---

## PHẦN C – PHÂN TÍCH LUỒNG HTTP REQUEST

### 1. Thứ tự 4 giai đoạn xử lý khi truy cập `test.internal-tool.com`
1. **Phân giải tên miền (DNS Resolution):** Trình duyệt gửi truy vấn tìm địa chỉ IP tương ứng với domain `test.internal-tool.com`.
2. **Thiết lập kết nối (TCP Handshake / TLS Negotiation):** Trình duyệt tạo kết nối an toàn tới máy chủ web qua cổng 80 (HTTP) hoặc 443 (HTTPS).
3. **Gửi HTTP Request & Nhận HTTP Response:** Trình duyệt gửi yêu cầu `GET /` kèm theo các Header; Máy chủ phản hồi mã trạng thái (ví dụ `200 OK`) và nội dung HTML.
4. **Biên dịch & Dựng giao diện (Parsing & Rendering):** Trình duyệt phân tích HTML, tải các tài nguyên phụ (CSS, JS, Images) và tiến hành dựng cấu trúc DOM / Render Tree lên màn hình.

---

### 2. Bảng so sánh mã trạng thái HTTP (HTTP Status Codes)

| Mã trạng thái | Ý nghĩa | Lỗi nằm ở đâu? | Hướng debug ban đầu |
| :---: | :--- | :--- | :--- |
| **`404 Not Found`** | Tài nguyên được yêu cầu không tồn tại trên máy chủ. | **Client-side** (gõ sai URL) hoặc **Server Route Configuration** (chưa định tuyến đúng). | • Kiểm tra lại tính chính xác của đường dẫn (URL Path).<br>• Kiểm tra lại file/endpoint trên Server xem đã được deploy hoặc cấu hình Router chưa. |
| **`500 Internal Server Error`** | Máy chủ gặp lỗi nội bộ không xác định và không thể hoàn tất Request. | **Server-side** (Lỗi mã nguồn backend, crash database, unhandled exception). | • Kiểm tra log hệ thống phía Backend (Server Logs/Application Logs).<br>• Kiểm tra kết nối tới Database hoặc dịch vụ thứ ba của Server. |

---

### 3. Lựa chọn Tab DevTools để xem mã trạng thái HTTP
* **Tab cần mở:** **Network Tab**.
* **Lý do không chọn các Tab còn lại:**
  * **Elements Tab:** Chỉ hiển thị cấu trúc cây DOM và mã hiệu chỉnh CSS.
  * **Console Tab:** Hiển thị nhật ký log, lỗi JS runtime và các câu lệnh xuất ra từ `console.log()`, không phản ánh đầy đủ thông số Header/Status code của toàn bộ các HTTP Request.
  * **Application Tab:** Chỉ dùng để quản lý bộ nhớ lưu trữ phía Client (Cookies, LocalStorage, Cache Storage, SessionStorage).

---

## PHẦN D – CHẨN ĐOÁN CACHE VÀ COOKIES

### 1. Chẩn đoán lỗi F5 vẫn thấy giao diện CSS cũ
* **Chẩn đoán nguyên nhân:** Trình duyệt đang tải lại file CSS từ **Browser Cache (Memory/Disk Cache)** tĩnh được lưu trữ từ phiên trước đó thay vì gửi yêu cầu tải bản mới từ Server.
* **Thao tác khắc phục:** 
  * Bấm tổ hợp phím **`Ctrl + F5`** (Windows) hoặc **`Cmd + Shift + R`** (Mac) để thực hiện **Hard Reload**.
  * Hoặc mở DevTools (F12) $\rightarrow$ Nhấp chuột phải vào nút Reload trên trình duyệt $\rightarrow$ Chọn **`Empty Cache and Hard Reload`**.

---

### 2. Rủi ro bảo mật khi Cookie `session_id` thiếu thuộc tính `HttpOnly`
* **Rủi ro bảo mật:** Mã script chạy ở phía Client (JavaScript) có thể đọc và truy xuất được giá trị của Cookie này thông qua câu lệnh `document.cookie`.
* **Loại tấn công có thể xảy ra:** Tấn công **XSS (Cross-Site Scripting)**. Kẻ tấn công có thể chèn mã độc JS vào trang web để lấy cắp `session_id` của bạn và mạo danh phiên đăng nhập (Session Hijacking).

---

### 3. Thao tác xóa phiên đăng nhập nhưng giữ nguyên Cache
* **Thao tác chính xác:** 
  1. Nhấn **F12** để mở DevTools $\rightarrow$ Chuyển sang **Application Tab**.
  2. Mở mục **Cookies** bên thanh menu trái $\rightarrow$ Chọn domain `test.internal-tool.com`.
  3. Tìm Cookie lưu phiên đăng nhập (ví dụ: `session_id`, `token`) $\rightarrow$ Chuột phải chọn **Delete** (hoặc bấm biểu tượng `Clear All Cookies`).
  * *Lưu ý: Không dùng tính năng Clear Browsing Data vì hành động đó sẽ xóa sạch cả Disk Cache/Images.*

---

## PHẦN E – DEBUG BẰNG INSPECT ELEMENT VÀ NETWORK TAB

### 1. Thử sửa màu nút "Đăng nhập" trên trình duyệt
1. Nhấp chuột phải vào nút **"Đăng nhập"** trên giao diện trang web $\rightarrow$ Chọn **Inspect (Kiểm tra)**.
2. Trình duyệt tự động chuyển đến tab **Elements** và highlight dòng mã HTML đại diện cho nút bấm.
3. Nhìn sang khung **Styles** ở cột bên phải, tìm thuộc tính `background-color` (hoặc `color`).
4. Nhấp vào ô mã màu hoặc trực tiếp thay đổi mã Hex/RGB sang mã màu chuẩn thiết kế (ví dụ `#0052CC`) để kiểm tra hiển thị trực quan trước khi gửi phản hồi cho Dev.

---

### 2. Quy trình 4 Bước Debugging trên tab Network cho Request API báo đỏ

```
+-----------------------------------------------------------------------------------+
|                        QUY TRÌNH 4 BƯỚC DEBUG NETWORK REQUEST                      |
+-----------------------------------------------------------------------------------+
|  [Bước 1: Check Headers]     --->  Kiểm tra Request URL, Method & Status Code     |
|  [Bước 2: Check Payload]     --->  Kiểm tra Params / Body JSON gửi đi             |
|  [Bước 3: Check Preview/Resp]--->  Đọc chi tiết thông điệp lỗi trả về từ Server   |
|  [Bước 4: Check Timing]      --->  Đánh giá thời gian phản hồi (Latency/Timeout)  |
+-----------------------------------------------------------------------------------+
```

* **Bước 1: Kiểm tra tổng quan (Headers Tab)**
  * Nhấp vào Request API báo đỏ. Kiểm tra **Request URL** (đường dẫn đã đúng chưa), **Request Method** (`GET`, `POST`, `PUT`...), và **Status Code** chính xác là mã bao nhiêu (`400`, `401`, `403`, `500`...).
* **Bước 2: Kiểm tra dữ liệu gửi đi (Payload Tab / Request Payload)**
  * Kiểm tra các tham số Query Parameters hoặc dữ liệu JSON trong Request Body xem có bị thiếu trường, sai định dạng dữ liệu (data type) hay không.
* **Bước 3: Kiểm tra phản hồi từ Server (Preview / Response Tab)**
  * Xem nội dung thông điệp lỗi do Server trả về (ví dụ: `{"error": "Invalid Token"}`, `{"message": "Field 'email' is required"}`) để xác định nguyên nhân chi tiết.
* **Bước 4: Kiểm tra thời gian xử lý (Timing Tab)**
  * Kiểm tra xem Request bị lỗi do máy chủ từ chối hay do **Time-out** (thời gian phản hồi vượt quá ngưỡng cho phép do mạng yếu hoặc Server xử lý quá lâu).

---

### 3. Thao tác lọc API và bỏ qua bộ nhớ đệm trên Network Tab
Để phục vụ bài test theo đúng yêu cầu:
1. **Lọc chỉ xem lời gọi API:** Tại thanh công cụ của Tab Network, nhấp vào bộ lọc **`Fetch/XHR`**. (Thao tác này sẽ ẩn toàn bộ các tệp tài nguyên tĩnh như `.png`, `.jpg`, `.css`, `.js`).
2. **Đảm bảo dữ liệu mới nhất (Không lấy từ Cache):** Tích chọn ô **`Disable cache`** nằm ngay trên thanh công cụ của Tab Network (Lưu ý: tính năng này có hiệu lực khi DevTools đang mở).