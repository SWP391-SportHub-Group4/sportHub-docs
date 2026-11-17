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
| [`architecture/`](architecture/) | [Thiết kế hệ thống](architecture/Center-Management-System-Design-v3.md) — kiến trúc, dữ liệu, phân quyền và luồng nghiệp vụ; [Architecture Decision Records](architecture/adr/README.md) — lý do lựa chọn công nghệ và quyết định kiến trúc |
| [`database/`](database/) | [Database Design](database/README.md) — ERD, schema, bảng/cột/khóa/ràng buộc và chỉ mục |
| [`api/`](api/) | [API contract](api/api-contract.md) — endpoint, DTO và quy tắc tích hợp |
| [`usecases/`](usecases/) | [Use Cases theo 6 flow của SRS](usecases/README.md) — mỗi flow có luồng chính, nhánh lỗi, điều kiện, nghiệm thu và truy vết UC/BR; có [mẫu](usecases/TEMPLATE.md) |
| [`operations/`](operations/) | [RUNBOOK](operations/RUNBOOK.md) và [kịch bản demo 6 flow](operations/LOCAL-DEMO-SCRIPT-6-FLOWS.md) |
| [`verification/`](verification/) | [AI Member/Coach — Implementation & Verification](verification/AI-Member-Coach-Implementation-Verification.md) |
| [`assets/screenshots/`](assets/screenshots/) | Ảnh giao diện dùng để minh họa dự án |

Khi thay đổi hành vi trong code, cập nhật tài liệu tương ứng: quy tắc ở `requirements/`, hợp đồng ở `api/`, luồng triển khai ở `usecases/`, cấu trúc hệ thống ở `architecture/`, quyết định kiến trúc ở `architecture/adr/`, schema ở `database/` và cách chạy ở `operations/`. Các liên kết tới mã nguồn trỏ về repository ứng dụng.

## Cấu trúc hiện có

```text
SportHub-Docs/
├── requirements/    Source of Truth, Business Rules, đặc tả chức năng và phi chức năng
├── architecture/    Thiết kế hệ thống, ADR
├── database/        ERD, schema, bảng/cột/ràng buộc, chỉ mục
├── api/             API contract
├── usecases/        6 flow UC theo SRS và mẫu UC
├── operations/      RUNBOOK, kịch bản demo
├── verification/    Bằng chứng đối chiếu AI Member/Coach
└── assets/          Ảnh minh họa
```

Các nhóm trong sơ đồ cấu trúc chưa có tài liệu tương ứng (`security/`, `ui-ux/`, `testing/`, `deployment/`) chưa được tạo thành thư mục rỗng hoặc file mẫu. `Troubleshooting` và `Maintenance Guide` cũng chưa có, nên mục `operations/` chỉ liệt kê tài liệu hiện hữu.
