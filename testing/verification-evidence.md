# Verification Evidence

### 8.1 Danh mục nguồn hiện có

| Evidence ID | Nguồn | Chứng minh được | Giới hạn |
| --- | --- | --- | --- |
| EV-01 | `backend/ci-local-test.log` | Tổng kết năm test project lịch sử trong [Test Report, mục 7.2](test-report.md#72-kết-quả-backend-lịch-sử) | Chưa gắn commit; chưa kiểm tra nội dung các TRX được dẫn |
| EV-02 | `frontend/ci-browser-test.log` | 42 test được log báo Pass | Chưa có metadata đủ để tái lập; không đại diện toàn bộ suite hiện tại |
| EV-03 | `.github/workflows/ci.yml` | CI có bước build/test, chuẩn bị PostgreSQL, upload TRX và job E2E | Cấu hình workflow không chứng minh job đã chạy thành công |
| EV-04 | `frontend/playwright.config.ts` | Cấu hình trace/screenshot/video và web server | Artifact chỉ tồn tại khi thực sự chạy và điều kiện lưu được kích hoạt |
| EV-05 | Test code được dẫn trong [Integration Testing](integration-testing.md) / [API Testing](api-testing.md) | Có assertion và tình huống đã được triển khai | Sự tồn tại của file không chứng minh Pass |
| EV-06 | Tài liệu AI Verification | Tiêu chí nghiệm thu, scope và giới hạn AI | Không phải report runtime của phiên bản hiện tại |

EV-01 đến EV-05 đang nằm trong repository ứng dụng tại workspace, chưa được đóng gói thành bộ evidence trong repository tài liệu. Các file log local có thể không được Git theo dõi; khi bàn giao phải đính kèm bản đã rà soát hoặc artifact CI có thể truy cập. Không coi đường dẫn local là bằng chứng đã được chia sẻ cho cả nhóm.

[Mục lục tài liệu kiểm thử](README.md)
