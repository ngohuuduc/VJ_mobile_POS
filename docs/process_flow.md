# VJ Mobile POS — Process Flow

> Trạng thái: Draft  
> Cập nhật: 2026-04-14  
> Phiên bản: 0.1

Chỉ bao gồm các luồng đã xác định đủ nội dung.  
Các luồng chưa xác định (Xuất kho, Thanh toán, Hủy đơn) xem tại [open_questions.md](open_questions.md).

---

## 1. Luồng Đăng nhập (Authentication)

```mermaid
sequenceDiagram
    actor User as Người dùng
    participant FE as Frontend (Vue)
    participant BE as Backend (FastAPI)
    participant Redis
    participant DB as PostgreSQL

    User->>FE: Nhập username / password
    FE->>BE: POST /auth/login
    BE->>DB: Kiểm tra credentials
    DB-->>BE: User record
    BE->>Redis: Lưu refresh token
    BE-->>FE: Access token + Refresh token
    FE->>FE: Lưu token (memory / secure storage)
    FE-->>User: Vào màn hình chính
```

---

## 2. Luồng Bán hàng — Tạo đơn & Xác nhận

> Luồng kết thúc tại bước Odoo tạo `stock.picking`.  
> Bước tiếp theo (xuất kho / thanh toán) chờ xác nhận — xem OQ-01, OQ-02, OQ-03.

```mermaid
flowchart TD
    A([Bắt đầu]) --> B[Chọn / Tìm khách hàng]
    B --> C{Khách có MST?}
    C -- Có --> D[Nhập MST → Tra cứu API]
    D --> E[Auto-fill thông tin KH]
    C -- Không --> F[Nhập thông tin thủ công\nhoặc chọn KH có sẵn]
    E --> G[Thêm sản phẩm vào đơn]
    F --> G
    G --> H{Còn thêm sản phẩm?}
    H -- Có --> G
    H -- Không --> I[Xem lại đơn hàng\ngiá / số lượng / chiết khấu]
    I --> J{Xác nhận đơn?}
    J -- Hủy bỏ --> K([Kết thúc / Xóa đơn nháp])
    J -- Xác nhận --> L[Backend: sale.order.action_confirm]
    L --> M[Odoo tạo stock.picking tự động]
    M --> N([Chờ bước tiếp theo\nxem OQ-01 / OQ-02 / OQ-03])
```

---

## 3. Luồng Tra cứu MST

```mermaid
sequenceDiagram
    actor User as Thu ngân
    participant FE as Frontend
    participant BE as Backend
    participant Redis
    participant MST as MST API (External)

    User->>FE: Nhập mã số thuế
    FE->>BE: GET /customers/mst/{tax_code}
    BE->>Redis: Kiểm tra cache
    alt Cache hit
        Redis-->>BE: Thông tin doanh nghiệp
    else Cache miss
        BE->>MST: Gọi API tra cứu
        MST-->>BE: Tên, địa chỉ, trạng thái hoạt động
        BE->>Redis: Lưu cache (TTL dài)
    end
    BE-->>FE: Thông tin doanh nghiệp
    FE-->>User: Auto-fill form khách hàng
```

---

## 4. Luồng Kiểm tra tồn kho

```mermaid
sequenceDiagram
    actor User as Thủ kho / Thu ngân
    participant FE as Frontend
    participant BE as Backend
    participant Redis
    participant Odoo

    User->>FE: Tìm kiếm sản phẩm
    FE->>BE: GET /inventory/stock?product_id=X&warehouse_id=Y
    BE->>Redis: Kiểm tra cache stock.quant
    alt Cache hit (TTL < 5 phút)
        Redis-->>BE: Số lượng tồn kho
    else Cache miss / Hết TTL
        BE->>Odoo: XML-RPC search_read stock.quant
        Odoo-->>BE: Dữ liệu tồn kho
        BE->>Redis: Lưu cache (TTL = 5 phút)
    end
    BE-->>FE: Tồn kho theo kho / vị trí
    FE-->>User: Hiển thị số lượng có thể bán
```

---

## 5. Luồng Làm mới Cache (Cache Invalidation)

```mermaid
flowchart TD
    A([Trigger]) --> B{Loại trigger}

    B -- Thủ công\nManager bấm Refresh --> C[POST /admin/cache/flush]
    B -- Tự động\nTTL hết hạn --> D[Redis tự xóa key]

    C --> E{Phạm vi flush}
    E -- Sản phẩm / Giá --> F[Xóa cache product:*]
    E -- Pricelist --> G[Xóa cache pricelist:*]
    E -- Tất cả --> H[Xóa toàn bộ namespace]

    F --> I([Lần gọi tiếp theo\nfetch mới từ Odoo])
    G --> I
    H --> I
    D --> I
```
