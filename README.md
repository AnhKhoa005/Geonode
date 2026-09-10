# WebGIS - Giao diện tùy chỉnh cho GeoNode

Đồ án Phát triển ứng dụng Web GIS: thiết kế lại giao diện GeoNode (trang chủ, Layers, Bản đồ, Tài liệu) theo phong cách hiện đại, đọc dữ liệu thật từ GeoNode API và mở viewer MapStore gốc.

> Repo này chỉ chứa phần giao diện tùy chỉnh. Toàn bộ backend (Django, GeoServer, PostGIS) dùng GeoNode gốc chạy bằng Docker.

## Tính năng

- **Trang chủ**: hero "Welcome to GeoNode", thống kê, lối tắt Tải lên Layer / Tạo Bản đồ mới.
- **Trang Layers** (`/datasets`): tìm kiếm, lọc Vector/Raster, sắp xếp (mới nhất, A-Z, phổ biến), badge loại layer, mở viewer chi tiết.
- **Trang Bản đồ** (`/maps`): lưới card thumbnail, số layer, ngày tạo, lượt xem, tự tạo thumbnail từ WMS nếu thiếu.
- **Trang Tài liệu** (`/documents`): icon theo định dạng file (PDF, Word, Excel...), badge đuôi file.
- Style thống nhất: tông navy + xanh dương, font Be Vietnam Pro + Inter, hỗ trợ tiếng Việt đầy đủ.

## Cấu trúc

```
webgis-custom/
├── templates/
│   └── geonode-mapstore-client/
│       ├── index.html          # Trang chủ
│       └── pages/
│           ├── datasets.html   # Trang Layers
│           ├── maps.html       # Trang Bản đồ
│           └── documents.html  # Trang Tài liệu
├── docker-compose.override.yml # Mount giao diện vào container GeoNode
└── README.md
```

## Cách chạy

Yêu cầu: đã có source GeoNode (bản 4.x) + Docker Desktop.

1. Clone GeoNode và checkout đúng bản đang dùng, sau đó copy giao diện vào:
   ```powershell
   Copy-Item -Recurse .\templates\ <duong-dan-geonode>\templates\
   Copy-Item .\docker-compose.override.yml <duong-dan-geonode>\docker-compose.override.yml
   ```
2. Khởi động GeoNode:
   ```powershell
   cd <duong-dan-geonode>
   docker compose up -d
   docker compose restart django
   ```
3. Mở trình duyệt: `http://localhost` (nhấn `Ctrl+F5` nếu vẫn thấy giao diện cũ do cache).

## Ghi chú

- Giao diện chỉ đọc API có sẵn của GeoNode (`/api/v2/datasets`, `/api/v2/maps`, `/api/v2/documents`), không sửa backend.
- Dữ liệu mẫu (bản đồ, tài liệu demo) nằm trong database Docker, không đi kèm repo này.
- Phát triển trên nền GeoNode (GPL-3.0).
