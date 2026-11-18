# Test Plan

### 3.1 Thông tin đợt kiểm thử

Đợt kiểm thử mới chưa được thực hiện. Các trường dưới đây phải điền khi chạy, không mặc định lấy ngày viết tài liệu làm ngày chạy test.

| Trường | Giá trị hiện tại |
| --- | --- |
| Run ID | Chưa cấp; đề xuất `RUN-YYYYMMDD-NN` |
| Commit, branch, trạng thái working tree | Ghi tại thời điểm chạy; SHA tham chiếu ở đầu tài liệu chưa phải SHA của kết quả lịch sử |
| Người thực hiện/người rà soát | Chưa phân công tên cụ thể |
| Thời gian bắt đầu/kết thúc | Chưa có |
| Môi trường và database | Chưa chọn cho đợt chạy mới |
| Provider mode | Ghi riêng từng dịch vụ: fake/mock/sandbox/live |

### 3.2 Môi trường và dữ liệu

- Backend: SDK/runtime .NET 10; năm test project Security, Scheduling, Payment, Administration và Training dùng xUnit.
- Database: PostgreSQL; CI chuẩn bị image `postgres:16-alpine`. Đọc factory của từng suite để xác định tạo database/container và vòng đời dữ liệu trước khi chạy.
- Frontend: Node `>=24 <25`, npm `>=11 <12` theo `frontend/package.json`; cài dependency bằng lockfile. Playwright dùng Desktop Chrome/Chromium và bản Next.js đã build.
- Cấu hình Playwright mặc định: frontend `http://127.0.0.1:3100`, API `http://127.0.0.1:5000`; có thể override bằng biến môi trường. Tránh dùng nhầm server đang chạy với dữ liệu khác.
- Python AI: dùng môi trường tách biệt và dependency của `ai-service`; xác định fake model hay provider thật trước khi diễn giải kết quả.
- Dùng database test độc lập, dữ liệu giả, không dùng tài khoản hoặc dữ liệu cá nhân thật. Không trỏ suite có seed/mutation vào database vận hành.
- Dữ liệu gồm năm role và Guest; tài khoản Active/Deactivated; lớp Draft/Published/InProgress/Completed/Cancelled; lớp còn chỗ/hết chỗ; phòng active/inactive và cho thuê/tắt thuê; ví đủ/thiếu điểm; invoice và attempt ở các trạng thái hợp lệ.
- Kiểm tra mốc giờ Việt Nam, sát thời điểm bắt đầu/kết thúc, cuối tháng/năm nhuận; snapshot giá và dữ liệu trước/sau thao tác.

### 3.3 Phân công kiểm thử theo phạm vi phụ trách

Nhóm thực hiện kiểm thử chủ yếu theo phạm vi chức năng được giao. Thành viên phụ trách phát triển các chức năng của vai trò nào sẽ trực tiếp kiểm tra các chức năng thuộc vai trò đó.

| Phạm vi phụ trách | Trách nhiệm kiểm thử |
| --- | --- |
| Member | Kiểm tra các chức năng dành cho hội viên và quyền truy cập tương ứng |
| Receptionist | Kiểm tra các nghiệp vụ tại quầy và quyền thao tác của lễ tân |
| Coach | Kiểm tra các chức năng huấn luyện, lớp được phân công và quyền truy cập học viên |
| Center Manager | Kiểm tra các chức năng quản lý và vận hành trung tâm |
| System Administrator | Kiểm tra quản trị tài khoản, phân quyền và nhật ký thao tác |

Khi phát hiện lỗi trong phần phụ trách, thành viên chỉnh sửa và kiểm tra lại. Với các luồng liên quan đến nhiều vai trò, các thành viên phụ trách cần phối hợp kiểm tra tính nhất quán giữa các bước. Yêu cầu phối hợp này không xác nhận nhóm đã hoàn thành kiểm thử tích hợp cho mọi luồng.

### 3.4 Trình tự thực hiện

Nhóm thực hiện kiểm thử tự động thông qua GitHub Actions của repository ứng dụng. Nhóm không tự đặt hoặc ghi Run ID riêng; pull request, commit và liên kết lần chạy GitHub Actions được dùng để xác định phiên bản và kết quả kiểm thử. GitHub tự cấp mã cho mỗi lần chạy workflow.

1. Thành viên hoàn thành thay đổi trong phạm vi phụ trách, commit và push code lên nhánh làm việc trên GitHub.
2. Tạo pull request vào `main` hoặc `develop`. Theo cấu hình `.github/workflows/ci.yml` hiện có, GitHub Actions tự kích hoạt workflow cho PR vào hai nhánh này; push trực tiếp lên hai nhánh cũng kích hoạt workflow.
3. GitHub Actions chuẩn bị môi trường và chạy các bước đã cấu hình, gồm kiểm tra deployment scripts, build/test backend, kiểm tra frontend và job E2E. Job E2E phụ thuộc kết quả backend/frontend, nên có thể không chạy nếu các job trước không đạt. Workflow chỉ kiểm tra những tình huống đã được cấu hình và viết test.
4. Thành viên xem tiến trình và kết quả trong phần **Checks** của PR hoặc tab **Actions**; khi cần, mở log và tải artifact kiểm thử được workflow lưu.
5. Nếu kiểm tra không đạt, thành viên phụ trách xem log, xác định nguyên nhân, chỉnh sửa rồi push commit mới lên cùng nhánh PR. GitHub Actions tự chạy lại với thay đổi mới; tiếp tục kiểm tra kết quả của commit mới nhất.
6. Khi tổng hợp Test Report, tham chiếu PR, commit và liên kết lần chạy Actions cùng log/artifact hiện có. Phân biệt test Pass/Fail với job bị lỗi môi trường, bị bỏ qua hoặc bị hủy; không coi workflow chưa hoàn tất là đã Pass.

Kiểm tra thủ công do thành viên phụ trách chức năng thực hiện như mục 3.3. GitHub Actions không tự chứng minh các thao tác thủ công hoặc tích hợp dịch vụ thật đã được nghiệm thu. Các mẫu Run ID ở những phần khác là đề xuất cho việc ghi nhận bổ sung; với kiểm thử qua CI của nhóm, có thể dùng liên kết lần chạy GitHub Actions thay cho mã tự đặt.

### 3.5 Điều kiện bắt đầu và kết thúc kiểm thử

**Điều kiện bắt đầu:** Thành viên push thay đổi lên GitHub và tạo hoặc cập nhật PR vào `main` hoặc `develop`. GitHub Actions được kích hoạt để chuẩn bị môi trường và chạy các bước kiểm tra theo cấu hình workflow.

**Điều kiện hoàn tất kiểm tra tự động cho PR:** Các job kiểm tra được yêu cầu trong workflow hoàn thành thành công trên commit mới nhất của PR. Nếu có bước thất bại, thành viên phụ trách kiểm tra nguyên nhân, chỉnh sửa và push lại để chạy kiểm tra mới. Các job bị bỏ qua, bị hủy hoặc không chạy được cần được xem xét, không tính là kiểm thử đã Pass.

[Mục lục tài liệu kiểm thử](README.md)
