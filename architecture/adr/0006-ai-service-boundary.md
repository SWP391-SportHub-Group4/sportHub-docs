# ADR-0006: Tách tác vụ AI sang FastAPI, giữ quyền ở backend

- Trạng thái: Đã áp dụng
- Ngày ghi nhận: 2026-11-17

## Bối cảnh

Trợ lý và gợi ý tập luyện gọi nhà cung cấp mô hình bên ngoài, có độ trễ và khả năng lỗi khác với API nghiệp vụ. Dữ liệu gửi cho AI phụ thuộc role và quan hệ Member/Coach; kết quả sinh ra không tự có thẩm quyền thay đổi giáo án hoặc dữ liệu thanh toán.

## Quyết định và lý do

Dùng **FastAPI** cho dịch vụ AI gọi **Gemini**. ASP.NET Core API là điểm vào có xác thực, tập hợp ngữ cảnh được phép và gọi AI service qua HTTP. Tách mã tích hợp mô hình để có thể đặt timeout, phân loại lỗi nhà cung cấp và thay đổi prompt/model mà không trộn với transaction nghiệp vụ. Kết quả AI là gợi ý/bản nháp; Coach hoặc người dùng có thẩm quyền xem xét trước khi lưu thao tác nghiệp vụ.

## Các lựa chọn đã cân nhắc

- **Gọi Gemini trực tiếp từ trình duyệt:** làm lộ thông tin tích hợp và khó kiểm soát dữ liệu ngữ cảnh theo quyền.
- **Đặt toàn bộ tích hợp AI trong API .NET:** giảm một service, nhưng trộn vòng đời và phụ thuộc thư viện AI với backend nghiệp vụ.

## Hệ quả

Cần vận hành thêm một service và xử lý lỗi mạng/timeout. AI service không được coi là nguồn xác thực hay nguồn dữ liệu nghiệp vụ; API phải giới hạn context trước khi gửi và kiểm quyền khi lưu kết quả. Tính năng nghiệp vụ cốt lõi vẫn cần xử lý được khi nhà cung cấp AI không sẵn sàng.

## Nguồn kiểm chứng

- [docker-compose.yml](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/docker-compose.yml), [ai-service/app/main.py](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/ai-service/app/main.py), [GeminiService](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/ai-service/app/services/gemini_service.py).
- [SportHub.AI](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/tree/main/backend/SportHub.AI), [AI Member/Coach verification](../../verification/AI-Member-Coach-Implementation-Verification.md).
