```mermaid
flowchart TD

%% ======================= 1. LOAD LOCAL ===========================
A1["🗄 loadProductsFromDB()"] -->|Đọc IndexedDB (STORE_PRODUCTS)| A2["Danh sách sản phẩm offline"]
A2 --> A3["Render bảng ProductsTable"]

%% ======================= 2. SYNC & UPDATE ===========================
B1["🌐 Gọi API hoặc nguồn sheet"] --> B2["syncProducts(products)"]
B2 --> B3["So sánh dữ liệu cũ - mới trong IndexedDB"]
B3 -->|Xoá mục cũ| B4["Xóa record không còn"]
B3 -->|Thêm/Cập nhật mới| B5["Lưu sản phẩm mới vào STORE_PRODUCTS"]

%% ======================= 3. CACHE ẢNH ===========================
B5 --> B6["ensureProductImageCachedByUrl(image_url)"]
B6 -->|Đã có trong IndexedDB| B7["Bỏ qua tải lại"]
B6 -->|Chưa có| B8["waitForImageLoadThenFetchBlob(url)"]
B8 -->|Fetch thành công| B9["Lưu Blob → STORE_PRODUCT_IMAGES"]
B8 -->|Fetch lỗi / CORS| B10["↻ Thử lại qua /api/image-proxy?url=..."]

%% ======================= 4. PREFETCH TOP ẢNH ===========================
B2 --> B11["prefetchProductImages(top5)"]
B11 --> B6

%% ======================= 5. HIỂN THỊ ===========================
A3 --> C1["useImageCacheTracker()"]
C1 -->|Có blob| C2["URL.createObjectURL(blob)"]
C1 -->|Không có blob| C3["Dùng image_url hoặc proxy_url"]

C2 --> C4["Hiển thị ảnh từ IndexedDB (offline-ready)"]
C3 --> C4

style A1 fill:#f7f7f7,stroke:#333,stroke-width:1px
style B1 fill:#e3f2fd,stroke:#1565c0,stroke-width:1px
style B8 fill:#ffebee,stroke:#b71c1c,stroke-width:1px
style B10 fill:#fff3cd,stroke:#b58900,stroke-width:1px
style C4 fill:#e8f5e9,stroke:#2e7d32,stroke-width:1px
```
