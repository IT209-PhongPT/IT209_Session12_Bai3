# IT209 - Session 12 - Bài tập 3: Tách biệt luồng CI/CD đa môi trường (Staging & Production) với GitHub Environments

- **Sinh viên:** Phạm Thanh Phong
- **MSSV:** N24DTCN120
- **Lớp:** IT209
- **Học phần:** IT209
- **Đường dẫn nộp bài:** `homework/session_12/ex3/`
- **Trạng thái:** Hoàn thành (100%)

---

## 1. Danh mục tệp tin trong thư mục bài tập

| Tên tệp tin | Mô tả chức năng |
| :--- | :--- |
| [.github/workflows/cd-multi-env.yml](./.github/workflows/cd-multi-env.yml) | **Tệp kịch bản GitHub Actions CD chính thức:** Quản lý hạ tầng đa môi trường với `environment: staging` và `environment: production` |
| [screenshots/github-environment-production-approval.png](./screenshots/github-environment-production-approval.png) | **Ảnh minh chứng nộp bài:** Cửa sổ Review deployments hiển thị trạng thái `Waiting for review` và nút `Approve and deploy` trên môi trường Production |
| [IT209_Session12_BT3_PhamThanhPhong_N24DTCN120.md](./IT209_Session12_BT3_PhamThanhPhong_N24DTCN120.md) | Báo cáo kỹ thuật chi tiết, phân tích kiến trúc Environment Protection Rules và giải thích cơ chế phê duyệt thủ công |
| [build.gradle](./build.gradle) & [settings.gradle](./settings.gradle) | Cấu hình dự án Spring Boot 3.2.5 và Gradle Wrapper |
| [gradlew](./gradlew) & [gradlew.bat](./gradlew.bat) | Bộ script thực thi Gradle Wrapper trên Linux và Windows |
| [src/](./src/) | Toàn bộ mã nguồn ứng dụng và bộ kiểm thử JUnit 5 |

---

## 2. Hướng dẫn thiết lập GitHub Environments trên Repository

Trước khi kích hoạt pipeline, thực hiện thiết lập 2 môi trường trên giao diện GitHub:
1. Truy cập Repository trên GitHub $\rightarrow$ **Settings** $\rightarrow$ **Environments**.
2. Nhấn **New environment** $\rightarrow$ Đặt tên: `staging` $\rightarrow$ Nhấn **Configure environment**.
3. Tiếp tục nhấn **New environment** $\rightarrow$ Đặt tên: `production` $\rightarrow$ Nhấn **Configure environment**.
4. Tại mục cấu hình của môi trường `production`:
   - Tích chọn mục **Required reviewers**.
   - Tìm và chọn tài khoản cá nhân của bạn (ví dụ: `PhamThanhPhong`) làm người phê duyệt.
   - Nhấn **Save protection rules**.

---

## 3. Nội dung tệp cấu hình `.github/workflows/cd-multi-env.yml`

```yaml
name: Multi-Environment CD Pipeline (Staging & Production)

on:
  push:
    branches:
      - main
      - develop

jobs:
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.example.com

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Triển khai tự động lên môi trường Staging
        run: |
          echo "Bắt đầu triển khai tự động lên môi trường Staging..."
          echo "Deploying branch: ${{ github.ref_name }} (Commit: ${{ github.sha }})"
          echo "Target URL: https://staging.example.com"
          echo "Triển khai Staging hoàn tất thành công!"

  deploy-prod:
    name: Deploy to Production
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://api.prod.example.com

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Triển khai an toàn lên môi trường Production
        run: |
          echo "Bắt đầu triển khai lên môi trường Production (Đã được phê duyệt)..."
          echo "Deploying branch: ${{ github.ref_name }} (Commit: ${{ github.sha }})"
          echo "Target URL: https://api.prod.example.com"
          echo "Triển khai Production an toàn hoàn tất thành công!"
```

---

## 4. Ảnh minh chứng cửa sổ phê duyệt Deployment trên môi trường Production

Ảnh chụp màn hình tab **Actions** xác nhận: Khi push vào nhánh `main`, pipeline tự động dừng tại Job **Deploy to Production** với trạng thái **Waiting for review** và mở hộp thoại **Review deployments** yêu cầu người duyệt xác nhận:

![Phê duyệt Deployment Production](./screenshots/github-environment-production-approval.png)

---

## 5. Hướng dẫn kiểm tra và chạy thử luồng triển khai

### 5.1. Kiểm thử triển khai tự động lên Staging (Nhánh `develop`)
```bash
# Tạo nhánh develop và push lên GitHub
git checkout -b develop
git push -u origin develop
```
- **Kết quả:** Job `deploy-staging` được kích hoạt và tự động triển khai thành công lên `staging.example.com` mà không cần duyệt.

### 5.2. Kiểm thử luồng phê duyệt Production (Nhánh `main`)
```bash
# Chuyển về nhánh main và đẩy mã nguồn
git checkout main
git push origin main
```
- **Kết quả:** 
  1. Job `deploy-prod` bắt đầu và tạm dừng ở trạng thái `Waiting for review`.
  2. Bấm vào nút **Review deployments** trên giao diện Actions $\rightarrow$ Tích chọn môi trường `production` $\rightarrow$ Nhấn **Approve and deploy**.
  3. Job tiếp tục thực thi và chuyển sang trạng thái **Tích xanh (Success)**.
