# SportHub Docs

Tài liệu kỹ thuật của [SportHub — hệ thống quản lý trung tâm thể thao](https://github.com/SWP391-SportHub-Group4/sports-center-management-system). Mã nguồn ứng dụng nằm ở repository liên kết; repository này chỉ chứa tài liệu và ảnh minh họa.

## Bắt đầu đọc

1. [Source of Truth](requirements/00-Source-of-Truth.md) — phạm vi, thuật ngữ và quyết định nghiệp vụ.
2. [Business Rules](requirements/SportManagement_BusinessRules_v2.0_updated.docx) — quy tắc nghiệp vụ bản Word.
3. [Functional & Nonfunctional Specification](requirements/Functional-and-Nonfunctional-Spec.md) — yêu cầu chức năng và chất lượng.

## Theo mục đích

| Thư mục | Tài liệu |
| --- | --- |
| [`requirements/`](requirements/) | [Source of Truth](requirements/00-Source-of-Truth.md), [Business Rules](requirements/SportManagement_BusinessRules_v2.0_updated.docx), [Functional & Nonfunctional Specification](requirements/Functional-and-Nonfunctional-Spec.md) |
| [`architecture/`](architecture/) | [Thiết kế hệ thống](architecture/Center-Management-System-Design-v3.md) — kiến trúc, dữ liệu, phân quyền và luồng nghiệp vụ |
| [`api/`](api/) | [API contract](api/api-contract.md) — endpoint, DTO và quy tắc tích hợp |
| [`operations/`](operations/) | [RUNBOOK](operations/RUNBOOK.md) và [kịch bản demo 6 flow](operations/LOCAL-DEMO-SCRIPT-6-FLOWS.md) |
| [`verification/`](verification/) | [AI Member/Coach — Implementation & Verification](verification/AI-Member-Coach-Implementation-Verification.md) |
| [`assets/screenshots/`](assets/screenshots/) | Ảnh giao diện dùng để minh họa dự án |

Khi thay đổi hành vi trong code, cập nhật tài liệu tương ứng: quy tắc ở `requirements/`, hợp đồng ở `api/`, cấu trúc hệ thống ở `architecture/` và cách chạy ở `operations/`. Các liên kết tới mã nguồn trỏ về repository ứng dụng.
