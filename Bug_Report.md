# Danh sách Bug (Defect Report)

Dưới đây là các lỗi phát hiện được trong quá trình chạy Pipeline kiểm thử 3 APIs, bao gồm các lỗi do hệ thống (SUT) và các lỗ hổng bảo mật mà AI đã bỏ sót.

## Issue 1: [Critical] SUT rò rỉ thông tin hệ thống (Information Disclosure / Stack Trace Leak)

- **Mức độ (Severity):** **CRITICAL**

- **Phân loại (Category):** **Security** (Information Exposure)

- **Mô tả chi tiết:** Khi người dùng gửi một payload JSON bị sai cú pháp (thiếu dấu ngoặc nhọn `}` ở cuối), thay vì bắt lỗi và trả về mã `400 Bad Request`, máy chủ (Express.js) lại không thể xử lý, dẫn đến crash nội bộ và trả về một trang HTML chứa toàn bộ Stack Trace báo lỗi. Lỗi này được phát hiện ở `TC_REG_28`.

- **Tác động (Impact):** Cực kỳ nguy hiểm. Hệ thống đã vô tình phơi bày đường dẫn tuyệt đối trên máy chủ của hệ thống (`D:\Software Testing\Seminar\eshop-sut\backend\node_modules\...`) cùng các thư viện đang sử dụng. Kẻ tấn công có thể dựa vào thông tin kiến trúc này để dò quét và khai thác các lỗ hổng sâu hơn.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/1

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-1.png)

## Issue 2: [Critical Security] Lỗ hổng bảo mật: Nhận mã độc XSS và Mass Assignment

- **Mức độ (Severity):** **CRITICAL (Security)**

- **Phân loại (Category):** **Security** (Injection & Broken Access Control)

- **Mô tả chi tiết:** Backend API hoàn toàn không thực hiện mã hóa (encode) hay thanh lọc (sanitize) dữ liệu đầu vào. Cụ thể:
  - Ở `TC_REG_23`, hệ thống chấp nhận chuỗi chứa mã độc HTML/JS `<script>alert('XSS')</script>` vào trường `name` và trả về `200 OK`.

  - Ở `TC_REG_30`, khi cố tình truyền thêm trường phân quyền `"role": "ADMIN"`, hệ thống cũng lưu trữ thành công mà không loại bỏ các trường thừa (Mass Assignment).

- **Tác động (Impact):** Lỗ hổng Stored XSS cho phép hacker chèn mã độc vào tên, khi Admin truy cập xem danh sách User, mã độc sẽ thực thi và đánh cắp Session/Cookie của Admin. Lỗ hổng Mass Assignment cho phép người dùng tự do thăng cấp tài khoản của mình lên làm Quản trị viên để chiếm quyền hệ thống.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/2

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-2.png)

## Issue 3: [Major] Lỗi Logic Nghiệp Vụ: Bỏ qua kiểm tra khớp Mật khẩu (Password Mismatch Ignored)

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Business Logic)

- **Mô tả chi tiết:** Lập trình viên thiết kế trường `confirm_password` ở phía Client nhưng lại không có code xử lý so sánh với trường `password` ở phía Server. Ở `TC_REG_19`, dù mật khẩu xác nhận được gửi lên là `"AnotherPassword1!"` (khác biệt hoàn toàn với mật khẩu gốc), hệ thống vẫn bỏ qua và trả về `200 OK`.

- **Tác động (Impact):** Gây rủi ro nghiêm trọng về trải nghiệm người dùng (UX). Nếu người dùng gõ nhầm mật khẩu ở ô đầu tiên, họ sẽ không hề hay biết và sau này bị mất quyền truy cập (không thể đăng nhập lại vào tài khoản vừa tạo).

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/3

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-3.png)

## Issue 4. [Major] Lỗi Logic Nghiệp Vụ: Cho phép đăng ký trùng Email (Duplicate Email)

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Data Integrity)

- **Mô tả chi tiết:** API thiếu khâu truy vấn kiểm tra dữ liệu tồn tại (Unique Constraint) trong Database. Ở request `TC_REG_01`, tài khoản với email `valid1@domain.com` được tạo thành công với ID là `3`. Tuy nhiên, khi gửi lại đúng email này ở `TC_REG_09`, hệ thống vẫn tiếp nhận và tạo ra một tài khoản mới tinh với ID là `11`.

- **Tác động (Impact):** Logic định danh (Identity) của toàn bộ hệ thống bị phá vỡ. Cơ sở dữ liệu sẽ chứa nhiều bản ghi trùng lặp email, dẫn đến việc chức năng Đăng nhập hoặc Quên mật khẩu phía sau không thể hoạt động chính xác vì không biết phải trỏ về User ID nào.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/4

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-4.png)

  <center><i>Tài khoản với email valid1@domain.com được tạo thành công lần 1</i></center>

  <br>

  ![alt text](images/image-5.png)

   <center><i>Tài khoản với email valid1@domain.com được tạo thành công lần 2</i></center>

## Issue 5. [Major] Thiếu hoàn toàn Data Validation (No Input Validation)

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Input Validation)

- **Mô tả chi tiết:** Server không có bất kỳ bộ kiểm tra ràng buộc (Validator) nào cho các trường đầu vào. Hệ thống dễ dàng trả về `200 OK` cho các kịch bản:
  - Payload rỗng hoàn toàn `{}` (`TC_REG_29`).

  - Email sai định dạng, thiếu tên miền hoặc thiếu ký tự `@` (`TC_REG_06`, `TC_REG_07`, `TC_REG_08`).

  - Mật khẩu yếu, rỗng hoặc thiếu các ký tự đặc biệt theo quy định (`TC_REG_10` đến `TC_REG_15`).

  - Truyền sai kiểu dữ liệu thành số nguyên hoặc boolean (`TC_REG_36`, `TC_REG_37`).

- **Tác động (Impact):** Dữ liệu rác (Garbage Data) tràn ngập trong cơ sở dữ liệu. Các dịch vụ nghiệp vụ phía sau (VD: Dịch vụ gửi Email tự động, Dịch vụ thanh toán) khi đọc phải các dữ liệu sai cấu trúc này sẽ gặp lỗi và có nguy cơ sập toàn bộ hệ thống (System Failure).

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/5

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-6.png)

   <center><i>Tài khoản với payload rỗng {} được tạo thành công</i></center>

  <br>

  ![alt text](images/image-7.png)

   <center><i>Tài khoản với email sai định dạng (thiếu @) được tạo thành công</i></center>

  <br>

  ![alt text](images/image-8.png)

   <center><i>Tài khoản với mật khẩu rỗng được tạo thành công</i></center>

  <br>

  ![alt text](images/image-9.png)

  <center><i>Tài khoản với mật khẩu là kiểu dữ liệu boolean được tạo thành công</i></center>
