# SportHub — AI Member/Coach: Triển khai và xác minh

Cập nhật: 2026-11-02. Phạm vi: Gợi ý AI cho Coach trong Flow 5 và điều hướng chatbot trong Flow 6 chỉ cho Member/Coach.

## Bằng chứng và giới hạn

- Dựa trên ZIP repository `(9)`, các ZIP bản vá AI và file xung đột `AiChatService(3).cs` mới nhất được cung cấp.
- ZIP repository là **snapshot**, không phải HEAD GitHub sau rebase. Không ghi đè tài liệu mới hơn của đồng đội khi chưa xem diff.
- `AiChatService.cs` đã merge hiện tại dùng `ISystemPromptReader` cho prompt đã xuất bản; trợ lý không nên quay lại cơ chế chọn prompt tĩnh.
- Đầu vào này không có lần chạy CI/E2E thành công trên HEAD cuối sau rebase; nội dung sau là **tiêu chí nghiệm thu**, không phải kết quả Pass đã được chứng nhận.

## Phạm vi vai trò

Member: chat và điều hướng về UI Member, dịch vụ, hóa đơn, ví, lịch, bài tập AI và tài khoản của mình. Coach: lớp/lịch, học viên, điểm danh, giáo án, tài khoản và ngữ cảnh huấn luyện. Cả hai role đều không được thực hiện hoặc nhận hướng dẫn giả mạo vai trò cho workflow chỉ dành cho nhân viên. Manager, Receptionist, Admin không phải role có UI chat AI.

## Chính sách Training Goal

- Input cơ bản: trim, chuẩn hóa Unicode, kiểm tra không rỗng, giới hạn độ dài; ghi chính xác ngưỡng độ dài được cấu hình. Mức tối thiểu cứng 12 ký tự loại một số input tiếng Việt ngắn có nghĩa và nên được sửa hoặc chấp nhận rõ ràng thành quy tắc sản phẩm.
- Kết luận ngữ nghĩa: VALID, VAGUE, INVALID. VAGUE có thể tiếp tục với mục tiêu tập được chuẩn hóa theo môn/trình độ/ngữ cảnh. INVALID được giải thích, không gọi generation. Provider lỗi/JSON sai phải chặn xử lý theo hướng an toàn, không tự tạo plan không có căn cứ.
- Trường hợp: ``tập kỹ thuật`` (VAGUE), ``cải thiện thể lực`` (VAGUE), ``tap ky thuat`` (VAGUE), ``Improve beginner footwork`` (VALID), ``dwadawdawdawdawdawd`` (INVALID), ``học SQL`` (INVALID), cam kết tăng chiều cao phi thực tế (INVALID hoặc yêu cầu sửa), đau/rủi ro y tế (làm rõ hoặc giới hạn an toàn), trộn ngôn ngữ, dấu Unicode tổ hợp, prompt injection và văn bản dài.
- Output: các item có cấu trúc phải có sets/reps hợp lệ và không chứa khẳng định phi lý; Coach chủ động rà soát và lưu.

## Yêu cầu kiến thức điều hướng

1. Liệt kê trang, tab, nút, mẫu route và nhãn hiển thị Member/Coach từ frontend.
2. Truy vết mỗi điều khiển UI có thể thao tác tới handler và API khi phù hợp; không đồng nhất endpoint API với nút.
3. Ghi role sở hữu, ID tính năng, nhãn nút, tiền điều kiện, trình tự bước, route backend và file nguồn.
4. Với workflow đã xác minh, trả lời tiếng Việt hoặc tiếng Anh theo câu hỏi; giữ nhãn nút UI bằng tiếng Anh.
5. Với điều hướng chưa khớp, thừa nhận chưa có hướng dẫn đã xác minh, không tự tạo các lần nhấp cụ thể; bộ chặn ngoài chủ đề không được từ chối câu hỏi điều hướng thông thường.

## Kiểm tra CI và merge

```powershell
git status
git diff --check
dotnet build .\backend\SportHub.sln --configuration Release
cd frontend
pnpm exec tsc --noEmit
pnpm build
```

Trong rebase, bảo đảm `AiChatService` chỉ có một lần trích xuất role và một định nghĩa của `isCoach`, `systemPrompt`, `promptVersion`. Giữ bộ đọc prompt đã xuất bản và fingerprint phạm vi truy cập. Hoàn thành rebase trước khi push.

## Ma trận nghiệm thu

| Tình huống | Kết quả mong đợi |
|---|---|
| Coach hỏi bằng tiếng Việt nút Personal Plan ở đâu | Đường dẫn Coach với nhãn UI đã xác minh; không từ chối vì ngoài chủ đề |
| Member hỏi hóa đơn ở đâu | Route và các bước đã xác minh chỉ dành cho Member |
| Member yêu cầu thao tác chỉ dành cho Coach | Không hướng dẫn Member dùng trang chỉ dành cho Coach |
| Coach hỏi quản lý tài khoản | Route `/account` dùng chung, không tự tạo bước menu |
| Goal rộng nhưng liên quan | VAGUE → gợi ý, không trả 400 chỉ vì quá rộng |
| Goal vô nghĩa | INVALID → giải thích, không gợi ý |
| Goal không liên quan môn đã chọn | Từ chối hoặc làm rõ; không đưa plan sai môn |
| Provider timeout hoặc kết luận sai định dạng | Không generation; phân biệt lỗi provider với input không hợp lệ của người dùng |
| Phiên bản prompt thay đổi | Xác thực lại việc tiếp tục tương tác trước |
| Người dùng hỏi ngoài phạm vi trung tâm | Phản hồi lịch sự trong phạm vi, tránh phân loại nhầm câu hỏi UI là ngoài chủ đề |

## Chính sách đồng bộ tài liệu

Các phụ lục bổ sung sửa đổi văn bản lịch sử mà không xóa dấu vết audit. Trước khi merge tài liệu, so sánh với `origin/feat/hao-ai-flow5-flow6` hoặc HEAD đích hiện tại; không ghi đè mù quáng tài liệu mới hơn của thành viên nhóm. Cập nhật `api-contract.md` từ OpenAPI đang chạy khi ngữ nghĩa và schema đã chốt. Khẳng định 100% coverage trước đây không có cơ sở.
