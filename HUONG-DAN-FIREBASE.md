# Hướng dẫn kết nối Firebase cho web Quỹ tạm ứng

Web (`quy-tam-ung.html`) đã được lập trình sẵn để đồng bộ dữ liệu 2 chiều qua
**Firebase Realtime Database** (số liệu tạm ứng/chi tiêu) và **Firebase Storage**
(ảnh/PDF hóa đơn), cộng thêm **auto-save**: mọi lần thêm/sửa/xóa đều tự lưu ngay,
không cần bấm nút "Lưu" riêng.

Hiện tại `firebaseConfig` trong file đang để trống, nên web chạy ở chế độ
**lưu tạm trên từng máy** (không đồng bộ nhiều thiết bị). Làm theo các bước dưới
đây để bật đồng bộ Firebase thật.

---

## Bước 1 — Tạo project Firebase (miễn phí)

1. Vào https://console.firebase.google.com/ và đăng nhập bằng tài khoản Google.
2. Bấm **"Add project" / "Thêm dự án"**.
3. Đặt tên, ví dụ `quy-tam-ung-jp`. Có thể tắt Google Analytics (không cần cho web này).
4. Bấm **Create project**, đợi vài chục giây.

## Bước 2 — Bật Realtime Database

1. Trong menu bên trái: **Build → Realtime Database**.
2. Bấm **Create Database**.
3. Chọn khu vực gần nhất (ví dụ `Singapore (asia-southeast1)`).
4. Ở bước chọn chế độ bảo mật, chọn **"Start in locked mode"** (mặc định khóa hết —
   ta sẽ tự cấu hình Rules đúng ở Bước 5, không dùng "test mode" mở toang).

## Bước 3 — Bật Anonymous Authentication (đăng nhập ẩn danh)

Web sẽ tự đăng nhập ẩn danh ở chế độ nền (không có màn hình đăng nhập cho người
dùng thấy) để Firebase biết yêu cầu đọc/ghi đến từ chính web này, không phải từ
người lạ gọi thẳng vào API.

1. Menu bên trái: **Build → Authentication → Get started**.
2. Tab **Sign-in method** → chọn **Anonymous** → bật **Enable** → **Save**.

## Bước 4 — Bật Firebase Storage (để đồng bộ ảnh/PDF hóa đơn)

1. Menu bên trái: **Build → Storage → Get started**.
2. Nếu được yêu cầu **nâng cấp lên gói Blaze (Pay as you go)**: đây là gói trả
   theo dùng của Google (bắt buộc phải khai báo thẻ khi tạo Storage cho project
   mới), **nhưng vẫn có mức miễn phí hàng tháng rất rộng** (5GB lưu trữ,
   1GB/ngày tải xuống) — với quy mô hóa đơn nội bộ công ty gần như không bao giờ
   phát sinh phí. Vẫn nên đặt **ngân sách cảnh báo (budget alert)** trong phần
   Billing để yên tâm.
3. Chọn khu vực giống Bước 2, bấm **Done**.

*(Chị chưa muốn khai báo thẻ thì bỏ qua bước này hoàn toàn — không sao cả. Bỏ
trống `storageBucket` ở Bước 6: web vẫn đồng bộ đầy đủ số tiền/ngày/ghi chú
qua Firebase như bình thường, chỉ riêng ảnh/PDF hóa đơn sẽ lưu tại máy nào
tải lên thì xem trên máy đó, không tự động hiện trên máy khác. Không cần
sửa gì thêm trong code — web tự nhận ra chưa có Storage và xử lý đúng như vậy.)*

## Bước 5 — Cấu hình Rules (bảo mật dữ liệu)

**Realtime Database → tab Rules**, thay nội dung bằng:

```json
{
  "rules": {
    "fundData": {
      ".read": "auth != null",
      ".write": "auth != null"
    }
  }
}
```

**Storage → tab Rules** (chỉ áp dụng nếu chị đã làm Bước 4; bỏ qua nếu chưa dùng Storage), thay nội dung bằng:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /receipts/{fileName} {
      allow read, write: if request.auth != null;
    }
  }
}
```

Bấm **Publish** ở cả hai. Rules này chỉ cho phép đọc/ghi khi đã đăng nhập
(kể cả ẩn danh) — tức là chỉ web này (sau khi qua màn hình PIN) mới ghi được,
người ngoài gọi thẳng API Firebase sẽ bị chặn.

## Bước 6 — Lấy `firebaseConfig` và dán vào web

1. Bấm biểu tượng **⚙️ (Project settings)** ở góc trên bên trái.
2. Kéo xuống mục **"Your apps"** → bấm biểu tượng **`</>`** (Web) → đặt tên bất kỳ
   → **Register app**.
3. Firebase hiện ra một đoạn `const firebaseConfig = {...}` — copy toàn bộ.
4. Mở file `quy-tam-ung.html`, tìm dòng `const firebaseConfig = {` (khoảng dòng
   630) và dán đè các giá trị (`apiKey`, `authDomain`, `databaseURL`,
   `projectId`, `storageBucket`, `messagingSenderId`, `appId`).
   - Lưu ý `databaseURL` không tự có trong đoạn code Firebase đưa — lấy từ
     trang Realtime Database (dạng
     `https://<project-id>-default-rtdb.<region>.firebasedatabase.app`).

Sau khi lưu file, mở lại web: nếu cấu hình đúng, góc trên bên phải khu vực
"Đã cập nhật" sẽ hiện **"Đã đồng bộ · ..."** — nghĩa là mọi thiết bị mở cùng
link này đều thấy dữ liệu giống nhau ngay lập tức, không cần tải lại trang.

---

## Đổi mã PIN truy cập

Trong `quy-tam-ung.html`, tìm dòng:

```js
const SHARED_PIN = "2609";
```

Đổi thành mã PIN chị muốn (chỉ nội bộ công ty biết). Đây là lớp chặn đơn giản
phía trình duyệt (mỗi lần mở web trên thiết bị/tab mới sẽ hỏi lại PIN) —
lớp bảo mật thật sự nằm ở Firebase Rules tại Bước 5.

## Dự phòng Google Sheets

Web vẫn giữ phương án dự phòng qua Google Sheets (Apps Script) đã cấu hình sẵn
trong file. Khi Firebase hoạt động bình thường, phương án này không được dùng
tới; nó chỉ tự kích hoạt khi Firebase gặp lỗi/mất mạng lúc lưu hoặc tải dữ
liệu, để tránh mất dữ liệu vừa nhập.

## Lưu ý triển khai

- File `quy-tam-ung.html` là một trang tĩnh — có thể mở trực tiếp, hoặc host
  miễn phí qua Firebase Hosting / GitHub Pages để có một đường link cố định
  gửi cho mọi người trong công ty.
- `apiKey` của Firebase Web không phải bí mật tuyệt đối (nó luôn lộ ra trong
  mã nguồn phía trình duyệt) — an toàn dữ liệu phụ thuộc vào **Rules** ở
  Bước 5, không phụ thuộc việc giấu `apiKey`.
