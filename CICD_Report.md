# CI/CD Integration Report

## 1. Pipeline Configuration

- **Nền tảng:** GitHub Actions.
- **Mô tả:** Pipeline được cấu hình bằng file `.yml` để tự động cài đặt Node.js, Newman, thư viện `htmlextra` reporter, và chạy bộ collection Postman cùng file Environment ngay khi có code được push lên nhánh `main`.

## 2. Sample Pipeline Runs

### 2.1. Pipeline Xanh (All Test Cases Passing)

- **Mô tả:** Commit chạy mượt mà, toàn bộ test cases đều Passed 100%.
- **GitHub Action Link:** `[Dán link GitHub Actions run vào đây]`
- **Screenshot:**
  `[Chèn ảnh chụp màn hình xanh lá]`

### 2.2. Pipeline Đỏ (One or More Test Cases Failing)

- **Mô tả:** Cố tình đưa vào một test case bị lỗi (hoặc phát hiện bug của hệ thống) để kiểm tra khả năng bắt lỗi của Pipeline.
- **GitHub Action Link:** `[Dán link GitHub Actions run vào đây]`
- **Screenshot:**
  `[Chèn ảnh chụp màn hình màu đỏ báo lỗi]`
