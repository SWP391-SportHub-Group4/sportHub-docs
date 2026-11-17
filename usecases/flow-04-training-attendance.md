# UC-F04 — Giảng dạy, điểm danh và kết quả tập luyện

> Căn cứ: *SportHub SRS Part I* §2.4, §4.2 UC-66–73. Đây là phạm vi mở rộng của SRS. [Source of Truth](../requirements/00-Source-of-Truth.md) đã xác nhận **không có PT 1:1**; Flow 4 áp dụng cho Coach của lớp cầu lông/bóng rổ được phân công. Quyền và contract xem [API contract](../api/api-contract.md), [thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md).

## Mục tiêu và actor

Coach xem lịch/lớp được giao, điểm danh, tạo giáo án, ghi kết quả/nhận xét, giao bài và gửi thông báo; Member xem lịch sử thuộc quyền của mình; Receptionist sửa điểm danh có lý do khi Member kiến nghị. CenterManager giải quyết báo vắng/đổi lịch qua [Flow 2](flow-02-class-booking-court-schedule.md). Gợi ý AI chỉ là đầu vào tùy chọn của [Flow 5](flow-05-ai-workout-recommendation.md).

| UC SRS | Tương tác | Kết quả |
| --- | --- | --- |
| UC-66 | Xem lịch dạy, lớp và học viên | Chỉ dữ liệu trong phạm vi phân công |
| UC-67–68 | Điểm danh, sửa điểm danh | Một trạng thái Present/Absent mỗi học viên/buổi, sửa có audit |
| UC-69–71 | Giáo án, kết quả, homework, thông báo | Nội dung gắn đúng lớp/buổi/Member và do Coach chịu trách nhiệm |
| UC-72 | Member xem hồ sơ tập | Chỉ đọc nội dung mình có quyền |
| UC-73 | Hồ sơ Coach | Công khai chuyên môn/thành tích được phép; Coach cập nhật chứng chỉ của mình |

## Tiền điều kiện và quyền dữ liệu

- Coach chủ nhiệm có quyền hồ sơ khi lớp `Published`/`InProgress` và Member còn Enrollment Confirmed chưa kết thúc. Coach dạy thay chỉ có quyền đối với buổi được giao, roster hiện hành và bản ghi của buổi đó; không được đọc toàn bộ hồ sơ sức khỏe/lịch sử khóa.
- Điểm danh gắn `Enrollment` + `ClassSession`, không gắn gói Gym hay booking từng buổi. Một Member có tối đa một bản ghi trạng thái cho mỗi buổi. Member vào lớp giữa chừng không bị đánh vắng các buổi đã qua.
- Member còn Enrollment Confirmed, `EndedAt` rỗng được đọc lịch sử buổi, giáo án, homework và kết quả của mình cả khi lớp Completed. Khi ghi danh kết thúc hoặc lớp Draft/Cancelled, quyền đọc nội dung bị thu hồi theo quy tắc hiện hành.
- Coach không tự sửa lịch khi báo vắng; chỉ tạo báo vắng có lý do và rút khi Pending. Quản lý phân công/dời/học bù thuộc Flow 2.

## Luồng chính

### F04-A — Lịch dạy và điểm danh (UC-66–68)

1. Coach mở lịch dạy và lớp/buổi được phân công. Backend lấy Coach từ JWT và đối chiếu `classId`/`sessionId` với phân công hiện hành; danh sách học viên chỉ gồm người có ghi danh hợp lệ cho buổi.
2. Sau buổi học, Coach ghi Present/Absent cho từng học viên; backend kiểm quyền buổi, trạng thái buổi, enrollment và bản ghi hiện có rồi lưu. Job cuối ngày nhắc Coach nếu buổi đã kết thúc mà còn thiếu điểm danh.
3. Member xem điểm danh và kiến nghị khi sai. Receptionist tìm đúng Member/buổi, sửa bản ghi với lý do bắt buộc 3–500 ký tự; hệ thống giữ audit trước/sau. Member thấy kết quả đã cập nhật.

### F04-B — Kế hoạch, kết quả và nội dung lớp (UC-69–71)

1. Coach chọn lớp, buổi và phạm vi cả lớp hoặc một Member hợp lệ; viết giáo án/kế hoạch thủ công. Có thể lấy bản nháp AI từ Flow 5, nhưng thao tác Save của Coach mới tạo nội dung chính thức.
2. Sau buổi, Coach ghi kết quả và nhận xét tiến bộ cho học viên Confirmed được đánh Present ở đúng buổi. Hệ thống ràng buộc kết quả với lớp, buổi và học viên; không cho ghi vào học viên ngoài roster.
3. Coach giao homework sau buổi cho cả lớp hoặc một học viên, gửi thông báo trong phạm vi được giao. Hệ thống tạo thông báo và lưu lịch sử; người nhận chỉ thấy nội dung mình có quyền.

### F04-C — Member xem và hồ sơ Coach (UC-72–73)

1. Member mở Training/Schedule, xem buổi, điểm danh, giáo án, homework, kết quả và nhận xét của chính mình theo quyền hiện hành; nội dung là read-only đối với Member.
2. Guest/Member xem hồ sơ Coach công khai gồm chuyên môn/thành tích được phép. Coach cập nhật chứng chỉ/thành tích của chính mình theo quy trình kiểm duyệt hiện hành; không tự sửa hồ sơ Coach khác.

## Nhánh thay thế và lỗi

| Mã | Điều kiện | Trạng thái cuối |
| --- | --- | --- |
| A1 | Coach không còn được giao, chỉ dạy thay buổi khác, lớp/Member ngoài phạm vi | Từ chối đọc/ghi; không lộ roster hay hồ sơ ngoài buổi. |
| A2 | Member đã rút, buổi sai hoặc chưa hợp lệ để điểm danh/kết quả | Không ghi bản ghi mồ côi; trả lỗi nghiệp vụ. |
| A3 | Ghi điểm danh lặp hoặc hai người sửa cùng lúc | Cập nhật bản ghi theo quy tắc hiện hành, không tạo hai điểm danh cho cùng enrollment/session; audit sửa đổi. |
| A4 | Ghi result cho Member Absent hoặc chưa Confirmed | Từ chối; không tạo kết quả. |
| A5 | Receptionist sửa điểm danh không có lý do | Từ chối; bản ghi cũ giữ nguyên. |
| A6 | Member đọc dữ liệu học viên khác hoặc ghi danh đã kết thúc | Từ chối; UI và API cùng phản ánh quyền đã thu hồi. |
| A7 | AI lỗi hoặc Coach bỏ qua bản nháp | Coach vẫn có thể viết thủ công; AI không tự lưu kế hoạch. |

## Hậu điều kiện và nghiệm thu

- Mỗi attendance/result/plan/homework có chủ thể, lớp/buổi và người ghi phù hợp; chỉnh sửa có audit. Member xem lịch sử Completed theo quyền, Coach mất quyền hồ sơ khi lớp hoàn thành hoặc quan hệ kết thúc.
- Kiểm thử: Coach chủ nhiệm và dạy thay, mất phân công khi trang đang mở, học viên vào giữa khóa, duplicate attendance, sửa có/không có lý do, result khi Absent, quyền lịch sử Completed, homework cá nhân/lớp, AI nháp chưa Save, job nhắc thiếu điểm danh.

## Truy vết

SRS §2.4/§4.2 UC-66–73; BR-43, BR-45, BR-55–59, BR-116. Xem [Source of Truth phần Flow 4](../requirements/00-Source-of-Truth.md), [Functional & Nonfunctional Specification](../requirements/Functional-and-Nonfunctional-Spec.md) và [API contract](../api/api-contract.md). Các mô tả PT 1:1 cũ không còn là yêu cầu hiện hành.

### Kiểm kê UC trong SRS

| Mã | Tên UC trong SRS | Actor chính |
| --- | --- | --- |
| UC-66 | View teaching schedule | Coach |
| UC-67 | Take attendance | Coach |
| UC-68 | Correct attendance | Member kiến nghị, Receptionist sửa |
| UC-69 | Create workout plan | Coach |
| UC-70 | Record workout result | Coach |
| UC-71 | Publish class content | Coach |
| UC-72 | View training records | Member |
| UC-73 | View Coach profile | Guest/Member/Coach |
