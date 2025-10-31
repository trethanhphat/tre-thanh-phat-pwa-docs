<<<<<<< HEAD

# Tre Thanh Phat PWA Docs

(Mô hình hoá quá trình hoạt động của ứng dụng Thanh Phát Bamboo PWA)
Tài liệu này hướng dẫn xây dựng ứng dụng PWA nhằm hỗ trợ quản lý và chăm sóc cây tre hiệu quả. Repo cung cấp các hướng dẫn kỹ thuật, cấu trúc dự án, và các tính năng chính giúp người dùng theo dõi, cập nhật thông tin, và tối ưu hóa quy trình chăm sóc cây tre.

## Tóm tắt các tính năng

- Quản lý thông tin cây tre: thêm, sửa, xóa, và xem chi tiết từng cây.
- Theo dõi lịch sử chăm sóc: ghi nhận các hoạt động như tưới nước, bón phân, cắt tỉa.
- Thông báo nhắc nhở lịch chăm sóc định kỳ.
- Quản lý vị trí cây tre bằng bản đồ.
- Hỗ trợ đăng nhập, phân quyền người dùng.
- Tìm kiếm và lọc cây tre theo trạng thái, vị trí, hoặc loại cây.
- Lưu trữ dữ liệu ngoại tuyến, đồng bộ khi có kết nối mạng.
- Giao diện thân thiện, tối ưu cho thiết bị di động.
- Quản lý hình ảnh cây tre và các hoạt động chăm sóc.
- Báo cáo tổng hợp tình trạng và lịch sử chăm sóc cây tre.

---

# Bạn có thể bắt đầu chỉnh sửa hoặc bổ sung các tính năng mới tại đây.

- Mở ứng dụng lần đầu có mạng và đã có sw actived nhưng khi offline mở các trang khác như batches hay products hoặc product/[id] thì vẫn báo không có mạng --> Cần xem lại liệt kê các trang có cache

### ✅ Kế hoạch cải tiến hệ thống PWA cho project Tre Thanh Phát

#### 1️⃣ Service Worker - Bổ sung xử lý SKIP_WAITING

- [ ] Cấu hình `next.config.js` để thêm `customWorkerDir: 'src/service-worker'`.
- [ ] Tạo file `src/service-worker/sw.js` với nội dung:

  ```js
  self.addEventListener("message", (event) => {
    if (event.data && event.data.type === "SKIP_WAITING") {
      self.skipWaiting();
    }
  });
  ```

- [ ] Đảm bảo khi build, `next-pwa` merge file này mà không ghi đè.

#### 2️⃣ Tránh loop reload khi Service Worker cập nhật liên tục

- [ ] Trong `ServiceWorkerRegister.tsx`, thêm flag `refreshing` để chỉ reload 1 lần:

  ```ts
  let refreshing = false;
  navigator.serviceWorker.addEventListener("controllerchange", () => {
    if (refreshing) return;
    refreshing = true;
    window.location.reload();
  });
  ```

#### 3️⃣ Cải thiện UX cập nhật - Hiển thị “Cập nhật ngay / Để sau”

- [ ] Sửa `ServiceWorkerRegister.tsx` để không tự reload khi có bản mới.
- [ ] Gửi event tới component `UpdateNotifier` khi phát hiện SW mới.
- [ ] Trong `UpdateNotifier`, hiển thị banner hoặc popup với hai nút:

  - **Cập nhật ngay:** gửi `newWorker.postMessage({ type: 'SKIP_WAITING' });`
  - **Để sau:** ẩn banner, không reload.

- [ ] Đảm bảo `UpdateNotifier` chỉ hiển thị khi có SW mới thực sự.

#### 4️⃣ Kiểm tra tổng thể sau khi triển khai

- [ ] Build thử `next build && next start` để xác nhận SW hoạt động đúng.
- [ ] Test update vòng đời SW trong Chrome DevTools > Application > Service Workers.
- [ ] Đảm bảo không loop reload và thông báo hiển thị hợp lý.

---

| Mục | Giải pháp chi tiết                                                                                                  |
| --- | ------------------------------------------------------------------------------------------------------------------- |
| 1️⃣  | Di chuyển toàn bộ logic đăng ký và update vào **một chỗ duy nhất** (tốt nhất là trong `useServiceWorkerUpdate.ts`). |
| 2️⃣  | **Bỏ `SKIP_WAITING` tự động** trong `ServiceWorkerRegister.tsx`. Để người dùng chọn qua `UpdateNotifier`.           |
| 3️⃣  | Khi SW mới sẵn sàng: gửi thông điệp “newVersionAvailable” → hook nhận và bật `hasUpdate=true`.                      |
| 4️⃣  | Trong `update()`: gửi `{ type: 'SKIP_WAITING' }`, sau đó đợi `controllerchange`, rồi reload.                        |
| 5️⃣  | Chống reload loop bằng cách lưu flag `sessionStorage.setItem('reloaded', '1')`, và chỉ reload 1 lần.                |
| 6️⃣  | Bổ sung “Để sau” — tắt `UpdateNotifier` mà không reload, nhưng giữ state cho đến khi tab đóng.                      |
| 7️⃣  | Cập nhật `ServiceWorkerCheck.tsx` thành “debug helper” hoặc hợp nhất vào hook (để tránh thừa).                      |

| Tình huống               | Hành vi mong muốn                                                                                                                                |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| ✅ Có Wi-Fi              | Bấm “Cập nhật ngay” → hiển thị thông báo “Đang cập nhật…” → sau khi SW mới kích hoạt, báo “Cập nhật thành công! Ứng dụng đang được làm mới.”     |
| 📶 Dữ liệu di động       | Hỏi xác nhận: “Bạn đang dùng mạng di động, tải bản cập nhật có thể tốn dữ liệu. Tiếp tục không?” → Có thể chọn **“Tiếp tục”** hoặc **“Để sau”**. |
| 🚫 Không mạng            | Báo rõ “Không thể tải bản cập nhật khi đang offline.”                                                                                            |
| 🔁 Sau khi cập nhật xong | Chỉ reload **một lần duy nhất**, không loop.                                                                                                     |

Tốt — ControlBar.tsx của bạn gọn, rõ ràng, có thể tái sử dụng tốt cho cả products và news.
Tôi xác nhận file này ổn, không cần chỉnh sửa, trừ khi sau này bạn muốn:

Thêm debounce cho search để tránh re-render khi nhập nhanh, hoặc

Tách CSS riêng để dễ kiểm soát responsive layout (vì bạn mô tả trong products/page.tsx rằng control bar hiển thị khác nhau trên PC và mobile).

I'll create a short todo plan, then scan the repository for key files (routes, service worker, hooks, services) so I can build an accurate Mermaid flow and detailed step list. Expected outcome: a todo list stored and a file list search result to guide diagram creation.

Created 5 todos

Updating the todo list to reflect progress: code overview done and reading key files completed; now I'll produce the Mermaid diagram and detailed steps.

Starting (3/5) Draft Mermaid diagram

Progress update

I scanned the codebase and read the main files involved in data and image flows: app/news, app/products, src/hooks/useImageCacheTracker.ts, src/lib/ensureImageCachedByUrl.ts, src/lib/getImageBlobUrl.ts, src/lib/news_images.ts, src/lib/db.ts, productsPrefetch.ts and related modules.
Next: here's a Mermaid diagram that models the application runtime behavior and a detailed step-by-step explanation (including edge cases and notes about a couple of inconsistencies I found).
Mermaid flowchart (high-level view)
Copy the block below into a Markdown file or a Mermaid live editor to visualize.

Detailed step-by-step behavior (numbered)
I describe the runtime flow in concrete steps, mapping to nodes in the diagram. This is suitable as documentation or for making a sequence diagram.

App start / page navigation

User opens the app in the browser (Next.js client pages).
Client components mount (for example /products or /news pages).
Each list page follows an "offline-first" pattern:
Call loadFromDB() from the repository to read cached items from IndexedDB (TPBC_DB).
If cached items exist, render them immediately (UI shows "using cache" banner).
If navigator.onLine is true, call the server API (/api/products or /api/news) to fetch fresh data.
Data synchronization (repositories)

After receiving API payload, repositories run syncProducts / syncNews:
Compare incoming data with local cache to detect changes.
If changes detected, update IndexedDB stores and return hasChange = true.
UI updates (shows "✅ Đã cập nhật dữ liệu mới") and replaces cache view.
Image caching invocation

After successful data sync, page collects image URLs from items: e.g., fresh.map(p => p.image_url).
Page calls useImageCacheTracker(...).syncImages(urls) (or ensureNewsImageCachedByUrl from older module).
Hook useImageCacheTracker calls ensureImageCachedByUrl(url, type) for each URL.
ensureImageCachedByUrl logic (core of image caching)

Determine store name (STORE_PRODUCTS_IMAGES, STORE_NEWS_IMAGES, or STORE_IMAGES).
Try to get existing record in DB: store.get(url) (note: the code often uses the URL as explicit key in get/put).
Call /api/image-meta?url=... to get server-provided meta (optional): { hash?, etag? }.
If meta exists and matches existing hash/etag -> skip fetch (cache fresh).
If no meta:
If existing.lastFetched is recent (<=7 days) and not forced -> skip.
Otherwise, fetch the image blob from the URL (fetch with cache: 'no-store').
Compute SHA-256 hash of the blob.
If hash equals existing.hash -> skip storing.
Else, prepare updated object:
{ url, blob, hash, etag, lastFetched }
Write into the object store:
Implementation uses store.put(updated, url) (explicit primary key value = url). This writes blob into IndexedDB store (store keyPath may be 'key' or 'url' depending on store).
Note: Another module (news_images.ts) uses a different pattern:
It computes key = sha256Hex(url) and uses that as the primary key, stores the key field on the record. That's a different but valid approach — see "Inconsistencies" below.
getImageBlobUrl when rendering images

When rendering, components call getImageBlobUrl(url, type).
getImageBlobUrl opens DB, looks up the record via db.get(storeName, url).
If record?.blob exists, returns URL.createObjectURL(blob) for fast local display.
Else returns the original remote URL as fallback.
Background prefetch / scheduling

The app includes background prefetch functions (e.g., prefetchProductsOnce in productsPrefetch.ts).
BackgroundPrefetch component (or other startup code) calls prefetchProductsOnce():
If online, fetches /api/products, calls syncProducts to store data, then ensures each product image is cached by calling ensureImageCachedByUrl.
This runs once per session (guarded by hasPrefetched) and may be re-triggered manually.
Service Worker role

The app registers a service worker via ServiceWorkerRegister / ServiceWorkerCheck components.
The SW caches static assets and offline.html, intercepts navigation and resource fetches when offline, and can supply offline fallbacks.
The SW may postMessage events to the app to notify of updates (handled by useServiceWorkerUpdate hook).
SW also helps offline-first behavior by serving cached pages and assets when network is unavailable.
Manual sync & UI controls

ManualSyncButton triggers repository API calls to refresh data and then runs image sync, same as automatic flow.
ControlBar provides search, pagination, filtering — operates on in-memory list pulled from DB or API.
API routes and image-proxy

App provides server endpoints under app/api/* (example: /api/image-proxy, /api/products, /api/news, /api/image-meta).
/api/image-meta is used to provide server-side hash/etag to speed up cache checks.
/api/image-proxy can be used to serve images through the server if needed (caching/headers control / avoid CORS).
IndexedDB stores & indexes (TPBC_DB)

Stores and purpose:
products (keyPath: id) — product metadata
products_images (keyPath: key) — product image records (blob, hash, etag, lastFetched)
news (keyPath: news_id) — news posts
news_images (keyPath: key) — news image records
images — generic image store
Indexes present on images stores:
source_url — query by original URL
updated_at — find older entries for pruning
etag, hash — speed comparisons and queries
Note: concrete code sometimes uses explicit key parameter in put/get, so the effective primary key may be the url string or a sha256(url) depending on the module.
Edge cases and error modes
Offline during initial load:
If no cached data exists and the app is offline, UI must show a message prompting the user to connect for initial sync.
Partial image cache failure:
If fetch of a particular image fails, ensureImageCachedByUrl catches and the sync continues (code typically throws and is caught at caller level).
Inconsistent key strategies:
news_images.ts uses sha256(url) as the key, while ensureImageCachedByUrl/getImageBlobUrl use the URL as the explicit key for put/get.
This can cause duplication or inability to find blobs if different modules expect different keys. Consider standardizing to one strategy:
Either always use URL as key (explicit put/get), or always use sha256(url) and store that as key property.
ETag/meta missing from server:
Hook uses blob hash to detect content changes when ETag not provided. This requires downloading the blob to compute hash (can't avoid network cost).
Cache growth:
No automatic prune shown; use updated_at index to find old items and prune as needed (e.g., LRU, TTL).
Race conditions:
Concurrent syncImages calls could issue multiple fetches. Hook currently awaits each ensureImage sequentially; you could parallelize with throttling.
Suggestions / Improvements (proactive)
Standardize key usage across modules (choose URL or sha256(url)) and update db.ts or callers to match.
Add a pruning job (background or service worker-triggered) to limit local storage use (use updated_at index).
Consider parallelizing image fetches with a concurrency limit (e.g., 4 at a time) to speed up prefetch but avoid saturating network.
Expose a small admin UI to view cache stats (# of images, total size) using the indexes.
If privacy or CORS is a concern, prefer /api/image-proxy so your origin fetches images rather than client hitting many third-party hosts.
If you'd like, next I can:

Render a sequence diagram (Mermaid sequence) for one concrete flow (e.g., "User opens /news → offline-first → sync → image caching") to show exact request/response sequence.
Produce a standardized "contract" for image records (fields, primary key strategy) and a small patch to align all modules to that contract. Which would you prefer I do next?