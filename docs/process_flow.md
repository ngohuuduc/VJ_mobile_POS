# VJ Mobile POS — Process Flow

> Trạng thái: Draft  
> Cập nhật: 2026-04-14  
> Phiên bản: 0.4

Chỉ bao gồm các luồng đã xác định đủ nội dung.  
Các câu hỏi còn mở xem tại [open_questions.md](open_questions.md).

---

## 1. Luồng Đăng nhập (Authentication)

```mermaid
sequenceDiagram
    actor User as Người dùng
    participant FE as Frontend (Vue)
    participant BE as Backend (FastAPI)
    participant DB as PostgreSQL

    User->>FE: Nhập username / password
    FE->>BE: POST /auth/login
    BE->>DB: Kiểm tra credentials
    DB-->>BE: User record + roles + locations
    BE->>DB: Lưu refresh token (bảng refresh_tokens)
    BE-->>FE: Access token + Refresh token + permissions
    FE->>FE: Lưu token (memory / secure storage)
    FE-->>User: Vào màn hình chính
```

---

## 2. Luồng Bán hàng — Tạo đơn & Xác nhận

> Warehouse tự động chọn theo location được gán của user.  
> Sau bước confirm → xem Luồng 6 (Xuất kho) và Luồng 8 (Thanh toán).

```mermaid
flowchart TD
    A([Bắt đầu]) --> B[Warehouse tự động chọn\ntheo location được gán của user]
    B --> C[Chọn / Tìm khách hàng]
    C --> D{Khách có MST?}
    D -- Có --> E[Nhập MST → Tra cứu API]
    E --> F[Auto-fill thông tin KH]
    D -- Không --> G[Nhập thông tin thủ công\nhoặc chọn KH có sẵn]
    F --> H[Thêm sản phẩm vào đơn]
    G --> H
    H --> I{Còn thêm sản phẩm?}
    I -- Có --> H
    I -- Không --> J[Xem lại đơn hàng\ngiá / số lượng]
    J --> K{Hành động}
    K -- Lưu nháp --> L[PostgreSQL: INSERT draft_orders\nexpires_at = now + 8 giờ]
    L --> M([Đơn nháp được lưu])
    K -- Xác nhận --> N[Backend: sale.order.action_confirm]
    N --> O[Odoo tạo stock.picking tự động]
    O --> P([Tiếp theo: Luồng 6 Xuất kho\nvà / hoặc Luồng 8 Thanh toán])
    K -- Hủy bỏ --> Q([Kết thúc / Xóa đơn nháp])
```

---

## 3. Luồng Tra cứu MST

```mermaid
sequenceDiagram
    actor User as Thu ngân
    participant FE as Frontend
    participant BE as Backend (TTLCache)
    participant MST as VietQR API

    User->>FE: Nhập mã số thuế
    FE->>BE: GET /customers/mst/{tax_code}
    alt Cache hit (TTL < 10 phút)
        Note over BE: Trả từ TTLCache in-memory
        BE-->>FE: Thông tin doanh nghiệp
    else Cache miss / Hết TTL
        BE->>MST: GET /v2/business/{tax_code}
        MST-->>BE: Tên, địa chỉ, trạng thái hoạt động
        Note over BE: Lưu vào TTLCache (10 phút)
        BE-->>FE: Thông tin doanh nghiệp
    end
    FE-->>User: Auto-fill form khách hàng
```

---

## 4. Luồng Kiểm tra tồn kho

```mermaid
sequenceDiagram
    actor User as Thủ kho / Thu ngân
    participant FE as Frontend
    participant BE as Backend (TTLCache)
    participant Odoo

    User->>FE: Tìm kiếm sản phẩm
    FE->>BE: GET /inventory/stock?product_id=X&location_id=Y
    Note over BE: location_id lấy từ locations\nđược gán cho user
    alt Cache hit (TTL < 5 phút)
        Note over BE: Trả từ TTLCache in-memory
        BE-->>FE: Tồn kho theo địa điểm
    else Cache miss / Hết TTL
        BE->>Odoo: XML-RPC search_read stock.quant
        Odoo-->>BE: Dữ liệu tồn kho theo location
        Note over BE: Lưu vào TTLCache (5 phút)
        BE-->>FE: Tồn kho theo địa điểm
    end
    FE-->>User: Hiển thị số lượng có thể bán
```

---

## 5. Luồng Làm mới Cache (Cache Invalidation)

```mermaid
flowchart TD
    A([Trigger]) --> B{Loại trigger}

    B -- Thủ công\nADMIN bấm Refresh --> C[POST /admin/cache/flush]
    B -- Tự động\nTTL hết hạn --> D[TTLCache tự expire\nkhông cần tác động]

    C --> E{Phạm vi flush}
    E -- Sản phẩm / Giá --> F[cache.clear product_cache]
    E -- Pricelist --> G[cache.clear pricelist_cache]
    E -- Tất cả --> H[cache.clear all]

    F --> I([Lần gọi tiếp theo\nfetch mới từ Odoo])
    G --> I
    H --> I
    D --> I
```

---

## 6. Luồng Xuất kho (Auto validate + Serial + Backorder)

```mermaid
flowchart TD
    A([sale.order được xác nhận]) --> B[Odoo tạo stock.picking tự động]
    B --> C[Backend: Kiểm tra stock.quant\ntheo location của đơn hàng]
    C --> D{Tồn kho đủ\ncho toàn bộ đơn?}

    D -- Đủ toàn bộ --> E{Sản phẩm có\nSerial Number?}
    D -- Thiếu một phần --> F[Cảnh báo: danh sách\nsản phẩm thiếu + số lượng]
    F --> G[Validate phần có đủ hàng\nTạo backorder cho phần thiếu]
    G --> H[stock.picking backorder\ntạo tự động trên Odoo]
    H --> I([Backorder → Luồng 9])

    E -- Không --> J[Auto validate picking\nstock.picking.button_validate]
    E -- Có --> K[Hệ thống gợi ý serial\ntừ stock.lot có sẵn theo sản phẩm]
    K --> L[POS user chọn hoặc nhập\nserial number]
    L --> J

    J --> M[Tồn kho cập nhật trên Odoo]
    M --> N[Invalidate product_cache\n& stock_cache trong TTLCache]
    N --> O([Hoàn tất xuất kho])
```

---

## 7. Luồng Hủy đơn hàng

```mermaid
flowchart TD
    A([Yêu cầu hủy đơn]) --> B{Trạng thái đơn?}

    B -- Draft / Nháp --> C[sale.order.action_cancel\nXóa draft_orders trong DB]
    C --> D[Ghi audit log]
    D --> E([Đơn đã hủy])

    B -- Đã xác nhận\nChưa xuất kho --> F{Role người dùng?}
    F -- ADMIN --> G[Hủy stock.picking liên quan\nstock.picking.action_cancel]
    G --> H[sale.order.action_cancel]
    H --> D
    F -- POS User --> I([Từ chối\nYêu cầu liên hệ ADMIN])

    B -- Đã xuất kho\nstock.picking done --> J[Không cho phép hủy thông thường]
    J --> K([Cần nhập hàng về qua Odoo\nNgoài phạm vi Mobile POS giai đoạn này])
```

---

## 8. Luồng Nhận thanh toán (Ghi nhận thủ công)

> Hỗ trợ nhiều phương thức trong 1 đơn.  
> `account.payment` tạo từ Mobile POS dựa trên thanh toán thực tế.  
> `account.move` (invoice) ở trạng thái draft — kế toán post trên Odoo.

```mermaid
flowchart TD
    START{Điểm vào} -- Đơn hàng mới\nvừa xác nhận --> B
    START -- Thu phần còn lại\nkhách quay lại --> SEARCH[Tìm đơn theo thông tin KH\ntên / SĐT / MST]
    SEARCH --> FOUND[Chọn đơn hàng\nHiển thị số tiền đã cọc\nvà số tiền còn lại]
    FOUND --> B

    B{Loại ghi nhận?}
    B -- Đặt cọc --> C[Nhân viên nhập số tiền cọc\ntự do]
    B -- Thanh toán đầy đủ / còn lại --> D[Hiển thị tổng tiền cần thu]

    C --> E[Chọn phương thức thanh toán]
    D --> E

    E --> F{Phương thức}
    F -- Tiền mặt --> G[Nhập số tiền nhận\nTính tiền thừa trả lại]
    F -- Chuyển khoản --> H[Ghi nhận số tiền\nvà ngân hàng]
    F -- VNPay / MoMo --> I[Thu ngân xác nhận thủ công\nsau khi khách thanh toán]
    F -- Thẻ tín dụng --> J[Thu ngân xác nhận thủ công\nsau khi máy POS xác nhận]

    G --> K{Còn thiếu?\nThêm phương thức?}
    H --> K
    I --> K
    J --> K

    K -- Thêm phương thức --> E
    K -- Hoàn tất --> L[Backend: Tạo account.move draft\ntrên Odoo]
    L --> M[Backend: Tạo account.payment\nlinked to invoice]
    M --> N[Ghi audit log vào PostgreSQL]
    N --> O([Giao dịch ghi nhận xong\nKế toán post invoice trên Odoo])
```

---

## 9. Luồng In phiếu bán hàng (Generate PDF)

> 3 loại phiếu: Xác nhận đơn, Đặt cọc, Thanh toán.  
> FastAPI render HTML/CSS (Jinja2) → WeasyPrint → PDF binary — toàn bộ trong Python.

```mermaid
sequenceDiagram
    actor User as Nhân viên
    participant FE as Frontend
    participant BE as FastAPI (WeasyPrint + Jinja2)
    participant DB as PostgreSQL

    User->>FE: Bấm "In phiếu" trên đơn hàng
    FE->>BE: GET /print/{receipt_type}?order_id=X
    Note over BE: receipt_type: order_confirm / deposit / payment
    BE->>DB: Lấy template HTML/CSS theo loại phiếu
    DB-->>BE: Template Jinja2
    BE->>BE: Render template với dữ liệu đơn hàng\n(số đơn, KH, SP, giá, PTTT, nhân viên, logo...)
    BE->>BE: WeasyPrint: HTML/CSS → PDF binary
    BE-->>FE: PDF file (Content-Type: application/pdf)
    FE-->>User: Mở PDF tab mới / download\n(xem OQ-I04)
```

---

## 10. Luồng Quản lý Backorder

```mermaid
flowchart TD
    A([Backorder được tạo\nsau Luồng 6]) --> B[Hiển thị trong danh sách\nBackorder của user]
    B --> C{Hành động}

    C -- Xem chi tiết --> D[Hiển thị: sản phẩm thiếu\nsố lượng còn lại / trạng thái]
    D --> C

    C -- Validate khi có hàng --> E[Kiểm tra stock.quant\ntheo location]
    E --> F{Đã đủ hàng?}
    F -- Chưa đủ --> G([Thông báo: vẫn thiếu hàng\nChờ nhập kho trên Odoo])
    F -- Đủ --> H{Sản phẩm có Serial?}
    H -- Không --> I[Validate backorder picking]
    H -- Có --> J[Gợi ý serial từ stock.lot]
    J --> K[POS user chọn / nhập serial]
    K --> I
    I --> L[Tồn kho cập nhật trên Odoo]
    L --> M[Invalidate stock_cache trong TTLCache]
    M --> N([Backorder hoàn tất])

    C -- Hủy backorder --> O{Role?}
    O -- ADMIN --> P[Hủy backorder picking trên Odoo]
    P --> Q([Backorder đã hủy])
    O -- POS User --> R([Từ chối\nYêu cầu liên hệ ADMIN])
```
