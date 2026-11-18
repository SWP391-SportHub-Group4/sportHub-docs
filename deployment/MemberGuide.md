# SportHub Member Guide — Kết nối hệ thống

**Cập nhật ngày 18/11/2026**

Tài liệu dành cho thành viên nhóm phát triển SportHub, hướng dẫn truy cập website, kết nối frontend với API, chạy hệ thống trên máy và dùng database AWS được cấp quyền. Cấu hình được đối chiếu với mã nguồn ứng dụng tại commit `8bbe1657`. Địa chỉ website và API bên dưới lấy từ tài liệu repository. Người vận hành xác nhận endpoint, chế độ DB và phiên bản đang triển khai trước khi thành viên dùng dữ liệu chung.

## 1. Chọn cách kết nối

| Nhu cầu | Cách kết nối | Phần cần chuẩn bị |
| --- | --- | --- |
| Trải nghiệm hệ thống đã triển khai | Trình duyệt → website Vercel → API AWS | Tài khoản ứng dụng đúng vai trò |
| Phát triển và kiểm thử trên dữ liệu riêng | Frontend local → API local → PostgreSQL local | Git, Node.js 24, npm 11, .NET SDK 10 và Docker Desktop |
| Chạy API local với dữ liệu website | Frontend local → API local → PostgreSQL AWS | Phiên bản mã tương thích schema, tài khoản DB riêng và kết nối TLS hoặc SSH tunnel |
| Xem DB bằng DBeaver/pgAdmin | Công cụ DB → PostgreSQL AWS | Tài khoản DB riêng, CA công khai hoặc SSH tunnel theo cấu hình được cấp |

Tài khoản đăng nhập website, tài khoản PostgreSQL và tài khoản SSH được cấp riêng cho từng mục đích. Xem vị trí các thành phần trong [Deployment Design](SportHub_Deployment_Design.md).

## 2. Truy cập website đang triển khai

1. Mở [website SportHub](https://sports-center-management-system-tawny.vercel.app/).
2. Đăng nhập bằng tài khoản được cấp. Hội viên có thể dùng luồng đăng ký trên website. Thành viên cần thử vai trò nhân sự nhận tài khoản tương ứng từ người quản trị.
3. Mở trang nghiệp vụ phù hợp với vai trò, chẳng hạn `/member`, `/coach`, `/receptionist`, `/manager` hoặc `/admin`.
4. Khi cần kiểm tra kết nối API, mở endpoint được tài liệu AWS ghi nhận là `https://sporthub-vie.duckdns.org/api/health`. Người vận hành xác nhận hostname hiện tại nếu địa chỉ này đã đổi.

Đăng nhập được và đọc được dữ liệu nghiệp vụ là bước kiểm tra sau khi endpoint health phản hồi. Khi báo lỗi, cung cấp thời điểm, URL, vai trò và mã HTTP. Che token, mật khẩu và dữ liệu cá nhân trong ảnh hoặc log gửi kèm.

## 3. Chuẩn bị mã nguồn và cấu hình local

Với máy chưa có mã nguồn, chạy lệnh sau trong thư mục làm việc bạn chọn.

```powershell
git clone https://github.com/SWP391-SportHub-Group4/sports-center-management-system.git
cd sports-center-management-system
```

Các lệnh còn lại dùng thư mục gốc repository ứng dụng làm điểm bắt đầu, trừ khi có bước `cd frontend`. Khi dùng DB website, người vận hành cung cấp commit hoặc nhánh tương thích với schema đang triển khai.

Kiểm tra các công cụ đã cài.

```powershell
git --version
node --version
npm --version
dotnet --version
docker compose version
```

Chỉ tạo file cấu hình khi file đó chưa tồn tại để giữ cấu hình cá nhân đã có.

```powershell
if (-not (Test-Path .env)) {
    Copy-Item .env.example .env
}
if (-not (Test-Path frontend/.env.local)) {
    Copy-Item frontend/.env.example frontend/.env.local
}
```

Backend đọc `.env` ở gốc repository. Next.js đọc `frontend/.env.local`. Điền giá trị vào các dòng có sẵn và giữ một giá trị hiệu lực cho mỗi biến. `JwtOptions__SecretKey` dùng cho backend local cần là chuỗi ngẫu nhiên riêng dài ít nhất 32 ký tự.

## 4. Chạy toàn bộ trên máy với database local

Trong `.env` ở gốc repository, đồng bộ mật khẩu và cổng PostgreSQL giữa các biến sau.

```dotenv
POSTGRES_USER=sporthub
POSTGRES_PASSWORD=MAT_KHAU_DB_LOCAL_CUA_BAN
POSTGRES_DB=sporthub
POSTGRES_PORT=5435
ConnectionStrings__Default=Host=localhost;Port=5435;Database=sporthub;Username=sporthub;Password=MAT_KHAU_DB_LOCAL_CUA_BAN
SharedDatabase__Enabled=false
JwtOptions__SecretKey=CHUOI_NGAU_NHIEN_RIENG_IT_NHAT_32_KY_TU
Frontend__BaseUrl=http://localhost:3000
Cors__AllowedOrigins__0=http://localhost:3000
AiService__BaseUrl=http://localhost:8001
```

Các giá trị viết hoa là chỗ để thay bằng cấu hình cá nhân. Nếu PostgreSQL đã có volume từ lần chạy trước, dùng tài khoản và mật khẩu đã khởi tạo volume đó. Đổi mật khẩu trong `.env` cần được đồng bộ với tài khoản PostgreSQL hiện có.

Khởi động Docker Desktop rồi chạy các dịch vụ nền. Để dùng các luồng Gemini, tạo `ai-service/.env` và điền `GEMINI_API_KEY` được cấp riêng. Compose đọc file này nếu có.

```powershell
docker compose up -d postgres ai-service
```

Nếu backend container đang chiếm cổng `5000`, dừng container đó trước khi chạy API bằng .NET.

```powershell
docker compose stop backend
dotnet run --project backend/SportHub.API/SportHub.API.csproj -c Release -- --environment=Development --SharedDatabase:Enabled=false --urls=http://localhost:5000 --DataProtection:KeysPath="$PWD/.sporthub-local/dataprotection-keys"
```

API Development với DB local tự chạy migration và có thể khởi tạo dữ liệu mẫu. Vì vậy, bước này dùng connection string của DB local ở trên.

Trong `frontend/.env.local`, đặt địa chỉ API rồi mở terminal thứ hai từ gốc repository để chạy frontend.

```dotenv
NEXT_PUBLIC_API_BASE_URL=http://localhost:5000
```

```powershell
cd frontend
npm ci
npm run dev
```

Mở `http://localhost:3000`. API health ở `http://localhost:5000/api/health`, Swagger ở `http://localhost:5000/swagger` trong môi trường Development. Để thử nghiệp vụ sau khi chạy, dùng tài khoản test của DB local hoặc làm theo phần khởi tạo dữ liệu trong [RUNBOOK](https://github.com/SWP391-SportHub-Group4/sportHub-docs/blob/main/operations/RUNBOOK.md).

## 5. Kết nối database AWS

Người vận hành cấp tên database, tài khoản DB riêng và xác nhận một trong hai cách kết nối dưới đây. Dữ liệu và giao dịch qua kết nối này tác động tới website thật. Dùng DB local hoặc CI riêng cho test tự động, migration và seed.

### 5.1. Endpoint public có TLS

Cách này áp dụng sau khi người vận hành đã bật profile `public`, chuẩn bị TLS và mở cổng tương ứng. Repository mặc định profile `private`. Mỗi thành viên nhận hostname, cổng và file CA công khai qua kênh được nhóm sử dụng để cấp cấu hình.

Lưu file CA được cấp, ví dụ tại `C:/Keys/sporthub-db-ca.crt`. Trong `.env` gốc, sửa hai giá trị sau, thay toàn bộ chỗ giữ chỗ bằng thông tin được cấp.

```dotenv
ConnectionStrings__Default=Host=HOST_DB_DUOC_CAP;Port=15432;Database=TEN_DB_WEBSITE;Username=USER_DB_RIENG;Password=MAT_KHAU_DB_RIENG;SSL Mode=VerifyFull;Root Certificate=C:/Keys/sporthub-db-ca.crt
SharedDatabase__Enabled=true
```

Hostname phải khớp chứng chỉ server. Tài liệu hiện ghi nhận `sporthub-vie.duckdns.org` làm ví dụ endpoint. Kiểm tra cổng theo hostname và cổng thực tế được cấp.

```powershell
Test-NetConnection HOST_DB_DUOC_CAP -Port 15432
```

`TcpTestSucceeded` xác nhận kết nối TCP. Đăng nhập bằng DB client và chạy câu lệnh ở mục 5.3 để kiểm tra tài khoản và đúng database.

### 5.2. Database private qua SSH tunnel

Nhận host SSH, user SSH, khóa cá nhân và tài khoản DB riêng từ người vận hành. Từ gốc repository, mở tunnel bằng script có sẵn.

```powershell
./scripts/Open-AwsDatabaseTunnel.ps1 -Server HOST_SSH_DUOC_CAP -SshUser USER_SSH_DUOC_CAP -IdentityFile 'C:\Keys\sporthub-team.pem'
```

Xác minh fingerprint SSH với người vận hành ở lần đầu kết nối. Script cần quyền xem thông tin container PostgreSQL trên host để tìm IP Docker hiện tại. Với user chỉ được port forwarding, dùng lệnh tunnel do người vận hành cung cấp theo tài liệu nguồn ở cuối trang.

Giữ terminal tunnel mở. Script chuyển tiếp `127.0.0.1:15432` đến PostgreSQL trong Docker trên AWS. Nếu container PostgreSQL được tạo lại, mở lại tunnel để lấy IP mới.

Trong `.env` gốc, đặt cấu hình tương ứng.

```dotenv
ConnectionStrings__Default=Host=127.0.0.1;Port=15432;Database=TEN_DB_WEBSITE;Username=USER_DB_RIENG;Password=MAT_KHAU_DB_RIENG
SharedDatabase__Enabled=true
```

### 5.3. DBeaver hoặc pgAdmin

| Trường | Public TLS | SSH tunnel đã mở bằng script |
| --- | --- | --- |
| Host | Hostname DB được cấp | `127.0.0.1` |
| Port | Cổng được cấp, ví dụ `15432` | `15432` hoặc LocalPort đã chọn |
| Database | Tên DB website được cấp | Tên DB website được cấp |
| User và Password | Tài khoản DB riêng | Tài khoản DB riêng |
| SSL | `verify-full`, chọn CA được cấp | Theo chính sách PostgreSQL được cấp cho kết nối tunnel |
| SSH trong công cụ | Theo cách kết nối TLS trực tiếp | Tắt vì script đã mở tunnel ngoài công cụ |

Sau khi kết nối, kiểm tra bằng truy vấn chỉ đọc.

```sql
SELECT current_database(), current_user;
```

## 6. Chạy API local với DB AWS

Hoàn tất mục 5, giữ JWT secret local, frontend origin và các cấu hình runtime trên máy. Chạy API trực tiếp bằng .NET với cờ dùng DB chung.

```powershell
dotnet run --project backend/SportHub.API/SportHub.API.csproj -c Release -- --environment=Development --SharedDatabase:Enabled=true --urls=http://localhost:5000 --DataProtection:KeysPath="$PWD/.sporthub-local/dataprotection-keys"
```

Log cần có `Shared database mode`, xác nhận migration, seed và các job nền trên API local đã tắt. API request vẫn ghi dữ liệu website và worker AWS vẫn có thể xử lý dữ liệu đó. Compose local ghi đè connection string sang `postgres` local, vì vậy cách kết nối này sử dụng lệnh `dotnet run` ở trên.

Frontend cấu hình `NEXT_PUBLIC_API_BASE_URL=http://localhost:5000` và chạy theo mục 4. Khi đăng nhập, dùng tài khoản ứng dụng trong database website. API local phát token theo JWT secret local của nó.

Email outbox dùng ASP.NET Data Protection. Luồng tạo email từ API local cần người vận hành thiết lập key ring và application discriminator tương thích worker AWS. Khi chưa có cấu hình này, thực hiện luồng gửi email trên website. AI, Cloudinary và VNPay cần cấu hình tích hợp tương ứng nếu chạy từ API local.

## 7. Frontend local gọi API AWS

Nếu chỉ sửa giao diện, người vận hành có thể cấp API HTTPS để frontend local gọi trực tiếp. Cập nhật `frontend/.env.local` theo endpoint đã xác nhận.

```dotenv
NEXT_PUBLIC_API_BASE_URL=https://HOST_API_DUOC_CAP
```

Khởi động lại `npm run dev` sau khi sửa file. API AWS cần cho phép origin frontend local, ví dụ `http://localhost:3000`, trong `Cors:AllowedOrigins`. Người vận hành cấu hình danh sách này trên server. Nếu đổi cổng frontend, cập nhật origin tương ứng. Các biến `NEXT_PUBLIC_*` đi vào mã gửi xuống trình duyệt, nên chỉ chứa cấu hình công khai như API URL.

## 8. Kiểm tra kết nối và xử lý lỗi

| Kiểm tra hoặc triệu chứng | Cách xác nhận và xử lý |
| --- | --- |
| Frontend mở được | Truy cập đúng URL và cổng đang chạy. Kiểm tra terminal Next.js nếu trang lỗi. |
| API phản hồi | Gọi `/api/health`, rồi `/api/classes` để kiểm tra đường đọc DB. |
| `Failed to fetch` hoặc lỗi CORS | Kiểm tra API URL trong Network, giao thức HTTPS và origin được API cho phép. Khởi động lại frontend sau khi sửa `.env.local`. |
| HTTP `401` | Đăng nhập lại trên frontend đang gọi đúng API. Token được cấp bởi API khác có thể dùng JWT secret khác. |
| HTTP `403` | Kiểm tra tài khoản có vai trò và quyền với nghiệp vụ đang mở. |
| DB timeout hoặc connection refused | Kiểm tra host, cổng, trạng thái tunnel và profile DB do người vận hành xác nhận. |
| Xác thực PostgreSQL thất bại | Kiểm tra tài khoản DB được cấp và connection string. Tài khoản website được dùng ở bước đăng nhập ứng dụng. |
| Lỗi CA hoặc hostname TLS | Chọn CA được cấp, dùng hostname khớp chứng chỉ và nhờ người vận hành kiểm tra hạn chứng chỉ. |
| Lỗi bảng/cột hoặc migration | Đối chiếu commit ứng dụng local với schema và release đang chạy. Người vận hành xử lý migration qua luồng triển khai. |
| API local không khởi động | Kiểm tra JWT secret, cổng `5000`, connection string và log .NET. |
| Email hoặc AI thất bại | Kiểm tra cấu hình tích hợp và key ring theo mục 6, rồi cung cấp mã request/log đã che bí mật để nhóm phụ trách kiểm tra. |

Khi kết thúc, dừng API và frontend bằng `Ctrl+C`, rồi đóng terminal tunnel sau cùng nếu đang dùng. Thành viên giữ cấu hình `.env`, khóa SSH và thông tin xác thực trong môi trường cá nhân.

## Nguồn đối chiếu

- [README ứng dụng và yêu cầu chạy local](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5/README.md)
- [Frontend environment](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5/frontend/.env.example) và [cấu hình backend local](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5/.env.example)
- [DB qua SSH tunnel](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5/deploy/aws/LOCAL-DATABASE.md) và [endpoint DB có TLS](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5/deploy/aws/PUBLIC-DATABASE.md)
- [Script mở tunnel](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5/scripts/Open-AwsDatabaseTunnel.ps1) và [CORS của API](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5/backend/SportHub.API/Extensions/CorsExtensions.cs)

Các bước kết nối có xác thực cần được kiểm tra trên tài khoản và môi trường thực tế được nhóm cấp. Trạng thái endpoint và quyền truy cập runtime chưa được xác minh trong lần biên soạn này.
