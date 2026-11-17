# UC-F05 — Gợi ý bài tập bằng AI

> Căn cứ: *SportHub SRS Part I* §2.5, §4.2 UC-74 (phạm vi mở rộng), đối chiếu [Source of Truth](../requirements/00-Source-of-Truth.md#ai-flow-56--cập-nhật-đối-chiếu-sau-các-bản-vá-02112026), [Functional & Nonfunctional Specification](../requirements/Functional-and-Nonfunctional-Spec.md#bổ-sung-yêu-cầu-ai-flow-56--02112026) và [bằng chứng AI Member/Coach](../verification/AI-Member-Coach-Implementation-Verification.md). Tài liệu này mô tả yêu cầu UC; trạng thái triển khai phải đối chiếu riêng.

## Mục tiêu và actor

Member nhận gợi ý tập cho chính mình từ mục tiêu/trình độ/ngữ cảnh phù hợp. Coach của lớp hợp lệ lấy bản nháp Session Plan hoặc Personal Plan cho học viên được phân công, chỉnh sửa và tự lưu nội dung chính thức theo [Flow 4](flow-04-training-attendance.md). AI service là hệ thống phụ trợ; không có quyền tự tạo plan, điểm danh hay thay đổi hồ sơ.

## Tiền điều kiện và dữ liệu vào

- Người gọi đã đăng nhập đúng role. Member chỉ dùng dữ liệu của mình. Với Personal Plan, Coach phải có quan hệ lớp đang hiệu lực với Member; với Session Plan, Coach phải được giao đúng lớp/buổi. Quyền phải kiểm lại ở backend tại thời điểm request và khi Save.
- Backend lấy mục tiêu tập luyện, môn, trình độ và lịch sử gần đây trong phạm vi được phép. Chỉ gửi trường cần thiết cho AI; không lấy dữ liệu học viên khác hay đưa khóa provider ra frontend.
- Training Goal qua hai lớp kiểm: độ dài/ký tự cơ bản và ngữ nghĩa `VALID`, `VAGUE`, `INVALID`. `VAGUE` nhưng có nghĩa được cụ thể hóa bằng môn/trình độ/ngữ cảnh; `INVALID` được giải thích và không gọi sinh gợi ý. Giới hạn độ dài hiện hành có thể loại một số câu ngắn tiếng Việt trước bước ngữ nghĩa và cần test riêng.

## Luồng chính UC-74

1. Member mở trang gợi ý tập và nhập/chọn mục tiêu; hoặc Coach mở form Session Plan/Personal Plan, chọn buổi hay Member hợp lệ rồi bấm yêu cầu gợi ý AI.
2. Backend xác thực role, quan hệ phân công/chủ sở hữu, dữ liệu bắt buộc và Training Goal. Với `VAGUE`, backend tạo mục tiêu đã chuẩn hóa theo ngữ cảnh; với `VALID`, tiếp tục trực tiếp.
3. Backend gọi AI service với context đã giới hạn. Dịch vụ trả các bài tập có thông số hợp lệ (ví dụ set/repetition) và giải thích. Backend kiểm kết quả, ghi AI log theo quy tắc hiện hành, trả về gợi ý có nhãn AI cho người dùng.
4. Member xem gợi ý như lời khuyên cá nhân, không biến thành giáo án Coach. Coach xem, sửa và chỉ khi chủ động Save thì hệ thống mới tạo/cập nhật Session Plan hoặc Personal Plan; API Save kiểm lại quyền, lớp, buổi và Member. Gợi ý AI chưa duyệt không hiển thị như plan chính thức.

## Nhánh thay thế và lỗi

| Mã | Điều kiện | Trạng thái cuối |
| --- | --- | --- |
| A1 | Thiếu hồ sơ/mục tiêu hoặc lỗi định dạng | Báo trường cần bổ sung; không gọi provider, không tạo plan. |
| A2 | Goal chung nhưng hợp lệ (`VAGUE`) | Tiếp tục với mục tiêu cụ thể hóa; không coi mặc định là input sai. |
| A3 | Goal vô nghĩa, ngoài môn, không an toàn hoặc cố prompt injection (`INVALID`) | Giải thích giới hạn; không gọi sinh gợi ý và không ghi plan. |
| A4 | Coach mất phân công, chọn học viên khác, Member xin dữ liệu người khác | Từ chối, không gửi context cho AI. |
| A5 | Provider timeout/quota/lỗi, JSON hoặc thông số bài tập không hợp lệ | Báo lỗi có kiểm soát; không dựng plan giả, không tự lưu. Coach có thể viết thủ công. |
| A6 | Coach rời form hoặc chỉnh bản nháp rồi không Save | Không tạo bản ghi chính thức; log AI không thay thế thao tác Save. |

## Hậu điều kiện và nghiệm thu

- Chỉ gợi ý thuộc scope dữ liệu người gọi; bản nháp phân biệt rõ với nội dung Coach đã lưu. AI log ghi nhận request/kết quả theo chính sách, không lộ dữ liệu nhạy cảm qua UI/log.
- Kiểm thử: Member tự hỏi; Coach Session/Personal Plan đúng và sai phân công; mục tiêu tiếng Việt có dấu/không dấu, tiếng Anh/hỗn hợp; `VALID`/`VAGUE`/`INVALID`; câu quá ngắn; prompt injection; provider lỗi; output set/reps bất hợp lý; Save sau khi quyền bị thu hồi.

## Truy vết

SRS §2.5/§4.2 UC-74; BR-59, BR-105; FR-AI-01/02 và NFR-AI-01. Các API cụ thể xem [API contract](../api/api-contract.md). Phạm vi kiểm chứng ngày 02/11/2026 tập trung vào Coach; không suy từ đó rằng mọi nhánh Member đã được nghiệm thu.
