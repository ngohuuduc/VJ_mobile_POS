# VJ Mobile POS — Process Flow

> Trạng thái: Draft  
> Cập nhật: 2026-04-14  
> Phiên bản: 0.5

Chỉ bao gồm các luồng đã xác định đủ nội dung.  
Các câu hỏi còn mở xem tại [open_questions.md](open_questions.md).

---

## Chú thích màu sắc

| Màu | Ý nghĩa |
|---|---|
| 🔵 Xanh dương | Thao tác của **người dùng** |
| ⬜ Xám nhạt | Xử lý **hệ thống** / Backend |
| 🟣 Tím nhạt | Thao tác trên **Odoo** (XML-RPC) |
| 🟢 Xanh lá | Truy vấn / ghi **Database** |
| 🟡 Vàng | Điểm **quyết định** |
| 🔴 Đỏ nhạt | **Cảnh báo** / Từ chối |

---

## 1. Luồng Đăng nhập (Authentication)

```mermaid
sequenceDiagram
    actor User as Người dùng
    participant FE as Frontend (Vue)
    participant BE as Backend (FastAPI)
    participant DB as PostgreSQL

    rect rgb(219, 234, 254)
        Note over User,FE: Thao tác người dùng
        User->>FE: Nhập username / password
        FE->>BE: POST /auth/login
    end
    rect rgb(241, 245, 249)
        Note over BE,DB: Xử lý hệ thống
        BE->>DB: Kiểm tra credentials
        DB-->>BE: User record + roles + locations
        BE->>DB: Lưu refresh token (bảng refresh_tokens)
        BE-->>FE: Access token + Refresh token + permissions
    end
    rect rgb(219, 234, 254)
        Note over FE,User: Phản hồi người dùng
        FE->>FE: Lưu token
        FE-->>User: Vào màn hình chính
    end
```

---

## 2. Luồng Bán hàng — Tạo đơn & Xác nhận

```mermaid
flowchart TD
    classDef user fill:#DBEAFE,stroke:#2563EB,color:#1e3a5f
    classDef system fill:#F1F5F9,stroke:#64748B,color:#1e293b
    classDef odoo fill:#EDE9FE,stroke:#7C3AED,color:#3b0764
    classDef db fill:#DCFCE7,stroke:#16A34A,color:#14532d
    classDef decision fill:#FEF9C3,stroke:#CA8A04,color:#713f12
    classDef terminal fill:#F8FAFC,stroke:#334155,color:#0f172a
    classDef warning fill:#FEE2E2,stroke:#DC2626,color:#7f1d1d

    A([Bắt đầu]):::terminal --> B[Warehouse tự động chọn\ntheo location của user]:::system
    B --> C[Chọn / Tìm khách hàng]:::user
    C --> D{Khách có MST?}:::decision
    D -- Có --> E[Nhập MST]:::user
    E --> F[Tra cứu VietQR API\nAuto-fill thông tin KH]:::system
    D -- Không --> G[Nhập thủ công\nhoặc chọn KH có sẵn]:::user
    F --> H[Thêm sản phẩm vào đơn]:::user
    G --> H
    H --> I{Còn thêm SP?}:::decision
    I -- Có --> H
    I -- Không --> J[Xem lại đơn hàng\ngiá / số lượng]:::user
    J --> K{Hành động}:::decision
    K -- Lưu nháp --> L[INSERT draft_orders\nexpires_at = now + 8h]:::db
    L --> M([Đơn nháp lưu thành công]):::terminal
    K -- Xác nhận --> N[sale.order.create\nsale.order.action_confirm]:::odoo
    N --> O[Odoo tự tạo stock.picking]:::odoo
    O --> P([Tiếp theo: Luồng 6 & Luồng 8]):::terminal
    K -- Hủy --> Q([Kết thúc]):::terminal
```

---

## 3. Luồng Tra cứu MST

```mermaid
sequenceDiagram
    actor User as Thu ngân
    participant FE as Frontend
    participant BE as Backend (TTLCache)
    participant MST as VietQR API

    rect rgb(219, 234, 254)
        Note over User,FE: Thao tác người dùng
        User->>FE: Nhập mã số thuế
        FE->>BE: GET /customers/mst/{tax_code}
    end
    rect rgb(241, 245, 249)
        Note over BE,MST: Xử lý hệ thống
        alt Cache hit (TTL < 10 phút)
            Note over BE: Trả từ TTLCache in-memory
        else Cache miss
            BE->>MST: GET /v2/business/{tax_code}
            MST-->>BE: Tên, địa chỉ, trạng thái hoạt động
            Note over BE: Lưu vào TTLCache (10 phút)
        end
        BE-->>FE: Thông tin doanh nghiệp
    end
    rect rgb(219, 234, 254)
        Note over FE,User: Phản hồi người dùng
        FE-->>User: Auto-fill form khách hàng
    end
```

---

## 4. Luồng Kiểm tra tồn kho

```mermaid
sequenceDiagram
    actor User as Nhân viên
    participant FE as Frontend
    participant BE as Backend (TTLCache)
    participant Odoo

    rect rgb(219, 234, 254)
        Note over User,FE: Thao tác người dùng
        User->>FE: Tìm kiếm sản phẩm
        FE->>BE: GET /inventory/stock?product_id=X&location_id=Y
    end
    rect rgb(241, 245, 249)
        Note over BE,Odoo: Xử lý hệ thống
        alt Cache hit (TTL < 5 phút)
            Note over BE: Trả từ TTLCache in-memory
        else Cache miss
            BE->>Odoo: XML-RPC search_read stock.quant
            Odoo-->>BE: Dữ liệu tồn kho theo location
            Note over BE: Lưu vào TTLCache (5 phút)
        end
        BE-->>FE: Tồn kho theo địa điểm
    end
    rect rgb(219, 234, 254)
        Note over FE,User: Phản hồi người dùng
        FE-->>User: Hiển thị số lượng có thể bán
    end
```

---

## 5. Luồng Làm mới Cache

```mermaid
flowchart TD
    classDef user fill:#DBEAFE,stroke:#2563EB,color:#1e3a5f
    classDef system fill:#F1F5F9,stroke:#64748B,color:#1e293b
    classDef decision fill:#FEF9C3,stroke:#CA8A04,color:#713f12
    classDef terminal fill:#F8FAFC,stroke:#334155,color:#0f172a

    A([Trigger]):::terminal --> B{Loại trigger}:::decision
    B -- Thủ công\nADMIN bấm Refresh --> C[POST /admin/cache/flush]:::user
    B -- Tự động\nTTL hết hạn --> D[TTLCache tự expire\nkhông cần tác động]:::system
    C --> E{Phạm vi flush}:::decision
    E -- Sản phẩm / Giá --> F[cache.clear product_cache]:::system
    E -- Pricelist --> G[cache.clear pricelist_cache]:::system
    E -- Tất cả --> H[cache.clear all]:::system
    F --> I([Lần gọi tiếp theo fetch từ Odoo]):::terminal
    G --> I
    H --> I
    D --> I
```

---

## 6. Luồng Xuất kho (Auto validate + Serial + Backorder)

```mermaid
flowchart TD
    classDef user fill:#DBEAFE,stroke:#2563EB,color:#1e3a5f
    classDef system fill:#F1F5F9,stroke:#64748B,color:#1e293b
    classDef odoo fill:#EDE9FE,stroke:#7C3AED,color:#3b0764
    classDef db fill:#DCFCE7,stroke:#16A34A,color:#14532d
    classDef decision fill:#FEF9C3,stroke:#CA8A04,color:#713f12
    classDef terminal fill:#F8FAFC,stroke:#334155,color:#0f172a
    classDef warning fill:#FEE2E2,stroke:#DC2626,color:#7f1d1d

    A([sale.order xác nhận]):::terminal --> B[Odoo tạo stock.picking tự động]:::odoo
    B --> C[Kiểm tra stock.quant\ntheo location của đơn hàng]:::system
    C --> D{Đủ hàng?}:::decision
    D -- Đủ toàn bộ --> E{Có Serial Number?}:::decision
    D -- Thiếu một phần --> F[Hiển thị cảnh báo\ndanh sách hàng thiếu + SL]:::warning
    F --> G[Validate phần có hàng\nTạo backorder cho phần thiếu]:::odoo
    G --> H([Backorder → Luồng 10]):::terminal
    E -- Không --> I[Auto validate picking\nbutton_validate]:::odoo
    E -- Có --> J[Gợi ý serial từ stock.lot\ntheo sản phẩm đã chọn]:::system
    J --> K[POS user chọn / nhập serial]:::user
    K --> I
    I --> L[Tồn kho cập nhật trên Odoo]:::odoo
    L --> M[Invalidate stock_cache]:::system
    M --> N([Hoàn tất xuất kho]):::terminal
```

---

## 7. Luồng Hủy đơn hàng

```mermaid
flowchart TD
    classDef user fill:#DBEAFE,stroke:#2563EB,color:#1e3a5f
    classDef system fill:#F1F5F9,stroke:#64748B,color:#1e293b
    classDef odoo fill:#EDE9FE,stroke:#7C3AED,color:#3b0764
    classDef db fill:#DCFCE7,stroke:#16A34A,color:#14532d
    classDef decision fill:#FEF9C3,stroke:#CA8A04,color:#713f12
    classDef terminal fill:#F8FAFC,stroke:#334155,color:#0f172a
    classDef warning fill:#FEE2E2,stroke:#DC2626,color:#7f1d1d

    A([Yêu cầu hủy đơn]):::terminal --> B[Nhân viên chọn đơn cần hủy]:::user
    B --> C{Trạng thái đơn?}:::decision
    C -- Draft --> D[sale.order.action_cancel]:::odoo
    D --> E[Xóa draft_orders trong DB]:::db
    E --> F[Ghi audit log]:::db
    F --> G([Đơn đã hủy]):::terminal
    C -- Đã xác nhận\nChưa xuất kho --> H{Role người dùng?}:::decision
    H -- ADMIN --> I[Hủy stock.picking\naction_cancel]:::odoo
    I --> J[sale.order.action_cancel]:::odoo
    J --> F
    H -- POS User --> K([Từ chối\nYêu cầu liên hệ ADMIN]):::warning
    C -- Đã xuất kho --> L([Không cho phép\nCần xử lý trên Odoo]):::warning
```

---

## 8. Luồng Nhận thanh toán (Ghi nhận thủ công)

```mermaid
flowchart TD
    classDef user fill:#DBEAFE,stroke:#2563EB,color:#1e3a5f
    classDef system fill:#F1F5F9,stroke:#64748B,color:#1e293b
    classDef odoo fill:#EDE9FE,stroke:#7C3AED,color:#3b0764
    classDef db fill:#DCFCE7,stroke:#16A34A,color:#14532d
    classDef decision fill:#FEF9C3,stroke:#CA8A04,color:#713f12
    classDef terminal fill:#F8FAFC,stroke:#334155,color:#0f172a

    START{Điểm vào}:::decision -- Đơn mới\nvừa xác nhận --> B
    START -- Thu phần còn lại\nkhách quay lại --> SEARCH[Tìm đơn theo\ntên / SĐT / MST]:::user
    SEARCH --> FOUND[Chọn đơn hàng\nHiển thị đã cọc & còn lại]:::system
    FOUND --> B
    B{Loại ghi nhận?}:::decision -- Đặt cọc --> C[Nhập số tiền cọc\ntự do]:::user
    B -- Thanh toán đầy đủ / còn lại --> D[Hiển thị tổng tiền cần thu]:::system
    C --> E[Chọn phương thức thanh toán]:::user
    D --> E
    E --> F{Phương thức}:::decision
    F -- Tiền mặt --> G[Nhập số tiền nhận\nTính tiền thừa]:::user
    F -- Chuyển khoản --> H[Ghi nhận số tiền\nvà ngân hàng]:::user
    F -- VNPay / MoMo --> I[Xác nhận thủ công\nsau khi khách thanh toán]:::user
    F -- Thẻ tín dụng --> J[Xác nhận thủ công\nsau khi máy POS xác nhận]:::user
    G --> K{Còn thiếu?\nThêm phương thức?}:::decision
    H --> K
    I --> K
    J --> K
    K -- Thêm phương thức --> E
    K -- Hoàn tất --> L[Tạo account.move draft\ntrên Odoo]:::odoo
    L --> M[Tạo account.payment\nlinked to invoice]:::odoo
    M --> N[Ghi audit log]:::db
    N --> O([Hoàn tất\nKế toán post invoice trên Odoo]):::terminal
```

---

## 9. Luồng In phiếu bán hàng (Generate PDF)

```mermaid
sequenceDiagram
    actor User as Nhân viên
    participant FE as Frontend
    participant BE as FastAPI (WeasyPrint + Jinja2)
    participant DB as PostgreSQL

    rect rgb(219, 234, 254)
        Note over User,FE: Thao tác người dùng
        User->>FE: Bấm "In phiếu" trên đơn hàng
        FE->>BE: GET /print/{receipt_type}?order_id=X
        Note over BE: receipt_type: order_confirm / deposit / payment
    end
    rect rgb(241, 245, 249)
        Note over BE,DB: Xử lý hệ thống
        BE->>DB: Lấy template HTML/CSS theo loại phiếu
        DB-->>BE: Template Jinja2
        BE->>BE: Render template với dữ liệu đơn hàng
        BE->>BE: WeasyPrint: HTML/CSS → PDF binary
        BE-->>FE: PDF file (Content-Type: application/pdf)
    end
    rect rgb(219, 234, 254)
        Note over FE,User: Phản hồi người dùng
        FE-->>User: Mở PDF tab mới / download
    end
```

---

## 10. Luồng Quản lý Backorder

```mermaid
flowchart TD
    classDef user fill:#DBEAFE,stroke:#2563EB,color:#1e3a5f
    classDef system fill:#F1F5F9,stroke:#64748B,color:#1e293b
    classDef odoo fill:#EDE9FE,stroke:#7C3AED,color:#3b0764
    classDef db fill:#DCFCE7,stroke:#16A34A,color:#14532d
    classDef decision fill:#FEF9C3,stroke:#CA8A04,color:#713f12
    classDef terminal fill:#F8FAFC,stroke:#334155,color:#0f172a
    classDef warning fill:#FEE2E2,stroke:#DC2626,color:#7f1d1d

    A([Backorder tạo sau Luồng 6]):::terminal --> B[Hiển thị danh sách backorder\ntheo user / location]:::system
    B --> C{Hành động}:::decision
    C -- Xem chi tiết --> D[Hiển thị SP thiếu\nSL còn lại / trạng thái]:::system
    D --> C
    C -- Validate khi có hàng --> E[Kiểm tra stock.quant\ntheo location]:::system
    E --> F{Đủ hàng?}:::decision
    F -- Chưa đủ --> G([Thông báo: vẫn thiếu\nChờ nhập kho trên Odoo]):::warning
    F -- Đủ --> H{Có Serial Number?}:::decision
    H -- Không --> I[Validate backorder picking]:::odoo
    H -- Có --> J[Gợi ý serial\ntừ stock.lot]:::system
    J --> K[POS user chọn / nhập serial]:::user
    K --> I
    I --> L[Tồn kho cập nhật trên Odoo]:::odoo
    L --> M[Invalidate stock_cache]:::system
    M --> N([Backorder hoàn tất]):::terminal
    C -- Hủy backorder --> O{Role?}:::decision
    O -- ADMIN --> P[Hủy backorder picking\ntrên Odoo]:::odoo
    P --> Q([Backorder đã hủy]):::terminal
    O -- POS User --> R([Từ chối\nLiên hệ ADMIN]):::warning
```
