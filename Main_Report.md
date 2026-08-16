# BÁO CÁO HW06 - Chi tiết luồng kiểm thử API (API Testing Pipeline)

## 1. Postman Features Utilized

Trong dự án này, em đã áp dụng các tính năng sau của Postman để tối ưu hóa quá trình kiểm thử:

- **Workspaces & Collections:** Tổ chức các test cases một cách khoa học.
- **Environments & Variables:** Sử dụng biến `{{base_url}}` để linh hoạt thay đổi môi trường.
- **Pre-request Scripts:** Tự động gán Header `X-Student-Id: 23127503` cho mọi request nhằm chống gian lận.
- **Data-driven Runs (Collection Runner):** Đọc dữ liệu từ file CSV để chạy hàng loạt các test cases phân vùng tương đương.
- **Tests (Assertions):** Viết script JavaScript để verify HTTP Status, Response Time và Schema Validation.

---

## 2. API 1: Account Registration (Pool A)

- **Endpoint:** `POST /api/register`

### 2.1. Generate

- **Số lượng TCs AI sinh ra:** **30**
- **Phạm vi bao phủ:** Phân vùng tương đương, Phân tích biên (luật mật khẩu, email), Schema Validation, và Security (SQL Injection, XSS, Buffer Overflow).

### Danh sách Test Cases cho API `POST /api/register` (Functional, Security & Schema)

| TC ID                    | Test Scenario                                                                         | Request Body (JSON)                                                                                                                                                   | Expected Status                 | Expected Response                                        | Audit Status   | Audit Reason                                                                                                                                                                     |
| :----------------------- | :------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------ | :------------------------------------------------------- | :------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **TC_REG_01**            | Happy Path: Mọi dữ liệu hợp lệ, password vừa đúng biên 8 ký tự.                       | `{`<br>`"name":"User A",`<br>`"email":"valid1@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                 | `200 OK`                        | `{"message":"User registered successfully","id":1}`      | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_02**            | Happy Path: Mọi dữ liệu hợp lệ, password dài hơn 8 ký tự.                             | `{`<br>`"name":"User B",`<br>`"email":"valid2@domain.com",`<br>`"password":"StrongPassword123!",`<br>`"confirm_password":"StrongPassword123!"`<br>`}`                 | `200 OK`                        | `{"message":"User registered successfully","id":2}`      | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_03**            | Tên (`name`) bị bỏ trống (Empty string).                                              | `{`<br>`"name":"",`<br>`"email":"test3@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                        | `400 Bad Request`               | `{"error":"Name is required"}`                           | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_04**            | Thiếu trường `name` trong payload (Missing field).                                    | `{`<br>`"email":"test4@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                                        | `400 Bad Request`               | `{"error":"Name is required"}`                           | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_05**            | Email bị bỏ trống (Empty string).                                                     | `{`<br>`"name":"User C",`<br>`"email":"",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                                  | `400 Bad Request`               | `{"error":"Email is required"}`                          | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_06**            | Email sai định dạng: Thiếu ký tự `@`.                                                 | `{`<br>`"name":"User D",`<br>`"email":"userdomain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                    | `400 Bad Request`               | `{"error":"Invalid email format"}`                       | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_07**            | Email sai định dạng: Thiếu tên miền sau `@`.                                          | `{`<br>`"name":"User E",`<br>`"email":"user@",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                             | `400 Bad Request`               | `{"error":"Invalid email format"}`                       | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_08**            | Email sai định dạng: Thiếu phần tên trước `@`.                                        | `{`<br>`"name":"User F",`<br>`"email":"@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                       | `400 Bad Request`               | `{"error":"Invalid email format"}`                       | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_09**            | Email đã tồn tại trong hệ thống (Duplicate).                                          | `{`<br>`"name":"User G",`<br>`"email":"valid1@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                 | `400 Bad Request`               | `{"error":"Email already exists"}`                       | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_10**            | Mật khẩu (`password`) bị bỏ trống (Empty string).                                     | `{`<br>`"name":"User H",`<br>`"email":"test5@domain.com",`<br>`"password":"",`<br>`"confirm_password":""`<br>`}`                                                      | `400 Bad Request`               | `{"error":"Password is required"}`                       | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_11**            | Password đúng biên 7 ký tự (Invalid Boundary).                                        | `{`<br>`"name":"User I",`<br>`"email":"test6@domain.com",`<br>`"password":"Pass12!",`<br>`"confirm_password":"Pass12!"`<br>`}`                                        | `400 Bad Request`               | `{"error":"Password must be at least 8 characters"}`     | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_12**            | Password đủ 8 ký tự nhưng THIẾU chữ hoa.                                              | `{`<br>`"name":"User J",`<br>`"email":"test7@domain.com",`<br>`"password":"password1!",`<br>`"confirm_password":"password1!"`<br>`}`                                  | `400 Bad Request`               | `{"error":"Password must contain uppercase letter"}`     | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_13**            | Password đủ 8 ký tự nhưng THIẾU chữ thường.                                           | `{`<br>`"name":"User K",`<br>`"email":"test8@domain.com",`<br>`"password":"PASSWORD1!",`<br>`"confirm_password":"PASSWORD1!"`<br>`}`                                  | `400 Bad Request`               | `{"error":"Password must contain lowercase letter"}`     | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_14**            | Password đủ 8 ký tự nhưng THIẾU chữ số.                                               | `{`<br>`"name":"User L",`<br>`"email":"test9@domain.com",`<br>`"password":"Password@!",`<br>`"confirm_password":"Password@!"`<br>`}`                                  | `400 Bad Request`               | `{"error":"Password must contain a number"}`             | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_15**            | Password đủ 8 ký tự nhưng THIẾU ký tự đặc biệt.                                       | `{`<br>`"name":"User M",`<br>`"email":"test10@domain.com",`<br>`"password":"Password12",`<br>`"confirm_password":"Password12"`<br>`}`                                 | `400 Bad Request`               | `{"error":"Password must contain a special character"}`  | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_16**            | Thiếu trường `password` trong payload.                                                | `{`<br>`"name":"User N",`<br>`"email":"test11@domain.com",`<br>`"confirm_password":"Password1!"`<br>`}`                                                               | `400 Bad Request`               | `{"error":"Password is required"}`                       | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_17**            | `confirm_password` bị bỏ trống.                                                       | `{`<br>`"name":"User O",`<br>`"email":"test12@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":""`<br>`}`                                           | `400 Bad Request`               | `{"error":"Confirm password is required"}`               | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_18**            | `confirm_password` không khớp: Sai chữ hoa/thường.                                    | `{`<br>`"name":"User P",`<br>`"email":"test13@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"password1!"`<br>`}`                                 | `400 Bad Request`               | `{"error":"Passwords do not match"}`                     | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_19**            | `confirm_password` không khớp: Hoàn toàn khác biệt.                                   | `{`<br>`"name":"User Q",`<br>`"email":"test14@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"AnotherPassword1!"`<br>`}`                          | `400 Bad Request`               | `{"error":"Passwords do not match"}`                     | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_20**            | Loại dữ liệu sai (Data type mismatch): Số thay vì chuỗi.                              | `{`<br>`"name":123,`<br>`"email":"test15@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                      | `400 Bad Request`               | `{"error":"Invalid data type"}`                          | **INCOMPLETE** | AI mới chỉ test data type cho trường `name`. Cần phải tạo thêm các test case truyền số/boolean vào trường `email` và `password` để đảm bảo độ phủ 100%.                          |
| **TC_REG_21**            | Security (SQLi): SQL Injection cơ bản vào trường `email`.                             | `{`<br>`"name":"User A",`<br>`"email":"admin@domain.com' OR 1=1 --",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                       | `400 Bad Request`               | `{"error":"Invalid email format"}`                       | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_22**            | Security (SQLi): SQL Injection thời gian (Time-based) vào `email`.                    | `{`<br>`"name":"User B",`<br>`"email":"test@...'; WAITFOR DELAY '0:0:5'--",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                | `400 Bad Request`               | `{"error":"Invalid email format"}`                       | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_23**            | Security (XSS): Chèn mã script độc hại vào trường `name`.                             | `{`<br>`"name":"<script>alert('XSS')</script>",`<br>`"email":"xss@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`             | `400 Bad Request`               | `{"error":"Invalid characters in name"}`                 | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_24**            | Security: Chuỗi siêu dài (>1000 ký tự) gây Buffer Overflow vào `name`.                | `{`<br>`"name":"A... (lặp 1000 chữ A)",`<br>`"email":"buffer@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                  | `400 Bad Request`               | `{"error":"Name exceeds maximum length"}`                | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_25**            | Security: Đổi HTTP Method từ POST sang GET.                                           | `{}`                                                                                                                                                                  | `405 Method Not Allowed`        | `{"error":"Method Not Allowed"}`                         | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_26**            | Schema: Header Content-Type là `text/plain` thay vì `application/json`.               | `{`<br>`"name":"User C",`<br>`"email":"text@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                   | `415 Unsupported Media Type`    | `{"error":"Unsupported Media Type"}`                     | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_27**            | Schema (Malformed JSON): Thiếu dấu phẩy ngăn cách các trường.                         | `{`<br>`"name":"User D" "email":"json@...",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                                | `400 Bad Request`               | `{"error":"Invalid JSON payload"}`                       | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_28**            | Schema (Malformed JSON): Thiếu dấu ngoặc nhọn đóng `}` ở cuối.                        | `{`<br>`"name":"User E",`<br>`"email":"json2@...",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>                                            | `400 Bad Request`               | `{"error":"Invalid JSON payload"}`                       | **INVALID**    | AI viết sai vì Postman sẽ báo lỗi "Syntax Error" ngay tại giao diện và chặn không cho gửi đi. Để gửi được request này, QA phải chỉnh chế độ Body sang "Raw Text" thay vì "JSON". |
| **TC_REG_29**            | Schema: Gửi payload rỗng (Empty body).                                                | `{}`                                                                                                                                                                  | `400 Bad Request`               | `{"error":"Name, email, and password are required"}`     | **VALID**      | -                                                                                                                                                                                |
| **TC_REG_30**            | Schema: Gửi các trường (fields) không tồn tại (Extra properties).                     | `{`<br>`"name":"User F",`<br>`"email":"extra@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!",`<br>`"role": "ADMIN"`<br>`}`            | `400 Bad Request`               | Phụ thuộc SUT                                            | **VALID**      | Case này rất hay để kiểm tra lỗ hổng Mass Assignment (chiếm quyền bằng cách tự inject trường phân quyền).                                                                        |
| **TC_REG_31 (Extended)** | **Security (Rate Limiting):** Gửi liên tục 50 requests đăng ký trong vòng 1 giây.     | `{`<br>`"name":"Spammer",`<br>`"email":"spam_limit@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}` <br> _(Thực thi liên tục)_ | `429 Too Many Requests`         | `{"error":"Too many requests. Please try again later."}` | **VALID**      | Tự viết thêm do AI thiếu tư duy về hạ tầng mạng và cơ chế chống Spam/DDoS.                                                                                                       |
| **TC_REG_32 (Extended)** | **Security (Password Trimming):** Thêm khoảng trắng (space) vào đầu và cuối mật khẩu. | `{`<br>`"name":"User Space",`<br>`"email":"space@domain.com",`<br>`"password":" Password1! ",`<br>`"confirm_password":" Password1! "`<br>`}`                          | `200 OK` hoặc `400 Bad Request` | `{"message":"User registered successfully"}` hoặc Error  | **VALID**      | Tự viết thêm vì AI hiểu máy móc theo regex mà không tính đến hành vi copy/paste bị dính khoảng trắng của người dùng thật.                                                        |
| **TC_REG_33 (Extended)** | **Functional (Unicode/Emoji):** Tên người dùng chứa Tiếng Việt có dấu và Emoji.       | `{`<br>`"name":"Nguyễn Văn 🤖",`<br>`"email":"unicode@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                         | `200 OK`                        | `{"message":"User registered successfully","id":...}`    | **VALID**      | Tự viết thêm để kiểm tra khả năng lưu trữ ký tự đặc biệt (chuẩn UTF-8) của Database.                                                                                             |
| **TC_REG_34 (Extended)** | **Security (Validation Bypass):** Thêm khoảng trắng vào cuối email đã tồn tại.        | `{`<br>`"name":"Hacker",`<br>`"email":"admin@domain.com  ",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                | `400 Bad Request`               | `{"error":"Email already exists"}`                       | **VALID**      | Tự viết thêm để xem hệ thống có trim() khoảng trắng trước khi check trùng lặp (duplicate) dưới CSDL hay không.                                                                   |
| **TC_REG_35 (Extended)** | **State Transition:** Đăng ký thành công, sau đó gửi lại chính xác payload đó lần 2.  | `{`<br>`"name":"User State",`<br>`"email":"state@domain.com",`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                              | `400 Bad Request`               | `{"error":"Email already exists"}`                       | **VALID**      | Tự viết thêm để kiểm tra sự chuyển đổi trạng thái của email từ "Có thể đăng ký" sang "Đã tồn tại".                                                                               |
| **TC_REG_36 (Extended)** | **Schema (Data type mismatch):** Truyền số (Number) thay vì chuỗi vào trường `email`. | `{`<br>`"name":"User R",`<br>`"email":456,`<br>`"password":"Password1!",`<br>`"confirm_password":"Password1!"`<br>`}`                                                 | `400 Bad Request`               | `{"error":"Invalid data type"}`                          | **VALID**      | Tự bổ sung để vá lỗ hổng INCOMPLETE của AI tại `TC_REG_20`.                                                                                                                      |
| **TC_REG_37 (Extended)** | **Schema (Data type mismatch):** Truyền Boolean thay vì chuỗi vào trường `password`.  | `{`<br>`"name":"User S",`<br>`"email":"test16@domain.com",`<br>`"password":true,`<br>`"confirm_password":true`<br>`}`                                                 | `400 Bad Request`               | `{"error":"Invalid data type"}`                          | **VALID**      | Tự bổ sung để vá lỗ hổng INCOMPLETE của AI tại `TC_REG_20`.                                                                                                                      |

### 2.2. Audit (Human Review)

- **Tóm tắt:** Đã đánh giá 30 test cases do AI sinh ra. Kết quả: **28 VALID**, **1 INVALID**, **1 INCOMPLETE**.
- **Lý do sửa đổi và đánh giá AI:**
  - **TC_REG_28 (INVALID):** AI thiết kế case kiểm tra Schema bằng cách gửi JSON thiếu dấu ngoặc nhọn `}`. Tuy ý tưởng đúng, nhưng AI không hiểu cơ chế của tool: Postman sẽ báo lỗi "Syntax Error" ngay trên UI và chặn không cho gửi request đi nếu để chế độ body là `application/json`. _Cách sửa:_ Tôi phải can thiệp bằng cách chuyển chế độ gửi trong Postman sang `Raw Text` để có thể ép server nhận payload lỗi này.
  - **TC_REG_20 (INCOMPLETE):** AI thiết kế case kiểm tra sai kiểu dữ liệu (Data type mismatch) bằng cách truyền số `123` vào trường `name`, nhưng lại dừng ở đó. AI quên không tạo các case tương tự để test việc truyền số/boolean vào trường `email` và `password`. Tôi phải ghi chú lại để tự cover nhằm đảm bảo độ phủ 100%.

### 2.3. Extend

- **Số lượng TCs tự viết thêm:** **7** (Từ TC_REG_31 đến TC_REG_37)
- **Chi tiết các case (Lấp lỗ hổng của AI):**
  - **Security (Rate Limiting - TC_REG_31):** AI chỉ tập trung vào bảo mật Payload (SQLi, XSS) mà thiếu tư duy về hạ tầng mạng. Tôi đã bổ sung case bắn liên tục 50 requests/giây để kiểm tra cơ chế phòng chống Spam/DDoS của API đăng ký.
  - **Security & Validation Bypass (TC_REG_32 & TC_REG_34):** AI xử lý validation một cách máy móc theo chuẩn Regex mà không lường trước hành vi người dùng thực tế (copy/paste bị dính khoảng trắng). Tôi đã thêm case cố tình chèn space vào mật khẩu (`" Password1! "`) và email (`"admin@domain.com  "`) để kiểm tra xem hệ thống có thực hiện hàm `trim()` hay không, qua đó chặn các rủi ro bypass logic kiểm tra trùng lặp.
  - **State Transitions (TC_REG_35):** AI có tạo case test email trùng lặp (TC*REG_09), nhưng đó chỉ là test dữ liệu tĩnh. Tôi đã bổ sung case mang tính chất chuyển trạng thái (State Lifecycle): Đăng ký thành công một user hợp lệ, sau đó \_ngay lập tức* gửi lại đúng payload đó lần thứ hai. Hệ thống phải nhận diện được sự thay đổi trạng thái của email từ "Có thể đăng ký" sang "Đã tồn tại" và từ chối request.
  - **Functional Unicode (TC_REG_33):** Bổ sung test case có chứa ký tự Tiếng Việt có dấu và Emoji (`"Nguyễn Văn 🤖"`) để kiểm thử khả năng hỗ trợ mã hóa UTF-8 ở tầng Database mà AI hoàn toàn bỏ quên.
  - **Schema Data Type (TC_REG_36 & TC_REG_37):** Đã bổ sung các case truyền số (Number) và Boolean vào trường email và password để hoàn thiện sự thiếu sót (INCOMPLETE) của AI ở TC_REG_20, đảm bảo độ phủ 100% về kiểm tra kiểu dữ liệu cho toàn bộ các trường bắt buộc.

---

### 2.4. Execute

Quá trình thực thi kiểm thử (Execution) cho API Đăng ký tài khoản (`POST /api/register`) được tự động hóa hoàn toàn thông qua phương pháp **Data-driven Testing**, chi tiết như sau:

- **Công cụ thực thi:**
  - **Postman Collection Runner:** Được sử dụng ở giai đoạn đầu để debug, kiểm tra logic script và đảm bảo các biến chạy đúng kịch bản.
  - **Newman CLI:** Được sử dụng để chạy chính thức toàn bộ kịch bản và xuất báo cáo dưới dạng giao diện web trực quan. Lệnh thực thi:
    `newman run HW06_API_Test_Suite.postman_collection.json -e "EShop - Local Environment.postman_environment.json" -d register_data.csv -r htmlextra`
- **Dữ liệu đầu vào:** File `register_data.csv` chứa 37 iterations, tương ứng với 37 Test Cases đã thiết kế (Bao gồm AI-generated và Human-extended).
- **Cơ chế Anti-Cheat:** Đã cấu hình thành công _Pre-request Script_ ở cấp độ Collection. Hệ thống tự động can thiệp và đính kèm Header `X-Student-Id: 23127503` vào toàn bộ 37 requests được gửi đi (Bằng chứng được lưu tại console log và báo cáo HTML).
- **Kết quả:** Quá trình chạy diễn ra thông suốt. Tuy nhiên, tỷ lệ Test Cases FAILED chiếm phần lớn. Phân tích đối chiếu log cho thấy **kịch bản test hoàn toàn chính xác**, sự thất bại của các assertions đến từ việc hệ thống SUT (System Under Test) hoạt động sai lệch nghiêm trọng so với đặc tả (API không có cơ chế bảo vệ), dẫn đến việc phát hiện 5 Bugs chí mạng.

---

### 2.5. Report Bugs

Dưới đây là danh sách 5 lỗi (Bugs) nghiêm trọng được bóc tách từ việc phân tích kỹ lưỡng file log (Console) và báo cáo Newman HTML. Các lỗi này đã được ghi nhận (Log) chính thức lên hệ thống GitHub Issues.

### 1. [Critical] SUT rò rỉ thông tin hệ thống (Information Disclosure / Stack Trace Leak)

- **Mức độ (Severity):** **CRITICAL**

- **Phân loại (Category):** **Security** (Information Exposure)

- **Mô tả chi tiết:** Khi người dùng gửi một payload JSON bị sai cú pháp (thiếu dấu ngoặc nhọn `}` ở cuối), thay vì bắt lỗi và trả về mã `400 Bad Request`, máy chủ (Express.js) lại không thể xử lý, dẫn đến crash nội bộ và trả về một trang HTML chứa toàn bộ Stack Trace báo lỗi. Lỗi này được phát hiện ở `TC_REG_28`.

- **Tác động (Impact):** Cực kỳ nguy hiểm. Hệ thống đã vô tình phơi bày đường dẫn tuyệt đối trên máy chủ của hệ thống (`D:\Software Testing\Seminar\eshop-sut\backend\node_modules\...`) cùng các thư viện đang sử dụng. Kẻ tấn công có thể dựa vào thông tin kiến trúc này để dò quét và khai thác các lỗ hổng sâu hơn.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/1

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-1.png)

### 2. [Critical Security] Lỗ hổng bảo mật: Nhận mã độc XSS và Mass Assignment

- **Mức độ (Severity):** **CRITICAL (Security)**

- **Phân loại (Category):** **Security** (Injection & Broken Access Control)

- **Mô tả chi tiết:** Backend API hoàn toàn không thực hiện mã hóa (encode) hay thanh lọc (sanitize) dữ liệu đầu vào. Cụ thể:
  - Ở `TC_REG_23`, hệ thống chấp nhận chuỗi chứa mã độc HTML/JS `<script>alert('XSS')</script>` vào trường `name` và trả về `200 OK`.

  - Ở `TC_REG_30`, khi cố tình truyền thêm trường phân quyền `"role": "ADMIN"`, hệ thống cũng lưu trữ thành công mà không loại bỏ các trường thừa (Mass Assignment).

- **Tác động (Impact):** Lỗ hổng Stored XSS cho phép hacker chèn mã độc vào tên, khi Admin truy cập xem danh sách User, mã độc sẽ thực thi và đánh cắp Session/Cookie của Admin. Lỗ hổng Mass Assignment cho phép người dùng tự do thăng cấp tài khoản của mình lên làm Quản trị viên để chiếm quyền hệ thống.

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/2

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-2.png)

### 3. [Major] Lỗi Logic Nghiệp Vụ: Bỏ qua kiểm tra khớp Mật khẩu (Password Mismatch Ignored)

- **Mức độ (Severity):** **MAJOR**

- **Phân loại (Category):** **Functional** (Business Logic)

- **Mô tả chi tiết:** Lập trình viên thiết kế trường `confirm_password` ở phía Client nhưng lại không có code xử lý so sánh với trường `password` ở phía Server. Ở `TC_REG_19`, dù mật khẩu xác nhận được gửi lên là `"AnotherPassword1!"` (khác biệt hoàn toàn với mật khẩu gốc), hệ thống vẫn bỏ qua và trả về `200 OK`.

- **Tác động (Impact):** Gây rủi ro nghiêm trọng về trải nghiệm người dùng (UX). Nếu người dùng gõ nhầm mật khẩu ở ô đầu tiên, họ sẽ không hề hay biết và sau này bị mất quyền truy cập (không thể đăng nhập lại vào tài khoản vừa tạo).

- **GitHub Issue Link:** https://github.com/Triszz/HW06-API_Testing/issues/3

- **Ảnh chụp (Screenshot):**

  ![alt text](images/image-3.png)

### 4. [Major] Lỗi Logic Nghiệp Vụ: Cho phép đăng ký trùng Email (Duplicate Email)

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

### 5. [Major] Thiếu hoàn toàn Data Validation (No Input Validation)

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

---

## 3. API 2: Apply Coupon (Pool B)

- **Endpoint:** `POST /api/apply-coupon`
- `[Cấu trúc lặp lại 5 bước tương tự như API 1...]`

---

## 4. API 3: Product Management (Pool C)

- **Endpoint:** `POST/PUT/DELETE /api/products`
- `[Cấu trúc lặp lại 5 bước tương tự như API 1...]`
