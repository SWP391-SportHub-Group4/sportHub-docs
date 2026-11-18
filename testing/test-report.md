# Test Results / Test Report

### 7.1 Kết quả lần biên soạn 18/11/2026

| Hoạt động | Kết quả |
| --- | --- |
| Đọc yêu cầu, use cases, API, verification và cấu hình/test code | Đã thực hiện để xây dựng tài liệu |
| Chạy lại unit/integration/API/Playwright/AI tests | Chưa thực hiện trong lần biên soạn |
| Chứng nhận Pass trên phiên bản hiện tại | Chưa có cơ sở |
| Xác nhận độ bao phủ phần trăm | Chưa có báo cáo coverage được thẩm định |

### 7.2 Kết quả backend lịch sử

Nguồn: `backend/ci-local-test.log` trong workspace ứng dụng. Tên TRX được log dẫn tới có dấu thời gian `20261104`; ngày 04/11/2026 được suy ra từ tên file, chưa chuẩn hóa timezone hay xác minh thời điểm từ nội dung TRX. Chưa xác định commit của lần chạy. Số liệu sau được chép từ phần tổng kết log, không phải kết quả chạy mới.

| Test project | Passed | Failed | Skipped | Total | Thời lượng theo log |
| --- | ---: | ---: | ---: | ---: | --- |
| SportHub.Administration.Tests | 29 | 0 | 0 | 29 | 4 giây |
| SportHub.Training.Tests | 12 | 0 | 0 | 12 | 10 giây |
| SportHub.Security.Tests | 308 | 0 | 0 | 308 | 2 phút 7 giây |
| SportHub.Payment.Tests | 382 | 0 | 1 | 383 | 3 phút 57 giây |
| SportHub.Scheduling.Tests | 341 | 0 | 0 | 341 | 4 phút 51 giây |
| **Tổng** | **1.072** | **0** | **1** | **1.073** | Không cộng thời lượng để suy ra elapsed time |

Lý do và tên test bị skipped chưa được xác minh từ log tổng kết. Cần đọc TRX tương ứng trước khi kết luận phạm vi bỏ qua. Không diễn giải 1.072 test Pass thành bao phủ 1.072 yêu cầu hoặc toàn bộ test hiện có.

### 7.3 Kết quả frontend lịch sử

Nguồn: `frontend/ci-browser-test.log`; log ghi `Running 42 tests using 1 worker` và `42 passed (2.0m)`. Các dòng đã xem gồm loading, public/auth và reports dashboard; không dùng kết quả này làm tổng kết toàn bộ thư mục `frontend/tests/` hiện tại. Chưa xác định thời điểm, commit, danh sách dependency và mode API đầy đủ của lần chạy.

| Passed | Failed được báo ở tổng kết | Phạm vi kết luận |
| ---: | --- | --- |
| 42 | Không thấy failure trong tổng kết đã đọc | Chỉ 42 trường hợp được log ghi nhận; bằng chứng lịch sử |

[Mục lục tài liệu kiểm thử](README.md)
