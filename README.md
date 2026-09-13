# WebGIS - Giao diện tùy chỉnh cho GeoNode

Đồ án Phát triển ứng dụng Web GIS: thiết kế lại giao diện GeoNode (trang chủ, Layers, Bản đồ, Tài liệu) theo phong cách hiện đại, đọc dữ liệu thật từ GeoNode API và mở viewer MapStore gốc.

> Repo này chỉ chứa phần giao diện tùy chỉnh. Toàn bộ backend (Django, GeoServer, PostGIS) dùng GeoNode gốc chạy bằng Docker.

## Tính năng

- **Trang chủ (phong cách Arsha)**: header cố định + menu mobile, hero xanh navy nhỏ gọn kèm ảnh minh họa và ô tìm kiếm, thống kê, 6 tính năng dạng Services (icon to, card đều nhau), Layers nổi bật, khối CTA nền ảnh, footer Arsha. Khung bản đồ MapStore gốc giữ nguyên bên dưới.
- **Trang Layers** (`/datasets`): tìm kiếm, lọc Vector/Raster, sắp xếp (mới nhất, A-Z, phổ biến), badge loại layer, mở viewer chi tiết.
- **Trang Bản đồ** (`/maps`): lưới card thumbnail, số layer, ngày tạo, lượt xem, tự tạo thumbnail từ WMS nếu thiếu.
- **Trang Tài liệu** (`/documents`): icon theo định dạng file (PDF, Word, Excel...), badge đuôi file.
- **Chat trực tuyến** (floating widget góc phải dưới): hộp chat lưu vào database, poll tin nhắn mới mỗi 5 giây (biến CSS tùy chỉnh `.gn-chat-*`), nhận diện tên người đăng nhập, khách lưu tên "Khách".
- **Thương hiệu GITC Portal**: thay nhãn "GeoNode" trên toàn bộ giao diện (navbar, title, footer, dropdown catalogue MapStore) thành "GITC Portal".
- Style thống nhất: tông navy + xanh dương, font **Roboto** (đồng bộ 4 trang + chatbox), hỗ trợ tiếng Việt đầy đủ.

## Cấu trúc

```
webgis-custom/
├── templates/
│   └── geonode-mapstore-client/
│       ├── index.html          # Trang chủ (Arsha)
│       ├── snippets/
│       │   └── chatbox.html    # Widget chat trực tuyến
│       └── pages/
│           ├── datasets.html   # Trang Layers
│           ├── maps.html       # Trang Bản đồ
│           └── documents.html  # Trang Tài liệu
├── arsha/                      # Assets Arsha (CSS scoped, JS, icons, ảnh) -> /static/arsha/
├── geonode/
│   └── chat/                   # Django app "chat" (backend cho chatbox)
├── docker-compose.override.yml # Mount giao diện + chat app + assets vào container
└── README.md
```

## Cách chạy

Yêu cầu: đã có source GeoNode (bản 4.x) + Docker Desktop.

1. Clone GeoNode và checkout đúng bản đang dùng, sau đó copy giao diện + chat app vào:
   ```powershell
   Copy-Item -Recurse .\templates\ <duong-dan-geonode>\templates\
   Copy-Item -Recurse .\geonode\chat\ <duong-dan-geonode>\geonode\chat\
   Copy-Item -Recurse .\arsha\ <duong-dan-geonode>\arsha\
   Copy-Item .\docker-compose.override.yml <duong-dan-geonode>\docker-compose.override.yml
   ```
2. Đăng ký app `chat` vào backend (chỉ cần 2 dòng):
   - `geonode/settings.py` — cuối file thêm:
     ```python
     INSTALLED_APPS += ("geonode.chat",)
     ```
   - `geonode/urls.py` — cuối file (trước `handler500`) thêm:
     ```python
     urlpatterns += [re_path(r"^api/v2/chat/", include("geonode.chat.urls"))]
     ```
3. (Tùy chọn) Đổi nhãn catalogue MapStore từ "GeoNode" thành "GITC Portal" — `geonode/settings.py` cuối file thêm:
   ```python
   MAPSTORE_CATALOGUE_SERVICES = {
       "GITC Portal": {"type": "geonode", "url": None, "autoload": True, "title": "GITC Portal"}
   }
   MAPSTORE_CATALOGUE_SELECTED_SERVICE = "GITC Portal"
   MAPSTORE_DASHBOARD_CATALOGUE_SERVICES = MAPSTORE_CATALOGUE_SERVICES
   MAPSTORE_DASHBOARD_CATALOGUE_SELECTED_SERVICE = "GITC Portal"
   ```
4. Khởi động GeoNode:
   ```powershell
   cd <duong-dan-geonode>
   docker compose up -d
   docker compose exec django python manage.py migrate   # tạo bảng chat
   docker compose restart django
   ```
4. Mở trình duyệt: `http://localhost` (nhấn `Ctrl+F5` nếu vẫn thấy giao diện cũ do cache).

## Ghi chú

- Giao diện chỉ đọc API có sẵn của GeoNode (`/api/v2/datasets`, `/api/v2/maps`, `/api/v2/documents`), không sửa backend ngoài app chat nhỏ.
- Chatbox dùng API riêng: `GET /api/v2/chat/messages/` (đọc tin mới nhất, mặc định 50) và `POST /api/v2/chat/send/` (gửi tin nhắn).
- Dữ liệu mẫu (bản đồ, tài liệu demo) nằm trong database Docker, không đi kèm repo này.
- Phát triển trên nền GeoNode (GPL-3.0).
- Giao diện trang chủ dựa trên template **Arsha** của BootstrapMade (giấy phép free, giữ credit trong footer). File CSS gốc đã được scope dưới `.gn-arsha` để không ảnh hưởng viewer MapStore; chỉ giữ lại CSS/JS/ảnh cần thiết (bỏ swiper, glightbox, isotope...).
