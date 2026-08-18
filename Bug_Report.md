# Danh sách Bug (Defect Report)

Dưới đây là các lỗi phát hiện được trong quá trình chạy Pipeline kiểm thử 3 APIs, bao gồm các lỗi do hệ thống (SUT) và các lỗ hổng bảo mật mà AI đã bỏ sót.

## Issue 1: [Critical] SUT rò rỉ thông tin hệ thống (Information Disclosure / Stack Trace Leak)

- **Endpoint:** `POST /api/register`

- **Mức độ (Severity):** **CRITICAL**

- **Phân loại (Category):** **Security** (Information Exposure)

- **Mô tả chi tiết:** Khi người dùng gửi một payload JSON bị sai cú pháp (thiếu dấu ngoặc nhọn `}` ở cuối), thay vì bắt lỗi và trả về mã `400 Bad Request`, máy chủ (Express.js) lại không thể xử lý, dẫn đến crash nội bộ và trả về một trang HTML chứa toàn bộ Stack Trace báo lỗi. Lỗi này được phát hiện ở `TC_REG_28`.

- **Tác động (Impact):** Cực kỳ nguy hiểm. Hệ thống đã vô tình phơi bày đường dẫn tuyệt đối trên máy chủ của hệ thống (`D:\Software Testing\Seminar\eshop-sut\backend\node_modules\...`) cùng các thư viện đang sử dụng. Kẻ tấn công có thể dựa vào thông tin kiến trúc này để dò quét và khai thác các lỗ hổng sâu hơn.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/1

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-1.png)

## Issue 2: [Critical Security] Lỗ hổng bảo mật: Nhận mã độc XSS và Mass Assignment

- **Endpoint:** `POST /api/register`

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

- **Endpoint:** `POST /api/register`

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Business Logic)

- **Mô tả chi tiết:** Lập trình viên thiết kế trường `confirm_password` ở phía Client nhưng lại không có code xử lý so sánh với trường `password` ở phía Server. Ở `TC_REG_19`, dù mật khẩu xác nhận được gửi lên là `"AnotherPassword1!"` (khác biệt hoàn toàn với mật khẩu gốc), hệ thống vẫn bỏ qua và trả về `200 OK`.

- **Tác động (Impact):** Gây rủi ro nghiêm trọng về trải nghiệm người dùng (UX). Nếu người dùng gõ nhầm mật khẩu ở ô đầu tiên, họ sẽ không hề hay biết và sau này bị mất quyền truy cập (không thể đăng nhập lại vào tài khoản vừa tạo).

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/3

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-3.png)

## Issue 4: [Major] Lỗi Logic Nghiệp Vụ: Cho phép đăng ký trùng Email (Duplicate Email)

- **Endpoint:** `POST /api/register`

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Data Integrity)

- **Mô tả chi tiết:** API thiếu khâu truy vấn kiểm tra dữ liệu tồn tại (Unique Constraint) trong Database. Ở request `TC_REG_01`, tài khoản với email `valid1@domain.com` được tạo thành công với ID là `3`. Tuy nhiên, khi gửi lại đúng email này ở `TC_REG_09`, hệ thống vẫn tiếp nhận và tạo ra một tài khoản mới tinh với ID là `11`.

- **Tác động (Impact):** Logic định danh (Identity) của toàn bộ hệ thống bị phá vỡ. Cơ sở dữ liệu sẽ chứa nhiều bản ghi trùng lặp email, dẫn đến việc chức năng Đăng nhập hoặc Quên mật khẩu phía sau không thể hoạt động chính xác vì không biết phải trỏ về User ID nào.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/4

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-4.png)

  <center><i>Tài khoản với email <code>valid1@domain.com</code> được tạo thành công lần 1</i></center>

  <br>

  ![alt text](images/image-5.png)

   <center><i>Tài khoản với email <code>valid1@domain.com</code> được tạo thành công lần 2</i></center>

## Issue 5: [Major] Thiếu hoàn toàn Data Validation (No Input Validation)

- **Endpoint:** `POST /api/register`

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

   <center><i>Tài khoản với payload rỗng <code>{}</code> được tạo thành công</i></center>

  <br>

  ![alt text](images/image-7.png)

   <center><i>Tài khoản với email sai định dạng (thiếu <code>@</code>) được tạo thành công</i></center>

  <br>

  ![alt text](images/image-8.png)

   <center><i>Tài khoản với mật khẩu rỗng được tạo thành công</i></center>

  <br>

  ![alt text](images/image-9.png)

  <center><i>Tài khoản với mật khẩu là kiểu dữ liệu boolean được tạo thành công</i></center>

## Issue 6: [Critical] Authentication Bypass & Lỗ hổng IDOR (Bỏ qua xác thực và Kiểm soát truy cập)

- **Endpoint:** `POST /api/apply-coupon`

- **Mức độ (Severity):** **CRITICAL**

- **Phân loại (Category):** **Security** (Broken Authentication & Access Control)

- **Mô tả chi tiết:** Backend hoàn toàn không kiểm tra Header `Authorization`. API trả về `200 OK` cho mọi trường hợp: không truyền Token (`TC_COUPON_12`), Token rác (`TC_COUPON_34`), và Token hết hạn (`TC_COUPON_35`). Hậu quả kéo theo là lỗi IDOR (`TC_COUPON_29`): Hệ thống tin tưởng tuyệt đối vào trường `user_id` truyền trong Body. Một người dùng bất kỳ có thể điền `user_id` của người khác để "xài chùa" mã giảm giá của họ.

- **Tác động (Impact):** Lỗ hổng bảo mật mức độ cao nhất. Hacker có thể viết script quét toàn bộ `user_id` từ 1 đến 100.000 để dùng sạch mã giảm giá của toàn bộ khách hàng trong hệ thống.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/6

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-13.png)

  <center><i>Request headers không truyền Authorization Bearer Token</i></center>

  <br>

  ![alt text](images/image-14.png)

  <center><i>Request headers truyền Authorization Bearer Token rác</i></center>

  <br>

  ![alt text](images/image-15.png)

  <center><i>Request headers truyền Authorization Bearer Token hết hạn</i></center>

## Issue 7: [Critical] Lỗ hổng Mass Assignment (Thao túng số tiền giảm giá)

- **Endpoint:** `POST /api/apply-coupon`

- **Mức độ (Severity):** **CRITICAL**

- **Phân loại (Category):** **Security** (Injection/Mass Assignment)

- **Mô tả chi tiết:** Ở `TC_COUPON_30`, khi cố tình chèn thêm trường `"discount_amount": 999999` vào Request Body, hệ thống vẫn chấp nhận (`200 OK`) thay vì từ chối (400 Bad Request).

- **Tác động (Impact):** Mặc dù cần kiểm tra thêm ở Database xem server có thực sự lấy con số bị thao túng này để trừ tiền hay không, nhưng việc API chấp nhận các trường tính toán từ phía Client là rủi ro cực lớn. Hacker có thể ép server giảm giá 100% cho mọi đơn hàng.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/7

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-16.png)

## Issue 8: [Critical] Lỗi sai Thuật toán tính toán phần trăm

- **Endpoint:** `POST /api/apply-coupon`

- **Mức độ (Severity):** **CRITICAL**

- **Phân loại (Category):** **Functional** (Business Logic/Math Error)

- **Mô tả chi tiết:** Ở `TC_COUPON_01` (và tất cả các case dùng mã Percent), khi áp dụng mã `SAVE10` (giảm 10%) cho đơn hàng 500.000 VNĐ. Kỳ vọng số tiền giảm là 50.000 VNĐ. Tuy nhiên, Server lại tính toán sai lệch hoàn toàn, trả về `"discount_amount": -4500000` và `"final_amount": 5000000`.

- **Tác động (Impact):** Đây là lỗi logic cốt lõi (Core feature broken). Người dùng sẽ phải thanh toán số tiền gấp 10 lần giá trị thực tế của đơn hàng, phá hủy hoàn toàn uy tín và doanh thu của doanh nghiệp.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/8

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-17.png)

## Issue 9: [Major] Bỏ qua điều kiện Giới hạn số lần sử dụng (C5 - Max Uses Bypass)

- **Endpoint:** `POST /api/apply-coupon`

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Business Logic)

- **Mô tả chi tiết:** Các Test Cases kiểm tra giới hạn lượt dùng mã giảm giá (`TC_COUPON_09`, `15`, `40`) đều bị Failed do server trả về `200 OK` thay vì từ chối `400`. Logic kiểm tra điều kiện C5 (`max_uses_per_user`) hoàn toàn không hoạt động.

- **Tác động (Impact):** Gây thiệt hại trực tiếp về tài chính. Một khách hàng có thể dùng đi dùng lại 1 mã giảm giá (vốn chỉ được dùng 1 lần) cho hàng trăm đơn hàng khác nhau.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/9

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-18.png)

  <center><i>Sử dụng voucher <code>SAVE10</code> lần 1</i></center>

  <br>

  ![alt text](images/image-19.png)

  <center><i>Sử dụng voucher <code>SAVE10</code> lần 2</i></center>

## Issue 10: [Major] Sai Logic Toán học ở Giá trị Biên (C3 - Minimum Amount Boundary)

- **Endpoint:** `POST /api/apply-coupon`

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Boundary Logic)

- **Mô tả chi tiết:** Theo đặc tả, đơn hàng chỉ cần **lớn hơn hoặc bằng (>=)** ngưỡng tối thiểu là được dùng mã. Tuy nhiên, ở `TC_COUPON_03` và `TC_COUPON_04` (Đơn hàng vừa đúng ngưỡng 300k và 500k), API lại báo lỗi 400. Điều này chứng tỏ Developer đã code sai toán tử: dùng dấu `>` thay vì `>=`.

- **Tác động (Impact):** Gây bức xúc cho người dùng. Khách hàng mua đúng 300k để được giảm giá nhưng hệ thống không cho, ép khách phải mua từ 300,001đ trở lên.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/10

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-20.png)

  <center><i>Áp dụng voucher <code>SAVE10</code> cho đơn hàng 300k</i></center>

  <br>

  ![alt text](images/image-21.png)

  <center><i>Áp dụng voucher <code>BIGBUY</code> cho đơn hàng 500k</i></center>

## Issue 11: [Major] Không Validate Schema và Kiểu Dữ Liệu

- **Endpoint:** `POST /api/apply-coupon`

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Schema Validation)

- **Mô tả chi tiết:** API bỏ qua mọi quy tắc kiểm tra cấu trúc dữ liệu đầu vào. Các Test cases Failed (`TC_COUPON_18`, `21`, `23`, `26`, `27`) cho thấy hệ thống trả về `200 OK` ngay cả khi: thiếu trường `user_id` bắt buộc, `total_amount` bị truyền sai kiểu thành dạng chuỗi (String), hoặc truyền số tiền siêu khổng lồ (Buffer Overflow).

- **Tác động (Impact):** Rác dữ liệu, dễ dẫn đến lỗi 500 Internal Server Error ở các module xử lý thanh toán phía sau (vì mong đợi kiểu Integer nhưng lại nhận String).

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/11

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-22.png)

  <center><i>Request body thiếu trường <code>user_id</code></i></center>

  <br>

  ![alt text](images/image-23.png)

  <center><i>Trường <code>total_amount</code> là kiểu chuỗi</i></center>

## Issue 12: [Medium] Lỗi Logic So Khớp Phân Biệt Hoa/Thường (Case-sensitive Issue)

- **Endpoint:** `POST /api/apply-coupon`

- **Mức độ (Severity):** **MEDIUM**

- **Phân loại (Category):** **Functional / UX**

- **Mô tả chi tiết:** Ở `TC_COUPON_38`, khi người dùng nhập mã giảm giá viết thường `"save10"`, hệ thống trả về lỗi `404 Not Found` (Kỳ vọng là `200 OK`). API đang so sánh chuỗi một cách cứng nhắc (Case-sensitive) thay vì tự động chuyển đổi sang chữ hoa (Uppercase) trước khi query Database.

- **Tác động (Impact):** Trải nghiệm người dùng (UX) kém. Khách hàng dễ bị bối rối và tưởng mã bị lỗi nếu điện thoại của họ không tự động bật phím CapsLock.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/12

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-24.png)

## Issue 13: [Medium] Thiếu cơ chế Rate Limiting (Chống Spam)

- **Endpoint:** `POST /api/apply-coupon`

- **Mức độ (Severity):** **MEDIUM**

- **Phân loại (Category):** **Security** (Rate Limiting)

- **Mô tả chi tiết:** Ở `TC_COUPON_33`, giả lập việc bắn 100 requests đồng thời trong 1 giây. Thay vì bị chặn bởi hệ thống với mã `429 Too Many Requests`, API lại tiếp nhận toàn bộ và trả về `200 OK`.

- **Tác động (Impact):** Kẻ xấu có thể sử dụng tool tự động để spam API, gây quá tải Server (DoS) hoặc khai thác lỗ hổng Race Condition để xài mã giảm giá nhiều lần cùng một lúc.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/13

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-25.png)

## Issue 14: [Critical] Authentication Bypass & Role Escalation (Thiếu hoàn toàn xác thực)

- **Endpoint:** `POST /api/products`

- **Mức độ (Severity):** **CRITICAL**

- **Phân loại (Category):** **Security** (Broken Access Control)

- **Mô tả chi tiết:** Đặc tả yêu cầu API thêm sản phẩm chỉ dành cho Admin (FR-15). Tuy nhiên, hệ thống trả về `200 OK` cho toàn bộ các kịch bản: Không truyền Token (`TC_PROD_22`), Truyền Token hết hạn (`TC_PROD_24`), và Truyền Token của khách hàng thường (`TC_PROD_21`).

- **Tác động (Impact):** Bất kỳ ai (kể cả khách vãng lai hoặc hacker) cũng có thể tự do gọi API này để bơm hàng triệu sản phẩm rác vào hệ thống EShop.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/14

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-27.png)

  <center><i>Request headers không truyền Authorization Bearer Token</i></center>

  <br>

  ![alt text](images/image-28.png)

  <center><i>Request headers truyền Authorization Bearer Token hết hạn</i></center>

  <br>

  ![alt text](images/image-29.png)

  <center><i>Request headers truyền Authorization Bearer Token của khách hàng thường</i></center>

## Issue 15: [Critical] Stored XSS qua trường tùy chọn (Optional Fields)

- **Endpoint:** `POST /api/products`

- **Mức độ (Severity):** **CRITICAL**

- **Phân loại (Category):** **Security** (Cross-Site Scripting)

- **Mô tả chi tiết:** Hệ thống lưu trữ trực tiếp dữ liệu thô vào CSDL mà không có bước mã hóa (Sanitization). Cụ thể, khi truyền `<script>alert(1)</script>` vào `name` (`TC_PROD_26`) hoặc chuỗi `javascript:alert(1)` vào `imageUrl` (`TC_PROD_37`), server vẫn khởi tạo sản phẩm thành công.

- **Tác động (Impact):** Mã độc này sẽ được lưu vào Database. Khi người dùng (hoặc Admin) mở trang xem danh sách sản phẩm, mã độc JS sẽ kích hoạt, dẫn đến việc hacker có thể chiếm đoạt Cookie/Session và cướp tài khoản Admin.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/15

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-30.png)

  <center><i>Request body truyền <code>&lt;script&gt;alert(1)&lt;/script&gt;</code> vào trường <code>name</code></i></center>

  <br>

  ![alt text](images/image-31.png)

  <center><i>Request body truyền chuỗi <code>javascript:alert(1)</code> vào trường <code>imageUrl</code></i></center>

## Issue 16: [Critical] Server Crash & Rò rỉ Stack Trace do lỗi Content-Type

- **Endpoint:** `POST /api/products`

- **Mức độ (Severity):** **CRITICAL**

- **Phân loại (Category):** **Security / Error Handling**

- **Mô tả chi tiết:** Tại `TC_PROD_28`, khi gửi payload JSON hợp lệ nhưng cố tình đổi Header `Content-Type` thành `text/plain`, server Express.js không xử lý được. Thay vì báo lỗi `400` hoặc `415`, server bị crash nội bộ và trả về một mã HTML 500 phơi bày toàn bộ mã nguồn đường dẫn nội bộ: `Cannot destructure property 'name' of 'req.body'... at D:\Software Testing\HWs\...`.

- **Tác động (Impact):** Lộ lọt thông tin kiến trúc thư mục máy chủ, tạo tiền đề cho hacker tìm kiếm các lỗ hổng hệ thống sâu hơn.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/16

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-32.png)

## Issue 17: [Major] Không có bất kỳ Schema Validation nào (Chấp nhận Payload rỗng)

- **Endpoint:** `POST /api/products`

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Input Validation)

- **Mô tả chi tiết:** Hệ thống bỏ qua mọi quy định về Data Schema.
  - Gửi Payload rỗng `{}` (`TC_PROD_19`) -> Vẫn tạo sản phẩm thành công.

  - Thiếu tất cả trường bắt buộc (`TC_PROD_08, 09, 10`) -> Thành công.

  - Truyền Object rỗng vào trường số (`price: {}`) -> Thành công.

- **Tác động (Impact):** Database sẽ chứa toàn dữ liệu rác, các dòng sản phẩm không có tên, không có giá. Gây sập các UI Frontend khi parse dữ liệu.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/17

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-33.png)

  <center><i>Request body gửi payload rỗng <code>{}</code></i></center>

  <br>

  ![alt text](images/image-34.png)

  <center><i>Request body gửi thiếu các trường bắt buộc</i></center>

  <br>

  ![alt text](images/image-35.png)

  <center><i>Request body truyền object rỗng <code>{}</code> vào trường <code>price</code></i></center>

## Issue 18: [Major] Chấp nhận Giá tiền Âm và Bằng 0

- **Endpoint:** `POST /api/products`

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Boundary Logic)

- **Mô tả chi tiết:** Dù đặc tả FR-15 ghi rõ `price` phải là số dương (> 0). Tuy nhiên ở `TC_PROD_04` (price = 0) và `TC_PROD_05` (price = -1), hệ thống vẫn trả về `200 OK`.

- **Tác động (Impact):** Gây lỗi nghiêm trọng ở module thanh toán và doanh thu. Người dùng có thể mua sản phẩm giá 0đ hoặc thậm chí số tiền âm (cửa hàng phải trả ngược tiền cho khách).

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/18

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-36.png)

  <center><i>Tạo sản phẩm với <code>price = 0</code></i></center>

  <br>

  ![alt text](images/image-37.png)

  <center><i>Tạo sản phẩm với <code>price = -1</code></i></center>

## Issue 19: [Major] Thiếu ràng buộc Khóa Ngoại (Foreign Key Constraint)

- **Endpoint:** `POST /api/products`

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Database Integrity)

- **Mô tả chi tiết:** Khi truyền `category_id` là `999` (Danh mục không hề tồn tại) hoặc `-1` (Số âm không hợp lệ), API vẫn báo tạo sản phẩm thành công (`TC_PROD_14`, `TC_PROD_38`).

- **Tác động (Impact):** Mất tính toàn vẹn dữ liệu (Data Integrity). Sản phẩm tạo ra không thuộc về bất kỳ danh mục nào, dẫn đến việc sản phẩm không bao giờ hiển thị được lên trang chủ.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/19

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-38.png)

  <center><i>Request body truyền <code>category_id</code> không tồn tại</i></center>
