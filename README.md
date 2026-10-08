# Xác thực người dùng với NextAuth.js v5

Bài tập đã hoàn thành phần cấu hình xác thực cho ứng dụng Next.js App Router. Người dùng đăng nhập bằng GitHub OAuth và quản lý phiên bằng JWT, không lưu session trong database.

## Công nghệ

- Next.js 16.3.8 (App Router), React 19 và TypeScript.
- NextAuth.js 5.0.0-beta.32.
- GitHub OAuth là phương thức đăng nhập duy nhất.

## Các phần đã hoàn thành

### Cấu hình xác thực — `auth.ts`

- Thay các hàm giả bằng cấu hình `NextAuth`.
- Sử dụng GitHub provider từ `next-auth/providers/github`.
- Đặt `session.strategy` là `jwt`.
- Export `handlers`, `auth`, `signIn` và `signOut`.
- Giữ payload JWT mặc định với thông tin người dùng cơ bản; không thêm OAuth access token hoặc secret vào payload.

### Route Handler — `app/api/auth/[...nextauth]/route.ts`

- Import `handlers` từ `@/auth`.
- Export `GET` và `POST` để NextAuth xử lý đăng nhập, callback, session và đăng xuất.

Giao diện `app/page.tsx` và cấu trúc thư mục được giữ nguyên theo yêu cầu bài tập. NextAuth tự đọc các biến môi trường `AUTH_SECRET`, `AUTH_GITHUB_ID` và `AUTH_GITHUB_SECRET`.

## Cài đặt và chạy

### 1. Cài dependencies

Clone repository fork của bạn và mở terminal tại thư mục dự án, sau đó chạy:

```bash
npm install
```

NextAuth đã được khai báo trong `package.json`, không cần cài riêng bằng `npm install next-auth@beta`.

### 2. Cấu hình môi trường

Tạo `.env.local` ở thư mục gốc theo `.env.example`:

```dotenv
AUTH_SECRET=""
AUTH_GITHUB_ID=""
AUTH_GITHUB_SECRET=""
AUTH_URL=http://localhost:3000
```

Sinh secret ngẫu nhiên bằng lệnh sau, rồi sao chép kết quả vào `AUTH_SECRET`:

```bash
node -e "console.log(require('node:crypto').randomBytes(32).toString('base64'))"
```

Nếu `.env.local` đã có `AUTH_SECRET`, giữ nguyên giá trị đó. Không chia sẻ hoặc commit `.env.local`; file này đã được bỏ qua trong `.gitignore`.

### 3. Tạo GitHub OAuth App

1. Đăng nhập GitHub, mở **Settings → Developer settings → OAuth Apps → New OAuth App** hoặc truy cập [OAuth Apps](https://github.com/settings/developers).
2. Điền các trường:

   | Trường | Giá trị |
   | --- | --- |
   | Application name | `NextAuth Local` hoặc tên bạn muốn |
   | Homepage URL | `http://localhost:3000` |
   | Application description | Có thể để trống |
   | Authorization callback URL / Redirect URI | `http://localhost:3000/api/auth/callback/github` |

3. Bấm **Register application**.
4. Sao chép **Client ID** vào `AUTH_GITHUB_ID`.
5. Bấm **Generate a new client secret** và sao chép giá trị vào `AUTH_GITHUB_SECRET`.

### 4. Khởi động ứng dụng

```bash
npm run dev
```

Nếu PowerShell báo `npm.ps1 cannot be loaded because running scripts is disabled`, dùng `npm.cmd`:

```powershell
npm.cmd install
npm.cmd run dev
```

Mở [http://localhost:3000](http://localhost:3000) khi terminal báo **Ready**. Khởi động lại server sau khi sửa `.env.local`. Nếu dùng cổng khác, cập nhật cả `AUTH_URL`, Homepage URL và callback URL cho khớp.

## Kiểm tra kết quả

### Kiểm tra đăng nhập thực tế

Sau khi điền thông tin GitHub OAuth App:

1. Mở trang chủ: hiển thị **Chưa đăng nhập**.
2. Bấm **Đăng nhập bằng GitHub** và chấp thuận quyền truy cập trên GitHub.
3. Sau khi callback về trang chủ, giao diện hiển thị **Đã đăng nhập thành công!**, tên và email mà GitHub cung cấp.
4. Tải lại trang để kiểm tra phiên được duy trì.
5. Bấm **Đăng xuất**: trang chủ trở lại trạng thái **Chưa đăng nhập**.

### Kết quả kiểm tra mã nguồn

Các kiểm tra đã thực hiện khi hoàn thành cấu hình:

- `npm run build`: đạt, bao gồm kiểm tra TypeScript.
- Lint riêng hai file xác thực: đạt.
- Trang chủ, `/api/auth/providers`, `/api/auth/session`, `/api/auth/csrf` và `/api/auth/signin`: phản hồi HTTP 200 khi có secret và URL localhost hợp lệ.
- Khi chưa đăng nhập, API session trả về `null` và trang đăng nhập hiển thị GitHub.
- POST `/api/auth/signout` với CSRF token hợp lệ: phản hồi HTTP 200 và trả URL chuyển hướng về trang chủ.

Luồng đăng nhập GitHub thực tế cần kiểm tra bằng OAuth App của bạn; các kiểm tra trên không xác nhận việc cấp quyền và callback với tài khoản GitHub thật.

Chạy lại build hoặc lint bằng:

```bash
npm run build
npm run lint
```

Lint toàn dự án còn báo lỗi `@next/next/no-html-link-for-pages` ở thẻ `<a>` trong `app/page.tsx`. File này được giữ nguyên theo ràng buộc bài tập. Có thể kiểm tra riêng phần xác thực bằng:

```bash
npx eslint auth.ts "app/api/auth/[...nextauth]/route.ts"
```

Trên PowerShell nếu bị chặn script, thay `npm` bằng `npm.cmd` và `npx` bằng `npx.cmd`. Build cần kết nối tới Google Fonts để tải các font Geist trong layout có sẵn. Khi chạy production, đặt `AUTH_URL` phù hợp với địa chỉ ứng dụng để tránh lỗi `UntrustedHost`.

## Nộp bài

Sau khi kiểm tra đăng nhập và đăng xuất, commit và push các file của bài tập lên repository fork:

```powershell
 git add auth.ts "app/api/auth/[...nextauth]/route.ts" package-lock.json README.md
 git diff --cached --stat
 git commit -m "Hoan thanh bai tap NextAuth"
 git push origin main
```

Nếu code đã được commit trước đó, lệnh trên sẽ tạo commit cho các thay đổi còn lại. Trên GitHub, mở repository fork → **Contribute → Open pull request** hoặc **Compare & pull request**, kiểm tra repository và nhánh đích, rồi bấm **Create pull request** để nộp bài.
