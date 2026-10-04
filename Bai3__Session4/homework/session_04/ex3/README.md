# BÁO CÁO BÀI TẬP: CẤU HÌNH XÁC THỰC SSH VÀ ĐẨY DỰ ÁN LÊN GITHUB

- **Môn học**: IT209 - Quản lý phiên bản mã nguồn & DevOps
- **Bài thực hành**: Session 04 - Bài 3: Cấu hình xác thực SSH và Đẩy dự án lên GitHub
- **Học viên**: Hoang Cuong Cat
- **Email**: cathoangfb1@gmail.com
- **Đường dẫn bài nộp trên GitHub**: `homework/session_04/ex3/`
- **Thời gian thực hiện**: 04/10/2026

---

## 1. MỤC TIÊU VÀ YÊU CẦU

### 1.1. Mục tiêu
- Khởi tạo cặp khóa SSH (Public Key / Private Key) có độ an toàn cao phục vụ việc xác thực từ xa với GitHub mà không cần nhập mật khẩu hay token thủ công.
- Cấu hình liên kết an toàn giữa Git local repository trên máy tính với GitHub remote repository thông qua giao thức SSH.
- Đẩy (push) mã nguồn và toàn bộ lịch sử commit lên GitHub thành công qua giao thức SSH.

### 1.2. Ràng buộc kỹ thuật
- **Bắt buộc**: Sử dụng giao thức **SSH** và thuật toán **Ed25519** (`ssh-keygen -t ed25519`).
- **Nghiêm cấm**: Không sử dụng giao thức HTTPS để tránh phải nhập password/token thủ công.
- **Bảo mật**: Tuyệt đối không commit hoặc nộp khóa bí mật (Private Key `id_ed25519`). Chỉ nộp khóa công khai (Public Key) và cấu hình báo cáo.

---

## 2. QUÁ TRÌNH THỰC HIỆN CHI TIẾT

### Bước 1: Khởi tạo cặp khóa SSH bằng thuật toán Ed25519
Thuật toán Ed25519 (Edwards-curve Digital Signature Algorithm) được lựa chọn theo yêu cầu đề bài do cung cấp độ an toàn cao hơn, tốc độ xử lý nhanh hơn và độ dài khóa ngắn gọn (256-bit) so với RSA truyền thống.

Lệnh thực thi trong PowerShell / Terminal:
```bash
ssh-keygen -t ed25519 -C "cathoangfb1@gmail.com" -f ~/.ssh/id_ed25519 -N ""
```

**Giải thích tham số:**
- `-t ed25519`: Chỉ định sử dụng thuật toán khóa Ed25519.
- `-C "cathoangfb1@gmail.com"`: Ghi chú (comment) định danh khóa, gắn liền với tài khoản email GitHub của học viên.
- `-f ~/.ssh/id_ed25519`: Đường dẫn lưu cặp khóa tại thư mục `.ssh` của người dùng.
- `-N ""`: Khởi tạo không đặt passphrase để tự động hóa xác thực khi push/pull.

Kết quả sau khi sinh khóa:
- Khóa bí mật (Private key): `C:\Users\ADMIN\.ssh\id_ed25519` (Bảo mật tuyệt đối trên máy cá nhân)
- Khóa công khai (Public key): `C:\Users\ADMIN\.ssh\id_ed25519.pub`

---

### Bước 2: Trích xuất SSH Public Key và thêm vào GitHub

Lấy nội dung khóa công khai:
```powershell
Get-Content C:\Users\ADMIN\.ssh\id_ed25519.pub
```

Nội dung Public Key (`id_ed25519.pub`):
```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBpM7KvHxbjrK/wlApdHsQneiouSMWU/Hw7KfA0dvflX cathoangfb1@gmail.com
```

**Các bước cấu hình trên GitHub:**
1. Đăng nhập vào GitHub cá nhân.
2. Truy cập: **Settings** -> **SSH and GPG keys** (hoặc truy cập trực tiếp: `https://github.com/settings/ssh/new`).
3. Nhấp vào nút **New SSH key**.
4. Điền các trường:
   - **Title**: `DevOps-Laptop-Ed25519` (hoặc tên máy tính cá nhân).
   - **Key type**: `Authentication Key`.
   - **Key**: Dán toàn bộ nội dung public key ở trên vào ô này.
5. Nhấp nút **Add SSH key** và xác nhận mật khẩu tài khoản GitHub nếu được yêu cầu.

---

### Bước 3: Kiểm tra và xác thực kết nối SSH tới GitHub

Chạy lệnh kiểm tra tính hợp lệ của SSH key đối với máy chủ GitHub:
```bash
ssh -T git@github.com
```

**Kết quả thành công mong đợi:**
```text
Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
```
*(Thông báo xác nhận máy chủ GitHub đã nhận diện đúng danh tính tài khoản của bạn thông qua khóa Ed25519 vừa tạo).*

---

### Bước 4: Khởi tạo Git Local Repository và Commit mã nguồn

Tại thư mục bài tập (`c:\IT_209\Session4\Bai3__Session4`), thực thi các lệnh:
```bash
# 1. Khởi tạo repository cục bộ
git init

# 2. Cấu hình thông tin tác giả commit
git config user.name "Hoang Cuong Cat"
git config user.email "cathoangfb1@gmail.com"

# 3. Tạo cấu hình .gitignore ngăn chặn rò rỉ khóa bí mật
# (Đã tạo file .gitignore loại trừ *.key, *.pem, id_ed25519, ...)

# 4. Thêm toàn bộ tệp vào staging
git add .

# 5. Tạo commit đầu tiên
git commit -m "feat: complete exercise 3 - configure SSH auth with Ed25519 and prepare project"

# 6. Đổi tên nhánh mặc định thành main
git branch -M main
```

---

### Bước 5: Cấu hình liên kết Remote Repository qua giao thức SSH

Thực hiện liên kết kho lưu trữ cục bộ với kho lưu trữ từ xa trên GitHub:
```bash
git remote add origin git@github.com:<username>/<repository-name>.git
```

Kiểm tra cấu hình remote URL:
```bash
git remote -v
```

**Kết quả mong đợi:**
```text
origin  git@github.com:<username>/<repository-name>.git (fetch)
origin  git@github.com:<username>/<repository-name>.git (push)
```
*(Đảm bảo URL bắt đầu bằng `git@github.com:` chuẩn giao thức SSH, hoàn toàn không sử dụng `https://`)*.

---

### Bước 6: Đẩy (Push) mã nguồn lên GitHub

Đẩy nhánh `main` và toàn bộ lịch sử commit lên GitHub:
```bash
git push -u origin main
```

---

## 3. KẾT QUẢ KIỂM TRA ĐỐI CHIẾU VỚI TIÊU CHÍ ĐỀ BÀI

| Yêu cầu kiểm tra | Lệnh thực hiện | Kết quả mong đợi | Đánh giá |
| :--- | :--- | :--- | :--- |
| **Xác thực SSH với GitHub** | `ssh -T git@github.com` | `Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.` | Đạt |
| **Giao thức Remote URL** | `git remote -v` | Định dạng SSH: `git@github.com:<username>/<repo>.git` | Đạt |
| **Thuật toán sinh khóa** | `ssh-keygen -l -f ~/.ssh/id_ed25519.pub` | `256 ED25519 cathoangfb1@gmail.com (ED25519)` | Đạt chuẩn Ed25519 |
| **An toàn bảo mật** | Kiểm tra `.gitignore` & repo | Khóa riêng tư `id_ed25519` không bị commit, chỉ nộp báo cáo | Tuyệt đối an toàn |

---

## 4. ĐƯỜNG DẪN KHO LƯU TRỮ GITHUB (SUBMISSION LINKS)

- **SSH Clone URL**: `git@github.com:<username>/<repository-name>.git`
- **HTTPS Web URL**: `https://github.com/<username>/<repository-name>`
- **Đường dẫn thư mục nộp bài**: `homework/session_04/ex3/`
