```mermaid
flowchart TD
A[User mở ứng dụng PWA] --> B{IndexedDB đã sẵn sàng?}
B -- Chưa có --> C[Tạo database & object stores]
B -- Đã có --> D[Tải metadata (createdAt, expireAt)]

    D --> E{Có dữ liệu hết hạn?}
    E -- Có --> F[Xoá dữ liệu hết hạn (cleanupExpired)]
    E -- Không --> G[Tiếp tục hoạt động bình thường]

    F --> G

    G --> H[User thao tác: lưu dữ liệu mới]
    H --> I[Ghi dữ liệu kèm createdAt và expireAt]

    I --> J{Định kỳ kiểm tra?}
    J -- Có --> K[Chạy cleanupIndexedDB()]
    J -- Không --> L[Kết thúc vòng hoạt động]

    K --> L

    style F fill:#ffdddd,stroke:#e44,stroke-width:2px
    style I fill:#ddffdd,stroke:#4b4,stroke-width:2px
    style K fill:#ffffcc,stroke:#cc0,stroke-width:2px
```
