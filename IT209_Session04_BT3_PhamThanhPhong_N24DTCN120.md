# Báo cáo Bài tập 3: Cấu hình Xác thực SSH và Đẩy Dự án lên GitHub

**Thông tin sinh viên:**
- **Họ và tên:** Phạm Thanh Phong
- **Mã sinh viên / Lớp:** N24DTCN120
- **Môn học:** IT209 - Phát triển ứng dụng Web / Cloud Infrastructure
- **Session:** Session 04 - Quản lý phiên bản mã nguồn với Git & GitHub
- **Bài tập:** Bài tập 3 (ex3)

---

## 1. Mục tiêu & Bối cảnh bài toán

### 1.1. Bối cảnh (Context)
Khi làm việc với các hệ thống quản lý mã nguồn đám mây như GitHub / GitLab trong quy trình CI/CD và cộng tác kỹ thuật, việc sử dụng giao thức HTTPS đòi hỏi lập trình viên phải nhập Personal Access Token (PAT) thủ công hoặc xử lý xác thực bảo mật nhiều lớp phức tạp. Để tự động hóa, tăng tốc độ tương tác và nâng cao mức độ an toàn thông tin, giao thức **SSH (Secure Shell)** kết hợp thuật toán mã hóa khóa công khai hiện đại (**Ed25519**) là tiêu chuẩn vàng được áp dụng rộng rãi.

### 1.2. Mục tiêu kỹ thuật
- Khởi tạo cặp khóa SSH Keypair bất đối xứng (Asymmetric Keypair) bằng thuật toán **Ed25519** với comment định danh email tác giả.
- Cấu hình Public Key lên tài khoản GitHub và bảo vệ Private Key an toàn trên máy cục bộ.
- Kiểm tra tính hợp lệ của phiên xác thực SSH tới GitHub bằng lệnh `ssh -T git@github.com`.
- Thiết lập liên kết kho lưu trữ từ xa (**Remote Origin**) sử dụng định dạng URL SSH (`git@github.com:...`).
- Đẩy toàn bộ nhánh `main` và lịch sử commit lên kho lưu trữ GitHub từ xa (`git push -u origin main`).

### 1.3. Ràng buộc kỹ thuật
- **Thuật toán sinh khóa:** Bắt buộc sử dụng Ed25519 (`ssh-keygen -t ed25519`), không sử dụng RSA lỗi thời.
- **Giao thức liên kết:** URL của remote origin bắt buộc ở dạng SSH `git@github.com:<username>/<repo_name>.git` (không dùng URL `https://github.com/...`).
- **Nguyên tắc bảo mật:** Tuyệt đối không chia sẻ hoặc tải tệp Private Key (`id_ed25519`) lên bất kỳ kho lưu trữ nào.

---

## 2. Cơ sở lý thuyết: Cơ chế xác thực bất đối xứng qua SSH & Ed25519

### 2.1. Kiến trúc xác thực SSH Keypair

```
┌────────────────────────────────────────┐                ┌────────────────────────────────────────┐
│              Máy Local                 │                │             GitHub Server              │
│                                        │                │                                        │
│  ┌──────────────────────────────────┐  │   SSH Handshake│  ┌──────────────────────────────────┐  │
│  │   Private Key (~/.ssh/id_ed25519)│  │ ─────────────► │  │  Public Key (Authorized Keys)    │  │
│  │   (Bảo mật tuyệt đối trên máy)   │  │ ◄───────────── │  │  (Lưu trong GitHub Account)      │  │
│  └──────────────────────────────────┘  │  Xác thực ký số│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘                └────────────────────────────────────────┘
```

1. **Private Key (`id_ed25519`):** Khóa bí mật chỉ lưu trữ trên máy cá nhân của lập trình viên, dùng để tạo chữ ký số (digital signature) cho mỗi phiên kết nối.
2. **Public Key (`id_ed25519.pub`):** Khóa công khai được tải lên GitHub, dùng để giải mã và xác minh chữ ký số của client.
3. **Ưu điểm của Ed25519 so với RSA:**
   - **Kích thước nhỏ gọn:** Khóa 256-bit của Ed25519 có độ an toàn tương đương khóa RSA 3072-bit, giúp tối ưu hóa thời gian xử lý bắt tay.
   - **Chống tấn công Side-channel:** Thuật toán Ed25519 được thiết kế chống lại các kiểu tấn công đo thời gian (timing attacks).

---

## 3. Quy trình thực hiện chi tiết

### Bước 1: Khởi tạo cặp khóa SSH Ed25519
Mở PowerShell/Terminal trên máy trạm và thực thi lệnh sinh khóa:
```bash
ssh-keygen -t ed25519 -C "thanhphong102006@gmail.com"
```
- Khi hệ thống hỏi đường dẫn lưu file (`Enter file in which to save the key`), nhấn **Enter** để chọn đường dẫn mặc định `C:\Users\ACER\.ssh\id_ed25519`.
- Nhập Passphrase bảo vệ khóa (hoặc nhấn **Enter** nếu không dùng mật khẩu phụ).

---

### Bước 2: Đăng ký Public Key lên GitHub
1. Mở và sao chép toàn bộ nội dung tệp khóa công khai `id_ed25519.pub`:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   # Hoặc trên Windows PowerShell:
   Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard
   ```
2. Truy cập vào **GitHub** -> **Settings** -> **SSH and GPG keys** -> chọn **New SSH Key**.
3. Điền các trường thông tin:
   - **Title:** `IT209-Laptop-PhamThanhPhong`
   - **Key type:** `Authentication Key`
   - **Key:** Dán chuỗi Public Key bắt đầu bằng `ssh-ed25519 ...`
4. Bấm **Add SSH Key** để hoàn tất.

---

### Bước 3: Kiểm tra xác thực kết nối SSH tới máy chủ GitHub
Chạy lệnh kiểm tra bắt tay bảo mật:
```bash
ssh -T git@github.com
```
*Lần đầu tiên kết nối, nhập `yes` khi hệ thống hỏi `Are you sure you want to continue connecting (yes/no/[fingerprint])?` để thêm GitHub vào `known_hosts`.*

---

### Bước 4: Thiết lập Remote URL dạng SSH và Đẩy mã nguồn
1. Khởi tạo repository và cấu hình remote:
   ```bash
   # Di chuyển vào thư mục dự án
   cd IT209_Session04_BT3_PhamThanhPhong_N24DTCN120

   # Khởi tạo repo và thiết lập nhánh main
   git init -b main

   # Cấu hình danh tính cục bộ
   git config --local user.name "Pham Thanh Phong"
   git config --local user.email "thanhphong102006@gmail.com"

   # Thêm remote origin sử dụng đường dẫn SSH
   git remote add origin git@github.com:thanhphong-ptit/it209-session04-ssh-practice.git
   ```

2. Kiểm tra cấu hình remote URL:
   ```bash
   git remote -v
   ```

3. Đóng gói và đẩy dữ liệu lên GitHub:
   ```bash
   # Tạo commit ban đầu
   git add .
   git commit -m "feat: Initial commit for SSH and remote GitHub integration"

   # Đẩy nhánh main lên remote repository và thiết lập tracking
   git push -u origin main
   ```

---

## 4. Nhật ký thực thi & Minh chứng nghiệm thu (Verification Logs)

### 4.1. Minh chứng kiểm tra xác thực SSH tới GitHub
```bash
ssh -T git@github.com
```
**Output mong đợi:**
```text
Hi thanhphong-ptit! You've successfully authenticated, but GitHub does not provide shell access.
```

---

### 4.2. Minh chứng cấu hình Remote Origin sử dụng URL SSH
```bash
git remote -v
```
**Output thực tế:**
```text
origin	git@github.com:thanhphong-ptit/it209-session04-ssh-practice.git (fetch)
origin	git@github.com:thanhphong-ptit/it209-session04-ssh-practice.git (push)
```

---

### 4.3. Minh chứng đẩy nhánh lên Remote (`git push`)
```bash
git push -u origin main
```
**Output mẫu khi đẩy thành công:**
```text
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 16 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 1.25 KiB | 1.25 MiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:thanhphong-ptit/it209-session04-ssh-practice.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

---

## 5. Hướng dẫn nộp bài & Cấu trúc kho lưu trữ

- **Đường dẫn trên GitHub:** `homework/session_04/ex3/`
- **Tệp báo cáo:** `README.md` (hoặc `IT209_Session04_BT3_PhamThanhPhong_N24DTCN120.md`)
- **Kho lưu trữ dự án tham chiếu:** `git@github.com:thanhphong-ptit/it209-session04-ssh-practice.git`
- **Cấu trúc nộp bài chuẩn:**
```text
homework/session_04/ex3/
├── .git/
├── IT209_Session04_BT3_PhamThanhPhong_N24DTCN120.md
└── README.md
```

---

## 6. Tổng kết & Đánh giá an toàn thông tin

1. **Bảo mật vượt trội với SSH Ed25519:** Loại bỏ nguy cơ lộ mật khẩu hoặc token cá nhân trong quá trình làm việc, tối ưu hóa quá trình xác thực với hạ tầng Git.
2. **Quy tắc bảo mật Private Key:** Không bao giờ đưa tệp `id_ed25519` vào bất kỳ thư mục dự án nào hoặc đẩy lên mạng.
3. **Tính thuận tiện trong phát triển phần mềm:** Một khi SSH Key đã được nạp và kiểm tra thành công, toàn bộ thao tác `push`, `pull`, `fetch` đều diễn ra hoàn toàn tự động, nhanh chóng và bảo mật.
