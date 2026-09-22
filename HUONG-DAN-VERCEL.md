# Hướng dẫn deploy web Quỹ tạm ứng lên Vercel

Web này chỉ là **một file tĩnh** (`index.html`, không có bước build nào),
nên đưa lên Vercel rất đơn giản — không cần cấu hình framework, không cần
build command.

## Cách 1 — Deploy qua trang web Vercel (khuyên dùng, không cần cài gì)

1. Vào https://vercel.com/ → đăng nhập (có thể đăng nhập thẳng bằng tài
   khoản GitHub cho tiện).
2. Bấm **Add New... → Project**.
3. Ở mục **Import Git Repository**, chọn repo `qu-t-m-ng-JP` (nếu chưa thấy,
   bấm **Adjust GitHub App Permissions** để cấp quyền cho Vercel đọc repo).
4. Ở màn hình cấu hình project:
   - **Framework Preset**: để **Other** (Vercel tự nhận đây là site tĩnh vì
     có sẵn `index.html` ở thư mục gốc).
   - **Build Command**: để trống.
   - **Output Directory**: để trống (mặc định là thư mục gốc).
   - **Root Directory**: để nguyên `.`
5. Bấm **Deploy**. Đợi khoảng 10–20 giây là xong, Vercel cho một link dạng
   `https://ten-du-an.vercel.app`.

Từ lần sau, **mỗi lần code trên nhánh này được cập nhật (push lên GitHub),
Vercel tự động deploy lại** — không cần làm lại từ đầu.

## Cách 2 — Deploy bằng Vercel CLI (nếu thích dùng terminal)

```bash
npm i -g vercel     # cài Vercel CLI (một lần)
cd qu-t-m-ng-JP
vercel               # lần đầu sẽ hỏi đăng nhập + chọn project, làm theo hướng dẫn
vercel --prod        # deploy bản chính thức
```

## Sau khi deploy — nhớ 2 việc

1. **Thêm domain Vercel vào Firebase** (nếu đã làm `HUONG-DAN-FIREBASE.md`):
   Firebase Console → **Authentication → Settings → Authorized domains** →
   **Add domain** → dán domain Vercel (`ten-du-an.vercel.app`, và cả domain
   riêng nếu gắn thêm). Thiếu bước này thì đăng nhập ẩn danh của web (dùng để
   xác thực với Realtime Database/Storage) có thể bị chặn trên domain mới,
   web sẽ báo lỗi đồng bộ dù cấu hình `firebaseConfig` đúng.
2. **Gắn domain riêng (tùy chọn)**: trong project trên Vercel → tab
   **Settings → Domains** → **Add** → nhập domain công ty (nếu có) và làm
   theo hướng dẫn trỏ DNS. Nhớ thêm domain riêng đó vào Authorized domains ở
   bước 1 luôn.

## Vì sao không cần `vercel.json`

Vì `index.html` nằm sẵn ở thư mục gốc và không có `package.json`, Vercel tự
nhận đây là site tĩnh và phục vụ trực tiếp — vào domain gốc là hiện đúng
trang Quỹ tạm ứng, không cần cấu hình rewrite/route gì thêm.
