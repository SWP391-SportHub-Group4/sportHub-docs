# ADR-0002: Next.js frontend và ASP.NET Core API

- Trạng thái: Đã áp dụng
- Ngày ghi nhận: 2026-11-17

## Bối cảnh

Sản phẩm có trang công khai và nhiều không gian làm việc theo vai trò. Giao diện cần điều hướng, form và tương tác nhiều; nghiệp vụ cần API kiểm quyền độc lập với trình duyệt và được kiểm thử tự động.

## Quyết định và lý do

Dùng **Next.js 16, React 19, TypeScript** cho ứng dụng web và **ASP.NET Core 10** cho HTTP API. Frontend tổ chức trang theo vai trò và gọi API; backend giữ kiểm tra nghiệp vụ, phân quyền và dữ liệu. Hợp đồng API được ghi riêng để frontend không suy diễn từ cấu trúc bảng. TypeScript hỗ trợ kiểm tra kiểu trên giao diện; .NET cho phép chia project nghiệp vụ và tích hợp EF Core, auth, background service trong cùng ứng dụng.

## Các lựa chọn đã cân nhắc

- **Ứng dụng server render duy nhất:** giảm số ứng dụng phải vận hành, nhưng khó giữ ranh giới rõ giữa giao diện tương tác và API nghiệp vụ đã dùng trong dự án.
- **SPA thuần không dùng Next.js:** đủ cho phần lớn màn hình nội bộ, nhưng cần tự xây thêm cấu trúc route và các khả năng phục vụ trang công khai.

## Hệ quả

Có hai ứng dụng và một hợp đồng HTTP cần đồng bộ. Mỗi thay đổi DTO/endpoint phải cập nhật [API contract](../../api/api-contract.md) và kiểm tra frontend. CORS, cấu hình URL API và phiên bản Node/.NET là một phần vận hành. Frontend không được coi là lớp bảo vệ quyền cuối cùng.

## Nguồn kiểm chứng

- [frontend/package.json](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/frontend/package.json), [frontend/src/app](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/tree/main/frontend/src/app).
- [global.json](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/global.json), [SportHub.API.csproj](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.API/SportHub.API.csproj), [docker-compose.yml](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/docker-compose.yml).
