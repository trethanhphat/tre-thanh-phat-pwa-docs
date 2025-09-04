```mermaid
flowchart TD
A["Người dùng"]
B["Danh sách lô"]

A --> |"Mở menu"| B
B --> |"Không có mang"| C["Dữ liệu offline"]
B --> |"Có internet"| D["Tải dữ liệu trong nền"]

```
