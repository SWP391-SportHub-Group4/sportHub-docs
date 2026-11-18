# Hợp đồng API refactor đa môn (backend)

## Phòng độc lập và quyền cho thuê — 26/10/2026

Form tạo phòng nhập tên và sức chứa; Allow rental mặc định tắt, Active luôn bật. Bật thuê sẽ hiện chọn Sport (bắt buộc, môn active ngoài Gym), giá và lịch tuần. `POST /api/rooms` nhận `allowRental=true` và `rentalSetup: {sportId, pricePerHour, hours:[{dayOfWeek,openTimeLocal,closeTimeLocal}]}`. Môn được lưu vào giá thuê riêng của phòng và bật dịch vụ CourtRental cho môn đó trong cùng transaction. Phòng vật lý vẫn dùng chung cho lớp, không gắn Sport. Thiếu/môn không hợp lệ trả `400 invalid_rental_sport`, không tạo phòng dở dang. Giá thuê hiện có sửa được Sport, giữ nguyên phòng và khung giờ; hóa đơn/lượt thuê cũ giữ snapshot. Không cần migration.

Contract hiện hành nằm tại [Independent Rooms Workflow](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/0fb80ae20787baeb926ba175b48fe4060d16ab73/docs/Independent-Rooms-Workflow.md#api-backend). `POST /api/rooms` chỉ cần tên và sức chứa; response bổ sung `allowRental`. `PUT /api/rooms/{roomId}/rental-access` nhận `{allowRental}` và chỉ CenterManager được gọi. Phòng lớp không bị lọc theo Sport; quyền thuê được kiểm tra riêng trong calendar, quote và transaction chiếm chỗ. Áp dụng migration `20261026100000_IndependentRooms` trước khi chạy backend mới.

## Landing page content

- Media hỗ trợ sửa trực tiếp tiêu đề và nội dung dạng văn bản thuần bằng cách nhấp đúp (Enter/F2 cũng mở chế độ sửa). Enter kết thúc, Shift+Enter chèn ngắt dòng có chủ ý, Escape khôi phục giá trị trước đó; nội dung sửa trống sẽ được hoàn tác. Tên môn trong danh mục và thông tin hồ sơ Coach được liên kết vẫn chỉ đọc. Văn bản nháp được áp dụng trong Preview, Save changes, Cancel và cơ chế bảo vệ xung đột phiên bản.
- `brand.logo.image` là trường thương hiệu duy nhất được hỗ trợ trong `values`. **Change logo** của Media tải JPG/PNG/WebP bằng API ảnh hiện có với `preset=logo`; preset này giữ tỷ lệ ảnh gốc và pixel trong suốt mà không cắt ảnh/chuyển sang JPEG. Các kiểm tra loại/chữ ký/kích thước, giới hạn lưu trữ và audit hiện có được giữ nguyên. Header trang chủ công khai và bản xem trước nháp lần lượt hiển thị logo đã xuất bản/logo nháp; navbar của Manager không thay đổi. Restore original xóa logo tùy chỉnh trong bản nháp.

- `GET /api/landing-page`: công khai, no-store; trả `{ revision, values, updatedAt }`. Khi chưa có bản ghi xuất bản, trả revision là GUID rỗng, các giá trị ghi đè rỗng và timestamp null; trang chủ giữ nội dung mặc định hiện có.
- `PUT /api/landing-page`: chỉ CenterManager. Gửi revision mà trình biên tập đã đọc và toàn bộ dictionary `values`. Các tiền tố mục được hỗ trợ: `hero`, `bookmarks`, `gallery`, `arena`, `booking`, `faq`. Nội dung là văn bản thuần, 1–2.000 ký tự mỗi trường, tối đa 300 trường và request 256 KB. Trường ảnh chấp nhận URL HTTPS hoặc tài nguyên local `/sporthub/`. API này không chỉnh sửa HTML, route, số lượng tài khoản hoặc giá vận hành.
- Lưu sẽ xuất bản ngay. Khóa trong phạm vi transaction tuần tự hóa lần tạo đầu và các lần cập nhật; revision cũ trả `409 landing_content_changed` mà không ghi đè nội dung đã xuất bản. `UPDATE_LANDING_PAGE` được ghi audit trong cùng transaction.
- Giao diện Manager: **Support & Communication → Media**, tại `/manager/media` (URL cũ `/manager/landing-page` chuyển hướng tới đây). Văn bản và ảnh được nhóm theo thẻ, ảnh thư viện, khu thể thao hoặc câu hỏi FAQ. Xem trước mục bao gồm các sửa đổi chưa lưu; **Restore section** chỉ thay đổi bản nháp cho tới khi chọn **Save & publish**. Dữ liệu Coach/cơ sở vật chất hiện có vẫn cung cấp giá trị mặc định cho bookmark khi có; các giá trị nội dung ghi đè được ưu tiên cho trường hiển thị tương ứng.
- `POST /api/manager/media/images`: chỉ CenterManager, có rate limit, các trường multipart `file` và `preset` (`landscape`, mặc định, hoặc `booking`); JPG/PNG/WebP tối đa 10 MB, được kiểm tra bằng content type và chữ ký file. Media mở hộp thoại cắt ảnh trước khi tải lên, dựng vùng cắt trực tiếp từ pixel ảnh gốc mà không phóng lớn (chất lượng JPEG 95%). Dùng `CLOUDINARY_URL` ở backend và trả `{ url, publicId, width, height }`. Ảnh được lưu trong `sporthub/media/<actor>/<unique-id>`, giới hạn 2.560 px và giữ tỷ lệ; thông tin xác thực không bao giờ tới trình duyệt. API kiểm tra kích thước đã lưu theo preset được chọn; tỷ lệ sai trả `400 media_aspect_ratio_invalid` và thử xóa ảnh tải lên không dùng. Tải lên thành công được ghi audit là `UPLOAD_MEDIA_IMAGE`. Tải lên thất bại giữ ảnh trước đó. Nếu lưu audit thất bại, hệ thống thử xóa tài nguyên mới.
- Tải lên chỉ cập nhật bản nháp local của trình biên tập; **Save & publish** lưu URL ảnh vào tài liệu trang chủ. Các ảnh đã xuất bản hoặc chưa dùng vẫn nằm trong Cloudinary Media Library để tái sử dụng; thay thế hoặc khôi phục một trường không xóa tài nguyên dùng chung. Trình biên tập chặn xuất bản hoặc chuyển mục khi đang tải lên.
- Media tải `/api/coaches` và `/api/public/overview` trước khi cho phép chỉnh sửa. Media dùng chung bộ phân giải bookmark từ dữ liệu thực tế của trang chủ, sau đó áp dụng các giá trị ghi đè đã xuất bản. Preview dùng cùng snapshot Coach/cơ sở vật chất đã tải và bản nháp hiện tại. Nếu một trong hai nguồn thực tế lỗi, trình biên tập cung cấp Retry thay vì cho sửa dựa trên giá trị mặc định tĩnh mâu thuẫn. Khôi phục một mục sẽ xóa các giá trị ghi đè của mục đó và trở về cùng giá trị mặc định từ dữ liệu thực tế mà trang chủ dùng.
- Coach nổi bật được lưu thành ba UUID khác nhau, có thứ tự, trong `values["bookmarks.coachIds"]`, phân cách bằng dấu phẩy. `PUT /api/landing-page` yêu cầu đúng ba tài khoản Coach đang Active có hồ sơ và kiểm tra lại điều kiện khi xuất bản (`400 featured_coaches_required` / `409 featured_coach_unavailable`). Media ưu tiên gợi ý Coach có avatar, hỗ trợ tìm theo tên/môn và chọn/sắp xếp thứ tự rõ ràng. Khi chưa có lựa chọn đã xuất bản, Media chọn sẵn tối đa ba gợi ý để rà soát; xuất bản vẫn yêu cầu đủ ba.
- Tên, avatar và môn trên thẻ Coach luôn lấy từ hồ sơ Coach hiện tại. Các giá trị ghi đè cũ `bookmarks.0.items.*` bị bỏ qua trên trang chủ và được xóa khi lưu. Chỉ tiêu đề/mô tả mục Coaches còn chỉnh sửa được. Nếu một Coach nổi bật sau đó không còn khả dụng, trang chủ bỏ Coach đó thay vì thay bằng người khác trong cùng thẻ; Media đánh dấu lựa chọn cần thay thế. `/api/coaches` trả thêm `sportNames` và loại tài khoản không có hồ sơ. Không cần migration schema cho việc chọn Coach.
- Khung chân dung Coach dùng `bookmarks.coachFrame.<coach UUID without hyphens>.{fit,x,y,zoom}`. Fit là `contain` (mặc định, toàn bộ ảnh) hoặc `cover`; x/y từ 0–100 và zoom từ 1–2. Tải avatar, trang chủ và Media dùng cùng khung vuông 1:1. Cấu hình khung được lưu cùng tài liệu trang chủ, không bao giờ ghi vào hồ sơ Coach. Media hiển thị cảnh báo cho ảnh raster nhỏ hơn 800 px; muốn khôi phục chi tiết đã mất trong ảnh bị giảm kích thước trước đó phải tải lại ảnh gốc.
- Tỷ lệ ảnh được dùng chung trong frontend `components/homepage/image-specs.ts`: Coach/avatar 1:1 (tối đa 1536×1536), ảnh thẻ/banner/arena 3:2 (tối đa 1800×1200), ảnh đặt sân 4:5 (tối đa 1200×1500). Vùng cắt khi tải lên, thumbnail Media, Preview và khung trang chủ dùng các preset này trên desktop và mobile. Ảnh hiện có được giữ nguyên và hiển thị trong khung cố định; tải lại cho phép Manager chọn vùng cắt mới.
- Ảnh thay thế khi ảnh người dùng không có hoặc lỗi dùng chung đồ họa vector local `/sporthub/avatar-default.svg` ở menu tài khoản, hồ sơ, danh sách và chân dung trang chủ; ảnh null/lỗi hiển thị đồ họa này mà không tạo URL avatar tải lên giả. Avatar mới dựng vùng cắt trực tiếp từ pixel ảnh gốc với kích thước tối đa 1.536 px (không phóng lớn), thay vì lấy mẫu lại canvas 240 px của trình biên tập. Cloudinary chỉ áp dụng giới hạn kích thước 1.536 px. Không có migration dữ liệu thay đổi ảnh hiện có.
- Migration `20261020162921_AddLandingPageContent` chỉ thêm bảng singleton `landing_page_content`. Tải lên Cloudinary không cần migration schema bổ sung và không thay đổi video hero.

## Manager workspace — 20 October 2026

Phần này xác định phạm vi và điều hướng workspace của Manager. Các quy tắc này thay thế những mô tả cũ về hủy, điều chỉnh ví và liên kết khóa tiếp theo.

- `GET /api/manager/people?role&keyword&page&pageSize`: chỉ CenterManager; giới hạn Member/Coach trước khi lọc và phân trang, giữ metadata kích hoạt và loại nhân viên kể cả khi truyền role nhân viên. UI dùng một danh bạ People; Schedule và Audit được truy cập qua dropdown hồ sơ.
- `GET /api/manager/requests/summary`: chỉ CenterManager; số lượng đang chờ `{coach,refunds,support,avatar}`. Các loại yêu cầu hiện có giữ API thay đổi dữ liệu riêng.
- `POST /api/avatars/reports`: tài khoản đã xác thực; `{targetUserId,avatarUrl,reason}` cho ảnh hiện tại của một Member/Coach khác. Báo cáo cùng người gửi/ảnh và còn mở được loại trùng. Audit và thông báo Manager được commit cùng báo cáo.
- `GET /api/manager/avatar-reports?status&search&page&pageSize`: hộp thư kiểm duyệt phân trang chỉ dành cho Manager, có danh tính/vai trò đối tượng, ảnh bị báo cáo/ảnh hiện tại và lý do.
- `POST /api/manager/avatar-reports/{reportId}/dismiss`: chỉ Manager, có tính idempotent sau khi xử lý.
- `DELETE /api/manager/users/{userId}/avatar`: chỉ Manager, xóa ảnh hiện tại của Member/Coach; xử lý các báo cáo còn mở và giữ lịch sử. Luồng xóa cũ chỉ dành cho Member vẫn tương thích.
- `POST /api/class-sessions/{sessionId}/incident-makeup`: chỉ Manager, `{fenceId,reason,makeup}`; yêu cầu Incident fence đang được giữ, khớp, chưa hết hạn và buổi thay thế hợp lệ trong tương lai. Cho phép lớp đã có học viên ghi danh. `/cancel` thông thường vẫn từ chối ghi danh Confirmed còn hiệu lực; `/incident` cho sự cố trong quá khứ cho phép học bù toàn bộ buổi trong cửa sổ Incident đã được xác thực.
  Với buổi tương lai bị ảnh hưởng, buổi thay thế có thể ở trước hoặc sau sự cố, kể cả trước buổi cuối khóa. Toàn bộ thời lượng phải nằm ngoài cửa sổ sự cố (được phép kết thúc đúng lúc sự cố bắt đầu hoặc bắt đầu đúng lúc sự cố kết thúc). Thời gian trong quá khứ, xung đột tài nguyên/Member, vi phạm sức chứa và giờ mở cửa vẫn bị từ chối. Buổi bù thông thường không thuộc sự cố vẫn phải sau buổi còn hiệu lực cuối cùng của khóa.
- Tạo/cập nhật phòng nhận trực tiếp `sportId`; thiết lập cho thuê không còn yêu cầu người dùng nhập `roomTypeId`. Response phòng thêm `sportId` có thể null. Các khóa loại/giá cũ được giữ nội bộ. Phòng đã có lịch/giá không thể âm thầm đổi môn.
- `GET /api/member-packages` thêm `packageId` tùy chọn; `GET /api/manager/classes` thêm `coachId`, bao gồm phân công chủ nhiệm và các buổi không bị hủy.
- Tạo điều chỉnh ví thủ công qua HTTP và duyệt/từ chối cộng điểm kiểu cũ trả 403. Các lệnh đọc ví/ledger, thanh toán và hoàn trả vẫn hoạt động.

## Tạo Sport kèm Room Type tùy chọn — 19 October 2026

`POST /api/manager/sports` bổ sung `createRoomType` (mặc định false) và `roomTypeName` (tên do Manager nhập, tối đa 100 ký tự). Khi chọn true, tên sau trim phải có nội dung và không trùng Room Type hiện có (không phân biệt hoa/thường); lỗi `invalid_room_type_name` hoặc `room_type_name_taken` không tạo Sport. Sport, Room Type Active, liên kết SportRoomType và audit được lưu cùng transaction. Khi false/bỏ trống, tên Room Type được bỏ qua và không tạo/sửa Room Type hay liên kết. `PUT` không hỗ trợ tạo Room Type (`room_type_creation_only` nếu bật flag). Response Sport giữ nguyên; Room Type mới xuất hiện trong `/api/room-types` kèm sportIds. Không tạo sân vật lý. Không cần migration.

## Tạo sân cho thuê trọn gói — 19 October 2026

Manager-only `POST /api/rooms/rental` lưu sân, giờ hoạt động, giá riêng và bật dịch vụ thuê trong một transaction. Court-rate request/response bổ sung `roomId` nullable; giá riêng được ưu tiên hơn giá chung theo loại sân. Xem contract, validation và verification.

## M06 class opening decision — 18 October 2026

Xem chính sách đã xác nhận và phần triển khai. Tái sử dụng các endpoint threshold/respond/waive/cancel/reschedule hiện có. Response lớp công khai thêm boolean `openingConfirmed` mà không lộ chi phí hoặc sĩ số hòa vốn. Chỉ lớp Met/Waived chuyển sang giảng dạy; lớp chưa xác nhận khi sát giờ mở cần Manager quyết định hoặc hoãn. Cấu hình phản hồi cho phép 1–48 giờ, đánh giá trước ít nhất 72 giờ và giữ khoảng cách 24 giờ trước khai giảng. Dời buổi đầu dự kiến chưa được xác nhận cập nhật ngày khai giảng/hạn đánh giá và không được lấn vào cửa sổ phản hồi hiện có. Điểm danh có thể trả 409 `class_opening_not_confirmed`; miễn ngưỡng hồi tố trả 409 `threshold_opening_time_passed`; dời lịch không hợp lệ trả 409 `threshold_response_window_conflict`.

## Incident recovery with manual scheduling — 18 October 2026

Chỉ Manager: POST `/api/class-sessions/{id}/incident`. UI tự tìm slot và endpoint gợi ý đã được gỡ theo yêu cầu người dùng. Manager nhập thủ công thời gian thay thế, phòng và Coach; lệnh lưu kiểm tra xung đột. Tái sử dụng các route lịch và sự cố hiện có; chỉnh sửa quá khứ thông thường vẫn bị chặn. Có thể báo cáo sự cố lớp trong quá khứ với buổi thay thế đủ thời lượng, dữ kiện bất biến và loại trùng khi retry. Các route sự cố liên quan được liệt kê bên dưới.


## Coach–Member assignment lifecycle update — 18 October 2026

M02 tái sử dụng các route tìm/tạo/kết thúc quan hệ hiện có. Tạo thủ công hiện yêu cầu ghi danh đã xác nhận trong khóa Published/InProgress do Coach đó chủ nhiệm (409 `relationship_requires_active_class`). Quan hệ từ mọi nguồn tự kết thúc khi khóa/ghi danh hỗ trợ cuối cùng kết thúc. Response thêm `endReason` và `activeClasses: [{classId,className}]`; bản ghi đã kết thúc được giữ làm lịch sử. Xem chi tiết vòng đời và transaction. Không thêm HTTP endpoint mới.

## Staff activation and audit policy update — 14 October 2026

Chính sách hiện hành được mô tả trong [UsersController](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.Administration/Api/UsersController.cs) và [AuditLogsController](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.Administration/Api/AuditLogsController.cs). Chính sách này thay thế giới hạn audit của Admin chỉ trong phạm vi tài khoản. Link nhân viên hết hạn tại thời điểm tạo tài khoản + 24 giờ. Admin đọc sự kiện Manager/Admin/Coach/System; Manager đọc sự kiện Manager/Receptionist/Coach/Member, theo vai trò bất biến tại thời điểm sự kiện. `GET /api/audit-logs` nhận `actorRole` chuẩn tùy chọn; mọi bộ lọc và danh mục hành động đều được giới hạn phạm vi phía server. Response thêm `actorRole` và metadata lời mời người dùng (`activationExpiresAtUtc`, `activationResendAvailableAtUtc`). Kích hoạt Member cũng giữ thời hạn 24 giờ.

## Admin account administration

- Trạng thái tài khoản có đúng hai giá trị chuẩn: `ACTIVE` (Active) và `DEACTIVATED` (Deactivated). Danh sách, chi tiết, bộ lọc trạng thái và tổng số dùng các giá trị này. Giá trị `BANNED` đã lưu kiểu cũ được chuyển sang `DEACTIVATED`; migration không bao giờ cấp quyền truy cập cho tài khoản từng bị chặn. Bộ lọc cũ `status=BANNED` chọn tài khoản Deactivated để tương thích; client mới gửi `DEACTIVATED`. Trạng thái cũ vẫn đọc được trong payload audit lịch sử.
- Admin Overview chỉ có Active accounts và Deactivated accounts làm KPI trạng thái tài khoản; kích hoạt nhân viên là mục riêng. Users & roles cung cấp All/Active/Deactivated trong dropdown trạng thái. Deep link activation-required/expiring hiện có vẫn được hỗ trợ và hiển thị thành chip bộ lọc Invitation riêng. Sửa tài khoản hiển thị một thao tác Activate hoặc Deactivate phù hợp, tiếp theo là lý do và rà soát; không có công tắc Account enabled trùng lặp hoặc thao tác Lock/Unlock.
- `GET /api/users/admin?status=ACTIVATION_REQUIRED` chọn tài khoản Active không phải Member, chưa có mật khẩu hoặc đăng nhập ngoài. `status=ACTIVATION_EXPIRING` chọn cùng nhóm tài khoản có hạn cố định `CreatedAt + 24 hours` lớn hơn giờ UTC hiện tại của server và không quá sáu giờ tới. Đây là bộ lọc lời mời, không phải giá trị `UserStatus` mới; cả hai hàng đợi đều giao với nhóm Active. Kích hoạt quyền truy cập tài khoản không xác minh email, tạo thông tin xác thực, gửi lại lời mời hoặc gia hạn lời mời.
- Lệnh thay đổi vai trò và trạng thái dùng chung advisory lock PostgreSQL trong phạm vi transaction `(105,24)` trước khi khóa tài khoản đích. Người thao tác được đọc lại và vẫn phải Active/SystemAdministrator sau khi chờ; nếu không trả 403 `administrator_access_changed`. Kiểm tra ngăn vô hiệu hóa Admin Active cuối cùng được thực hiện trong ranh giới này. Các quy tắc chặn tự vô hiệu hóa, vai trò Admin bất biến và tách Member/nhân viên vẫn được áp dụng.
- `POST /api/users/{userId}/activate` và `POST /api/users/{userId}/deactivate` yêu cầu lý do 3–500 ký tự, lần lượt đặt `ACTIVE` và `DEACTIVATED`. Thay đổi trạng thái thực tế commit trạng thái, security stamp mới, audit và outbox cùng nhau; phiên cũ mất hiệu lực tại request đã xác thực tiếp theo và vẫn mất hiệu lực sau khi kích hoạt lại. Hành động audit mới là `ACTIVATE_USER_ACCOUNT` và `DEACTIVATE_USER_ACCOUNT`; hành động lock/unlock lịch sử vẫn hiển thị. Deactivate chặn đăng nhập và sử dụng dịch vụ, bao gồm mua/check-in/điểm danh có nhân viên hỗ trợ, đồng thời giữ booking, quyền lợi và lịch sử ví hiện có. Mọi hủy/hoàn trả dùng workflow có audit riêng.
- `CoachCoverageRequired` dùng `transitionId` mới làm `SourceEntityId` của thông báo; cùng ID xuất hiện trong `newValue` của audit trạng thái tài khoản. ID xác định lần chuyển trạng thái, không phải Coach. `targetId` của audit vẫn là ID tài khoản. Lặp lại trạng thái hiện tại không gây thay đổi, còn lần Deactivate sau tạo sự kiện mới. Hàng đợi bố trí người thay của Manager vẫn xử lý qua API hiện có.
- Chi tiết tài khoản Admin liên kết tới `/admin/audit-log?targetEntity=UserAccount&targetId=<userId>`. Bộ lọc audit dùng chung lưu ngày lịch `dateFrom`/`dateTo` trong URL trang và gửi `fromUtc` (bao gồm nửa đêm Việt Nam) / `toUtc` (không bao gồm nửa đêm sau ngày cuối được chọn) tới `GET /api/audit-logs`. Preset: All time, Today, Last 7 days, This month. Khoảng ngày sai/đảo ngược bị từ chối ở UI. Bộ lọc loại/ID bản ghi, hành động, người thao tác và vai trò tại sự kiện giữ quy tắc hiển thị phía server.
- Phản hồi tạo/gửi lại/sửa thông tin nhân viên cho biết email đã được đưa vào hàng đợi. Gửi lại/sửa không bao giờ kéo dài `CreatedAt + 24 hours`; không thể gia hạn lời mời đã hết hạn bằng các thao tác này. Nhân viên chưa kích hoạt đã hết hạn và đủ điều kiện được job hết hạn hiện có xóa trước khi lời mời mới có thể dùng lại email. Migration hai trạng thái không thay đổi hạn kích hoạt nhân viên hoặc quyền đọc audit.


SportHub là **hệ thống quản lý trung tâm thể thao với ba môn Gym, cầu lông và bóng rổ; có thể mở rộng thêm môn trong tương lai**.

Các bảng mô tả contract; phần route baseline chỉ để tra cứu đường đọc tương thích. Không dùng baseline để khôi phục luồng ghi legacy. Trạng thái triển khai và nghiệm thu xem mục 13 của thiết kế hệ thống.

## Contract giao diện nghiệp vụ

### Hồ sơ học viên Coach và lịch sử cùng môn

| Verb | Route | Contract |
|---|---|---|
| GET | `/api/coaches/me/teaching/students?page&search&sportId` | Coach; học viên duy nhất từ các lớp đang phân công, 10 mục/trang; `items[{memberId,memberName,email,classes[]}]`, `sports[]`, `page`, `pageSize`, `totalCount`. |
| GET | `/api/coaches/me/teaching/students/{memberId}` | Coach; thông tin học viên và lớp/môn hiện tại trong phạm vi phân công. |
| GET | `/api/coaches/me/teaching/students/{memberId}/history?classId&page` | Coach; `RESULT` của chính học viên từ lớp trước cùng môn, mới nhất trước, 20 mục/trang; mỗi mục gồm `record,className,sportName,coachName,sessionNo,sessionStartAtUtc`. Yêu cầu phân công hiện tại và ghi danh Confirmed tại classId; lịch sử chỉ đọc. |
| GET | `/api/coaches/me/teaching/students/{memberId}/history/classes?classId&page&search` | Coach; lớp có đánh giá trong phạm vi lịch sử hợp lệ, 8 lớp/trang, `classId,className,startDate,assessmentCount,averageScore,lastRecordedAtUtc`. Sắp xếp theo đánh giá gần nhất; điểm trung bình null nếu chưa có điểm. Endpoint `history` nhận thêm `sourceClassId` để tải đánh giá riêng cho lớp đã chọn. |

Các endpoint `teaching/classes/{classId}/members` và `records` hỗ trợ query `memberId` và `sessionId`. HLV dạy thay bắt buộc cung cấp `sessionId` đúng buổi được giao; records chỉ trả nội dung buổi đó. Không đổi schema/migration. Chi tiết quyền và route ở các mục Coach bên dưới.

### Sắp xếp danh sách Admin

- `GET /api/users/admin`: thêm query tùy chọn `sortBy=fullName|email|role|status`, `sortDirection=asc|desc`; mặc định `email/asc`.
- `GET /api/audit-logs`: thêm query tùy chọn `sortBy=timestamp|actorEmail|action|targetEntity`, `sortDirection=asc|desc`; mặc định `timestamp/desc`.
- Server sắp xếp trước `Skip/Take`, thêm ID làm khóa thứ hai để phân trang ổn định. Role/status dùng thứ tự enum nghiệp vụ. Cột hoặc chiều không hợp lệ trả 400 `invalid_sort_column`/`invalid_sort_direction`. Quyền và phạm vi Audit log giữ nguyên.

Các API dưới đây bổ sung đúng dependency của frontend P2.06–P2.10, không đổi schema/migration. Quy định mới về hủy khóa thay phần mô tả P1.05 bên dưới.

| Verb | Route | Actor / contract |
|---|---|---|
| POST | `/api/gym-checkins` | Receptionist; `{targetMemberId}`. Member cần gói Gym còn hiệu lực; Gym đóng trả 409 `gym_closed`. Mỗi Member chỉ được ghi thêm lượt sau ít nhất 10 giây; gửi đồng thời hoặc trong cooldown trả 409 `gym_checkin_cooldown`. Lịch sử chỉ ghi thời điểm vào, không có thao tác check-out. |
| PUT | `/api/manager/coaches/{userId}` | Manager; bổ sung `fullName?` (2–100 ký tự, validator hiện có), `phone?` (đúng định dạng, unique); null giữ giá trị cũ, chuỗi phone rỗng xóa số. Không đổi email/role/UserStatus. |
| GET | `/api/manager/classes?thresholdStatus=AT_RISK` | Manager; thêm filter threshold, giữ sport/lifecycle/search/pagination hiện có. Enum wire UPPER_SNAKE_CASE. |
| GET | `/api/manager/classes/{classId}/holds` | Manager; phân trang `holdId,memberId,memberName,invoiceId,status,expiresAtUtc,createdAt`. |
| GET | `/api/manager/classes/{classId}/enrollments` | Manager; paged `enrollmentId,memberId,memberName,invoiceItemId,status,enrolledAt`; lấy invoice bằng endpoint by-item hiện có. |
| GET | `/api/manager/classes/{classId}/threshold-responses` | Manager; paged `responseId,memberId,memberName,choice,targetClassId,resolutionStatus,additionalInvoiceId,deadlineUtc`. Không trả token/hash. |
| GET | `/api/manager/classes/{classId}/cancellation-preview` | Manager; `canCancel,version,totalSessions,sessionsNotProvided,confirmedCount,activeHoldCount,refundPoints,previewToken,members[],requiresMemberDecision`. Thông thường không thể hủy lớp đã có ghi danh. Ngoại lệ: Published + AtRisk + sĩ số đã xác nhận dưới ngưỡng, không có buổi Completed, có item đã thanh toán. Ngoại lệ tính số điểm có thể hoàn và yêu cầu Member lựa chọn sau khi hủy. |
| POST | `/api/manager/classes/{classId}/cancel` | Manager; `{reason,previewToken?}`. Ghi danh còn hiệu lực thông thường: 409 `class_has_enrollments`. Ngoại lệ dưới ngưỡng yêu cầu preview mới, hủy buổi/nhả occupancy và hold, đưa yêu cầu lựa chọn qua chuông/email cho từng Member đã trả tiền còn lại vào hàng đợi một cách nguyên tử. Manager hủy lớp dưới ngưỡng đã có học viên chỉ thông báo: không cộng điểm tại thời điểm hủy, sau đó từng Member chọn điểm hoặc yêu cầu hoàn tiền (Phần H, `api/refunds/center-cancellations`). Phản hồi giữ chỗ/chờ chuyển hiện có trở thành quyết định Pending mới, với toàn bộ thời hạn phản hồi theo cấu hình (1–48 giờ, mặc định 48) tính từ lúc hủy. Chỉ cho phép REFUND hoặc WAIT_NEXT_COURSE với lớp đã hủy; cả hai hoàn giá trị còn lại bằng điểm, WAIT còn đăng ký Course interests. Không phản hồi: tự hoàn điểm sau hạn. Retry hủy/lựa chọn có tính idempotent. Preview cũ: 409 `class_cancellation_changed`. |
| POST | `/api/manager/classes/{classId}/enrollment/close` | Manager; body `{reason}` bắt buộc, 3–500 ký tự (thiếu/ngắn → 400 `enrollment_close_reason_required`). Chỉ đóng tuyển sinh lớp Published: checkout mới nhận 409 `enrollment_closed`; hold Active và ghi danh hiện có không bị hủy, hold hợp lệ vẫn hoàn tất thanh toán. Luôn gửi thông báo + email tiếng Anh cho Member đã ghi danh. Response `ClassManagerResponse` thêm `enrollmentClosedReason`, `enrollmentClosedAtUtc`. Đóng lặp là no-op; không hủy lớp hoặc hoàn điểm. |
| POST | `/api/manager/classes/{classId}/enrollment/open` | Legacy compatibility; không còn nút trong giao diện Manager. Không mở lại lớp đủ sĩ số (`class_full`). |
| GET | `/api/members/me/class-holds` | Member; danh sách hold lớp còn hiệu lực của chính Member: `[{classId,invoiceId,expiresAtUtc}]` (Active, chưa hết hạn, có invoice). UI dùng để hiện “Hoàn tất thanh toán” cho lớp đã đóng tuyển sinh sau khi Member giữ chỗ; hold vẫn thanh toán được tới `expiresAtUtc`. |
| POST | `/api/manager/notices` | Manager; body hiện có, thêm header `Idempotency-Key` UUID. Cùng Manager/key/payload nhận cùng noticeId và không tạo thêm outbox; đổi payload trả 409 `notice_idempotency_conflict`. Client cũ không có key vẫn được hỗ trợ. |
| GET | `/api/manager/notices` | Manager; lịch sử của chính mình, query `search` theo tiêu đề, `fromDate`/`toDate` theo ngày Việt Nam (bao gồm hai đầu), `page`/`pageSize` (tối đa 50). Trả `PagedResult` với tiêu đề, người gửi, thời điểm, số người nhận và kênh đã chọn; mới nhất trước, tie-break bằng noticeId. |
| GET | `/api/manager/notices/{noticeId}` | Manager đã gửi; giữ `{noticeId,delivery}`, thêm `subject,message,createdAtUtc,senderName,recipientCount,sendInApp,sendEmail,isLegacy`. Manager khác nhận 404. |
| GET | `/api/manager/notices/{noticeId}/recipients` | Manager đã gửi; phân trang `page`/`pageSize` (tối đa 50). Mỗi người có `userId,name,email,inApp,emailDelivery`; mỗi kênh có `status,retryCount,sentAt,lastAttemptAt`, hoặc null khi không có bản ghi. Không trả payload email hay lỗi kỹ thuật của SMTP. Manager khác nhận 404. |
| GET | `/api/manager/notices/by-key/{key}` | Manager đã gửi; phục hồi receipt sau timeout/F5, chờ transaction gửi cùng key hoàn tất. |

**Quy tắc Manager hủy lớp:** Manager không thể hủy lớp có bất kỳ ghi danh `Confirmed` còn hiệu lực (`EndedAt == null`). Preview trả `canCancel=false`, `refundPoints=0` và `members=[]`. Cả `POST /api/manager/classes/{classId}/cancel` và các nhánh hủy kèm học bù (gồm xử lý yêu cầu Coach và thay thế do sự cố) trả HTTP `409`, lỗi `class_has_enrollments`. Kiểm tra chạy sau khi khóa dòng lớp và không tin số đếm cache hoặc preview trước đó. Dời lịch, đổi phòng và phân công Coach đủ chuyên môn khác vẫn khả dụng. Xử lý tự động dưới ngưỡng dùng các service riêng hiện có và không thay đổi.

Migration `AddManualNoticeHistory` tạo `manual_notice_records`. Lần gửi mới lưu nguyên văn nội dung và snapshot tên/email người nhận/người gửi trong cùng transaction với audit/outbox; thay hồ sơ sau này không đổi lịch sử. Khôi phục lịch sử cũ từ audit `SEND_MANUAL_NOTICE` và outbox ManualNotice: tiêu đề/ngày gửi/người gửi, người nhận còn có dữ liệu và nội dung in-app nếu còn; không giải mã email. Bản cũ có `isLegacy=true`, `message=null` nếu không khôi phục được, tên có thể lấy từ hồ sơ hiện tại. Không gửi lại thông báo trong migration. `recipientCount` là số người; `delivery.total` và các trạng thái đếm theo từng kênh, không phải số người. `Read` của chuông không chứng minh email đã được mở.
| GET | `/api/manager/incidents/{incidentId}/notifications` | Manager; delivery tổng các notice IncidentResolution của rental thuộc incident. |
| GET | `/api/manager/incidents?page=1&pageSize=20` | Manager; danh sách sự cố đã ghi nhận, mới nhất trước, paging (pageSize 1–100). Mọi Manager xem được; trả `{items,page,pageSize,totalCount}`. |
| GET | `/api/manager/incidents/{incidentId}` | Manager; chi tiết `{incidentId,scope,roomId,roomName,startAtUtc,endAtUtc,reason,resolutionSummary,createdAtUtc}`; 404 `incident_not_found` khi không tồn tại. Trang Incident dùng bảng cuộn để chọn sự cố và mở chi tiết/trạng thái gửi thông báo. |
| GET | `/api/court-rentals/policy` | Member; `slotMinutes,maxHours,advanceDays,cancelFreeHours,serverNow`; không mở quyền đọc mọi system setting. |
| GET | `/api/court-rentals/{rentalId}` | Member chủ thuê; `rental,roomName,sportName,blocks,cancelReason,cancelledAtUtc,refundedPoints`. Rental summary thêm nullable invoiceId kể cả PendingPayment. Người khác 404; blocks là snapshot giá, refundedPoints từ ledger SystemEvent. |

Hủy khóa tính `floor(remainingPaidValueVnd × sessionsNotProvided / totalSessions / 1000)`; giá trị còn lại trừ các lần refund/transfer trước. Buổi gốc hoặc buổi bù đã Completed được coi đã cung cấp theo authority scheduling; không lấy giờ browser để tính hoàn. Ghi danh legacy thiếu paid item chặn hủy để tránh hoàn sai. Pending/AwaitingPayment threshold responses được hết hạn cùng transaction, tiền gateway đến muộn đi qua cơ chế compensation hiện có.

Delivery có `total,pending,sending,sent,failed,read`; đây là trạng thái outbox/in-app. SMTP vẫn at-least-once; receipt/idempotency không có nghĩa người nhận đã đọc email hoặc exactly-once SMTP.

## Contract tích hợp frontend

Các endpoint dưới đây phục vụ frontend API-backed. Không thay schema/migration. ID lớp/package/room/sport là int; ID invoice/enrollment/request là UUID. Backend tiếp tục kiểm role, ownership, trạng thái và đồng thời.

| Verb | Route | Hợp đồng / authority |
|---|---|---|
| GET / PUT | `/api/users/me`, `/api/users/me/profile` | Thêm `sportIds: number[]`, nullable `approvalStatus`. Role UPPER_SNAKE_CASE. F5 refresh profile trước cấp quyền trên UI; regression Security đã pass. |
| GET | `/api/membership-packages/public` | Anonymous, chỉ catalog active; endpoint quản trị/auth cũ giữ chính sách riêng. |
| GET | `/api/classes?fromDate=&toDate=&sportId=&page=&pageSize=` | DateOnly YYYY-MM-DD theo startDate khóa; validate khoảng ngày; danh sách public gồm khóa Published và InProgress (nhận ghi danh giữa chừng khi còn chỗ). Mỗi khóa có `sessionsPassed` (buổi đã bắt đầu, không tính buổi hủy) và `sessionsRemaining` (buổi Scheduled sắp tới). Đăng ký/checkout lớp không bị chặn vì đã qua buổi nào; chỉ trả 409 `class_started` khi không còn buổi sắp tới. Giá luôn là cả khóa; hạn giữ chỗ không vượt quá giờ bắt đầu buổi kế tiếp. |
| GET | `/api/classes/{classId}/public-sessions` | Anonymous, chỉ khóa Published, lịch buổi/room/coach không có roster/attendance. |
| GET | `/api/members/me/classes/{classId}/sessions` | Member có ghi danh còn hiệu lực (`Confirmed`, chưa `EndedAt`) trong khóa Published/InProgress/Completed (xem lại lịch sử sau khi lớp kết thúc); khóa Cancelled/Draft hoặc ghi danh đã kết thúc trả 403 `class_access_ended`. Không trả roster. |
| GET | `/api/coaches?sportId=` | Member/Receptionist/Manager: Coach active cùng specialty (`sportId`), chỉ userId/fullName/sportIds. Không trả email hoặc credential. |
| GET | `/api/checkouts/by-key?key=`, `/api/checkouts/by-reference?reference=` | Recovery timeout/return theo invoice của beneficiary hoặc initiator; kiểm scope lại. Query VNPay không là chứng cứ Paid. |
| POST | `/api/checkouts/{invoiceId}/confirm-points` | Xác nhận cash=0, invoice lock, ownership, hold còn hạn, không reconciliation/verified pending; idempotent Paid, không tạo gateway attempt. |
| GET | `/api/checkouts/{invoiceId}` | Thêm beneficiaryUserId/initiatorUserId/serverNowUtc và ptMemberPackageId/ptCoachId/ptFrequency để resume/re-quote khi retry. |
| POST | `/api/checkouts/{invoiceId}/attempts` | Response thêm gatewayMode MOCK/VNPAY theo provider thực. Cash>0 mới có QR/link. Cả attempts và confirm-points chặn `point_confirmation_pending`. |
| GET | `/api/invoices/{invoiceId}/point-confirmations/current` | Receptionist; audited beneficiary, revision, points, failedAttempts, expiry/resendAtUtc và status PENDING/LOCKED/EXPIRED; không trả code/hash/salt. |
| GET | `/api/invoices/by-item/{itemId}` | Lookup invoice từ own enrollment invoiceItemId; dùng lại quyền đọc invoice. Own invoice list hỗ trợ filter status. |
| GET | `/api/refunds/quote/{itemId}` | Ownership + paid/benefit eligibility; `{systemCalculatedPoints}` do backend tính. |
| POST | `/api/refunds/class-cancellations` | Body `{invoiceItemId}`. Member hủy ghi danh của mình, Lễ tân/Manager hủy thay. Chỉ trước buổi đầu: còn ≥ 5 ngày hoàn 100%, dưới 5 ngày hoàn 50% (tự động bằng điểm, ghi `PaymentAdjustment` Completed). Lớp đã bắt đầu → 409 `class_already_started`, dùng `POST /api/refunds` để Manager quyết định. |
| POST | `/api/refunds/membership-cancellations` | Body `{invoiceItemId}`. Hủy Membership đang hiệu lực (Member, hoặc Lễ tân/Manager thay): còn từ 2/3 thời hạn hoàn 50% bằng điểm, ít hơn 409 `refund_points_zero`; còn gói gia hạn nối tiếp → 409 `membership_has_renewal` (hủy gói nối tiếp trước). |
| POST | `/api/refunds` | Item lock và tối đa một request REQUESTED chưa xử lý trên item; submit lặp trả request hiện có. Không cho FE tự gửi số điểm duyệt. |
| GET | `/api/class-threshold-responses/mine`, `/{id}`, `/by-token?token=` | Member owner; course/sport/remaining paid value/deadline/choice/status/additionalInvoiceId/serverNowUtc. Token chỉ hash ở backend, không ghi analytics. |
| GET | `/api/class-threshold-responses/{id}/transfer-quote?targetClassId=` | Owner, response còn mở; cùng môn, target Published/chưa bắt đầu. Backend tính cashDifference/walletCreditPoints; giá trị chia hết cho 1.000. |
| POST | `/api/class-threshold-responses/{id}` | `{choice:REFUND|TRANSFER,targetClassId}`; dùng cùng transaction/lock/idempotent-final-choice như token route. Retry AwaitingPayment giữ nguyên đích. |
| GET | `/api/coach-member-relationships` | Member/Coach/Manager; Member/Coach bị clamp theo owner. Không có endpoint members/me/relationships giả. |
| GET | `/api/wallet/me/ledger?page=&pageSize=&entryType=` | Array WalletLedgerResponse, không Paged; timestamp createdAtUtc. Filter HOLD/RELEASE/SPEND/EARN/ADJUSTMENT áp trước pagination; filter sai 400. Ledger staff cũng hỗ trợ filter; chỉ Receptionist ghi audit lần xem. |

## API bổ sung cho Coach lớp nhóm

Các endpoint được giới hạn theo lớp/buổi được phân công, hỗ trợ lưu giáo án, kết quả, bài tập/thông báo, điểm danh và Gemini suggestion. Coach chủ nhiệm chỉ đọc hồ sơ học viên Confirmed của lớp Published/InProgress; HLV dạy thay chỉ đọc roster và bản ghi của buổi được giao. Member có feed khi Enrollment Confirmed, `EndedAt` rỗng và lớp Published/InProgress/Completed, kể cả sau buổi cuối; lớp Cancelled/Draft hoặc ghi danh đã kết thúc bị chặn. Có migration `AddClassTeachingWorkspace`; extension này được chủ sản phẩm xác nhận cho phép Coach ghi điểm danh lớp nhóm qua route mới.

- `POST /api/coaches/me/teaching/classes/{classId}/suggestion` chỉ nhận `scope: "session"` với `sessionId` bắt buộc thuộc lớp đó. Coach chủ nhiệm chỉ được tạo gợi ý cho buổi Scheduled có thời điểm bắt đầu còn ở tương lai; scope khác/thiếu buổi trả `400 suggestion_context_invalid`, buổi đã qua, bắt đầu, hoàn thành hoặc hủy trả `409 session_suggestion_unavailable`. Gợi ý chỉ áp dụng buổi đó và có sets, reps dương cho mọi bài tập. UI không còn cung cấp tạo lộ trình khóa mới; các plan hiện có vẫn khả dụng. Danh sách lớp và buổi đặt mục hiện tại/sắp tới trước (gần nhất trước), rồi mục đã qua (mới nhất trước).
- Các nơi hiển thị avatar người dùng lấy ảnh hiện tại qua `GET /api/avatars/{userId}` và chia sẻ cập nhật tải lên/xóa giữa các view đang mount. Receptionist Active đã xác thực có thể đọc avatar cùng các role nhân viên khác; điều này không cấp quyền tải lên hoặc xóa avatar. Truy cập công khai không xác thực vẫn chỉ giới hạn avatar Coach.

`hold.minutes` cho checkout mới: Manager đặt số nguyên **10–30**, mặc định **15**. Hạn lưu trên checkout/attempt hiện có là snapshot, không thay đổi khi Manager cập nhật setting. Lớp, sân và chuyển lớp có thể cắt hạn sớm theo giờ bắt đầu hoặc deadline nghiệp vụ; client dùng `expiresAtUtc` trả về, không tự cộng số phút cấu hình.

## Quy ước wire và dữ liệu

- Enum JSON hiện tại: **UPPER_SNAKE_CASE** (`ISSUED`, `PAID_AFTER_RECONCILIATION`, `GROUP_COURSE`, `VN_PAY`). Enum số không được chấp nhận. DTO string biểu diễn enum có `WireEnum` converter; query enum chấp nhận canonical và tên nội bộ để tương thích. JWT role claim vẫn dùng tên nội bộ, không tự chuyển JWT.
- Enum DB hiện tại: lưu int mặc định EF (không có `HasConversion`). Khi thêm giá trị phải append, không đổi số cũ. `UserRole` hiện: CenterManager=0, Coach=1, Member=2, Receptionist=3, SystemAdministrator=4 (đúng 5 role, BR-61).
- `InvoiceStatus` DB: Issued=0, Paid=2, Void=3, PaidAfterReconciliation=4. Wire: `ISSUED`, `PAID`, `VOID`, `PAID_AFTER_RECONCILIATION`.
- Auth: JWT Bearer; policy trong `backend/SportHub.BuildingBlocks/Api/SportHubPolicies.cs`.
- Error body thông thường `{error,message}`, mã lỗi chữ thường snake_case; lỗi occupancy có thêm `conflicts[]`. ID vẫn đúng kiểu int/Guid, thời gian UTC ISO-8601; tham số ngày báo cáo/lịch dùng ngày Việt Nam.

## Bản chốt API — lịch sân, incident, payment và snapshot


### Lịch sân và xử lý sự cố

| Verb | Route / request | Actor | Response / lỗi |
|---|---|---|---|
| GET | `/api/manager/court-schedule?fromDate=YYYY-MM-DD&toDate=YYYY-MM-DD&roomId=` | Manager, Receptionist | 200 `CourtScheduleEntry[]`; hai ngày inclusive, tối đa 31 ngày; 400 `invalid_range`, 403 `court_schedule_forbidden`. |
| GET | `/api/coaches/me/court-schedule?fromDate=&toDate=&roomId=` | Coach | 200 cùng DTO nhưng chỉ lớp được giao. |
| POST | `/api/manager/incidents/preview` body `{scope:"ROOM"|"CENTER",roomId?,startAtUtc,endAtUtc,reason}` | Manager | 200 `{scope,roomId,startAtUtc,endAtUtc,impacts[],canResolve,blockReason,members[],expectedRefundPointsTotal}`. Mỗi impact có `{sourceType,sourceId,startAtUtc,endAtUtc,resolutionOptions[]}`. `members[]` do server tính: `{memberId,memberName,sourceType,sourceId,outcome,expectedRefundPoints?}`; `outcome` = `REFUNDED_AS_POINTS` (rental đã thanh toán; khi resolve chỉ thông báo, Member chọn nhận 100% = floor(giá trị còn lại/1.000) điểm hoặc gửi yêu cầu hoàn tiền, xem Phần H), `CHECKOUT_RELEASED` (đơn chưa thanh toán, nhả hold), `AWAITS_SCHEDULE_DECISION` (hội viên lớp có buổi bị ảnh hưởng, chờ dời/bù, `expectedRefundPoints` null). Số điểm là báo giá tại thời điểm preview; resolve tính lại và khớp công thức. |
| POST | `/api/manager/incidents/resolve` body như preview | Manager | 200 `{incidentId}`; 409 `incident_requires_schedule_resolution` nếu còn lớp/block, `incident_schedule_changed` hoặc `occupancy_conflict` nếu lịch đổi; lỗi 400 `invalid_incident_scope`, `incident_reason_required`, `invalid_incident_range`; 404 `incident_room_not_found`. |
| POST | `/api/manager/incidents/fences` body như preview | Manager | 200 `IncidentFence`. Bắt đầu xử lý: giữ khung giờ (phòng hoặc toàn trung tâm) để không ai đặt mới trong lúc dời/bù lớp. Hiệu lực 2 giờ, mở lại cùng khung giờ là gia hạn chứ không tạo thêm. Mọi đặt mới (thuê sân, buổi lớp, dời buổi) chồng khung giờ bị `409 room_under_incident_review`; room block của chính sự cố được miễn. |
| GET | `/api/manager/incidents/fences` | Manager | Các khung giờ đang giữ: `{fenceId,scope,roomId,roomName,startAtUtc,endAtUtc,reason,createdAtUtc,expiresAtUtc,isExpired,remainingImpacts,affectedMemberCount,canResolve}`. `remainingImpacts` được đọc lại lúc gọi nên Manager thấy còn gì phải xử lý; `isExpired` = hết giữ (cho đặt lại) nhưng vẫn liệt kê để làm tiếp. |
| POST | `/api/manager/incidents/fences/{fenceId}/extend` | Manager | Giữ thêm 2 giờ; 409 `incident_fence_closed` nếu đã resolve/nhả hoặc khung giờ đã qua. |
| POST | `/api/manager/incidents/fences/{fenceId}/release` | Manager | 204. Nhả khung giờ không resolve; lặp lại là no-op; 409 `incident_fence_closed` nếu đã resolve. `resolve` tự đóng các fence nằm trong khung giờ của nó. |
| GET | `/api/manager/incidents/fences/{fenceId}/steps` | Manager | Nhật ký `{stepId,kind,className,sessionNo,beforeStartUtc,afterStartUtc,state,undoNote,canUndo}`; `kind` = `RESCHEDULE`/`CANCEL_WITH_MAKEUP`/`REMOVE_ROOM_BLOCK`/`REVIEWED_COMPLETED`, `state` = `APPLIED`/`UNDONE`/`UNDO_FAILED`. Chỉ ghi các bước thuộc fence đang mở; thay đổi ngoài khung giờ không được hoàn tác. Buổi đã học đủ chỉ ghi nhận: `REVIEWED_COMPLETED` luôn có `canUndo=false`, không tính vào `appliedSteps` cần hoàn tác. |
| POST | `/api/manager/incidents/fences/{fenceId}/abandon` | Manager | Bỏ hẳn sự cố trong một transaction: nhả khung giờ rồi hoàn tác các bước từ mới nhất về cũ nhất. Mỗi bước chỉ hoàn tác khi buổi vẫn đúng như bước đó để lại và phòng/coach gốc còn trống; room block đã gỡ được tạo lại theo ID cũ nếu còn hợp lệ và khung giờ còn trống. Ngược lại giữ nguyên và trả trong `notUndone[]` kèm `undoNote`. Trả `{fenceId,undone,notUndone[]}`. Hội viên được báo khi buổi quay lại giờ ban đầu hoặc buổi bù bị hủy. 409 `incident_fence_closed` nếu đã resolve/nhả. Bỏ qua `REVIEWED_COMPLETED`: giữ lịch sử và buổi Completed, không báo đây là hoàn tác thất bại. |

Migration `20261027100000_DistinguishCompletedIncidentSteps` tách mã nội bộ `ReviewedCompleted = 3` khỏi `RemoveRoomBlock = 2`. Dữ liệu cũ có `kind=2`, `session_id` khác null và `room_block_id` null chuyển sang 3; giữ nguyên mã bản ghi, trạng thái và ghi chú lịch sử. Dòng gỡ room block giữ mã 2. Áp dụng migration cùng phiên bản backend này.
| GET | `/api/reports/revenue/reconciliation?fromDate&toDate` | Manager | Đối soát: tính lại từng số của báo cáo doanh thu trực tiếp từ payment, ledger điểm, refund và `checks[{name,reported,actual,delta,ok}]`, cùng hóa đơn Paid có tiền mặt/điểm không khớp thanh toán (`invoiceMismatchCount`, mẫu tối đa 50). Chỉ dùng aggregate trong DB; chạy theo yêu cầu, không tự chạy. |

**Sự cố Gym tạm thời:** không gia hạn Membership, không hoàn điểm hoặc hủy day pass trong resolve; từ chối `membershipExtensionDays` khác 0. `GET /api/manager/incidents/{id}` trả `closesGym`, `dayPassMembers` (snapshot passId/memberId/memberName/useDate/expiresAtUtc), `dayPassNoticeSentAtUtc`. Manager chủ động gửi thông báo qua `POST /api/manager/incidents/{id}/day-pass-notice` (204, idempotent); sau đó dùng point-gifts cho Member có Membership Active giao với khoảng đóng Gym hoặc người có day pass trong snapshot. Người có day pass trong snapshot phải đồng ý nhận điểm. Mỗi khoản tặng gắn incident + Member để chống tặng trùng; ledger/audit/outbox cùng transaction.

**Manager tặng điểm trực tiếp:** ngoài sự cố cũng được tặng cho người đủ điều kiện. `/manager/points` cho chọn từng người nhận, nhập điểm nguyên dương và lý do bắt buộc, xem lại danh sách người nhận, số điểm từng người và lý do trước khi gửi; không có dòng tổng trong Review gift. Chính Manager xác nhận thì cộng ngay, không cần Manager khác duyệt. Dùng `/api/manager/point-gifts` bên dưới; Receptionist không có quyền (403).
| GET | `/api/manager/operations/health` | Manager | Sức khỏe vận hành: `status` Healthy/Degraded/Unhealthy, `jobs[]` (lần thành công/lỗi cuối, số lỗi liên tiếp, `unhealthy` khi im lặng quá 3 chu kỳ hoặc lỗi ≥3 lần; lịch sử job nằm trong bộ nhớ, reset khi khởi động lại), `outbox` (pending, kẹt khi gửi, failed, failed ≥5 lần), `payments` (sự kiện cổng chưa xử lý/lỗi, hóa đơn đã thanh toán nhưng chưa kích hoạt (`paidNotActivated`, đếm duy nhất theo hóa đơn trong hàng đợi Unactivated), attempt chờ đối soát, hóa đơn bị đánh dấu, seat hold hết hạn chưa nhả), `support.overdueRequests` (SLA đã đến hạn, chưa DONE/CLOSED/REJECTED). Hai chỉ số tồn đọng mới làm health Degraded; bỏ `manualCompensationRequired`. |
`CourtScheduleEntry`: `{sourceType,sourceId,roomId,startAtUtc,endAtUtc,coachId,coachName,title,status,classId,participants[]}`. Source type: `CLASS_SESSION`, `COURT_RENTAL`, `ROOM_BLOCK`. Participant: `{memberId,memberName,enrollmentId,attendanceStatus,recordedAtUtc}`. Rental/block không có roster; Member chỉ gọi availability và lịch thuê của mình, không gọi hai endpoint lịch nội bộ.

`resolutionOptions` trả `{action,method,path}`. Action identifiers: `Reschedule`, `CancelWithMakeup`, `CancelByCenter`, `RemoveExistingBlock`, `AutoCancelAndRefundOnResolve`; đây là identifier thao tác, giữ đúng chuỗi API trả, không phải enum. Frontend mở form tương ứng và gọi route được trả: class reschedule/cancel kèm makeup, xóa block không gắn incident. Sau khi xử lý, gọi preview lại rồi resolve. Resolve không tự chọn lịch thay người dùng; chỉ hủy/hoàn rental và tạo incident/block khi kiểm lại không còn conflict. Email và audit cùng transaction với bước nghiệp vụ tương ứng.

### Checkout và invoice

POST checkout trả 201 `CheckoutResponse`, yêu cầu header `Idempotency-Key` (không rỗng, tối đa 120 ký tự):

| Route | Body | Actor |
|---|---|---|
| `/api/checkouts/membership` | `{packageId,targetMemberId?,allowStacking:false,stackingApprovalReason?}` | Member hoặc FrontDesk; stacking cần quyền Manager và lý do. |
| `/api/checkouts/class` | `{classId,targetMemberId?}` | Member hoặc FrontDesk. |
| `/api/checkouts/court-rental` | `{sportId,roomId,startUtc,endUtc}` | Member tự mua. |
| `/api/checkouts/wallet-top-up` | `{points,targetMemberId?}` (1–10.000) | Member tự nạp, hoặc Receptionist nạp hộ một Member được chọn rõ ràng tại quầy (quyết định 25/10/2026: `targetMemberId` bắt buộc với staff, 400 `target_member_required`; điểm vào ví Member đó, audit ghi Receptionist, Member, số điểm, invoice). Cùng luồng và giới hạn với Member tự nạp; 1 điểm = 1.000 VND, chỉ thanh toán VNPay. Receptionist không tặng, trừ hay điều chỉnh điểm (403). |

FrontDesk là Manager/Lễ tân; staff mua cho Member phải có targetMemberId, Member chỉ mua cho mình. Lỗi chung: 400 `idempotency_key_required`/`target_member_required`, 403 `target_member_forbidden`/`checkout_not_owned`; conflict nghiệp vụ trả 409. GET checkout trả 200; attempts trả 200 `PaymentAttemptResponse` (`paymentAttemptId,invoiceId,transactionReference,cashAmount,pointsApplied,expiresAtUtc,paymentUrl,state`); không gửi giá do client tính.

`CheckoutResponse`: `{invoiceId,checkoutSessionId,revision,kind,state,totalAmount,pointsApplied,cashAmount,expiresAtUtc,resourceHoldId,invoiceStatus,fulfillmentOutcome,reconciliationRequired}`. Số tiền server quyết định; FE không tự đánh dấu paid từ redirect return hoặc chỉ từ `state`.

Wallet top-up dùng checkout/invoice/attempt/IPN/QueryDR và ledger hiện có. Chỉ cộng điểm sau khi VNPay xác minh khoản thu; callback lặp không cộng lại. Member không thể dùng điểm để thanh toán top-up. Giới hạn FE: 10.000–10.000.000 ₫ (bội số 1.000 ₫; BE chấp nhận 1–10.000 điểm). Mỗi Member chỉ có một top-up mở: `POST /api/checkouts/wallet-top-up` với key mới sẽ hủy top-up `Issued` chưa thanh toán trước đó (trả 409 `payment_reconciliation_pending` nếu top-up cũ đang chờ đối soát); cùng `Idempotency-Key` trả lại đúng checkout. "Change amount" ở FE gọi `POST /api/checkouts/{id}/cancel` rồi tạo checkout mới; thanh toán muộn trên checkout đã hủy vẫn được cộng điểm một lần. FE phục hồi bước thanh toán từ `?invoice=<id>` trên `/member/wallet/top-up` và chỉ tin trạng thái từ server (poll `GET /api/checkouts/{id}`). Khoản thu xác minh sau khi checkout hết hạn vẫn cộng điểm vào đúng ví và ghi `PAID_AFTER_RECONCILIATION` với `FULFILLED`.

| fulfillmentOutcome | Ý nghĩa |
|---|---|
| `PENDING` | Chưa cấp quyền lợi; xem thêm invoiceStatus/state để biết đã hủy/hết hạn hay còn chờ. |
| `FULFILLED` | Đã cấp quyền lợi, kể cả thanh toán muộn reacquire thành công. |
| `COMPENSATED` | Tiền được xác minh nhưng không cấp được quyền lợi; đã bồi hoàn bằng điểm. |
| `RECONCILIATION_REQUIRED` | Cần đối soát thủ công; không hiển thị như đã mua thành công. |

`InvoiceSummaryResponse` có `beneficiaryUserId` (Member), alias cũ `memberId`, `fulfillmentOutcome` cùng các field tiền/trạng thái hiện có. `paidVia` canonical: `POINTS`, `VN_PAY`, `VN_PAY_AND_POINTS`, `VN_PAY_AFTER_RECONCILIATION`, `VN_PAY_COMPENSATED`, `VN_PAY_MANUAL_COMPENSATION`. Không suy quyền lợi chỉ từ `PAID_AFTER_RECONCILIATION`.

`InvoiceItemResponse` bổ sung nullable `classId`, `courtRentalId`, `memberPackageId`, `sportId`, `sportName`, `sourceInvoiceItemId`. `relatedEntityId` vẫn đọc được cho lịch sử. Giá/item, sport name và thời hạn Membership được snapshot khi tạo checkout; sửa catalog không đổi hợp đồng đã mua. Membership không có sport riêng.

Membership catalog mới từ chối `sessionLimit != null` bằng 400 `membership_session_limit_removed`; giá phải dương, bội số 1.000 VND (400 `membership_price_invalid`). Những field quota legacy còn đọc được nhưng không ảnh hưởng access/expiry.

Invoice lịch sử vẫn đọc theo ownership/FrontDesk. Discount/correction không còn tạo/approve/reject (409 `legacy_adjustment_read_only`); refund dùng `/api/refunds`, không payout tiền mặt. Gửi payment thủ công trả 409 `checkout_requires_verified_payment`. Email invoice created/payment received/refund completed được queue cùng transaction; API không chờ gửi SMTP.

## Phần I — P1.10 Court Rental (đã có PostgreSQL regression ở P1.13)

| Verb | Path | Actor / contract |
|---|---|---|
| POST | `/api/checkouts/court-rental` | Member; `Idempotency-Key`; body `{sportId, roomId, startUtc, endUtc}` (không khai báo số người). Giá server tính theo giờ, giữ occupancy rồi trả checkout; payment mới chuyển Confirmed. |
| GET | `/api/court-rentals/availability?sportId&startUtc&endUtc` | Member; chỉ trả sân trống + báo giá giờ, không trả lớp/Member/nguồn lịch bận. |
| GET | `/api/court-rentals/calendar?sportId&date&hours` | Member đang hoạt động; ngày Việt Nam `yyyy-MM-dd`, thời lượng nguyên giờ (mặc định 1). Trả `{sportId,date,serverNowUtc,rooms:[{roomId,name,slots:[{startUtc,endUtc,status,totalPrice,blocks}]}]}`. Trạng thái `AVAILABLE/BOOKED/HELD/SCHEDULED/BLOCKED/UNAVAILABLE/NO_RATE`; chỉ `AVAILABLE` có báo giá và được chọn. Không trả danh tính khách, ID lượt đặt hay nguồn lịch bận. Checkout vẫn kiểm tra lại giá, giờ mở cửa và tranh chấp occupancy. UI chỉ báo thành công khi invoice `PAID` hoặc `PAID_AFTER_RECONCILIATION` và `fulfillmentOutcome=FULFILLED`. |
| GET | `/api/court-rentals/mine?fromUtc&toUtc` | Member; lượt thuê của chính họ. |
| POST | `/api/court-rentals/{rentalId}/check-in` | Chỉ Receptionist (BR-68); chỉ lượt `Confirmed`, từ 5 phút trước giờ bắt đầu đến trước giờ kết thúc. Chuyển sang `InUse` (Checked-in); ngoài cửa sổ hoặc bấm lặp trả 409. Tách biệt check-in Gym. |
| POST | `/api/court-rentals/{rentalId}/no-show` | Chỉ Receptionist (BR-68); chỉ lượt `Confirmed` và đã qua giờ bắt đầu (409 `rental_not_started`). Chuyển sang `NoShow` và giải phóng occupancy. Hết giờ kết thúc mà chưa đánh dấu thì job `CompleteDueAsync` tự ghi `NoShow` (Confirmed → NoShow, InUse → Completed). `NoShow` không hoàn tiền hay điểm. |
| POST | `/api/court-rentals/{rentalId}/cancel` | Chủ thuê; ≥24h trước giờ bắt đầu hoàn 100%, dưới 24h 0%; cùng transaction hủy occupancy. |
| GET | `/api/manager/court-schedule/rentals?roomId&fromUtc&toUtc` | Manager/Receptionist qua `FrontDesk`; chỉ metadata thuê cần cho lịch, không roster Member. |
| POST | `/api/manager/court-rentals/{rentalId}/cancel` | CenterManager; body `{reason}`; hủy lượt thuê do trung tâm và release occupancy nguyên tử. **Chỉ gửi thông báo (chuông + email) cho Member, không cộng điểm lúc hủy**; Member chọn cách hoàn tại `api/refunds/center-cancellations` (Phần H). |

**Cửa sổ thuê sân (25/10/2026; đã triển khai ở backend, chưa có test):** Member tự thuê và Receptionist đặt hộ (`targetMemberId`) chỉ thuê trong khung 06:00 đến 22:00 giờ Việt Nam (lượt thuê phải nằm trọn trong khung); ngoài khung trả 409 `court_outside_hours`. Calendar, availability và checkout chỉ nhận ngày trong 3 ngày lịch (hôm nay và 2 ngày kế tiếp); cửa sổ chuyển ngày lúc 06:00 (từ 06:00 ngày 26: 26, 27, 28; từ 06:00 ngày 27: 27, 28, 29; trước 06:00 vẫn tính theo ngày hôm trước). Ngày ngoài cửa sổ trả 409 `court_outside_window` (checkout) hoặc 400 `invalid_rental_window` (calendar). `GET /api/court-rentals/policy` trả `advanceDays = 2`; calendar chỉ trả slot 06:00 đến 22:00. Giới hạn này thay `advanceDays` 30 ngày và không áp dụng cho Lịch sân của CenterManager và Receptionist (`/api/manager/court-schedule/rentals`).

Invoice checkout giữ phòng bằng cùng occupancy constraints với class/block. Hết hạn/hủy nhả đúng một lần; IPN muộn không Spend lại điểm đã release, chỉ reacquire khi slot còn hợp lệ; nếu không thì theo cơ chế bồi hoàn khoản cash đã xác minh. Migration và PostgreSQL concurrency/ownership đã qua gate; chưa áp migration trên DB phát triển/chia sẻ.

Job trạng thái chạy mỗi phút: lượt `InUse` hết giờ thành `Completed`, lượt `Confirmed` chưa nhận sân hết giờ thành `NoShow`. Cả hai đều nhả occupancy; `NoShow` không tự hoàn tiền.

## Phần A — Route baseline (trước refactor)

### AuditLogsController — `api/audit-logs`

| Verb | Path |
|---|---|
| GET | `api/audit-logs` |
| GET | `api/audit-logs/actions` |

`GET api/audit-logs/actions` trả mảng mã Action khác nhau, bỏ mã rỗng/chỉ khoảng trắng và sắp xếp tăng dần, không phụ thuộc phân trang của bảng. Chỉ CenterManager/SystemAdministrator được gọi. Admin nhận đầy đủ danh mục `AccountAuditActions`, kể cả mã chưa phát sinh log, cộng các mã đã ghi trong phạm vi Manager/Admin/Coach/System. Manager nhận các mã đã ghi trong phạm vi Manager/Receptionist/Coach/Member. Chọn Action chưa có log trả bảng rỗng hợp lệ; danh mục không giới hạn vào `UserAccount`, không tạo log giả và không mở rộng quyền đọc.

Danh mục Admin hiện có: `CREATE_STAFF_ACCOUNT`, `CHANGE_USER_ROLE`, `LOCK_USER_ACCOUNT`, `UNLOCK_USER_ACCOUNT`, `RESEND_STAFF_ACTIVATION`, `CHANGE_PENDING_STAFF_EMAIL`, `FRONT_DESK_CREATE_MEMBER`, `RESEND_MEMBER_ACTIVATION`, `CHANGE_PENDING_MEMBER_EMAIL`, `EXPIRE_UNACTIVATED_STAFF_ACCOUNT`, `UPDATE_COACH_SPECIALTIES`.

Ô Action mở danh sách cuộn khi focus/click, hỗ trợ gõ để thu hẹp gợi ý không phân biệt hoa/thường (nhiều từ có thể nhập khác thứ tự), chọn bằng chuột hoặc ArrowUp/ArrowDown + Enter, Escape/Tab đóng danh sách. Chọn mã rồi Apply filters gửi nguyên mã đó; backend vẫn lọc Action bằng so sánh chính xác. All actions/ô trống không lọc Action. Danh sách có trạng thái tải, lỗi/thử lại và rỗng; quyền đọc audit không được mở rộng.

Với sự kiện `targetEntity = UserAccount`, mỗi item bổ sung `targetFullName`,
`targetEmail` (thông tin hiện tại của tài khoản đích) và `targetAccountExists`.
Các trường này không phải snapshot tại thời điểm sự kiện. Tài khoản không còn
tồn tại trả `targetAccountExists = false`, tên/email null; sự kiện trên entity
khác trả cả ba trường null. `targetId` vẫn được giữ để đối chiếu.
Việc bổ sung không thay đổi quyền đọc audit theo role tại thời điểm sự kiện hay policy G10.

### ReportExportsController — `api/reports/exports`

| Verb | Path |
|---|---|
| GET | `api/reports/exports` |
| POST | `api/reports/exports` |
| GET | `api/reports/exports/{reportExportId:guid}/download` |
| POST | `api/reports/exports/{reportExportId:guid}/retry` |
| DELETE | `api/reports/exports/{reportExportId:guid}` |

### SystemSettingsController — `api/system-settings`

| Verb | Path |
|---|---|
| GET | `api/system-settings` |
| PUT | `api/system-settings/{key}` |

### UsersController — `api/users`

| Verb | Path |
|---|---|
| GET | `api/users` |
| GET | `api/users/admin` |
| GET | `api/users/{userId:guid}` |
| POST | `api/users/admin/staff` |
| POST | `api/users/{userId:guid}/resend-activation` |
| PUT | `api/users/{userId:guid}/email` |
| PUT | `api/users/{userId:guid}/role` |
| POST | `api/users/{userId:guid}/activate` |
| POST | `api/users/{userId:guid}/deactivate` |
| POST | `api/users/{userId:guid}/lock` (alias tương thích của `/deactivate`) |
| POST | `api/users/{userId:guid}/unlock` (alias tương thích của `/activate`) |

Activate/Deactivate là thao tác chỉ dành cho SystemAdministrator với body `{reason}` (3–500 ký tự, bắt buộc). Alias tương thích dùng cùng kiểm tra lý do, phân quyền, bảo vệ tự vô hiệu hóa/Admin Active cuối cùng, hành động audit và vô hiệu phiên; không tạo trạng thái thứ ba. Client mới dùng route chuẩn. Lặp lại trạng thái hiện tại không thay đổi trạng thái và sự kiện. Activate thành công chỉ thay đổi trạng thái truy cập; yêu cầu lời mời/mật khẩu đang chờ vẫn áp dụng.

`POST api/users/admin/staff` (SystemAdministrator; `{email, fullName, phone?, roleId, specialties?}`): tạo tài khoản staff **không có mật khẩu**, gửi email kích hoạt (link dùng một lần, 24 giờ). Với Coach (`roleId` 2) bắt buộc `specialties` chứa **ít nhất một môn đang hoạt động**; **một Coach có thể được gán nhiều môn ngay khi tạo tài khoản**. Role khác không được gửi `specialties`. **Manager không tạo được Coach** (không còn `POST api/manager/coaches`): Manager chỉ sửa hồ sơ/chuyên môn (`PUT api/manager/coaches/{id}`).

**Đã đồng bộ:** `UserAdminService.CreateStaffAsync` chấp nhận 1–10 môn khi tạo Coach (không còn trả `coach_single_specialty_required`). Mỗi môn phải đang hoạt động và có dịch vụ `GROUP_COURSE` đang bật; môn Gym (chỉ `MEMBERSHIP_ACCESS`) hoặc môn chỉ thuê sân bị từ chối `400 sport_not_coachable`. Môn thêm sau này tự đủ điều kiện khi bật lớp nhóm, không cần sửa mã. UI chỉ liệt kê các môn đủ điều kiện.

`UserAdminResponse.activationPending` (bool): tài khoản (staff hoặc Member đăng ký tại quầy) chưa đặt mật khẩu và chưa liên kết Google. Hai thao tác dưới đây chỉ dành cho SystemAdministrator và chỉ áp dụng cho tài khoản `activationPending`; Member hoặc staff đã kích hoạt trả **409 `account_not_pending_activation`**.

- `POST api/users/{userId}/resend-activation` (không body): phát hành link mới, vẫn hết hạn tại `CreatedAt + 24 giờ`, vô hiệu link cũ; còn trong 60 giây kể từ lần gửi trước trả **429 `activation_resend_too_soon`**; ghi audit `RESEND_STAFF_ACTIVATION`.
- `PUT api/users/{userId}/email` (`{email, reason}`, lý do 3–500 ký tự): sửa email gõ sai. Email trùng (không phân biệt hoa thường) trả 409 `email_already_exists`, email không đổi trả 400 `email_unchanged`. Link gửi tới email cũ bị vô hiệu ngay (người nhận nhầm không kích hoạt được), link mới gửi tới email mới; ghi audit `CHANGE_PENDING_STAFF_EMAIL` kèm email cũ/mới và lý do.

Đăng nhập Google với email của staff chưa kích hoạt bị từ chối (401), vì staff chỉ vào được sau khi tự đặt mật khẩu bằng link.

`PUT api/users/{userId}/role` (SystemAdministrator; body `{role, reason, sportIds?}`; `sportIds` bắt buộc khi đổi sang Coach). Chỉ đổi **giữa các role staff** (SystemAdministrator, CenterManager, Receptionist, Coach).

Khi tài khoản đã là Coach và `role` vẫn là `COACH`, gửi `sportIds` để lưu chuyên môn (1–10 môn hợp lệ, có GroupCourse). Ghi audit `UPDATE_COACH_SPECIALTIES` kèm lý do và danh sách trước/sau; giữ nguyên phiên đăng nhập và lớp đã phân công, giống cập nhật chuyên môn của Manager. Nếu không gửi `sportIds` hoặc danh sách không đổi thì không ghi thay đổi.

- **Member ↔ Staff không được phép:** đổi từ Member sang bất kỳ role staff nào, hoặc từ staff về Member, trả **409 `member_staff_role_change_not_allowed`**. Người muốn làm staff phải được Admin tạo tài khoản staff bằng **email khác** (`POST api/users/admin/staff`, luồng kích hoạt qua email); tài khoản Member cũ giữ nguyên.
- **Coach → role staff khác:** trả **409 `role_change_blocked`** kèm `reasons: string[]` khi còn lớp chưa kết thúc do Coach làm chính, buổi lớp tương lai còn chiếm lịch (`CoachOccupancy`), hoặc quan hệ huấn luyện Active.
- **Mọi tài khoản hiện là SystemAdministrator:** không được đổi sang role khác, gồm chính Admin đang thao tác và Admin khác, không phụ thuộc số lượng Admin hoặc trạng thái tài khoản (409 `administrator_role_change_not_allowed`). Chỉ gửi lại role hiện tại thì không thay đổi dữ liệu. Quy tắc chặn tự Deactivate và bảo vệ Admin Active cuối cùng vẫn được kiểm riêng.

Khi đổi thành công: rời Coach thì giữ `CoachProfile` làm lịch sử nhưng xóa chuyên môn (`UserSportSpecialty`); đổi sang Coach thì Admin phải chọn chuyên môn; `SecurityStamp` đổi nên token cũ mất hiệu lực.

### Manager request inbox and unread badges

Navbar Manager hiển thị cùng badge yêu cầu chưa đọc ở **Exception** và **Requests**, trên desktop và mobile. `GET /api/manager/requests/summary` giữ tổng đang chờ `coach`, `refunds`, `support` và thêm `unread`: yêu cầu Coach đang chờ, hoàn trả Requested và ticket hỗ trợ còn mở chưa có xác nhận đọc của Manager đang xác thực. Ticket hỗ trợ Done, Rejected và Closed bị loại. Giá trị 0 ẩn badge; trên 99 hiển thị `99+`. Làm mới mỗi 60 giây, khi cửa sổ được focus và sau khi đọc/thao tác yêu cầu.

`POST /api/manager/requests/{type}/{id}/read` (chỉ Manager; type `coach`, `refunds` hoặc `support`) lưu xác nhận đọc riêng từng Manager có tính idempotent, trả 204; type sai hoặc yêu cầu không tồn tại trả 404. Mở drawer Coach hoặc chi tiết hỗ trợ đánh dấu đã đọc. Hoàn trả hiển thị thẻ chi tiết đầy đủ, nên được đánh dấu đọc khi thẻ vào viewport. Chỉ mở menu hoặc danh sách yêu cầu Coach không đánh dấu yêu cầu đã đọc. Xác nhận đọc không đổi trạng thái nghiệp vụ. Migration `20261024100000_ManagerRequestReadReceipts` thêm `manager_request_reads`; yêu cầu đang mở hiện có ban đầu chưa có xác nhận đọc.

### Coach coverage and lead-coach changes

Khi tài khoản Coach trở thành Deactivated, mỗi lần chuyển trạng thái thực tế đưa một thông báo `COACH_COVERAGE_REQUIRED` riêng vào hàng đợi cho các Manager Active. `actionUrl` mở trực tiếp `/manager/coaches/{coachId}?tab=overview`. Retry không gây thay đổi không đưa thêm thông báo vào hàng đợi. ID chuyển trạng thái được tra qua bản ghi audit tài khoản; thông báo cũ có nguồn là ID Coach cũng mở cùng hồ sơ.

Overview hồ sơ Coach hiển thị bảng khẩn cấp viền đỏ với nền tông ấm. Dòng **Lead coach** biểu thị lớp chưa kết thúc; dòng **Substitute coach** biểu thị buổi Scheduled tương lai trong lớp do Coach khác chủ nhiệm. Các dòng buổi thuộc phân công chủ nhiệm đã được bao phủ bởi dòng lớp và không lặp lại. Buổi đã hoàn thành, hủy hoặc bắt đầu không thể phân công lại bằng workflow này.

| Verb | Path | Behavior |
|---|---|---|
| GET | `/api/manager/coach-coverage` | Hàng đợi chỉ Manager được đọc. Trả `mainClasses[]` và `sessions[]`; sessions có `sessionId,classId,startAtUtc,endAtUtc,sessionNo,roomId,roomName,coachId,isSubstitute`. |
| GET | `/api/manager/classes/{classId}/lead-coach/candidates` | Chỉ Manager. Trả `{version,currentCoachId,remainingSessionCount,coaches:[{coachId,fullName}]}`. Ứng viên là Coach Active khác, đúng chuyên môn và không xung đột occupancy trong **mọi** buổi Scheduled tương lai. |
| POST | `/api/manager/classes/{classId}/lead-coach` | `{coachId,expectedVersion,reason}`; lý do 3–500 ký tự. Chỉ lớp Published/InProgress. Kiểm tra lại điều kiện và lịch trống, rồi đổi Coach chủ nhiệm cùng mọi buổi Scheduled tương lai một cách nguyên tử. Giữ thời gian, phòng, ghi danh và lịch sử giảng dạy; cập nhật occupancy, audit, thông báo, quan hệ huấn luyện và yêu cầu đổi buổi Pending qua workflow lịch dùng chung. Trả 204. |
| GET | `/api/availability/coaches` | Với buổi dạy thay: query `sportId,startUtc,endUtc,excludeSessionId`. Trả các Coach đủ chuyên môn, còn trống cho buổi đó. |
| POST | `/api/class-sessions/{sessionId}/reschedule` | Với buổi dạy thay: `{startAtUtc,coachId,reason}`, giữ thời điểm bắt đầu gốc. Chỉ thay Coach của buổi, giữ Coach chủ nhiệm và các buổi khác. Kiểm tra lại lịch trống khi xác nhận. |

`POST /api/manager/coach-coverage/{coachId}/classes/{classId}/reassign-main` đã được **gỡ bỏ**. Cả phân công lại lớp thông thường và khi Coach Deactivated hiện dùng API Coach chủ nhiệm chung. Lớp Draft dùng trình sửa lớp hiện có. Không có endpoint duyệt yêu cầu Member tự động đổi Coach.

### FrontDeskMembersController — `api/members/front-desk`

Receptionist và CenterManager (policy `FrontDesk`) đăng ký Member mới tại quầy. Chỉ tạo role **Member** (role không nhận từ body); staff chỉ do SystemAdministrator tạo.

| Verb | Path |
|---|---|
| POST | `api/members/front-desk` |
| POST | `api/members/front-desk/{userId:guid}/resend-activation` |
| PUT | `api/members/front-desk/{userId:guid}/email` |

- `POST api/members/front-desk` (`{email, fullName, phone?}`): email nhập **một lần** (không còn `confirmEmail`; gõ sai thì dùng luồng sửa email bên dưới). Tạo Member **không mật khẩu**, tạo ví điểm 0 (BR-83), gửi email kích hoạt (link dùng một lần, 24 giờ, `setup=member`), audit `FRONT_DESK_CREATE_MEMBER`. Trả 201 `UserAdminResponse` với `activationPending=true`. Email trùng (không phân biệt hoa thường) 409 `email_already_exists`; số điện thoại trùng 409 `phone_already_exists`.
- `POST api/members/front-desk/{userId}/resend-activation`: chỉ Member chờ kích hoạt. Cooldown 60 giây (429 `activation_resend_too_soon`), link cũ bị vô hiệu, audit `RESEND_MEMBER_ACTIVATION`.
- `PUT api/members/front-desk/{userId}/email` (`{email, reason}`): sửa email gõ sai khi Member chờ kích hoạt (giống Admin sửa email staff chờ kích hoạt); link cũ bị vô hiệu ngay, link mới gửi tới email mới, audit `CHANGE_PENDING_MEMBER_EMAIL` (email cũ, mới, lý do). 400 `email_unchanged`, 409 `email_already_exists`.
- Member chờ kích hoạt quá **7 ngày kể từ lúc đăng ký** (không có mật khẩu, không liên kết Google) bị gỡ khỏi hệ thống, email được giải phóng, audit `EXPIRE_UNACTIVATED_MEMBER_ACCOUNT` (actor `SYSTEM`, giữ bản ghi đăng ký). Sửa email không kéo dài 7 ngày. Tài khoản đã có hóa đơn, ghi danh, thuê sân, gói hoặc giao dịch điểm **không** bị gỡ vì dữ liệu đó thuộc trung tâm; tài khoản đó tiếp tục chờ kích hoạt. Job `MemberInvitationExpiryJob` chạy mỗi 10 phút. Link kích hoạt vẫn hết hạn sau 24 giờ; hết link thì lễ tân gửi lại.
- Member đã kích hoạt trả 409 `account_not_pending_activation`; tài khoản không phải Member trả 404 `user_not_found`.
- Trước khi kích hoạt Member chưa đăng nhập được bằng mật khẩu nhưng tài khoản Active, nên quầy vẫn tìm (`GET api/users?role=Member`), ghi nhận thanh toán và bán dịch vụ. Member kích hoạt bằng link (đặt mật khẩu qua `POST api/auth/password/reset`) hoặc đăng nhập Google bằng chính email đó.

### Vé Gym 1 ngày (day pass) và mã check-in

- Vé ngày là gói Membership `packageId = 50000`, 1 ngày, seed mặc định 50.000đ bằng migration `GymDayPass`. Mua qua `POST api/checkouts/membership` như gói thường. Manager sửa được giá/tên/mô tả qua `PUT api/membership-packages/50000`; đổi thời hạn trả 409 `day_pass_duration_fixed`, ngừng bán trả 409 `day_pass_fixed`.
- Hiệu lực (quyết định 24/10/2026): chỉ bán từ 06:00 đến trước 22:00 giờ Việt Nam, ngoài khung trả 409 `day_pass_outside_hours`; hết hạn 22:00 cùng ngày mua (`expiresAtUtc` trong `MemberPackageResponse`; `endDate` chỉ là ngày giờ Việt Nam của mốc này; check-in, `isUsable` và job hết hạn đều xét `expiresAtUtc`; không cộng dồn với gói dài). Đã có vé hôm nay mà mua thêm trả 409 `day_pass_already_active`. Gym đang đóng cửa vì sự cố (mọi phòng Gym bị block) trả 409 `gym_closed_today`; check-in lúc Gym đóng trả 409 `gym_closed` kể cả với người đã mua vé, vé vẫn hết hạn 22:00 và không được gia hạn hay hoàn điểm qua luồng hoàn hiện có (quyết định 25/10/2026).
- Thanh toán xong: thông báo và email "SportHub - Your Gym day pass" nhắc vé hết hạn lúc 22:00 cùng ngày. Trong 60 phút cuối trước 22:00 checkout ghi chú Gym đóng cửa lúc 22:00 (cờ `closingSoon`, `closesAtUtc`). Thanh toán thành công sau 22:00 không cấp vé (đi theo `RECONCILIATION_REQUIRED`).
- Check-in: chỉ lễ tân thực hiện qua `POST api/gym-checkins` (Member không tự check-in; endpoint `api/gym-checkins/self` đã gỡ). Mã QR do server cấp và ký: Member gọi `GET api/members/me/checkin-code` (`{code, expiresAtUtc}`, mã dạng `SPORTHUB-CODE:{memberId}.{exp}.{chữ ký}`, hết hạn 00:00 giờ Việt Nam hôm sau, client không tự dựng được); lễ tân/Manager (policy `FrontDesk`) quét mã rồi gọi `POST api/members/checkin-code/resolve` (`{code}`) nhận `{memberId}`; mã giả/sai chữ ký trả 400 `checkin_code_invalid`, mã hết hạn 409 `checkin_code_expired`. Mã chỉ định danh Member; quyền vào Gym vẫn do check-in kiểm gói Active hôm nay.
- Không nhắc hết hạn (`PackageExpiring`), không gia hạn khi sự cố đóng Gym, không hoàn trả (`RefundCalculator` trả 0).
- Báo cáo: `GET api/reports/membership-summary` và `membership-period` không tính vé ngày vào hội viên Membership; thêm `membersWithDayPassToday` và `dayPassesSold`.

### AiController — `api/ai`

| Verb | Path |
|---|---|
| POST | `api/ai/workout-suggestions/{memberId:guid}` |
| POST | `api/ai/chat` |
| GET | `api/ai/logs` |

Gợi ý bài tập có `items: [{ exercise, sets, reps, notes }]` với
sets/reps dạng số cho từng bài tập. `exercises: string[]` hiện có vẫn được giữ
cho client cũ. Bài tập theo thời gian dùng reps làm số lần giữ/vòng;
thời lượng đặt trong `notes` (ví dụ plank: 3 sets × 1 lần giữ, mỗi lần 30 giây).
Lời khuyên huấn luyện chung đặt trong `rationale`, không biến thành một bài tập.
Trình biên tập đọc trực tiếp `items` và cho Coach chỉnh giá trị trước khi lưu.
Query parameter `sport` tùy chọn chọn `Gym` (mặc định), `Badminton` hoặc
`Basketball`; giá trị không được hỗ trợ trả 400. Môn được chọn có trong
response và input AI log. Vẫn yêu cầu quyền của Coach (quan hệ Active với Member).
Mọi item được tạo có sets/reps dương, bao gồm phần khởi động
và thả lỏng. Gợi ý Coach lớp nhóm có các giá trị này trong từng dòng bài tập.

### HealthController — `api/[controller]`

| Verb | Path |
|---|---|
| GET | `api/[controller]` |

### Coach credentials and awards — `api/coaches`

- `GET /api/coaches/me/credentials` chỉ dành cho Coach, liệt kê tài liệu chưa xóa của Coach hiện tại, kể cả mục không hiển thị công khai. `POST /api/coaches/me/credentials` nhận multipart `file`, `kind` (`CERTIFICATE` hoặc `AWARD`), `title` bắt buộc (tối đa 200 ký tự), `issuer` tùy chọn (200), `issuedOn` ISO tùy chọn và `isPublished` (mặc định true). Chấp nhận JPG/PNG/WebP/PDF có chữ ký nội dung khớp, không rỗng và tối đa 10 MiB; mỗi Coach tối đa 20 tài liệu chưa xóa. Tải lên dùng `CLOUDINARY_URL` ở backend, giữ bản gốc và không dùng cắt avatar. Tài nguyên PDF dùng loại raw và có `.pdf` trong public ID. Cloudinary có thể hạn chế phân phối PDF trực tiếp qua CDN; thay vào đó ứng dụng đọc PDF gốc qua endpoint file bên dưới bằng truy xuất có xác thực phía server.
- `PUT /api/coaches/me/credentials/{credentialId}` nhận metadata cùng `version` được trả gần nhất. Sửa trên phiên bản cũ trả `409 credential_changed`. `DELETE` ẩn tài liệu ngay và xóa tài nguyên Cloudinary; xóa thất bại vẫn được xếp hàng trong bản ghi chứng chỉ cho job dọn dẹp mỗi mười phút. Chỉ chủ sở hữu được sửa/xóa; Coach khác nhận 404 và role không phải Coach không được vào endpoint tự quản lý. Thay đổi thành công được ghi audit. Nếu ghi database thất bại sau tải lên, hệ thống thử dọn tài nguyên không dùng.
- `GET /api/coaches/{coachId}/credentials` không yêu cầu xác thực, chỉ trả tài liệu đã xuất bản, chưa xóa của Coach hiện đang Active. Hồ sơ không tồn tại, không hoạt động hoặc không phải Coach trả 404. Item response chứa `credentialId,kind,title,issuer,issuedOn,fileName,fileUrl,contentType,sizeBytes,isPublished,version,createdAtUtc` cùng `contentUrl` có thể null: đường dẫn file tương đối của ứng dụng cho PDF, null cho ảnh. Không trả secret Cloudinary API, URL tải có chữ ký của provider hoặc `filePublicId` nội bộ. Link tài liệu công khai được dùng để chia sẻ; tắt hiển thị loại tài liệu khỏi hồ sơ, không xóa bản sao URL công khai mà khách trước đó đã có. Đây là tài liệu do Coach cung cấp, không có trạng thái hoặc workflow xác minh.
- `GET /api/coaches/{coachId}/credentials/{credentialId}/file` trả byte PDF gốc với `Content-Type: application/pdf`, tên file hiển thị inline, `X-Content-Type-Options: nosniff`, `Cache-Control: private, no-store` và hỗ trợ byte-range. Khách không xác thực chỉ đọc PDF đã xuất bản, chưa xóa của Coach Active; Coach chủ sở hữu đã xác thực còn có thể đọc PDF chưa xuất bản của mình. Chủ sở hữu khác, tài liệu đã xóa, không phải PDF và hồ sơ không hoạt động/không phải Coach trả 404. Backend chỉ truy xuất ID tài liệu đã lưu qua API tải raw có xác thực của Cloudinary, bằng request có chữ ký hiệu lực 120 giây; thông tin xác thực provider ở phía server. Lượt đọc kiểm tra chữ ký PDF, giới hạn 10 MiB và deadline 120 giây. Cache riêng trong tiến trình giữ byte PDF tối đa năm phút trong giới hạn 32 MiB, gộp các lượt đọc đồng thời cùng file và được nạp sau khi tải PDF lên. Xóa tài liệu làm mất hiệu lực mục cache; mỗi lượt đọc API vẫn kiểm tra trạng thái xuất bản, chủ sở hữu và trạng thái Coach hiện tại trước khi truy cập byte cache. Hủy một request trình duyệt chỉ hủy lượt chờ đó, cho phép request tiếp theo dùng lại lượt đọc đang xử lý có giới hạn. Bản ghi PDF hiện có hoạt động mà không cần migration hoặc tải lại.
- Hộp thoại PDF trong tài khoản và hồ sơ công khai Coach tải `contentUrl` với quyền xác thực của chủ sở hữu khi có, rồi dùng PDF Blob local cho trình xem nhiều trang tích hợp, **Open original** và **Download PDF**. Trình xem hiển thị trạng thái tải, lỗi và Retry, hủy tải khi đóng và giải phóng Blob. Trình duyệt không hỗ trợ PDF nhúng có thể mở hoặc tải bản gốc. Response API cũ không có `contentUrl` giữ cách tải URL trực tiếp không xác thực; thông tin xác thực không bao giờ được chuyển tới Cloudinary.
- `/api/coaches` thêm `credentialCount` chỉ đếm tài liệu hiển thị công khai. Hồ sơ tài khoản hỗ trợ tải lên, sửa, ẩn và xóa; hồ sơ Coach công khai hiển thị thư viện tài liệu với chế độ xem ảnh không cắt và link file gốc. Bản xem trước khi hover Coach liên kết tới thư viện khi có tài liệu.
- Migration `20261023162010_CoachCredentials` thêm bảng `coach_credentials`, FK chủ sở hữu, chỉ mục ID file, trạng thái xuất bản, phiên bản sửa và timestamp dọn tài nguyên bền vững. Không thay đổi hồ sơ Coach hiện có hoặc tạo thành tích mẫu.

### Avatar Cloudinary — `api/avatars`

Quy tắc nghiệp vụ, quyền xem, xóa ảnh cũ và Google avatar: [AvatarsController](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.Identity/Api/AvatarsController.cs). `GET /api/users/me`, response đăng nhập và danh sách Coach có `avatarUrl` nullable. Member/Coach/Center Manager/System Administrator tự tải multipart `file` qua `POST /api/avatars/me` (JPG/PNG/WebP, tối đa 5 MiB) và xóa qua `DELETE /api/avatars/me`; chủ ảnh luôn lấy từ người dùng đã xác thực. `GET /api/avatars/{userId}` chỉ công khai avatar Coach; API avatar Member/Manager/Admin yêu cầu đăng nhập. Admin quản lý ảnh cá nhân tại Account bằng cùng trình cắt ảnh Cloudinary; thay/xóa ảnh cập nhật avatar trên header, ghi audit và dọn ảnh cũ theo luồng hiện có, không cần migration mới. Center Manager gỡ trực tiếp ảnh hiện tại của Member qua `DELETE /api/manager/members/{memberId}/avatar`; hệ thống ghi audit và xếp tác vụ dọn ảnh cũ trên Cloudinary. Endpoint này không áp dụng cho ảnh Coach.

### AccountController — `api/users/me`

| Verb | Path |
|---|---|
| GET | `api/users/me` |
| PUT | `api/users/me/profile` |
| POST | `api/users/me/password/otp` |
| POST | `api/users/me/password` |

### AuthController — `api/auth`

| Verb | Path |
|---|---|
| POST | `api/auth/register/otp` |
| POST | `api/auth/register` |
| POST | `api/auth/login` |

### GoogleAuthController — `api/auth`

| Verb | Path |
|---|---|
| POST | `api/auth/google` |
| POST | `api/auth/google/link` |

### MemberPackagesController — `api`

| Verb | Path |
|---|---|
| GET | `api/members/me/packages` |
| GET | `api/members/{memberId:guid}/packages` |
| GET | `api/member-packages` |

### MembershipPackagesController — `api/membership-packages`

| Verb | Path |
|---|---|
| GET | `api/membership-packages` |
| POST | `api/membership-packages` |
| PUT | `api/membership-packages/{packageId:int}` |
| POST | `api/membership-packages/{packageId:int}/discontinue` |
| POST | `api/membership-packages/{packageId:int}/reactivate` |

### MembershipReportsController — `api/reports`

| Verb | Path |
|---|---|
| GET | `api/reports/membership-summary` |

### TrainingProfilesController — `api`

| Verb | Path |
|---|---|
| GET | `api/members/me/training-profile` |
| PUT | `api/members/me/training-profile` |
| GET | `api/members/{memberId:guid}/training-profile` |

### NotificationsController — `api/notifications`

| Verb | Path |
|---|---|
| GET | `api/notifications` |
| GET | `api/notifications/inbox?unreadOnly=false&search=&page=1&pageSize=25` |
| GET | `api/notifications/{notificationId:guid}` |
| GET | `api/notifications/unread-count` |
| POST | `api/notifications/{notificationId:guid}/read` |
| POST | `api/notifications/read-all` |

Hộp thư và chi tiết khả dụng với mọi role đã xác thực, chỉ trả thông báo InApp của người dùng hiện tại; payload email/OTP không bao giờ bị lộ. Hộp thư trả `{items,totalCount,page,pageSize}`, mới nhất trước; `page` từ 1–100000, `pageSize` từ 1–50 và `search` tối đa 150 ký tự. Tìm kiếm khớp nội dung thông báo và tiêu đề thông báo thủ công được liên kết. Endpoint danh sách gốc giữ hành vi lấy 100 mục mới nhất cho xem trước ở chuông. DTO thông báo thêm `title` và `senderName` có thể null từ bản ghi tại thời điểm gửi của thông báo thủ công được tham chiếu; sự kiện hệ thống dùng nhãn sự kiện của UI và người gửi SportHub. Chi tiết trả 404 cho ID không tồn tại, thuộc người khác hoặc không phải InApp. Đọc chi tiết không thay đổi trạng thái; UI gửi lệnh đọc hiện có sau khi mở.

Chuông mở `/notifications?message=<notificationId>` cho mọi thông báo, kể cả không có `actionUrl`. Hộp thư trình bày toàn bộ văn bản trước link hành động liên quan riêng, giới hạn theo workspace của role người nhận. Trạng thái chưa đọc/đã đọc được lưu qua các lệnh hiện có và đồng bộ với chuông; xác nhận đọc thất bại vẫn cho đọc thông báo và cung cấp retry. Back/Forward trình duyệt khôi phục thông báo đang chọn, bộ lọc, tìm kiếm và trang. Không cần migration.

`COACH_ATTENDANCE_REMINDER` là tổng kết InApp cuối ngày cho Coach Active còn thiếu điểm danh trong buổi đã kết thúc và còn được phép thao tác. Worker chạy mỗi phút đưa tối đa một thông báo mỗi Coach/ngày Việt Nam vào hàng đợi sau 22:00 theo mặc định (`CoachAttendanceReminders:HourLocal`). Có thể xử lý bù ngày hôm qua sau gián đoạn, chỉ trong cửa sổ điểm danh 24 giờ của từng buổi. Thông báo liệt kê lớp/buổi bị ảnh hưởng, số mục thiếu và deadline; `actionUrl` là `/coach/schedule` và hành động trong hộp thư là **Complete attendance**. Xác nhận thông báo hiện có, chỉ mục duy nhất event/recipient/source và khóa producer ngăn thông báo lặp khi retry/restart/nhiều replica. Không bao giờ tự ghi vắng hoặc gửi email; không cần migration hoặc endpoint công khai mới.

### InvoicesController — `api`

| Verb | Path |
|---|---|
| GET | `api/invoices` |
| GET | `api/members/me/invoices` |
| GET | `api/invoices/{invoiceId:guid}` |
| POST | `api/member-packages/purchase` |
| POST | `api/invoices/{invoiceId:guid}/payments` |
| POST | `api/invoices/{invoiceId:guid}/adjustments` |

### PaymentAdjustmentsController — `api/payment-adjustments`

| Verb | Path |
|---|---|
| GET | `api/payment-adjustments` |
| POST | `api/payment-adjustments/{adjustmentId:guid}/approve` |
| POST | `api/payment-adjustments/{adjustmentId:guid}/reject` |

Refund mới không còn tạo/duyệt qua route adjustment chung; refund legacy vẫn được đọc để đối soát. Không còn endpoint xác nhận payout tiền.

### RevenueReportsController — `api/reports`

| Verb | Path |
|---|---|
| GET | `api/reports/revenue` |

### AttendanceController — `api/attendance`

| Verb | Path |
|---|---|
| POST | `api/attendance/{enrollmentId:guid}` |
| GET | `api/attendance/sessions/{sessionId:guid}` |

### ClassesController — `api/classes`

| Verb | Path |
|---|---|
| GET | `api/classes` |
| GET | `api/classes/{classId:int}` |
| POST | `api/classes` |
| PUT | `api/classes/{classId:int}` |
| POST | `api/classes/{classId:int}/archive` |
| POST | `api/classes/{classId:int}/reactivate` |
| POST | `api/classes/{classId:int}/recurrences` |
| DELETE | `api/classes/{classId:int}/recurrences/{recurrenceId:int}` |
| POST | `api/classes/{classId:int}/generate-sessions` |

### ClassSessionsController — `api/class-sessions`

| Verb | Path |
|---|---|
| GET | `api/class-sessions` |
| GET | `api/class-sessions/mine` |
| GET | `api/class-sessions/{sessionId:guid}` |
| GET | `api/class-sessions/{sessionId:guid}/roster` |
| POST | `api/class-sessions` |
| PUT | `api/class-sessions/{sessionId:guid}` |
| POST | `api/class-sessions/{sessionId:guid}/cancel` |
| POST | `api/class-sessions/{sessionId:guid}/reschedule` |

### ClassUtilizationReportsController — `api/reports`

| Verb | Path |
|---|---|
| GET | `api/reports/class-utilization` |

Đăng ký được mở tự động trong transaction xuất bản. Ghi danh đã trả tiền và được xác nhận cuối cùng làm lớp đóng khi đủ sức chứa. Rút ghi danh mở lại lớp từng tự đóng do đầy chỉ khi buổi đầu chưa bắt đầu. Hold checkout còn hiệu lực vẫn được tính vào số chỗ khả dụng, và nhả hoặc hết hạn hold không cần Manager thao tác. UI Manager không còn cung cấp điều khiển ghi danh thủ công. Các lớp đã xuất bản, sắp diễn ra hiện có được chuẩn hóa bằng migration dữ liệu AutomaticClassEnrollment.

### EnrollmentsController — `api`

| Verb | Path |
|---|---|
| POST | `api/enrollments` |
| POST | `api/enrollments/{enrollmentId:guid}/cancel` |
| GET | `api/members/me/enrollments` |
| GET | `api/members/me/schedule` |
| GET | `api/members/{memberId:guid}/enrollments` |

### GymCheckInsController — `api/gym-checkins`

| Verb | Path |
|---|---|
| POST | `api/gym-checkins` |

### MemberGymCheckInsController — `api/members`

| Verb | Path |
|---|---|
| GET | `api/members/me/gym-checkins` |
| GET | `api/members/{memberId:guid}/gym-checkins` |

### MemberCheckInCodesController — `api/members`

| Verb | Path |
|---|---|
| GET | `api/members/me/checkin-code` |
| POST | `api/members/checkin-code/resolve` |

### RoomsController — `api/rooms`

| Verb | Path |
|---|---|
| GET | `api/rooms` |
| POST | `api/rooms` |
| PUT | `api/rooms/{roomId:int}` |
| DELETE | `api/rooms/{roomId:int}` |

### CoachMemberRelationshipsController — `api/coach-member-relationships`

| Verb | Path |
|---|---|
| GET | `api/coach-member-relationships` |
| POST | `api/coach-member-relationships` |
| POST | `api/coach-member-relationships/{relationshipId:guid}/end` |


## Phần B — Contract đích theo chặng

| Chặng | Trạng thái | Ghi chú |
|---|---|---|
| P1.01–P1.02 Ports/model/migration | Hoàn tất backend | Typed FK và snapshots; giữ lịch sử, nâng cấp và DB trắng qua gate. |
| P1.03–P1.05 Identity/catalog/course/Gym | Hoàn tất backend | Phần C/D/F; JSON enum theo quy ước chốt ở đầu tài liệu. |
| P1.06–P1.08 Wallet/checkout/refund | Hoàn tất backend | Phần E/G/H; sandbox VNPay thật do người dùng phụ trách. |
| P1.09 Threshold/transfer | Hoàn tất backend | Phần I, expiry/transfer/late payment integration đã chạy. |
| P1.10–P1.11 Rental/calendar/incident/outbox | Hoàn tất backend | Bản chốt ở đầu tài liệu và Phần J. |
| P1.12 Báo cáo/seed/config | Hoàn tất backend | Xem phần Contract báo cáo và tích hợp. |
| P1.13 Kiểm thử | 465/465 pass | Xem evidence cuối; frontend và dịch vụ bên ngoài chưa thuộc chứng nhận này. |

AI (`api/ai/*`) chỉ sửa tối thiểu để build.

## Phần C — Route mới đã triển khai (P1.03)

| Verb | Path | Actor | Request | Response / lỗi chính |
|---|---|---|---|---|
| POST | `api/auth/password/forgot` | ẩn danh, 3/phút/IP | `{email}` | 204 luôn (trung tính; có tài khoản Active thì gửi email chứa link `/reset-password?email=&token=`) |
| POST | `api/auth/password/reset` | ẩn danh, 3/phút/IP | `{email, token, newPassword, confirmNewPassword}` | 204; 400 `otp_invalid` `otp_expired` `otp_already_used` `otp_attempts_exceeded` `password_*` |
| POST | `api/users/me/password` | đã đăng nhập | `{currentPassword?, newPassword, confirmNewPassword}` | 200 `{accessToken}`; 400 `current_password_required` `new_password_same_as_current` `password_*`; 401 `invalid_credentials` |

JWT có thêm claim `sst`; middleware từ chối token thiếu/sai `sst` hoặc sai role (401).
Ghi chú: `coachCategory` trong response là DEPRECATED.

## Phần D — Route mới đã triển khai (P1.04)

Actor viết tắt: **M** = CenterManager (policy `CatalogManage`), **FD** = Manager + Receptionist, **Staff** = Manager, Receptionist, Coach, **Auth** = mọi người đã đăng nhập.

| Verb | Path | Actor | Request | Response / lỗi chính |
|---|---|---|---|---|
| GET | `api/sports?service=` | ẩn danh | — | `SportResponse[]` chỉ môn active, mỗi môn chỉ gồm service đang bật, không có `readiness`. `service` lọc theo `MEMBERSHIP_ACCESS`, `GROUP_COURSE`, `COURT_RENTAL`. `SportResponse`: `{sportId, code, name, description, imageUrl, sortOrder, isActive, services[{serviceType,isEnabled,defaultSessionMinutes?,defaultMaxCapacity?}], readiness?[{serviceType,ready,missing[]}]}` với `missing` gồm `room_type`, `room`, `opening_hours`, `court_rate`. CAT-01: không còn `operationType`. |
| GET | `api/manager/sports?service=` | M | — | mọi môn, đủ service và `readiness` (chỉ GROUP_COURSE và COURT_RENTAL) |
| GET | `api/manager/sports/name-availability?name=&excludeSportId=` | M | Tên đang nhập; `excludeSportId` chỉ khi sửa | `{name,isValid,isAvailable}`; kiểm cả môn inactive và mã sinh từ tên khi tạo |
| POST | `api/manager/sports` | M | `{name, description?, imageUrl?, services[{serviceType,isEnabled,defaultSessionMinutes?,defaultMaxCapacity?}]}` | 201; tự sinh code từ tên (bỏ dấu, chữ thường, phân cách `_`, tối đa 32 ký tự), bỏ qua code do client cũ gửi; 400 `sport_name_invalid` `service_type_invalid` `service_not_allowed_for_sport` (Membership ngoài môn Gym) `sport_group_course_defaults_required` `service_defaults_not_allowed`; 409 `sport_code_taken` `sport_name_taken` — môn đã tồn tại, không tự thêm hậu tố |
| PUT | `api/manager/sports/{id}` | M | như trên, không nhận đổi `code`. Service không liệt kê bị TẮT, không bị xóa | 200; 400 `sport_code_immutable` và các lỗi như trên; 404 `sport_not_found` |
| POST | `api/manager/sports/{id}/deactivate`, `/activate` | M | — | `SportResponse` |
| POST | `api/manager/sports/{id}/services/{serviceType}/enable`, `/disable` | M | — | `SportResponse`; 404 `service_not_configured`. Tắt chỉ chặn giao dịch mới |
| GET | `api/room-types` | Auth | — | `[{roomTypeId, name, sportIds[]}]` |
| POST | `api/manager/room-types` | M | `{name}` | 201; 409 `room_type_name_taken` |
| PUT | `api/manager/room-types/{id}` | M | `{name}` | 200 |
| PUT | `api/manager/room-types/{id}/sports` | M | `{sportIds[]}` (thay toàn bộ) | 200; 400 `invalid_sport` |
| DELETE | `api/manager/room-types/{id}` | M | — | 204; 409 `room_type_in_use` |
| GET | `api/rooms` | Staff | — | `RoomResponse` thêm `roomTypeId`, `isActive` |
| POST, PUT | `api/rooms`, `api/rooms/{id}` | M | `{name, capacity, roomTypeId?, isActive?, allowRental?}`; không cần Sport; tạo mới mặc định không cho thuê | 400 `invalid_room_type`; 409 `room_name_taken` |
| PUT | `api/rooms/{id}/rental-access` | M | `{allowRental}`; cấu hình độc lập với Active, giữ lớp/booking hiện có; inactive vẫn không nhận booking | 404 `room_not_found` |
| DELETE | `api/rooms/{id}` | M | — | 204; 409 `room_in_use` |
| GET | `api/rooms/{roomId}/opening-hours` | Auth | — | `[{dayOfWeek(0=CN..6), openTimeLocal "HH:mm", closeTimeLocal}]` |
| PUT | `api/manager/rooms/{roomId}/opening-hours` | M | `{hours:[{dayOfWeek, openTimeLocal, closeTimeLocal}]}` (thay toàn bộ, ≤7) | 200; 400 `invalid_opening_hours` `duplicate_opening_day` `invalid_time` |
| GET | `api/manager/room-blocks?roomId&fromUtc&toUtc` | FD | — | `RoomBlockResponse[]`; chỉ `roomId` không kèm ngày: tất cả block chưa kết thúc của sân; truy vấn lịch: mặc định 31 ngày, tối đa 62 |
| POST | `api/manager/room-blocks` | M | `{roomId, startAtUtc, endAtUtc, reason}` | 201; **409 `occupancy_conflict` + `conflicts[]`** `{resource, conflictSourceType, conflictSourceId, startUtc, endUtc}`; 400 `block_in_the_past` `invalid_range` |
| PUT | `api/manager/room-blocks/{blockId}` | M | `{roomId, startAtUtc, endAtUtc, reason}` | 200 `RoomBlockResponse`; sửa thời gian/lý do trên cùng sân, giữ nguyên ID; kiểm tra occupancy và cập nhật nguyên tử; 409 `occupancy_conflict`, `room_block_from_incident`, `room_under_incident_review`; không thành công giữ block cũ |
| DELETE | `api/manager/room-blocks/{blockId}` | M | — | 204; 409 `room_block_from_incident` |
| GET | `api/court-rates?roomTypeId` | Auth | — | khung giá active |
| GET | `api/manager/court-rates?roomTypeId` | M | — | gồm cả inactive |
| POST, PUT | `api/manager/court-rates`, `.../{rateId}` | M | `{roomTypeId, sportId?, daysOfWeek["MON".."SUN"], startTimeLocal, endTimeLocal, pricePerHour, isActive}` | 400 `invalid_price` `invalid_rate_window` `invalid_days` `sport_not_compatible`; 409 `court_rate_overlap` |
| DELETE | `api/manager/court-rates/{rateId}` | M | — | 204 |

Snapshot audit CourtRate (`CREATE_COURT_RATE`, `UPDATE_COURT_RATE`, `DELETE_COURT_RATE`) chứa `roomTypeId`, `roomTypeName`, `sportId`/`sportName` có thể null, `days` (mã ngày phân cách bằng dấu phẩy), `startTimeLocal`/`endTimeLocal` (`HH:mm`, giờ Việt Nam), `price` và `active`. Tên và thời gian được ghi tại thời điểm sự kiện; không điền bù snapshot lịch sử từ danh mục hiện tại. Snapshot cũ có thể thiếu tên/thời gian. Cập nhật giữ cả snapshot cũ và mới; tạo chỉ có mới, xóa chỉ có cũ. Không có migration schema hoặc thay đổi request/response giá.
| GET | `api/availability/rooms?sportId&startUtc&endUtc` | Staff | — | `[{roomId,name,roomTypeId,capacity}]`; khoảng tối đa 12 giờ |
| GET | `api/availability/coaches?sportId&startUtc&endUtc&excludeSessionId?` | Staff | — | `[{coachId, fullName}]` |
| GET | `api/availability/rooms/{roomId}/busy?fromUtc&toUtc` | FD | — | `[{resource, sourceType, sourceId, startAtUtc, endAtUtc}]`; tối đa 31 ngày |

Mã lỗi DB chưa được service bắt (mọi endpoint): 409 `occupancy_conflict`, 409 `duplicate_value`, 409 `reference_violation`, 409 `concurrency_conflict`, 400 `constraint_violation`.
Mọi thời điểm trong request/response là UTC; giờ mở cửa và khung giá là giờ địa phương Asia/Ho_Chi_Minh (UTC+7 cố định).

## Phần E — P1.06 Wallet và xác nhận dùng điểm

Các endpoint sau đã có source và integration test PostgreSQL. 1 điểm = 1.000 VND; điểm là integer, tiền là decimal. `Confirmed` ở xác nhận điểm chưa phải Invoice Paid.

| Verb | Path | Actor | Request / Response |
|---|---|---|---|
| GET | `api/wallet/me` | Member | `{ownerUserId, availablePoints, heldPoints, vndPerPoint}`; subject lấy từ JWT |
| GET | `api/wallet/me/ledger?page&pageSize` | Member | Mảng ledger, mới nhất trước; page >=1, pageSize 1..100 |
| GET | `api/members/{memberId}/points`, `.../points/ledger` | Receptionist/Manager | Chỉ Member; Receptionist ghi audit lần xem, Manager chỉ đọc và không ghi audit |
| GET | `api/manager/wallets/{ownerId}`, `.../ledger` | Manager | Số dư/lịch sử của Member; xem, refresh, lọc và phân trang không ghi audit |
| POST | `api/wallets/{ownerId}/adjustments` | Manager | Luồng điều chỉnh cũ bị khóa (403); dùng point-gifts để tặng điểm, không mở lại trừ điểm thủ công. |
| GET | `api/point-credit-requests?status=Pending&page=1&pageSize=20` | Manager | `PagedResult<PointCreditRequestResponse>` gồm người nhận, người yêu cầu, điểm, lý do, trạng thái và tên/email hiện tại để người duyệt kiểm tra. |
| POST | `api/point-credit-requests/{requestId}/approve` | Manager khác người yêu cầu | Luồng phê duyệt cũ bị khóa (403), giữ lịch sử. |
| POST | `api/point-credit-requests/{requestId}/reject` | Manager khác người yêu cầu | Luồng phê duyệt cũ bị khóa (403), giữ lịch sử. |
| GET | `api/invoices/{invoiceId}/point-selection` | Chủ invoice hoặc Receptionist | `{invoiceId, memberId, pointsApplied, cashAmount, holdExpiresAtUtc, status, revision}`; Receptionist chỉ xem Member, có audit |
| POST | `api/invoices/{invoiceId}/point-confirmations` | Receptionist | `{memberId, points:int>0, revision:int}`; trả `{confirmationId, invoiceId, memberId, points, expiresAtUtc, holdExpiresAtUtc, status:"Pending", revision}` |
| POST | `api/point-confirmations/{confirmationId}/verify` | Lễ tân đã yêu cầu mã | `{code:"6 digits"}`; trả PointSelectionResponse, status `Confirmed`; giữ điểm đúng một lần |
| POST | `api/invoices/{invoiceId}/point-confirmations/clear` | Receptionist | `{memberId, revision}`; release điểm của cycle cũ, vô hiệu OTP, tăng revision; dùng khi bỏ chọn/đổi Member tại UI |
| POST | `api/wallet/me/checkouts/{invoiceId}/points` | Member | `{points:int>=0}`; 0 = bỏ điểm; invoice phải thuộc JWT subject, không nhận owner từ client, không cần OTP |

Ledger trả `id, entryType, points, availableDelta, heldDelta, availableAfter, heldAfter, referenceType, referenceId, note, createdAtUtc`. Entry types: `HOLD/RELEASE/SPEND/EARN/ADJUSTMENT`. Actor được lưu ở ledger; không có endpoint rút/chuyển điểm. Nạp điểm chỉ qua `POST /api/checkouts/wallet-top-up` (Member, hoặc Receptionist nạp hộ); tặng điểm chỉ qua CREDIT của Manager.

Cột **Tham chiếu** trên lịch sử điểm hiển thị loại/mã nguồn. Khoản tặng dùng `ManagerGift` + `giftId`, ledger `Earn`; lịch sử ManagerAdjustment cũ được giữ. Retry cùng batch không sinh thêm ledger hoặc audit.

Manager mở/xem ví không ghi VIEW_OWNER_WALLET hoặc VIEW_MEMBER_WALLET. Tặng điểm ghi GIFT_POINTS với Manager, Member, incident (nếu có), điểm, lý do, ghi nhận đồng ý và ledger; validation/thất bại/retry đã áp dụng không sinh thêm log. Audit phê duyệt/điều chỉnh cũ được giữ.

**Luồng và lỗi:**
- Đọc point-selection lấy revision trước khi yêu cầu OTP. Stale revision → 409 `checkout_revision_changed`. Hóa đơn legacy không có cycle, đã Paid/Void hoặc quá hạn → 409 `checkout_unavailable`; khác Member → 403 `invoice_not_owned`.
- OTP 6 số, PBKDF2 có salt, hết hạn `min(now+5 phút, holdExpiresAtUtc)`. Gửi lại bằng POST request với revision mới; cùng lựa chọn trong 60s → 409 `point_confirmation_cooldown`. Rate limit request 3/phút theo IP + token → 429. Đổi điểm/gửi lại/bỏ chọn không kéo dài hạn invoice.
- Chỉ lễ tân yêu cầu mã được verify. Sai mã → 400 `point_confirmation_invalid`; 5 lần sai đã commit → 409 `point_confirmation_locked`; hết hạn/vô hiệu → 409 `point_confirmation_expired`. Gọi đúng mã lặp/song song không Hold hai lần.
- Verify kiểm lại role/active Member, revision/cycle, invoice chưa trả, hạn và số dư. Thiếu điểm → 409 `insufficient_points`; OTP chưa consumed. Invoice/hold/OTP rollback cùng nhau khi lỗi sau Hold.
- Đã có payment/attempt/adjustment → 409 `payment_already_started`, không sửa số tiền của QR cũ. Thu tiền thủ công khi đang chọn/giữ điểm → 409 `point_checkout_requires_gateway`.
- Self selection chỉ đổi điểm trên invoice của chính mình; points > total/1000 bị 400 `invalid_points`. Đổi/bỏ điểm release cycle cũ và tạo reference mới, không âm available/held.
- Job mỗi phút release điểm trên hóa đơn Issued đã quá hạn hoặc Void. Expired state được suy từ thời gian server ngay cả khi job chưa xử lý.

Chu kỳ có snapshot trên Invoice (`CheckoutCycleId/CheckoutRevision/HoldExpiresAtUtc`) và lịch sử trong CheckoutSession. `PointsApplied` là điểm chọn/giữ; `CashAmount=TotalAmount-PointsApplied*1000`. Endpoint attempts thực hiện Spend + fulfillment nguyên tử khi cash=0, không phát QR 0đ. Course/rental dùng quote/reserve ports; callback/reconcile và Spend/Release gắn reference CheckoutSession + cycle ID. Xem Phần G và bản chốt outcome ở đầu tài liệu.

**Email:** chọn `Email:Provider=Smtp` (mặc định, giữ cấu hình `Smtp:*`/`Email:Smtp:*`) hoặc `Brevo` (HTTPS API, cấu hình `Email:Brevo:ApiKey`, `FromAddress`, `FromName`; sender cần được xác minh). Brevo không yêu cầu cấu hình SMTP và dùng được khi nền tảng chặn SMTP outbound. Không ghi OTP, link kích hoạt hoặc API key vào log; `Email:DemoLoggingEnabled` không thay thế email delivery. Không tự giữ điểm hoặc báo xác nhận thành công khi chưa xác minh mã. Outbox email/retry bền vững thuộc P1.11.

## Phần F — P1.05 Lớp theo khóa, điểm danh và Gym

| Verb | Path | Actor | Hành vi chính |
|---|---|---|---|
| GET | `api/classes`, `api/classes/{classId}` | Public | Chỉ `Published`, không trả chi phí/ngưỡng nội bộ; giá và chỗ còn theo cả khóa. |
| GET/POST/PUT | `api/manager/classes`, `api/manager/classes/{classId}`, `api/manager/classes/{classId}/publish`, `.../cancel` | Manager | Soạn Draft; publish khóa + đủ buổi + occupancy + audit + thông báo Coach cùng transaction; hủy với preview/refund/hold release theo bổ sung P2.06–P2.10 đầu tài liệu. |
| GET | `api/classes/{classId}/sessions`, `api/class-sessions/{sessionId}`, `.../roster` | Nhân viên; Coach đúng lớp | Buổi của cả khóa và roster Confirmed; Coach chỉ đọc lớp được giao. |
| POST | `api/class-sessions/{sessionId}/reschedule`, `.../cancel` | Manager | Kiểm lại room/coach/opening/capacity/lịch Member; hủy buộc có buổi bù hợp lệ. |
| PUT | `api/class-sessions/{sessionId}/attendance/{enrollmentId}` | Coach được phân công buổi và Receptionist (BR-56); `{status:"Present"|"Absent", reason?}`. Coach ghi từ 5 phút trước đầu buổi đến hết 24 giờ sau cuối buổi. Receptionist chỉ chỉnh bản ghi Coach đã tạo khi Member kiến nghị: bắt buộc `reason` 3 đến 500 ký tự (400 `attendance_correction_reason_required`), chưa có bản ghi trả 409 `attendance_not_recorded`, không bị giới hạn 24 giờ; mọi lần ghi hoặc sửa có audit (kèm lý do). |
| GET | `api/members/me/enrollments`, `.../schedule` | Member | Ghi danh và lịch cá nhân; không có endpoint tự ghi danh từng buổi. |

`IClassEnrollmentFulfillment` cung cấp quote, giữ chỗ, confirm, release và cancel trong transaction của checkout/fulfillment caller. Lớp nhóm không tự tạo Present/Absent; điểm danh do Receptionist hoặc Coach được phân công ghi.

## Phần G — P1.07 Checkout hiện hành

| Verb | Path | Actor | Hành vi |
|---|---|---|---|
| POST | `api/checkouts/membership`, `api/checkouts/class` | Member, Receptionist, Manager | Bắt buộc `Idempotency-Key`; staff chỉ định `targetMemberId`, Member chỉ mua cho mình. Trả CheckoutResponse với invoice/cycle/hạn/số tiền. PT 1:1 không thuộc phạm vi hiện hành. |
| GET | `api/checkouts/{invoiceId}` | Chủ invoice hoặc staff | Trạng thái chu kỳ checkout hiện tại. |
| POST | `api/checkouts/verify-reference` | Chủ checkout hoặc staff | Body `{reference}` lấy từ `vnp_TxnRef` trên Return URL. Backend kiểm ownership rồi gọi QueryDR cho đúng attempt; chỉ phản hồi VNPay đã xác minh mới cập nhật payment. Trả CheckoutResponse mới nhất. Rate limited. |
| POST | `api/checkouts/cancel-reference` | Chủ checkout hoặc staff | Body `{fields}` là toàn bộ query của Return URL. Backend kiểm ownership và chữ ký VNPay; chỉ khi `vnp_ResponseCode=24` (khách hủy) mới đóng attempt `Pending` (chưa có verified event) thành `Failed/Cancelled` để lần thanh toán sau tạo `vnp_TxnRef` mới. Hóa đơn vẫn `Issued` đến khi hết hạn. Sai chữ ký → 400, mã khác 24 → 409. Rate limited. |
| POST | `api/checkouts/{invoiceId}/attempts` | Chủ invoice hoặc staff | Tạo/đọc PaymentAttempt snapshot tiền/điểm và URL VNPay; 100% điểm hoàn tất ngay, không tạo attempt VNPay 0đ. |
| POST | `api/checkouts/{invoiceId}/cancel`, `.../retry` | Chủ invoice hoặc staff | Hủy nhả hold; retry sau hết hạn cấp invoice/cycle mới và `Idempotency-Key` mới. |
| GET | `api/payments/vnpay/return`, `api/payments/vnpay/ipn` | Public (VNPay) | Return chỉ đọc; IPN xác minh chữ ký/tham chiếu/số tiền rồi lưu event và thực hiện fulfillment. |
| POST | `api/invoices/{invoiceId}/reconcile` | FrontDesk | QueryDR xác minh giao dịch theo attempt mới nhất. |
| POST | `api/dev/payments/{reference}/simulate` | FrontDesk, Development | Mock callback qua cùng pipeline IPN; không có ở môi trường khác. |

Lỗi truyền tải QueryDR trả `502 vnp_query_http_error` (message chứa HTTP status upstream), `502 vnp_query_connection_failed`, `502 vnp_query_invalid_response` hoặc `504 vnp_query_timeout`. Đây là lỗi xác minh, không phải kết luận giao dịch thất bại. Không trả raw response, chữ ký hoặc secret của gateway; chữ ký, merchant, reference và số tiền vẫn phải được xác minh trước khi ghi payment.

Request QueryDR gửi `User-Agent: SportHub/1.0` và `Accept: application/json`. Kiểm tra sandbox ngày 15/10/2026 cho thấy cùng POST JSON rỗng nhận HTTP 403 với HttpClient không có User-Agent, và HTTP 200 kèm JSON lỗi nghiệp vụ khi có header này; probe rỗng chỉ kiểm tra khả năng truy cập, không xác nhận một giao dịch thanh toán.

Route ghi payment thủ công không còn tạo khoản thu mới. `PAID_AFTER_RECONCILIATION` biểu thị khoản thu muộn, cần đọc `fulfillmentOutcome`: `FULFILLED` khi reacquire/cấp quyền lợi được; `COMPENSATED` khi bồi hoàn điểm; `RECONCILIATION_REQUIRED` khi không thể đổi chính xác tiền thực thu sang điểm. Khi hold lớp hết hạn, thanh toán kết hợp điểm và VNPay không được tự tái giữ chỗ; hệ thống nhả điểm đã giữ và bù phần tiền VNPay thực thu vào ví theo 1.000đ/điểm. Hóa đơn đã đóng không còn số tiền cần thanh toán (`outstanding = 0`), còn `totalAmount` giữ nguyên giá trị đơn ban đầu. `CheckoutSession` và `VerifiedGatewayEvent` giữ lịch sử/idempotency. CourtRental dùng `/api/checkouts/court-rental`.

## Phần H — P1.08 Refund bằng điểm

Refund dùng `PaymentAdjustment` hiện có, không thêm bảng Refund; refund mới có `InvoiceItemId`, `SystemCalculatedPoints`, `ApprovedPoints`, `CenterFault` và ledger reference. `Amount`/`RequestedAmount` là 0 cho refund mới, để không bị tính nhầm thành tiền hoàn trong báo cáo legacy. Approval ghi `Completed`, cộng điểm vào ví và hủy entitlement trong cùng transaction. Không có xác nhận chi tiền hay VNPay refund.

| Verb | Path | Actor | Hành vi mục tiêu |
|---|---|---|---|
| GET | `api/refunds?status=&invoiceId=&invoiceItemId=&page=&pageSize=` | Receptionist, Manager | Trả item, điểm tính/duyệt, ledger reference và trạng thái. |
| POST | `api/refunds` | Item owner (Member), Receptionist, Manager | Body `{invoiceItemId, reason}`; item owner chỉ yêu cầu item của mình. Server tính điểm, không nhận điểm từ client. |
| POST | `api/refunds/{adjustmentId}/approve` | Manager khác người yêu cầu | Body `{centerFault, percent?, reason}`. `percent` (0–100) chỉ được giảm so với số điểm hệ thống tính lại cho dịch vụ, không được vượt cap; cần căn cứ ≥ 10 ký tự. Bỏ trống thì theo quy tắc. Khóa invoice/item/refund; Earn + hủy quyền lợi + Completed nguyên tử. |
| POST | `api/refunds/{adjustmentId}/reject` | Manager khác người yêu cầu | Body `{reason}`; từ chối yêu cầu chưa xử lý. |
| GET | `api/refunds/center-cancellations/mine` | Member | Danh sách ghi danh lớp / lượt thuê sân bị **trung tâm** hủy trong 90 ngày: `[{invoiceItemId,invoiceId,kind:"Class"\|"CourtRental",name,cancelledAtUtc,reason,refundPoints,remainingValueVnd,autoRefundAtUtc,status,creditedPoints}]`. `status` = `Offer` (chưa chọn), `PointsCredited`, `RefundRequested` (đã gửi Support Request PAYMENT). `refundPoints` = floor(giá trị đã trả còn lại/1.000). |
| POST | `api/refunds/center-cancellations/{invoiceItemId}/points` | Member sở hữu mục | Chọn nhận 100% giá trị còn lại bằng điểm: **hoàn tự động ngay**, idempotent (gọi lại trả `alreadyApplied=true`), ghi `POINT_LEDGER` `SystemEvent` với note bắt đầu `CenterCancel:`. 404 `center_cancellation_not_found`, 403 `center_cancellation_not_owned`, 409 `center_cancellation_refund_requested` (hóa đơn đã có Support Request PAYMENT chưa bị từ chối), 409 `center_cancellation_nothing_to_refund`. |

**Hoàn song song theo phương thức gốc (25/10/2026):** request PAYMENT hoàn phần VNPay qua payout của Manager và tự cộng lại phần điểm đã dùng trong cùng giao dịch; hóa đơn 100% điểm không gửi được request ngân hàng. Chi tiết, mã lỗi và endpoint đề xuất `refund-breakdown` ở [Payment-Support-Workflow](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/d78ec839d92eb04d602b4e9f86ff3f41288c7f22/docs/Payment-Support-Workflow.md) mục 5.7.

**Hủy do trung tâm (lớp dưới ngưỡng do Manager hủy, hủy lượt thuê sân, sự cố sân):** lúc hủy hệ thống chỉ gửi thông báo cho Member, **không cộng điểm**. Mỗi mục Member chỉ chọn **một** cách, và cách còn lại bị từ chối: (1) nhận điểm → hoàn tự động; (2) gửi Support Request loại PAYMENT gắn hóa đơn (`/member/help?topic=PAYMENT&invoiceId=`) → Manager xử lý theo luồng payout hiện có. `SupportInvoiceGuard` trả 409 `center_cancellation_points_chosen` nếu hóa đơn đã nhận điểm. Quá 2 ngày kể từ lúc hủy mà Member chưa chọn thì job `CenterCancellationAutoRefundJob` (10 phút/lần, quét 60 ngày gần nhất) tự hoàn điểm. Trạng thái được suy ra từ ledger + yêu cầu hỗ trợ, không có bảng mới.

Đã có calculator cho Membership (50% khi RemainingDays×3 >= TotalDays×2; lỗi trung tâm theo phần ngày chưa dùng; vé Gym 1 ngày không hoàn), Class (trước buổi đầu 100%, center cancellation theo buổi chưa cung cấp, loại buổi hủy đã có buổi bù khỏi tỷ lệ), Rental (>=24h 100%, dưới 24h 0, center cancel 100%). Quy đổi trên giá trị item đã trả gồm cash + points trừ refund trước, floor VND/1.000. `RefundCreditService` hỗ trợ SystemEvent credit, threshold/incident/rental caller và ghi InvoiceItemId trên ledger.

Membership theo BR-88 tính cả ngày đầu/cuối: `TotalDays = EndDate − StartDate + 1`, `RemainingDays = clamp(EndDate − RefundRequestedDate + 1, 0, TotalDays)`, theo ngày Việt Nam. "Đã dùng quá mức" ở Membership là đã qua hơn 1/3 thời hạn, không tính số lần tới Gym hoặc số buổi còn lại. Gói gia hạn chưa bắt đầu vẫn được xét hoàn chuẩn tối đa 50%; gói Expired/Cancelled không có điểm hoàn. Khi duyệt, dùng ngày tạo yêu cầu, không thay bằng ngày duyệt. Ngoại lệ Center Fault cần lý do và căn cứ, hoàn theo phần ngày chưa dùng. Theo quyết định 25/10/2026, đóng Gym tạm thời để sửa chữa không gia hạn Membership và không phải refund Membership; Manager chọn người bị ảnh hưởng và đề xuất tặng điểm theo luồng CREDIT có chính Manager xác nhận. API sự cố từ chối membershipExtensionDays khác 0.

Trong luồng **refund** Membership, Manager không nhập số điểm tùy ý: chỉ chọn center-fault có lý do rồi server tính lại theo cap. Khoản **tặng điểm do sự cố Gym tạm thời** ở trên là luồng CREDIT riêng; Manager được đặt số điểm cho từng người nhận và chính Manager xác nhận thì cộng ví ngay. PostgreSQL gate cho refund/cap/concurrency/rollback và migration upgrade đã qua; xem evidence.

## Phần I — P1.09 Ngưỡng hoàn vốn và phản hồi

| Verb | Path | Actor | Hành vi hiện có |
|---|---|---|---|
| PUT | `api/manager/classes/{classId}/threshold/pricing` | Manager | Body `{price,costAmount,reason}`; chỉ lớp Published trước deadline, tính lại ceil(cost/price), không đổi invoice snapshot. |
| POST | `api/manager/classes/{classId}/threshold/waive` | Manager | Body `{reason}`; đặt WaivedByManager có audit. |
| POST | `api/class-threshold-responses` | Member chủ ghi danh | Body `{token,choice,targetClassId?}`; token SHA-256 được lưu hash; lựa chọn cuối cùng, kiểm member sở hữu và deadline. Refund tự động cộng điểm; chuyển bằng/rẻ hơn đổi enrollment nguyên tử và hoàn chênh; chuyển đắt hơn trả `additionalInvoiceId`, giữ ghi danh nguồn đến khi invoice difference được fulfill. Gửi lại Transfer trả checkout đang mở hoặc tạo retry khi invoice cũ Void/expired. |

Job evaluator đặt AtRisk sau deadline nếu ConfirmedCount thấp ngưỡng, snapshot response deadline theo setting và queue secure link cho mỗi member (kể cả ghi danh mới khi AtRisk). Job expiry hoàn điểm response Pending quá hạn, sau đó reevaluate và hủy khóa/hoàn các ghi danh còn lại nếu chưa đạt ngưỡng và không được waive. Hết hạn AtRisk thì không nhận thêm ghi danh.

Invoice `CLASS_TRANSFER_DIFFERENCE` giữ chỗ lớp đích, release/retry khi hết hạn và chỉ đổi enrollment sau payment fulfillment. Add-on invoice item trỏ item gốc; hoàn lớp đích tính item chênh đã trả trừ refund chênh trước. PostgreSQL gate đã qua, gồm transfer thấp/bằng/cao, ownership/replay và expiry.

## Phần J — P1.10/P1.11 Thuê sân, sự cố và outbox

| Verb | Path | Actor | Contract |
|---|---|---|---|
| POST | `api/checkouts/court-rental` | Member | `Idempotency-Key`, body gồm môn/phòng/start/end; invoice/item giữ snapshot giá và occupancy cho tới fulfill/cancel/expiry. |
| GET | `api/court-rentals/availability` | Member | `sportId,startUtc,endUtc`; chỉ phòng trống + giá, không trả lịch Member/lớp. |
| GET | `api/court-rentals/mine` | Member | Rental của chính người gọi, lọc `fromUtc/toUtc`. |
| POST | `api/court-rentals/{rentalId}/cancel` | Chủ rental | Self-cancel theo ngưỡng setting 24 giờ; trả 204. |
| GET | `api/manager/court-schedule/rentals` | FrontDesk | Rental theo khoảng thời gian/phòng; lịch tổng hợp dùng `/api/manager/court-schedule`. |
| POST | `api/manager/court-rentals/{rentalId}/cancel` | Manager | `{reason}`; hủy do trung tâm, chỉ thông báo Member; việc hoàn do Member chọn (Phần H). |
| POST | `api/manager/incidents/preview`, `api/manager/incidents/resolve` | Manager | `IncidentRequest` gồm scope, room/time, lý do; resolve kiểm tra tác động lại, rental cancel/refund + block nguyên tử; lớp chồng thời gian trả conflict để xử lý lịch trước. |
| POST | `api/manager/notices` | Manager | `{subject,message,recipientUserIds,sendInApp,sendEmail}`; in-app/email vào outbox theo người nhận, giới hạn 1–200 người. |

Email không xuất hiện trong danh sách/read-all InApp. OTP email được mã hóa ở outbox và gửi bởi dispatcher; status email Sent chỉ sau sender thành công, retry có backoff/lease và gửi at-least-once.

Email sender hỗ trợ cả SMTP và Brevo HTTPS qua cùng `IEmailSender`, gồm file đính kèm. Brevo gọi `POST https://api.brevo.com/v3/smtp/email`; chỉ trả thành công khi HTTP thành công và có message identifier. HTTP lỗi hoặc phản hồi không hợp lệ được đưa về cơ chế outbox retry hiện có, không tự fallback sang SMTP. `Sent` nghĩa provider đã chấp nhận, không xác nhận giao tới Inbox. Không thay đổi route API, schema hoặc migration.

`GET /api/reports/revenue` giữ các field legacy, đồng thời thêm `legacyCashCollected`, `reconciliationCashCollected`, `reconciliationCashCount`, `pointsRedeemed`, `pointsRedeemedVnd` (điểm×1.000 VND), `pointsIssued` (mọi khoản điểm cộng vào ví: hoàn điểm + bù cổng thanh toán), `pointsRefunded` (chỉ phần hoàn điểm: yêu cầu hoàn, huỷ do trung tâm, chênh lệch chuyển lớp; tập con của `pointsIssued`), `managerPointAdjustment`, `outstandingPoints` (available+held tại lúc chạy), và `bySource[{source,cashCollected,pointsRedeemed}]`. Daily rows thêm `legacyCashCollected` và `reconciliationCashCollected`; Cash refunded (`totalRefunded`) là tổng `MemberSupportPayout.Amount > 0` của các khoản hoàn chuyển khoản đã ghi nhận qua Support Request, quy kỳ theo `RecordedAtUtc` giờ VN. `netRevenue = totalCollected - totalRefunded`; daily `refunded`/`net`, đối soát và export dùng cùng phạm vi lọc. Hoàn điểm không làm giảm thu ròng; khoản chỉ hoàn điểm không tăng `refundCount`. `bankRefundPaid` và `bankRefundCount` được giữ làm alias tương thích của `totalRefunded`/`refundCount`, không cộng thêm hoặc hiển thị riêng; daily `bankRefundPaid` là alias của `refunded`; `totalCollected` cộng cả cash reconciliation đã xác minh để đối soát. Group-by-Sport và export dùng cùng tổng hợp theo phần P1.12/P1.13 bên dưới.

Lịch phòng tổng hợp và incident workflow hiện hành được mô tả ở bản chốt đầu tài liệu; PostgreSQL concurrency/privacy đã qua gate. Manager cần chọn phương án dời/hủy/bù hợp lệ trước khi resolve incident có lớp.
# Contract báo cáo và tích hợp


| Verb | Route | Actor | Hợp đồng |
|---|---|---|---|
| GET | `/api/reports/revenue?fromDate=YYYY-MM-DD&toDate=YYYY-MM-DD&sportId=&source=&memberId=` | CenterManager | Bộ lọc theo chiều tùy chọn áp dụng đồng thời cho tổng, các dòng theo ngày, `bySource` và `bySportAndSource`. Các chiều chứa `source`, `sportId/sportName/memberId` có thể null, `cashCollected`, `legacyCashCollected`, `pointsRedeemed`. Các dòng tiền thu cộng thành `totalCollected`, tính Reconciliation và LegacyUnclassified đúng một lần. |
| GET | `/api/reports/court-rental-revenue?fromDate=...&toDate=...` | CenterManager | `{fromDate,toDate,rows}`; lấy các dòng Rental của cùng bộ tổng hợp doanh thu, theo ngày thực thu VN. |
| GET | `/api/reports/membership-period?fromDate=...&toDate=...` | CenterManager | Các trường kỳ hiện có cùng `membersPurchasingMembershipInPeriod`, `dayPassesSold`, `daily[{date,buyers}]`. Người mua là Member không trùng có hóa đơn Membership đã thanh toán, loại vé ngày; người mua theo ngày không trùng trong từng ngày Việt Nam và có thể lặp giữa các ngày. Vé ngày đếm vé liên kết đã trả tiền có trạng thái Active/Expired theo ngày thanh toán hóa đơn. Vé cũ không có bằng chứng thanh toán không được tính là đã bán. |
| GET | `/api/reports/class-enrollment?fromDate=...&toDate=...&sportId=3` | CenterManager | `activeHoldCount` chỉ đếm Active có expiry lớn hơn server clock; `availableSeats=capacity-confirmed-activeHold`. |
| POST | `/api/reports/exports` | Theo policy export hiện hữu | `{reportType,fromDate,toDate,columns,format:"Csv" hoặc "Pdf",sportId?:3}`. `sportId` áp dụng cho `CLASS_ENROLLMENT`. |

Các loại export mới:

- `REVENUE_DAILY`: `date,collectedAmount,refundedAmount,obligationReduction,netCollected,legacyCashCollected,reconciliationCashCollected`.
- `REVENUE_DIMENSIONS` / `COURT_RENTAL_REVENUE`: `source,sportId,sportName,memberName,memberId,collectedAmount,legacyCashCollected,pointsRedeemed,pointsRedeemedVnd`. Manager chọn cột `memberName` (Member) để xuất tên hội viên trong CSV/PDF. `memberId` (Member ID) còn được API hỗ trợ để tương thích; bộ lọc vẫn dùng ID. Tên đọc từ hồ sơ tại thời điểm xuất, truy vấn chung cho các hội viên trong kết quả; không thay đổi cách nhóm hay tổng tiền/điểm. Dòng `memberId = null` để trống tên, không gán một hội viên cho dòng tổng hợp. Nếu có ID nhưng thiếu tên, hiện `Name unavailable`. File đã xuất giữ nguyên dữ liệu tại thời điểm tạo.
- `MEMBERSHIP_PERIOD`: `fromDate,toDate,newMembers,membersPurchasingMembershipInPeriod,activeMembersAtPeriodEnd`. `newMembers` đếm tài khoản Member tạo trong kỳ. `membersPurchasingMembershipInPeriod` đếm MemberId riêng biệt trên hóa đơn có InvoiceItem Membership, đã fulfillment thành công (`Paid` hoặc `PaidAfterReconciliation` với `PaidVia=VnPayAfterReconciliation`), theo `Invoice.PaidAtUtc` trong kỳ theo ngày Việt Nam; bao gồm thanh toán bằng điểm, không tính giao dịch chỉ bù điểm và không dùng ngày dự phòng khi PaidAtUtc null.
- Overview dùng `GET /api/reports/membership-summary?asOfDate=YYYY-MM-DD` với ngày hôm nay theo giờ Việt Nam cho **Active Gym members**: số Member riêng biệt có gói Active với `StartDate <= asOfDate <= EndDate`, loại vé ngày, PendingPayment, Expired và Cancelled. Không phải số người đến trung tâm hay số người có gói từng hiệu lực trong tháng. Giữ nguyên báo cáo Membership theo kỳ và các cột export hiện có.
- `GET /api/reports/class-status` chỉ dành cho CenterManager. `fromDate` và `toDate` tùy chọn (có cả hai hoặc không có, tối đa 366 ngày tính cả hai đầu) lọc ngày bắt đầu lớp; `sportId` tùy chọn lọc môn. Trả `{open,completed,unpublished,classes[]}`. Mỗi lớp có `classId,name,sportId,sportName,startDate,status,capacity,confirmedCount,thresholdStatus`. Open = Published + InProgress; Completed = Completed; Unpublished = Draft; loại Cancelled. Không truyền ngày thì Overview giữ ý nghĩa tất cả lớp hiện có. Ghi danh cộng số chỗ đã xác nhận/sức chứa trên các lớp không phải Draft; loại hold chưa trả tiền. So sánh kỳ là so các nhóm lớp theo ngày bắt đầu dựa trên trạng thái hiện tại, không phải snapshot lịch sử.
- `CLASS_ENROLLMENT`: `classId,code,name,sportId,sportName,status,capacity,confirmedCount,activeHoldCount,availableSeats,fillRatio,breakEvenThreshold,thresholdStatus,firstSessionStartUtc`.

`REVENUE` cũ vẫn là danh sách hóa đơn theo ngày phát hành; không dùng loại này để so với tổng thu theo ngày thanh toán. `MEMBER_SUMMARY` cũ giữ tương thích. Export mới gọi cùng service API, không nhân bản truy vấn tài chính. Phạm vi doanh thu tối đa 366 ngày, trả `range_too_large`; report export thất bại giữ trạng thái Failed và failure reason theo contract hiện hữu. Điểm không cộng thành cash; outstanding là available+held hiện tại. Membership sport null; item chưa có sport reference chỉ được phân loại khi có đúng một môn OneOnOne.

- `REVENUE_SUMMARY`: `fromDate,toDate,collectedAmount,refundedAmount,netCollected,legacyCashCollected,reconciliationCashCollected,pointsRedeemed,pointsRedeemedVnd,pointsIssued,managerPointAdjustment,outstandingPoints`.

Trình bày báo cáo: `pointsRedeemed` là số điểm đã dùng trong kỳ được chọn; `pointsRedeemedVnd` là giá trị VND tương đương, không bao giờ là khoản thu tiền bổ sung. `pointsIssued` đếm mọi lần ghi có ví trong kỳ (hoàn trả và bồi hoàn từ gateway); `pointsRefunded` chỉ đếm điểm được ghi có do hoàn trả. Cả hai đều không phải tiền thu, và `netRevenue`/`netCollected` = tiền thu − tiền hoàn, nên không gồm biến động điểm và không phải tổng doanh thu. Điều chỉnh của Manager được giữ riêng. Số dư ví hiện tại (`outstandingPoints`, khả dụng + đang giữ) hiển thị tách khỏi biến động trong kỳ. `/api/reports/revenue` còn trả `outstandingPointsAsOfUtc`, thời điểm UTC quan sát số dư hiện tại đó; `REVENUE_SUMMARY` hỗ trợ cột xuất tùy chọn này. Đây không phải số dư cuối kỳ lịch sử. Response kỳ cũ giữ `activeMembersAtPeriodEnd`; dashboard Reports thay vào đó ghi nhãn Active Gym members bằng summary riêng cho hôm nay. `REVENUE` cũ có tiêu đề "Invoices by issue date". `refundedAmount` và `netCollected` cũ giữ ý nghĩa số liệu hiện có nhưng nhãn PDF chỉ rõ hoàn tiền và loại hoàn điểm; bộ chọn cột xuất chính của Manager không đưa các cột cũ này ra. File đã xuất hiện có không thay đổi.

## Manager operations read contracts

- `GET /api/audit-logs` nhận `targetId` tùy chọn cùng `targetEntity` (bộ lọc Target trên UI). `targetEntity` dùng so khớp chuỗi con không phân biệt hoa thường bằng SQL `LIKE '%term%'`; khoảng trắng hai đầu được loại và `%`, `_`, `\` trong input được hiểu là ký tự nguyên bản. Ví dụ, `court` khớp `CourtRate` và `CourtRental`. `targetId` vẫn khớp chính xác. Hai bộ lọc đều áp dụng trước phân trang/đếm. Phạm vi hiển thị dùng role người thao tác tại thời điểm sự kiện: Admin đọc sự kiện Manager/Admin/Coach/System, Manager đọc Manager/Receptionist/Coach/Member. Member/Coach/Receptionist không thể có quyền audit qua các tham số này.
- Các dòng dịch vụ của `GET /api/manager/sports` chứa `offeringId` thực tế. Client không được suy ra ID dịch vụ từ ID môn. `GET /api/sports` công khai không có `offeringId`.
- Không thêm migration và không triển khai lệnh G03/G06/G07/G13 mới. Preview/action/recheck/resolve sự cố hiện tại vẫn là chuỗi thao tác riêng; resolve timeout yêu cầu đối soát trước lần thử mới.

## Audit target display context

`GET /api/audit-logs` thêm `currentTargetLabel` và `referenceNames` (`Entity:ID` → tên hiển thị hiện tại) tùy chọn cho kết quả mà Admin và Manager được xem. Tên chỉ được tra cho đối tượng và tham chiếu được phép trên trang đã lọc, phân trang, với một query có giới hạn cho mỗi loại entity liên quan. Cả hai bên đọc giữ các trường tài khoản đích hiện có. Tra cứu hiển thị không vượt qua giới hạn theo role sự kiện hoặc cấp quyền vào endpoint chi tiết vận hành.

UI Manager ưu tiên danh tính đã ghi (`name`, `targetName`, tên loại phòng trong giá sân, tiêu đề thông báo hoặc loại báo cáo). Nếu chưa ghi danh tính, UI hiển thị nhãn hiện tại và đánh dấu là hiện tại. Đối tượng không tồn tại/đã xóa/không hợp lệ giữ loại và ID cùng thông báo không có tên. Tên tham chiếu cũng là ngữ cảnh hiển thị hiện tại, không phải bằng chứng lịch sử. Không ghi lại hoặc điền bù `oldValue` hay `newValue`. Giá, trạng thái, thời gian, sức chứa và giá trị thao tác khác đã thay đổi chỉ lấy từ sự kiện.

Metadata dùng allowlist trường rõ ràng, bản địa hóa nhãn, so sánh giá trị đã ghi và hiểu các mảng cũ cho giờ mở cửa phòng và liên kết môn của loại phòng, trường service PascalCase và wrapper `{value, reason}`. Object không nhận diện và trường secret được bỏ qua. Chi tiết trống nói rõ không có chi tiết được ghi. Giá trị thiếu ở dữ liệu cũ được hiển thị là chưa ghi nhận, không bao giờ suy từ entity hiện tại.

Snapshot môn mới gồm tên/mã, mô tả/URL ảnh, hoạt động/thứ tự và thời lượng/sức chứa dịch vụ; thay đổi hoạt động cũng ghi tên/mã. Log tương thích dịch vụ gồm cả danh sách loại phòng trước và sau. Snapshot giờ mở cửa và tương thích loại phòng hiện ghi tên phòng/loại cùng các mảng tương ứng. Snapshot tạo/xóa room-block ghi tên phòng, ID phòng, thời điểm bắt đầu/kết thúc và lý do. Audit xuất báo cáo ghi loại báo cáo cùng tham số gốc. Các phần thêm vào payload này không đổi bản thân thao tác và không cần migration bảng audit.

## Automatic sport display order


`sortOrder` vẫn trong `SportResponse` nhưng không còn là input tạo/cập nhật. Request cũ có `sortOrder` bị bỏ qua. Backend gán thứ hạng dương liên tiếp `1..N` cho mọi môn: active trước, inactive sau. Tạo hoặc kích hoạt lại môn thêm môn vào cuối nhóm active; vô hiệu hóa thêm vào cuối nhóm inactive và thu gọn thứ hạng. Lặp activate/deactivate không ảnh hưởng vị trí. Sửa tên hoặc dịch vụ không di chuyển môn. Bộ lọc trạng thái/tìm kiếm của Manager giữ thứ hạng toàn cục này; không đánh số lại kết quả đã lọc. Danh sách công khai/Member chỉ gồm môn active. Tạo và đổi hoạt động được tuần tự hóa trong một transaction database, gồm cả ghi audit. Không bổ sung xóa môn hoặc đổi ID/mã môn.

Migration `20261007110000_AutomaticSportOrder` chuẩn hóa thứ hạng cũ âm, trùng và thưa, đồng thời giữ thứ tự tương đối trước đó trong từng nhóm hoạt động (`sortOrder`, tên, ID). Snapshot audit lịch sử không đổi.

## Manager class-slot preview

`GET /api/manager/class-schedule/availability` chỉ dành cho Manager. Query: `sportId`, `roomId`, `coachId` (tùy chọn), `capacity` dương, `startUtc`/`endUtc` UTC, `excludeSessionId` tùy chọn. Slot phải có thời lượng dương và tối đa 12 giờ. Response:

```json
{"available":false,"roomName":"Court A","coachName":"Coach Linh","reasons":["outside_opening_hours","coach_busy"]}
```

Lý do: `sport_inactive`, `room_not_found`, `room_inactive`, `room_incompatible`, `room_capacity_exceeded`, `outside_opening_hours`, `room_busy`, `coach_required`, `coach_inactive`, `coach_specialty_mismatch`, `coach_busy`. Tên mô tả bản ghi hiện tại. Giờ mở cửa theo giờ Việt Nam của phòng phải bao phủ toàn bộ slot. Occupancy phòng và Coach đều xét các lượt giữ chỗ active chồng lấn từ mọi loại nguồn.

`excludeSessionId` phải xác định một buổi lớp Scheduled hiện có thuộc môn được yêu cầu; nếu không endpoint trả `400 invalid_preview_session`. Tham số chỉ loại occupancy có source type là ClassSession và source ID là buổi đó, cho phép xem trước phương án thay mà không coi lượt giữ chỗ hiện tại là xung đột. Các lượt giữ chỗ khác vẫn được kiểm tra. Không có môn trả `404 sport_not_found`; thời lượng/sức chứa sai trả `400`.

Danh sách Coach còn trống cũng nhận excludeSessionId: phải xác định buổi lớp Scheduled cùng môn (nếu không trả 400 invalid_preview_session). Chỉ loại occupancy ClassSession của nguồn đó; các chồng lấn khác vẫn được tính. Cả hai endpoint availability chỉ mang tính tham khảo. Endpoint này không giữ tài nguyên, đổi buổi, ghi audit hoặc yêu cầu migration. Nó kiểm tra tài nguyên còn trống, không phải mọi quy tắc của lệnh cuối. Publish/reschedule/makeup vẫn kiểm tra vòng đời, học viên, ngày và occupancy trong transaction hiện có. Frontend còn đánh dấu chồng lấn trong bản nháp đề xuất là `draft_overlap`; đây là mã validation local, không phải lý do API. Hợp đồng `/api/availability/rooms`, `/coaches` và room-busy hiện có không đổi.

## Coach yêu cầu thay đổi buổi học lớp nhóm

Phạm vi đã chốt: Coach chỉ gửi báo vắng `{type:"ABSENCE",reason}` và rút báo vắng `PENDING`; không có thao tác sửa báo vắng hoặc đề xuất phương án từ Coach trên giao diện hiện hành. Center Manager chọn dạy thay, dời lịch hoặc hủy kèm học bù, kiểm tra lại Coach thay đang Active, đúng chuyên môn và còn trống trong toàn bộ buổi trước khi lưu. JWT xác định Coach thao tác; `sessionId` là ID buổi để kiểm tra phân công. Lịch và trạng thái yêu cầu cập nhật nguyên tử. Các API yêu cầu hiện có và dữ liệu lịch sử loại cũ được mô tả tại Yêu cầu vắng buổi học. PUT sửa yêu cầu đã được gỡ. Backend hiện còn nhận ba loại đề xuất cũ cho tạo mới; yêu cầu chặn chúng từ mọi client được theo dõi tại [COACH-01, thiết kế §13.2](../architecture/Center-Management-System-Design-v3.md#132-chức-năng-và-contract-còn-thiếu). Giữ lịch sử/rút Pending và các phương án xử lý của Manager là yêu cầu độc lập với việc chặn tạo mới loại cũ. Migration nền: `20261011025932_AddClassSessionChangeRequests`.

Cập nhật chính sách, 2026-10-18: yêu cầu Coach đã gửi là bất biến. Nút/form sửa, phương thức cập nhật client, service UpdateAsync và route PUT đã được gỡ. PUT tới URL GET yêu cầu hiện có trả 405 Method Not Allowed. Chủ sở hữu vẫn có thể POST `.../{id}/withdraw` khi Pending, không chịu hạn tạo trước sáu giờ. Yêu cầu đã được xử lý trả `409 class_change_not_pending`; yêu cầu của người khác trả `404 class_change_not_found`. Tạo yêu cầu khác sau khi rút cần requestId mới và điều kiện tạo hợp lệ; nội dung lịch sử không bao giờ được ghi lại.

Hộp thoại yêu cầu của Manager hiện dùng GET /api/availability/coaches với sportId, startUtc/endUtc đích và excludeSessionId tùy chọn. SUBSTITUTION dùng toàn bộ khoảng giờ gốc và loại Coach vắng khỏi lựa chọn. Reschedule/makeup dùng khoảng mới với thời lượng gốc và xóa lựa chọn cũ khi khoảng giờ đổi. Thiếu thời gian, đang tải/lỗi hoặc lựa chọn không khả dụng chặn xác nhận; kết quả rỗng gợi ý thời gian/cách xử lý khác. Lỗi tra cứu hỗ trợ làm mới. Lỗi 409 khi lưu giữ lý do, xóa lựa chọn và làm mới ứng viên. Backend vẫn xác thực đối tượng/occupancy trong transaction; xung đột giữ Pending và lịch.

UI Coach hiện gửi `{requestId,type:"ABSENCE",reason,proposedStartAtUtc:null,proposedEndAtUtc:null}`. Tạo yêu cầu phải trước buổi Scheduled được phân công ít nhất 6 giờ (`409 class_absence_deadline`); đề xuất vắng chứa thời gian trả `400 absence_proposal_not_allowed`. Rút vẫn khả dụng khi Pending. Các loại đề xuất hiện có vẫn được nhận cho client cũ với validation trước đây; lịch sử yêu cầu không bị ghi lại. Manager resolve chỉ nhận `SUBSTITUTE`, `RESCHEDULE`, `CANCEL_WITH_MAKEUP`, hoặc dùng endpoint reject riêng. Migration `20261017090000_AddCoachAbsenceRequests` mở rộng ràng buộc loại yêu cầu; Absence không phải loại xử lý.

Response yêu cầu trả thêm `outcome: {action,sessionId,sessionNo,startAtUtc,endAtUtc,coachId,coachName,roomId,roomName,handledBy,reason,source,sessionStatus}` có thể null. Migration `20261017100000_AddClassRequestOutcomeSnapshots` lưu chi tiết lịch và quyết định trong transaction cho các lượt xét duyệt và thay đổi trực tiếp của Manager (`source: DECISION`), gồm `ROOM_CHANGED` và `COURSE_CANCELLED`. Thay đổi lịch về sau không ghi đè snapshot này. Yêu cầu trước đó không có snapshot có thể chỉ trả lịch hiện tại được tham chiếu (`source: CURRENT_SCHEDULE`), được UI ghi nhãn rõ; không tự tạo ngày/tên lịch sử. Tự đóng vẫn là WITHDRAWN/SCHEDULE_UPDATED (UI: Closed), được phân biệt với Coach rút và duyệt.

## Hợp đồng bàn giao An/Hào

### Coach assignment access and schedule updates (2026-10-15)

- Quyền Coach truy cập danh tính, sức khỏe, gói và context AI của học viên yêu cầu phân công chủ nhiệm lớp Published/InProgress có ghi danh Confirmed. Quan hệ Coach–Member cũ/thủ công không cấp quyền học viên. Kết thúc một ghi danh không làm mất quyền qua lớp đủ điều kiện khác. Endpoint giảng dạy lớp đã kết thúc vẫn trả 403.
- Phân công dạy thay chỉ cấp quyền vào buổi không bị hủy được giao cho Coach đó, danh sách học viên hiện tại và bản ghi của buổi. Không cấp quyền danh bạ học viên, hồ sơ sức khỏe, đánh giá trước hoặc AI học viên. Cả `GET /api/coaches/me/teaching/classes/{classId}/members` và `.../records` yêu cầu `sessionId` với người dạy thay (`403 session_scope_required` nếu thiếu; `403 session_not_assigned` nếu khác lớp, khác Coach hoặc Cancelled). Bản ghi dạy thay chỉ có nội dung buổi đó, loại plan toàn khóa; roster bỏ mục tiêu/trình độ/ghi chú sức khỏe và chỉ đếm điểm danh buổi đó. Ghi danh Confirmed có `EndedAt` khác null bị loại khỏi quyền Coach vào học viên/hồ sơ/roster/bản ghi cá nhân và ghi dữ liệu giảng dạy. Coach chủ nhiệm có thể bỏ `sessionId` để truy cập toàn lớp; khi có `sessionId`, bản ghi gồm buổi đó và context PLAN toàn khóa. API list/get/roster/session-write áp dụng cùng ranh giới phân công. URL lớp trên frontend mở buổi dạy thay được giao thay vì màn quản lý toàn lớp. Danh sách khóa và xem trước buổi công khai vẫn tách khỏi dữ liệu giảng dạy được bảo vệ.
- POST/PUT bản ghi giảng dạy còn kiểm tra phạm vi ghi trong `ClassTeachingService.SaveAsync`: buổi được truyền phải thuộc lớp (`400 session_not_in_class`), được giao cho người thao tác (`403 session_not_assigned`) và không Cancelled (`409 session_cancelled`). Không có `sessionId` (bỏ qua hoặc null), chỉ Coach chủ nhiệm được ghi; người dạy thay nhận `403 class_plan_not_assigned`, kể cả PLAN/NOTICE và PLAN cá nhân có memberId. RESULT yêu cầu sessionId/memberId (`400 result_scope_required`); HOMEWORK yêu cầu sessionId (`400 homework_session_required`). PUT còn kiểm tra tác giả/version và giữ phạm vi kind/session/member; không sửa NOTICE/HOMEWORK. Xem hợp đồng phạm vi ghi giảng dạy.
- Kết quả buổi yêu cầu học viên có ghi danh Confirmed còn hiệu lực và điểm danh Present đã lưu trong buổi đó. Cả POST và PUT RESULT trả 409 `result_attendance_required` khi thiếu điểm danh hoặc Absent. Coach Record và trình sửa kết quả bị vô hiệu cho tới khi xác nhận Present và buổi đã bắt đầu. Lệnh điểm danh Coach và Receptionist trả 409 `attendance_result_exists` khi yêu cầu Absent cho học viên/buổi đã có RESULT lưu. Ghi kết quả và điểm danh được tuần tự hóa bằng cùng khóa dòng buổi, ngăn lưu đồng thời mâu thuẫn. Bản ghi hiện có không bị ghi lại và không cần migration.
- Không có endpoint cập nhật yêu cầu Coach. `POST /api/coaches/me/teaching/change-requests/{id}/withdraw` là thao tác thay đổi duy nhất trên yêu cầu hiện có được cung cấp cho Coach chủ sở hữu. Yêu cầu Pending và dùng chung khóa transaction với xét duyệt/đổi lịch trực tiếp. Nếu xét duyệt/cập nhật lịch trực tiếp hoàn thành trước: rút trả 409 và UI làm mới; nếu rút thắng, Manager không thể resolve.
- Dời lịch trực tiếp, dạy thay, hủy kèm bù hoặc hủy toàn khóa thành công tự đóng yêu cầu Pending bị ảnh hưởng thành `WITHDRAWN`, với `resolutionType: "SCHEDULE_UPDATED"`, người xét/thời gian/ghi chú và buổi kết quả. Duyệt thông thường vẫn là `RESOLVED`. Đóng, lịch, audit và outbox chuông/email commit nguyên tử. Xung đột giữ Pending.
- Đổi lịch thông báo cho Coach chủ nhiệm, Coach cũ và Coach mới một lần mỗi người nhận/kênh, cùng các Member ghi danh. Hành động ở chuông dẫn tới `/coach/schedule`; kết quả yêu cầu dẫn tới `/coach/change-requests`. Xử lý lớp thủ công liên quan sự cố dùng cùng các lệnh. Không cần migration mới.



### Member class communication and content access — 2026-10-16

- `CheckoutResponse.isDayPass` được tính từ item membership của hóa đơn. Giá trị giữ qua refresh/recovery và khiến checkout hiển thị điều khoản trong ngày/không hoàn trước thanh toán, kể cả truy cập trực tiếp `/checkout/{invoiceId}`.
- `GET /api/invoices/{invoiceId}/receipt.pdf` vẫn yêu cầu xác thực và giới hạn chủ sở hữu với Member. Chi tiết thanh toán và kết quả đã xác nhận hiện cung cấp **Download invoice**.
- Buổi và bản ghi giảng dạy của Member yêu cầu ghi danh Confirmed, không có `endedAt` và lớp Published/InProgress/Completed. Member có thể đọc lịch sử buổi của mình sau buổi cuối hoặc khi lớp Completed. Ghi danh kết thúc hoặc lớp Cancelled/Draft trả `403 class_access_ended`; lịch sử ghi danh/tài chính độc lập. View lớp Member đang mở kiểm tra lại quyền khi focus và mỗi 30 giây. Query lớp đang diễn ra hẹp hơn cho nhắc nhở và context AI vẫn yêu cầu buổi tương lai không bị hủy.
- Job liên lạc Member mỗi phút tạo nhắc chuông ở mốc 24 giờ và 1 giờ. Bỏ buổi hủy, ghi danh không còn hiệu lực và cửa sổ nhắc trước lúc ghi danh. Xác nhận duy nhất `(session,member,start,hours)` và khóa worker trong transaction ngăn ghi outbox trùng giữa replica/restart. Thời điểm bắt đầu sau dời lịch có định danh nhắc mới.
- AI Member nhận hồ sơ sức khỏe người dùng hiện tại, giáo án khóa/cá nhân được phép, đánh giá/nhận xét Coach gần đây, homework/thông báo, tổng kết điểm danh và buổi sắp tới đã ghi danh. Context có giới hạn gồm ID/ngày nguồn và cờ cắt bớt. Loại học viên khác và nội dung lớp đã kết thúc. ID tiếp tục của provider chỉ được chấp nhận khi tương tác thành công của chính người dùng có cùng phạm vi ghi danh hiện tại; nếu không bắt đầu context mới.
- Schema: `20261016045838_MemberClassCommunication`, sau migration giá gốc hiện có. Không xóa dòng dữ liệu vận hành. Quy tắc truy cập được mô tả trong phần này.

- **Đã gỡ:** course interests (`/api/course-interests`, `/api/members/me/course-interests`, `/api/manager/course-interests`), lựa chọn ngưỡng `WAIT_NEXT_COURSE`, `continuationOfClassId`, thông báo `NextCourseAvailable`/`CourseInterestOpened`. Migration `RemoveCourseInterest` xóa bảng/cột, đổi phản hồi `WAIT_NEXT_COURSE` đã lưu thành `REFUND` và xóa các thông báo đó. Phản hồi ngưỡng chỉ nhận `REFUND`, `TRANSFER` và `KEEP_PLACE`.
- Khi hết hạn phản hồi mà dưới ngưỡng và chưa được miễn, `ThresholdResponseExpiryService` hủy lớp, hoàn 100% giá trị còn lại có thể hoàn bằng điểm và đưa một thông báo `CLASS_CANCELLED` qua chuông/email cho mỗi Member vào hàng đợi, nêu số điểm hoàn và tối đa 3 khóa tương tự tại thời điểm đó (cùng môn, Published, mở đăng ký, còn chỗ, chưa bắt đầu, Member chưa ghi danh; giá gần nhất trước). Gợi ý không được lưu hoặc gửi lại.

## Hỗ trợ Member và đối soát

| Verb | Route | Hợp đồng / authority |
|---|---|---|
| GET | `/api/member-support-requests/queue?status&search&assignedToUserId&page&pageSize` | FrontDesk xem toàn bộ; Admin/Coach chỉ yêu cầu được giao; `status` ∈ OPEN/ASSIGNED/IN_PROGRESS/CLOSED. Search tối đa 200 ký tự trong subject/description/tên/email Member, không phân biệt hoa thường. Response đọc thêm memberName/memberEmail/assignedToName hiện tại. |
| GET | `/api/member-support-requests/assignees` | FrontDesk; Receptionist/CenterManager đang Active. Chỉ người gọi CenterManager được nhận thêm SystemAdministrator đang Active trong danh sách. |
| POST | `/api/member-support-requests/{id}/assign`, `/progress`, `/close` | FrontDesk; khóa dòng `FOR UPDATE`, ghi audit. Progress/close yêu cầu đã phân công. CLOSED chặn assign/progress với 409 `support_request_closed`; close lặp trả 200 kết quả cũ, không đổi kết quả/audit. Mọi nhân viên FrontDesk đều thao tác được, không giới hạn theo người được giao (FR-22). |
| POST | `/api/member-support-requests` | FrontDesk tạo hộ member: `memberId, category, channel (COUNTER/PHONE/FANPAGE), description`, `subject` tùy chọn (tự sinh từ loại + mô tả), tùy chọn `assigneeUserId` + `handoverNote` để bàn giao ngay khi tạo. |
| POST | `/api/member-support-requests/{id}/assign` (body `assigneeUserId, note`) | Chuyển lên vai trò cao hơn người thao tác (Receptionist → Manager → Administrator) bắt buộc `note` ≥ 3 ký tự, lỗi 400 `support_transfer_note_required`. |
| GET | `/api/member-support-requests/{id}/events` | FrontDesk hoặc Admin/Coach được giao; lịch sử xử lý (CREATED, ASSIGNED, PROGRESS, DONE, REJECTED) kèm người thực hiện, ghi chú và thời điểm. |
| POST | `/api/support/requests` (multipart) | Member tự gửi từ trang Trợ giúp: `category, description, files[]` (tối đa 3 ảnh JPG/PNG/WebP, 5 MB/ảnh); kênh ONLINE; tạo thông báo và email xác nhận. |
| GET | `/api/support/requests/mine`, `/api/support/attachments/{id}` | Member xem yêu cầu của mình; ảnh đính kèm chỉ chủ yêu cầu, Receptionist, CenterManager hoặc Admin/Coach đang được phân công xem được. |

**Quyết định 24/10/2026 (đề xuất, chưa triển khai):** SLA 48 giờ (`slaDueAtUtc`), `POST /api/support/requests/{id}/remind`, request PAYMENT có `invoiceId` và 1–3 ảnh bắt buộc (không có trường ngân hàng; Lễ tân tạo hộ Member qua `POST /api/member-support-requests` kèm `invoiceId`, gọi được `POST /api/member-support-requests/{id}/remind` giúp Member, không xử lý được: 403 `payment_request_manager_only`), `POST /api/member-support-requests/{id}/payout` chỉ CenterManager (`amount`, `note` bắt buộc, 1–3 ảnh chứng từ), Member `confirm-received` hoặc `report-not-received`, `GET /api/members/me/unactivated-payments`, `POST /api/invoices/{invoiceId}/convert-to-points`, báo cáo `bankRefundPaid`. Hợp đồng đầy đủ, mã lỗi và dữ liệu: [Payment-Support-Workflow](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/d78ec839d92eb04d602b4e9f86ff3f41288c7f22/docs/Payment-Support-Workflow.md) mục 6–7. Không dùng như endpoint hiện hữu cho đến khi được triển khai.

Assign cho SystemAdministrator chỉ cho phép người gọi CenterManager; Receptionist bị từ chối 403 `support_admin_assignment_forbidden`, không thay đổi yêu cầu hoặc ghi audit phân công. Các role ngoài FrontDesk vẫn bị chặn bởi policy. Người nhận phải đang Active.

Hộp thư Admin và Coach dùng tổng kết trạng thái của Receptionist: **Received** (OPEN), **Assigned** (ASSIGNED), **In progress** (IN_PROGRESS), **Finished** (DONE/CLOSED) và **Declined** (REJECTED), với bộ lọc và màu badge dùng chung. Với mọi role, `stats.done` và `queue?status=FINISHED` loại REJECTED; `stats.rejected` và `queue?status=REJECTED` báo yêu cầu bị từ chối riêng. Yêu cầu bị từ chối trong lịch sử giữ kết quả và lý do từ chối gốc. Bộ lọc tùy chọn `queue?status=NOT_STARTED` vẫn gộp OPEN/ASSIGNED; `stats.notStarted` trả tổng của chúng để tương thích. Overdue là bộ lọc deadline riêng. Admin/Coach vẫn chỉ giới hạn yêu cầu hiện được giao và chỉ được thêm ghi chú tiến độ hoặc đánh dấu hoàn thành, không được tạo, phân công hoặc từ chối. Progress và Done yêu cầu ghi chú 3–1000 ký tự, khóa dòng, kiểm tra lại chủ sở hữu và từ chối yêu cầu đã kết thúc. Không đổi schema. Avatar hồ sơ công khai không còn nút Report photo; bản ghi kiểm duyệt hiện có giữ nguyên.

Khi Manager phân công cho admin, giao dịch lưu thêm thông báo InApp `SUPPORT_REQUEST_ASSIGNED` cho đúng admin đó, gồm tên Manager/Member, tiêu đề, loại và nội dung yêu cầu. Admin đọc qua chuông và `/notifications`; chỉ chủ thông báo được xem/đánh dấu đã đọc. Gán lặp cùng người nhận không tạo thêm thông báo/audit và không đưa tiến độ về ASSIGNED. Chuyển đi rồi gán lại là lần phân công mới. Thông báo mở `/admin/support?request={id}` khi yêu cầu vẫn được giao cho Admin đó; Coach mở `/coach/support?request={id}`. Admin và Coach được đọc hàng đợi, chi tiết, thống kê, lịch sử, ảnh đính kèm và gọi `/progress`, `/done` chỉ với yêu cầu được phân công; quyền này không mở rộng sang hồ sơ/gói tập Member. Không gửi email cho Admin.
| GET | `/api/users`, `/api/users/{userId}` | Receptionist luôn bị ép `role=Member`; xem chi tiết tài khoản khác Member trả 404. |
| GET | `/api/front-desk/payment-reconciliation?memberId&page&pageSize` | FrontDesk; gồm thanh toán đã xác minh chưa cấp dịch vụ và giao dịch chờ xác minh, kể cả sau khi hold hết hạn. |
| POST | `/api/front-desk/payment-reconciliation/{invoiceId}/retry` | FrontDesk; thử cấp lại dịch vụ hoặc xác minh với cổng; ghi audit `FRONT_DESK_PAYMENT_RECONCILIATION_RETRY`. |
| GET | `/api/members/me/class-holds` | Member; các giữ chỗ còn hạn có hóa đơn để tiếp tục thanh toán. |

### Recorded attendance

- `GET /api/reports/recorded-attendance?fromDate=YYYY-MM-DD&toDate=YYYY-MM-DD` chỉ dành cho CenterManager. Mặc định từ đầu tháng tới hôm nay theo giờ Việt Nam; khoảng sai, kết thúc trong tương lai hoặc dài hơn 366 ngày trả `400 invalid_date_range`.
- Response: `{fromDate,toDate,daily:[{date,uniqueMembers,gymCheckIns,classAttendances}]}`. Bao gồm mọi ngày trong khoảng, kể cả không có hoạt động. `uniqueMembers` loại trùng ID Member giữa check-in Gym và điểm danh lớp Present trong ngày đó. Lượt check-in và điểm danh buổi vẫn là số sự kiện riêng, không được cộng để ước tính số người không trùng.
- Điểm danh lớp được tính theo ngày bắt đầu buổi ở Việt Nam, không phải ngày nhập/sửa. Loại buổi và check-in tương lai; sửa điểm danh về sau có thể đổi tổng ngày trước đó. Không ước tính khách thuê sân và lượt tới không ghi nhận. Không trả dữ liệu cấp hồ sơ và không cần migration.

### Court booking volume

- `GET /api/reports/court-bookings?fromDate=YYYY-MM-DD&toDate=YYYY-MM-DD` chỉ dành cho CenterManager, mặc định toàn tháng hiện tại theo lịch Việt Nam và nhận tối đa 366 ngày lịch gồm ngày thuê tương lai. Khoảng sai trả `400 invalid_date_range`. Manager Overview yêu cầu rõ ngày hôm nay ở Việt Nam cho cả hai đầu khoảng và hiển thị giờ đã đặt theo môn. Reports mặc định cả tháng; khoản thu và điểm danh đã ghi nhận trên Overview vẫn từ đầu tháng tới hôm nay.
- Response: `{fromDate,toDate,totalBookings,totalBookedHours,bySport:[{sportId,sportName,bookings,bookedHours}]}`. Đếm mỗi lượt thuê đã trả tiền một lần theo ngày bắt đầu thuê ở Việt Nam, không phải ngày đặt/thanh toán. Hóa đơn liên kết phải Paid/PaidAfterReconciliation hoặc giữ `PaidAtUtc` đã ghi; hóa đơn trả hoàn toàn bằng điểm đủ điều kiện. Trạng thái thuê không loại lượt đặt đã trả tiền trong lịch sử: tính InUse, NoShow, hủy đã trả tiền và đơn đã trả tiền đang chờ cấp dịch vụ. Không tính hold chưa trả/hết hạn và hủy chưa trả tiền. Số giờ là tổng toàn bộ thời lượng đã đặt, gồm booking đã trả tiền cho ngày thuê sắp tới trong khoảng chọn; không xác định có mặt thực tế hoặc thời gian đã sử dụng.
- Bao gồm mọi môn active trong danh mục, kể cả môn không có dịch vụ thuê hoặc không có booking (số lượt/giờ bằng 0), cùng môn inactive lịch sử có lượt thuê đã trả tiền đủ điều kiện. Thêm môn khiến nó tự xuất hiện; không có danh sách Badminton/Basketball cố định. Không cần migration schema.

### Manager Reports dashboard — filtered analysis

- Mọi kỳ báo cáo dùng ngày lịch Việt Nam bao gồm hai đầu, tối đa 366 ngày. Reports mặc định toàn tháng hiện tại cho ngày thuê và ngày bắt đầu lớp. Request tài chính/mua hàng kết thúc hôm nay; không hiển thị ngày tương lai là doanh thu bằng 0. `Compare with: Previous month` căn chỉnh số ngày tài chính đã qua khi kỳ chính là tháng hiện tại, còn so sánh vận hành dùng trọn tháng lịch. Kỳ so sánh tùy chỉnh tường minh giữ ngày đã chọn (ngày kết thúc tài chính vẫn giới hạn hôm nay). Thời lượng không bằng nhau và mốc gốc bằng 0 chỉ hiển thị chênh lệch tuyệt đối; chênh lệch ghi danh dùng điểm phần trăm.
- Tiền thu của Revenue dùng ngày thanh toán; hoàn tiền là các khoản chuyển khoản Support Request đã ghi (`MemberSupportPayout.Amount > 0`), dùng `RecordedAtUtc` kể cả khi thanh toán gốc ở kỳ khác. Tính ngay sau ghi nhận, không chờ Member xác nhận. Payout liên kết item hóa đơn dùng các chiều của item đó; payout cấp hóa đơn không quy được nguồn đáng tin cậy giữ Unclassified và bị loại bởi bộ lọc môn/nguồn trừ khi nguồn đó khớp. Không cộng hoàn tiền PaymentAdjustment cũ vào KPI này. Hoàn điểm không làm giảm `netRevenue`. `bankRefundPaid`/`bankRefundCount` chỉ là alias tương thích; chỉ số Bank payouts recorded riêng đã được gỡ. Số dư ví hiện tại chỉ giới hạn theo Member (khi chọn), không theo ngày, môn hoặc nguồn.
- Court bookings nhận `sportId` và `memberId` tùy chọn, trả `daily[{date,bookings,bookedHours}]` cùng tổng/bySport hiện có. Cùng điều kiện hóa đơn đã thanh toán áp dụng cho dòng ngày, tổng, dòng môn và export, độc lập trạng thái thuê. Toàn bộ thời lượng theo lịch tính là giờ đặt, không phải thời gian sử dụng đo được. Bao gồm ngày thuê tương lai trong kỳ chọn. Khoản thu thuê dùng khoảng ngày thanh toán đã qua riêng. Nhãn KPI Reports và cột export là `Paid bookings`.
- Active Gym members dùng `/api/reports/membership-summary?asOfDate=<today>` độc lập kỳ chọn/so sánh: Member không trùng có Membership Active hợp lệ hôm nay, loại vé ngày. Không có chênh lệch kỳ trước.
- Loại export `COURT_BOOKINGS` (`sportName,bookings,bookedHours`) và `CLASS_STATUS` (`name,sportName,startDate,status,confirmedCount,capacity,thresholdStatus`) dùng chung bộ đọc với API tương ứng. `MEMBERSHIP_PERIOD` hỗ trợ thêm `dayPassesSold`. Export doanh thu summary/daily/dimension nhận Sport/Source/Member bằng cùng cách tổng hợp của dashboard. PDF và CSV dùng dòng dữ liệu giống nhau; export chính và so sánh là file riêng có kỳ thực tế và bộ lọc đã áp dụng. Không đổi schema hoặc migration.
- Nhãn Reports: `Money received` = `totalCollected`; `Cash refunded` = `totalRefunded`; `After refunds` = `netRevenue`. After refunds loại biến động điểm và không trừ chi phí vận hành. Tên trường API không đổi; Overview giữ nhãn khoản thu riêng, gồm `Net collected`.
- Biểu đồ ngày dùng đầy đủ lịch của tháng/kỳ tùy chỉnh đã chọn, gồm ranh giới tháng và ngày tương lai. Ngày đã ghi nhận không hoạt động giữ 0; ngày ngoài dữ liệu tài chính/mua hàng được trả để trống và ghi `No data yet`, không bao giờ vẽ là 0 hoặc doanh thu dự báo. Request và tổng tài chính vẫn kết thúc hôm nay. Chuỗi so sánh đã qua giữ ngày riêng và dừng khi hết dữ liệu. Lựa chọn hoàn toàn ở tương lai hiển thị biểu đồ lịch trống.
- Manager Overview hiển thị `Overdue requests` trong dải KPI đầu trang bằng `GET /api/member-support-requests/stats.overdue`. Làm mới cùng các thống kê Overview khác, hiển thị Unavailable khi lỗi và dùng hành động Retry chung. Link yêu cầu quá hạn riêng dưới dải đã được gỡ.

`daily[]` của Membership-period trả thêm `dayPassesSold`: gói vé ngày hợp lệ đã thanh toán được đếm một lần mỗi gói theo ngày thanh toán ở Việt Nam, dùng cùng điều kiện với tổng Day passes sold của kỳ. `buyers` vẫn là số người mua membership không trùng mỗi ngày, loại vé ngày. Biểu đồ Members Gym purchases by day hiển thị hai chuỗi riêng về người và vé; cả hai không đo điểm danh. Thiếu dữ liệu vé ngày theo ngày từ API cũ hiển thị Unavailable, không phải 0. Không đổi schema/migration.

Hồ sơ Coach công khai nhóm tài liệu đã xuất bản thành **Certificates** (chứng nhận chuyên môn) và **Awards** (công nhận/thành tích), với trạng thái trống rõ ràng khi chưa chia sẻ tài liệu. Quản lý tải lên của Coach nhóm hai loại và hiển thị giới hạn chung: 20 tài liệu chưa xóa, mỗi file tối đa 10 MiB (JPG/PNG/WebP/PDF). Tài liệu chưa xuất bản/đã xóa vẫn bị loại khỏi API và thư viện công khai.

Hồ sơ Coach công khai dùng giao diện Court & Volt hiện tại với bookmark Overview, Certificates và Awards (`?tab=overview|certificates|awards`). Overview hiển thị giới thiệu và phương pháp huấn luyện đã lưu. Bookmark chứng chỉ/giải thưởng hiển thị tài liệu đã xuất bản tương ứng; bản xem trước khi hover Coach liên kết thẳng tới Certificates. Bố cục hồ sơ cũ được thay thế.

Chi tiết Coach công khai là view độc lập không có navbar trang chủ. Back dùng đường dẫn local `returnTo` đã xác thực; link từ bản xem trước Coach giữ pathname/query/hash nguồn, xem trước tài khoản quay về `/account`. Đổi bookmark hồ sơ giữ `returnTo`. Đích ngoài/không hợp lệ và link quay lại cùng hồ sơ chuyển về mục SportHub ở trang chủ (`/#sporthub`). Trang danh bạ `/coaches` được gỡ; hồ sơ công khai riêng `/coaches/{coachId}` vẫn khả dụng.

Tên/chân dung Coach trong view lớp vẫn là bản xem trước khi hover/focus và là link thực tới hồ sơ Coach công khai. Nhấp/chạm hoặc Enter mở hồ sơ; `returnTo` giữ URL lớp/dịch vụ/lịch nguồn. Avatar và tên bên trong bản xem trước cũng liên kết tới hồ sơ.

Chi tiết tài khoản Admin chỉ cung cấp **Remove photo** cho tài khoản Member/Coach có avatar tải lên. Hộp thoại xác nhận gọi `DELETE /api/admin/users/{userId}/avatar`; thành công thì tải lại tài khoản và hiển thị avatar mặc định. Giới hạn role/đối tượng ở backend, ghi audit và dọn Cloudinary bền vững vẫn được áp dụng; không cần có báo cáo trước khi xóa.

### Private avatar images in Admin audit details

- Entry UPDATE_AVATAR, DELETE_AVATAR, IMPORT_GOOGLE_AVATAR và MANAGER_REMOVE_MEMBER_AVATAR mới ghi ID ảnh riêng tư trước/sau cùng metadata hết hạn trong cùng lần lưu database với thay đổi avatar. Byte ảnh được giữ trong `avatar_audit_images`, không bao giờ trong giá trị JSON audit hoặc URL Cloudinary công khai. Tối đa 5 MiB mỗi ảnh, chỉ JPG/PNG/WebP; sau tải lên, ảnh gửi được giữ, ảnh hiện có được sao từ evidence đã lưu hoặc Cloudinary tenant đã cấu hình trước khi xóa thông thường. Ảnh cũ không còn khả dụng không ngăn kiểm duyệt avatar; hiển thị Image no longer available.
- GET `/api/admin/audit-avatar-images/{imageId}` chỉ dành cho Admin, trả ảnh với `Cache-Control: private, no-store` và nosniff. Ảnh không có/hết hạn trả 404. Bắt buộc xác thực và policy SystemAdministrator, kể cả khi biết ID ảnh. Quyền truy cập hết hạn sau 30 ngày; worker dọn avatar hiện có xóa dòng hết hạn mỗi mười phút. URL avatar thông thường tiếp tục được xóa qua hàng đợi xóa Cloudinary bền vững hiện có.
- Admin Audit log → View details hiển thị panel ảnh Before/After, tải blob có xác thực và thu hồi URL bộ nhớ khi đóng. Avatar đã xóa/mặc định hiển thị No profile photo. Sự kiện cũ không có evidence ảnh vẫn đọc được nhưng không thể khôi phục file đã xóa. Sự kiện avatar Member chỉ được thêm vào phạm vi audit của Admin; sự kiện vận hành Member khác vẫn ngoài phạm vi. Manager không được truy cập ảnh riêng tư.
- Áp dụng migration `AvatarAuditImages` trước khi triển khai tính năng. Chỉ thêm bảng lưu ảnh và chỉ mục; không điền bù hoặc thay đổi bản ghi audit hiện có.

### Manager avatar, support settings and current operational health

- Center Manager và System Administrator có thể tải lên/đổi/xóa avatar của mình trong Account qua trình cắt Cloudinary và API avatar hiện có (5 MiB, JPG/PNG/WebP). API avatar của họ yêu cầu xác thực. Lưu ảnh audit riêng tư dùng cùng luồng evidence avatar. Endpoint tự phục vụ luôn dùng ID tài khoản đang xác thực; endpoint kiểm duyệt hiện có giữ giới hạn đối tượng riêng.
- Operating settings cung cấp đúng ba khóa hỗ trợ hiện có: `support.sla_hours` (1–168 giờ, mặc định 48), `support.remind_cooldown_hours` (1–168 giờ, mặc định 48), `support.payout_confirm_days` (1–30 ngày, mặc định 3). Yêu cầu Member/nhân viên mới, cập nhật tiến độ thực tế và yêu cầu payout mở lại dùng SLA đã cấu hình. Đổi cấu hình hoặc chuyển giao yêu cầu không ghi lại deadline đã lưu. Nhắc của Member/nhân viên và job SLA dùng chung cooldown cấu hình và loại trùng theo từng người nhận. Yêu cầu thông thường thông báo người được giao đang Active, hoặc các Manager và Receptionist Active khi không có người được giao Active; PAYMENT chỉ thông báo Manager được giao đang Active, hoặc mọi Manager Active. Không thêm khóa thứ tư hoặc seed cấu hình mới.
- Health vận hành Manager thay số đếm bồi hoàn gateway-event cũ bằng `payments.paidNotActivated`, đếm hóa đơn không trùng có sự kiện Unactivated đã xác minh thành công, khớp hàng đợi quầy. `support.overdueRequests` đếm yêu cầu đã quá hạn SLA và không DONE/CLOSED/REJECTED. Health ít nhất là Degraded khi một trong hai tồn đọng khác 0. Quyền đọc health hiện có vẫn chỉ dành cho Manager.
- Tên audit bao phủ tường minh PAYMENT_NOT_ACTIVATED, REMIND_SUPPORT_REQUEST, SUPPORT_PAYOUT_RECORDED/CONFIRMED/NOT_RECEIVED/AUTO_CONFIRMED, COMPLETE/REJECT_SUPPORT_REQUEST và CONVERT_UNACTIVATED_PAYMENT_TO_POINTS. Sự kiện payout/xác nhận/nhắc/chuyển điểm mới có tham chiếu yêu cầu/hóa đơn an toàn, số tiền/điểm, kết quả xác nhận hoặc số lần nhắc, không có thông tin ngân hàng. Admin xem được sự kiện xác nhận, nhắc và chuyển điểm Member trong phạm vi hẹp cùng sự kiện ảnh; thao tác Member không liên quan vẫn bị loại. Nhãn sự kiện cũ đọc được kể cả khi sự kiện cũ thiếu chi tiết.

### API Manager tặng điểm

Tất cả route sau yêu cầu CenterManager.

**Điều kiện người nhận:** tài khoản Member Active đã mua gói Gym, gồm cả Gym day pass. Backend xét MemberPackage Active/Expired hoặc Cancelled có hóa đơn Paid/PaidAfterReconciliation; không hiện tài khoản chưa mua gói hoặc chỉ chờ thanh toán. Khi có incidentId, chỉ lấy Membership Active giao với thời gian đóng Gym hoặc người có day pass trong snapshot sự cố; requiresAgreement đánh dấu người có vé trong snapshot và alreadyGifted đánh dấu đã tặng cho sự cố đó. POST kiểm lại điều kiện trước khi cộng ví.

**Giao diện Gift points:** chỉ tìm tên/email và lọc gói; không tìm Member ID, không có Membership status, Affected period from hoặc Through date. Danh sách có vùng cuộn ẩn thanh cuộn, ô chọn bên phải và không có cột status. Không có Apply to selected. Common points amount là ô nhập văn bản, không tăng/giảm bằng cuộn chuột; báo lỗi ngay với số âm, số lẻ, số không hợp lệ hoặc vượt maxPoints. Review gift mở hộp thoại gồm danh sách người nhận, điểm từng người và lý do; không hiển thị dòng Total recipients/points. Các tham số lọc nâng cao bên dưới vẫn được backend hỗ trợ, không bắt buộc có trên UI.

**Giới hạn động:** PointGiftLimitService gọi IRevenueReportService.GetAsync từ đầu tháng đến hôm nay theo lịch Việt Nam, lấy NetRevenue cùng nguồn Net collected của Overview. maxPoints = min(int.MaxValue, floor(max(0, netCollectedVnd) / vndPerPoint)); hiện vndPerPoint = 1.000. Đây là giới hạn cho từng người nhận, không phải ngân sách tổng đợt tặng. Frontend lấy limits và báo lỗi ngay khi nhập vượt; POST tính lại giới hạn, trả 400 point_gift_limit_exceeded nếu vượt. Không code cứng doanh thu hoặc maxPoints.


| Method | Route | Hợp đồng |
|---|---|---|
| GET | `/api/manager/point-gifts/limits` | `{fromDate,toDate,netCollectedVnd,vndPerPoint,maxPoints}`; giới hạn điểm động theo Net collected tháng hiện tại. |
| GET | `/api/manager/point-gifts/filters` | Danh sách packageId/name cho bộ lọc. |
| GET | `/api/manager/point-gifts/recipients` | `incidentId?,keyword?,packageId?,membershipStatus?,fromDate?,toDate?,page,pageSize`. PagedResult gồm memberId/name/email/requiresAgreement/alreadyGifted. Bộ lọc không tự chọn người nhận. |
| POST | `/api/manager/point-gifts` | `{batchId,incidentId?,reason,recipients:[{memberId,points,memberAgreed}]}`; 1–100 người khác nhau, lý do 3–1000 ký tự, points nguyên dương. 200 `{batchId,recipients,totalPoints}`. Cộng ngay trong một transaction. Cùng batch/payload retry trả kết quả cũ; khác payload 409 point_gift_batch_conflict. Incident + Member trùng 409 incident_member_already_gifted; cần đồng ý mà chưa ghi nhận 400 day_pass_gift_agreement_required. |
| GET | `/api/manager/point-gifts` | Lịch sử phân trang, lọc incidentId, có người nhận, Manager, điểm, lý do, thời gian, ledgerEntryId. |

---

## AI Member/Coach — contract bổ sung cho bản vá 02/11/2026

> Đây là ghi chú đối chiếu **bản vá dự kiến tích hợp**, không thay thế OpenAPI runtime. Khi khác nhau, kiểm tra controller đang được deploy. Các đoạn cũ bên dưới ghi `scope=session` duy nhất phản ánh phiên bản trước khi mở rộng Personal Plan.

| Luồng | Route | Quyền và hành vi cần kiểm tra |
|---|---|---|
| Coach AI lesson suggestion | `POST /api/coaches/me/teaching/classes/{classId}/suggestion` | Coach được phân công. `scope=session` gắn buổi hợp lệ; `scope=personal` gắn `memberId` thuộc lớp/Coach hợp lệ. Kết quả là suggestion, không tự lưu. |
| Coach member workout suggestion (legacy/khác luồng) | `POST /api/ai/workout-suggestions/{memberId}` | Coach có quan hệ cho phép; không nhầm API này với nút AI lesson suggestion của class teaching editor. |
| AI chatbot | `POST /api/ai/chat` | Chỉ bật UI chatbot cho Member/Coach; truy vấn navigation phải lọc theo role từ backend context và không mở quyền API nghiệp vụ mới. |
| AI logs | `GET /api/ai/logs` | Quyền endpoint xác định ở backend, không diễn giải là một nút giao diện Member/Coach. |

**Goal validation:** giới hạn input cơ bản → phân loại `VALID`, `VAGUE`, `INVALID` → chỉ sinh giáo án cho VALID hoặc VAGUE. VAGUE phải dùng context môn và trình độ, INVALID trả message, không gọi bước tạo giáo án. Không được coi AI validator là cơ chế xác thực quyền, và cần kiểm thử xử lý lỗi provider/JSON/timeout.

**Navigation Knowledge:** API-contract chỉ mô tả endpoint và quyền; câu hỏi “button nằm ở đâu?” cần thêm mapping `role → page → tab → button → handler → API`. Chỉ đưa các mapping đã xác nhận vào câu trả lời. Kiểm tra đường dẫn thực tế của frontend trước khi khẳng định route đã có nút.



### Manager Reports — Cash and Points

- Reports tách Revenue thành Cash (`tab=revenue`, giữ cho link hiện có) và Points (`tab=points`), tiếp theo là Courts, Classes và Gyms. Ngày và kỳ so sánh dùng chung. More filters dùng cùng phạm vi trên dashboard và trang export: Cash = Sport / Collection source / Member; Points = Member; Courts = Sport / Member; Classes = Sport; Gyms = không có bộ lọc nâng cao. Lựa chọn môn dùng đầy đủ danh mục Manager, gồm môn inactive cho báo cáo lịch sử. Nhãn nguồn thu dùng chung với biểu đồ và giữ nhóm reconciliation, transfer, unclassified riêng.
- Mỗi bookmark giữ riêng bộ lọc nâng cao nháp/đã áp dụng và trạng thái mở rộng. Đổi trường không tải lại dữ liệu báo cáo cho tới khi Apply; thay đổi đang chờ hiển thị Changes not applied và chặn Export. Chip và số bộ lọc đã áp dụng vẫn hiển thị khi đóng More filters. Reset filters xóa ngay chỉ bộ lọc nâng cao của bookmark đang mở, giữ ngày/so sánh chung và bookmark khác. Lỗi tra cứu cung cấp Retry và không bao giờ âm thầm bỏ bộ lọc đã chọn.
- Link export chỉ mang kỳ/so sánh đã áp dụng của bookmark đang mở và ID bộ lọc được hỗ trợ trong URL, không có tên/email Member. Trang export khôi phục loại báo cáo và bộ lọc đúng phạm vi, tra Member đã chọn để hiển thị và dùng cùng quy tắc pending/Apply/Reset. Cash summary gửi môn/nguồn/Member tới API export hiện có, không chỉ export theo chiều. Courts cố định nguồn Rental; Points chỉ gửi Member; Classes chỉ gửi môn; Gyms không gửi cả hai. Kỳ so sánh và kỳ chính được export thành job riêng với ngày báo cáo thực tế (hoạt động tài chính giới hạn hôm nay, so sánh tháng trước theo phần thời gian đã qua khi phù hợp; ghi danh lớp giữ toàn kỳ chọn). Không thể export kỳ tài chính hoàn toàn ở tương lai. Số dư điểm hiện tại vẫn là quan sát hiện tại, độc lập ngày hoạt động được export.
- Cash hiển thị tiền nhận, khoản hoàn chuyển khoản đã ghi và tiền sau hoàn, với chi tiết thu chỉ về tiền. Hoạt động điểm không còn bị lặp trong Cash.
- Points thay summary chung trước đây hiển thị trong Point Management: điểm dùng, toàn bộ điểm phát hành, điểm hoàn (một phần của điểm phát hành), điều chỉnh Manager và số dư khả dụng+đang giữ hiện tại. Số dư hiện tại có timestamp quan sát và không so sánh với kỳ lịch sử. Point Management giữ tặng điểm, tìm Member và thao tác ví/ledger cá nhân, cùng link Reports · Points.
- Dòng Revenue theo ngày trả thêm `pointsRedeemed`, `pointsIssued`, `pointsRefunded` và `managerPointAdjustment`; tổng của chúng khớp tổng kỳ tương ứng. Biểu đồ Points dùng đơn vị điểm nguyên, không phải VND.
- Export `POINTS_SUMMARY` hỗ trợ `fromDate,toDate,pointsRedeemed,pointsIssued,pointsRefunded,managerPointAdjustment,outstandingPoints,outstandingPointsAsOfUtc`, theo phạm vi ngày/Member. Export Cash dùng `REVENUE_SUMMARY` với cột thu/hoàn/ròng. Cột export cũ và file đã lưu vẫn được hỗ trợ; không cần migration schema.

### Shared database development mode

- `SharedDatabase__Enabled=true` là chế độ Development bật theo lựa chọn cho API local kết nối database server dùng chung. Bỏ migration/seed tự động lúc khởi động và không đăng ký background job API (kể cả dọn ảnh dịch vụ); server đã triển khai vẫn phụ trách thông báo/email, đối soát, hết hạn, nhắc, hoàn trả và dọn dẹp. Mặc định false; hành vi server và local độc lập thông thường không đổi. Production từ chối chế độ này trước khi mở database; lệnh seed/bootstrap cũng từ chối khi chế độ được bật.
- Chế độ này không phải chỉ đọc: request API đã xác thực vẫn ghi bình thường vào database dùng chung. Cần worker server đang chạy để xử lý tự động. Payload email do một trong hai API tạo vẫn cần khóa Data Protection/cấu hình ứng dụng tương thích để worker giải mã. Không thay đổi xác minh chữ ký VNPay hoặc vô hiệu endpoint xác minh checkout tường minh. Không đổi schema hoặc migration.


### Service cover and gallery images

- Chủ sở hữu thư viện: `membership` (MembershipPackage), `class` (Class), `room` (Room vật lý; độc lập với môn). Một ảnh bìa và tối đa năm ảnh bổ sung.
- `GET /api/service-images?kind=class&ids=1,2`: tối đa 100 ID dương mỗi request; trả `{items: ServiceGallery[]}` cho dịch vụ hiển thị công khai.
- `GET /api/service-images/{kind}/{id}`: thư viện đã xuất bản cho dịch vụ đủ điều kiện. Dịch vụ hiện có chưa có ảnh trả thư viện rỗng. Không công khai dịch vụ riêng tư/nháp.
- `GET /api/coaches/me/class-images?ids=1,2`: đọc hàng loạt chỉ dành cho Coach, tối đa 100 ID lớp dương. Dùng cùng phạm vi phân công lớp Published/InProgress như `/api/coaches/me/classes`, gồm phân công dạy thay không bị hủy. Chỉ trả ảnh đã xuất bản; bỏ lớp không liên quan, Draft và Completed. Không cấp quyền dữ liệu học viên hoặc sửa ảnh. Coach và Member dùng lại ảnh bìa/thứ tự Manager đã lưu, làm mới lượt đọc ảnh khi quay lại cửa sổ trình duyệt.
- `GET /api/manager/service-images/{kind}/{id}`: thư viện đã xuất bản chỉ dành cho CenterManager, gồm chủ sở hữu nháp/inactive.
- `POST /api/manager/service-images/{kind}/{id}`: multipart `file`, `altText` tùy chọn; đưa ảnh vào vùng tạm mà không đổi thư viện Member. JPG/PNG/WebP, tối đa 10 MB, khung ngang 3:2. Trả `ServiceImage`.
- `PUT /api/manager/service-images/{kind}/{id}`: `{version, imageIds, coverImageId, altTexts?}` xuất bản lựa chọn có thứ tự một cách nguyên tử. Thư viện không rỗng cần chọn một ảnh bìa; thư viện rỗng cần ảnh bìa null. Trả thư viện đã lưu. Phiên bản cũ trả 409 `service_image_version_conflict`.
- `DELETE /api/manager/service-images/{kind}/{id}/{imageId}`: bỏ ảnh tải tạm của Manager gửi request. Ảnh đã xuất bản được xóa qua PUT.
- `ServiceGallery`: `{kind, id, version, coverImageId, images}`. Version là chuỗi số thập phân bắt đầu từ `"0"`. `ServiceImage`: `{imageId, url, altText, sortOrder, width, height}`.
- Giới hạn vùng tạm: 12 mỗi chủ sở hữu; ảnh tải không dùng hết hạn sau 24 giờ. Tài nguyên được tham chiếu/đã gỡ dùng cơ chế dọn dẹp bền vững có retry; tham chiếu môn/landing dùng chung được bảo vệ. Chế độ SharedDatabase vô hiệu worker dọn dẹp.
- Thay đổi thư viện không đổi vòng đời lớp, lịch, sức chứa hoặc giá. Manager có thể sửa ảnh lớp đã xuất bản.
- Response item hóa đơn được phép truy cập thêm `serviceImageKind` và `serviceImageId` có thể null; liên kết membership/phòng được tra hàng loạt và giới hạn theo người hưởng hóa đơn. Các trường này không thay đổi snapshot tài chính đã lưu.
- Áp dụng migration EF bổ sung trước khi dùng các endpoint này. Không cần seed hoặc reset database dùng chung.
