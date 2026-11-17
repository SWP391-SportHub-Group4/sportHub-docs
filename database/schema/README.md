# Từ điển schema

> Nguồn: `SportHubDbContextModelSnapshot.cs` tại commit `8bbe16574197a8863150dab3a1b37e6d2fbf5bc5` của `main` repo ứng dụng. Mỗi trang ghi tất cả cột, kiểu PostgreSQL, nullable, PK, FK, index và check constraints do EF biết. Một số constraint SQL thủ công nằm trong [Database Design](../README.md).

| Module | Bảng | Cột | FK | Index phụ | Check |
| --- | ---: | ---: | ---: | ---: | ---: |
| [administration](administration.md) | 3 | 23 | 2 | 2 | 2 |
| [ai](ai.md) | 2 | 16 | 1 | 2 | 2 |
| [api](api.md) | 1 | 4 | 1 | 0 | 0 |
| [audit](audit.md) | 1 | 10 | 1 | 2 | 0 |
| [identity](identity.md) | 16 | 122 | 23 | 31 | 3 |
| [membership](membership.md) | 4 | 44 | 6 | 7 | 3 |
| [notification](notification.md) | 2 | 25 | 2 | 5 | 0 |
| [payment](payment.md) | 12 | 154 | 38 | 49 | 15 |
| [scheduling](scheduling.md) | 25 | 246 | 57 | 71 | 29 |
| [training](training.md) | 3 | 25 | 7 | 7 | 0 |
| **Tổng** | **69** | **669** | **138** | **176** | **54** |

Các FK ghi theo entity đích và tên bảng vật lý đã ánh xạ. Những cột không có FK thật nhưng chứa ID theo quy ước nghiệp vụ vẫn được ghi là cột thường; không suy diễn quan hệ DB từ tên cột.
