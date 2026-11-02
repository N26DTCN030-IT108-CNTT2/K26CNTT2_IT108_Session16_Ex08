# K26CNTT2_IT108_Session16_Ex08
|
# 🏥 Bài 8: Vá Lỗ Hổng Bảo Mật Cookie Bệnh Án Và Khắc Phục Lỗi Cache Thuốc Cũ

> **Môn học:** Nhập Môn Công Nghệ Thông Tin (IT108)  
> **Sinh viên thực hiện:** Lê Hồ Thanh Thảo  
> **Mã sinh viên:** N26DTCN030  
> **Lớp:** D26TXCN02-N (K26CNTT2)  
> **Hệ thống:** Phòng khám Đa khoa CarePulse (`carepulse.vn`)

---

## 📌 PHẦN 1: PHÂN TÍCH LỖ HỔNG & NGUYÊN NHÂN SỰ CỐ

### 1. Cơ chế Browser Cache khiến trình duyệt hiển thị bảng giá cũ
* **Cơ chế hoạt động:** Trình duyệt mặc định áp dụng cơ chế **Browser Cache** để lưu bản sao của các tệp tài nguyên tĩnh (`medicine_catalog.css`, `pricing_data.json`) trên bộ nhớ tạm (Disk Cache hoặc Memory Cache) ở máy khách nhằm tối ưu tốc độ và giảm số lượng HTTP Request gửi đến máy chủ.
* **Nguyên nhân dính dữ liệu cũ:** Khi bác sĩ nhấn `F5` (Normal Reload), trình duyệt không gửi yêu cầu lấy file mới từ Server mà tự động nạp file từ bộ nhớ đệm cục bộ (hiển thị cột Size là `from disk cache` hoặc `from memory cache` với thời gian nạp cực nhanh 2ms). Do Server chưa cấu hình cơ chế vô hiệu hóa cache (như `Cache-Control: no-cache` hoặc Cache Busting), trình duyệt tiếp tục dùng bản sao giá cũ của tuần trước.

---

### 2. Hiểm họa an ninh từ việc thiếu cờ HttpOnly & Secure (Session Hijacking)

#### 🔹 Câu lệnh JavaScript đánh cắp Token khi bị tấn công XSS:
Nếu hacker nhúng được mã script độc hại vào trang web, hacker có thể trích xuất chuỗi Token phiên làm việc bằng câu lệnh:
```javascript
// Đọc toàn bộ Cookie không có cờ HttpOnly từ bộ nhớ trình duyệt
const doctorToken = document.cookie;
console.log("Token bị đánh cắp:", doctorToken);
```
⚠️ Hiểm họa Session Hijacking đối với hồ sơ bệnh án:
Khi thiếu cờ HttpOnly: Mã độc XSS có thể truy cập trực tiếp vào document.cookie để lấy cắp token carepulse_doctor_session, gửi về server của hacker. Kẻ tấn công có thể giả mạo bác sĩ đăng nhập hệ thống mà không cần mật khẩu.

Khi thiếu cờ Secure: Mẩu tin Cookie bị truyền qua các kết nối HTTP không mã hóa (như Wi-Fi công cộng). Kẻ tấn công trên cùng mạng có thể nghe lén gói tin (Sniffing/Man-in-the-Middle) để lấy cắp Session Token.

Hậu quả: Hacker chiếm toàn bộ quyền kiểm soát tài khoản bác sĩ, có thể xem, chỉnh sửa, xóa hồ sơ bệnh án bảo mật của bệnh nhân hoặc sửa đổi đơn thuốc, gây ra hậu quả y tế và pháp lý nghiêm trọng.


PHẦN 2: HOÀN THIỆN GIẢI PHÁP & THAO TÁC DEVTOOLS
3. Hướng dẫn thao tác làm mới cưỡng bức (Hard Reload)
Để cập nhật danh mục thuốc mới ngay lập tức mà không cần xóa toàn bộ lịch sử duyệt web:

Thao tác phím tắt: Bấm Ctrl + Shift + R (hoặc Ctrl + F5; trên macOS: Cmd + Shift + R).

Tác dụng: Ép trình duyệt bỏ qua bộ nhớ đệm (Bypass Cache) và gửi HTTP Request trực tiếp lên máy chủ để tải về phiên bản file medicine_catalog.css và pricing_data.json mới nhất.

4. Dòng lệnh phản hồi Set-Cookie chuẩn an ninh y tế
``` http
Set-Cookie: carepulse_doctor_session=TOKEN_XYZ883921983; Domain=carepulse.vn; Path=/; Expires=Wed, 21 Oct 2026 07:00:00 GMT; HttpOnly; Secure; SameSite=Strict
```
HttpOnly: Ngăn mã JavaScript (XSS) truy cập Cookie qua document.cookie.

Secure: Bắt buộc Cookie chỉ được gửi qua kết nối mã hóa HTTPS.

SameSite=Strict: Chống tấn công giả mạo yêu cầu từ trang khác (CSRF).


| Bước | Thao tác trên DevTools | Nội dung & Mục đích kiểm tra |
| :--- | :--- | :--- |
| **Bước 1** | **Tab Application** | Mở DevTools (`F12`) $\rightarrow$ Chọn tab **Application** $\rightarrow$ Chọn **Cookies** $\rightarrow$ Click tên miền `carepulse.vn`. Kiểm tra cột **HttpOnly** và **Secure** đã được tích chọn (✅) hay chưa. |
| **Bước 2** | **Tab Network (Vô hiệu hóa Cache)** | Chuyển sang tab **Network** $\rightarrow$ Tích chọn ô **`Disable cache`** trên thanh công cụ. Giúp trình duyệt luôn tải mới dữ liệu từ Server mỗi khi mở DevTools. |
| **Bước 3** | **Tab Network (Kiểm tra nguồn tải)** | Nhấn `F5` tải lại trang $\rightarrow$ Kiểm tra cột **Size** của tệp `medicine_catalog.css` và `pricing_data.json`. Đảm bảo hiển thị dung lượng thực tế (KB) thay vì `(from disk cache)` hay `(from memory cache)`. |






















