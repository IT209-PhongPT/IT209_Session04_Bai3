# IT209 - Session 04 - Bài tập 3: Cấu hình xác thực SSH và Đẩy dự án lên GitHub

- **Sinh viên:** Phạm Thanh Phong
- **MSSV:** N24DTCN120
- **Lớp / Học phần:** IT209 - Phát triển ứng dụng Web / Cloud Infrastructure
- **Remote Origin (SSH):** `git@github.com:thanhphong-ptit/it209-session04-ssh-practice.git`

---

## Tóm tắt nội dung thực hiện

1. **Tạo cặp khóa SSH Ed25519:**
   - Lệnh: `ssh-keygen -t ed25519 -C "thanhphong102006@gmail.com"`
   - Thuật toán Ed25519 cung cấp độ dài khóa ngắn gọn (256-bit), tốc độ mã hóa cao và an toàn vượt trội so với RSA truyền thống.
2. **Đăng ký Public Key lên GitHub:**
   - Copy nội dung `~/.ssh/id_ed25519.pub` và dán vào **GitHub Settings -> SSH and GPG keys**.
3. **Kiểm tra xác thực kết nối:**
   - Lệnh: `ssh -T git@github.com`
   - Xác nhận phản hồi: `Hi <username>! You've successfully authenticated...`
4. **Cấu hình Remote URL dạng SSH:**
   - Lệnh: `git remote add origin git@github.com:thanhphong-ptit/it209-session04-ssh-practice.git`
   - Kiểm tra: `git remote -v`
5. **Đẩy mã nguồn lên Remote Repository:**
   - Lệnh: `git push -u origin main`

---

> 📄 **Xem báo cáo chi tiết đầy đủ:** [IT209_Session04_BT3_PhamThanhPhong_N24DTCN120.md](./IT209_Session04_BT3_PhamThanhPhong_N24DTCN120.md)
