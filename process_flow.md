# VJ Mobile POS — Process Flow

> Trạng thái: Draft  
> Cập nhật: 2026-04-14  
> Phiên bản: 0.6

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

## Mục lục

### Phần A — Business Flows (Nghiệp vụ)

| # | Luồng | Mô tả |
|---|---|---|
| A1 | Bán hàng — Tạo đơn & Xác nhận | Chọn SP, serial, xác nhận hoặc lưu nháp |
| A2 | Nhận thanh toán | Chọn KH (nếu chưa), ghi nhận thủ công, multi-payment, đặt cọc |
| A3 | Xuất kho | Auto validate picking + serial + backorder |
| A4 | Quản lý Backorder | Theo dõi, validate, hủy backorder |
| A5 | Hủy đơn hàng | Hủy theo trạng thái + phân quyền |

### Phần B — Technical Flows (Kỹ thuật)

| # | Luồng | Mô tả |
|---|---|---|
| B1 | Đăng nhập | JWT auth + hr.employee info |
| B2 | Làm mới Cache | TTL auto-expire + flush thủ công |
| B3 | Quản lý User | Tạo user liên kết hr.employee, phân quyền |
| B4 | Reset Password | Gửi email token qua AWS SES |
| B5 | Kiểm tra tồn kho | Tra cứu stock.quant theo location |
| B6 | Tra cứu MST | VietQR API + TTLCache |
| B7 | In phiếu bán hàng | Generate PDF (A4/A5) từ template |

---

# Phần A — Business Flows

---

## A1. Luồng Bán hàng — Tạo đơn & Xác nhận

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
    B --> C[Tìm kiếm sản phẩm\ntên / SKU / barcode / serial]:::user
    C --> D[Thêm SP vào đơn\nChọn serial nếu có]:::user
    D --> E{SP có serial?}:::decision
    E -- Có --> F[Gợi ý serial từ stock.lot\nUser chọn / nhập]:::user
    E -- Không --> G[Thêm trực tiếp]:::system
    F --> H{Còn thêm SP?}:::decision
    G --> H
    H -- Có --> C
    H -- Không --> I[Xem lại đơn hàng\ngiá / số lượng]:::user
    I --> J{Hành động}:::decision
    J -- Lưu nháp --> K[INSERT draft_orders\nexpires_at = now + 8h]:::db
    K --> L([Đơn nháp lưu thành công]):::terminal
    J -- Xác nhận --> M[sale.order.create\nsale.order.action_confirm]:::odoo
    M --> N[Odoo tự tạo stock.picking]:::odoo
    N --> O([Tiếp theo: Luồng A3 & A2]):::terminal
    J -- Hủy --> P([Kết thúc]):::terminal
```

**Ghi chú:**
- KH không bắt buộc ở bước này — chỉ bắt buộc trước khi thanh toán (A2)
- SP hết hàng: cho phép thêm kèm cảnh báo, ghi `**` trong tên SP (OQ-Q04)
- Serial chọn ngay khi thêm SP (OQ-Q04)

---

## A2. Luồng Nhận thanh toán (Ghi nhận thủ công)

```mermaid
flowchart TD
    classDef user fill:#DBEAFE,stroke:#2563EB,color:#1e3a5f
    classDef system fill:#F1F5F9,stroke:#64748B,color:#1e293b
    classDef odoo fill:#EDE9FE,stroke:#7C3AED,color:#3b0764
    classDef db fill:#DCFCE7,stroke:#16A34A,color:#14532d
    classDef decision fill:#FEF9C3,stroke:#CA8A04,color:#713f12
    classDef terminal fill:#F8FAFC,stroke:#334155,color:#0f172a
    classDef warning fill:#FEE2E2,stroke:#DC2626,color:#7f1d1d

    START{Điểm vào}:::decision -- Đơn mới\nvừa xác nhận --> CK
    START -- Thu phần còn lại\nkhách quay lại --> SEARCH[Tìm đơn theo\ntên / SĐT / MST / email]:::user
    SEARCH --> FOUND[Chọn đơn hàng\nHiển thị đã cọc & còn lại]:::system
    FOUND --> B

    CK{Đơn có KH chưa?}:::decision
    CK -- Chưa có --> SELKH[Chọn / Tạo khách hàng\ntên, SĐT, email]:::user
    CK -- Đã có --> B
    SELKH --> MST{KH có MST?}:::decision
    MST -- Có --> LOOKUP[Tra cứu VietQR API\nAuto-fill thông tin]:::system
    MST -- Không --> B
    LOOKUP --> B

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

**Ghi chú:**
- **Bắt buộc chọn/tạo KH trước khi thanh toán** — nếu đơn chưa có KH sẽ mở dialog chọn KH
- Hỗ trợ đặt cọc nhiều lần (OQ-O01)
- Đơn cọc không hết hạn (OQ-O02)
- Nhiều phương thức trong 1 đơn (OQ-C03)

---

## A3. Luồng Xuất kho (Auto validate + Serial + Backorder)

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
    G --> H([Backorder → Luồng A4]):::terminal
    E -- Không --> I[Auto validate picking\nbutton_validate]:::odoo
    E -- Có --> J[Gợi ý serial từ stock.lot\ntheo sản phẩm đã chọn]:::system
    J --> K[POS user chọn / nhập serial]:::user
    K --> I
    I --> L[Tồn kho cập nhật trên Odoo]:::odoo
    L --> M[Invalidate stock_cache]:::system
    M --> N([Hoàn tất xuất kho]):::terminal
```

---

## A4. Luồng Quản lý Backorder

```mermaid
flowchart TD
    classDef user fill:#DBEAFE,stroke:#2563EB,color:#1e3a5f
    classDef system fill:#F1F5F9,stroke:#64748B,color:#1e293b
    classDef odoo fill:#EDE9FE,stroke:#7C3AED,color:#3b0764
    classDef db fill:#DCFCE7,stroke:#16A34A,color:#14532d
    classDef decision fill:#FEF9C3,stroke:#CA8A04,color:#713f12
    classDef terminal fill:#F8FAFC,stroke:#334155,color:#0f172a
    classDef warning fill:#FEE2E2,stroke:#DC2626,color:#7f1d1d

    A([Backorder tạo sau Luồng A3]):::terminal --> B[Hiển thị danh sách backorder\ntheo user / location]:::system
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

---

## A5. Luồng Hủy đơn hàng

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

# Phần B — Technical Flows

---

## B1. Luồng Đăng nhập (Authentication)

```mermaid
sequenceDiagram
    actor User as Người dùng
    participant FE as Frontend (Vue)
    participant BE as Backend (FastAPI)
    participant DB as PostgreSQL
    participant Odoo as Odoo (hr.employee)

    rect rgb(219, 234, 254)
        Note over User,FE: Thao tác người dùng
        User->>FE: Nhập username / password
        FE->>BE: POST /auth/login
    end
    rect rgb(241, 245, 249)
        Note over BE,Odoo: Xử lý hệ thống
        BE->>DB: Kiểm tra credentials + check is_active + is_locked
        alt Sai >= 5 lần
            BE-->>FE: 401 ACCOUNT_LOCKED
        else Tài khoản bị vô hiệu hóa
            BE-->>FE: 401 ACCOUNT_DISABLED
        end
        DB-->>BE: User record + hr_employee_id + roles + locations
        BE->>Odoo: XML-RPC hr.employee.search_read(hr_employee_id)
        Odoo-->>BE: Tên, email, SĐT, phòng ban, chức danh
        BE->>DB: Lưu refresh token (bảng refresh_tokens)
        BE-->>FE: Access token + Refresh token + user info + employee info
    end
    rect rgb(219, 234, 254)
        Note over FE,User: Phản hồi người dùng
        FE->>FE: Lưu token (memory) + refresh token (localStorage)
        FE->>FE: Khởi động idle watcher (5 phút)
        FE-->>User: Vào màn hình chính
    end
```

**Ghi chú:**
- Credentials quản lý trên PostgreSQL local — không dùng tài khoản Odoo
- Thông tin cá nhân (tên, email, SĐT...) lấy từ `hr.employee` trên Odoo
- Khóa tài khoản sau 5 lần sai → ADMIN mở thủ công (OQ-M04, OQ-W03)
- Idle timeout 5 phút → tự đăng xuất (OQ-M03)

---

## B2. Luồng Làm mới Cache

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
    E -- Danh mục SP --> GA[cache.clear category_cache]:::system
    E -- Tồn kho --> GB[cache.clear stock_cache]:::system
    E -- Khách hàng --> GC[cache.clear customer_cache]:::system
    E -- Tất cả --> H[cache.clear all]:::system
    F --> I([Lần gọi tiếp theo fetch từ Odoo]):::terminal
    G --> I
    GA --> I
    GB --> I
    GC --> I
    H --> I
    D --> I
```

**TTL mặc định:**

| Cache | TTL |
|---|---|
| Sản phẩm + giá | 30 phút |
| Pricelist | 1 giờ |
| Danh mục SP (`product.category`) | 8 tiếng |
| Tồn kho (`stock.quant`) | 5 phút |
| Khách hàng (`res.partner`) | 60 phút |
| MST (VietQR) | 10 phút |

---

## B3. Luồng Quản lý User (ADMIN)

```mermaid
flowchart TD
    classDef user fill:#DBEAFE,stroke:#2563EB,color:#1e3a5f
    classDef system fill:#F1F5F9,stroke:#64748B,color:#1e293b
    classDef odoo fill:#EDE9FE,stroke:#7C3AED,color:#3b0764
    classDef db fill:#DCFCE7,stroke:#16A34A,color:#14532d
    classDef decision fill:#FEF9C3,stroke:#CA8A04,color:#713f12
    classDef terminal fill:#F8FAFC,stroke:#334155,color:#0f172a
    classDef warning fill:#FEE2E2,stroke:#DC2626,color:#7f1d1d

    A([ADMIN tạo user]):::terminal --> B[Tìm kiếm nhân viên\nGET /admin/employees/search]:::user
    B --> C[Chọn hr.employee từ Odoo]:::user
    C --> D{Employee đã có\ntài khoản POS?}:::decision
    D -- Có --> E([Từ chối\nEmployee đã liên kết]):::warning
    D -- Chưa --> F[Backend lấy thông tin\ntên, email, SĐT, phòng ban]:::odoo
    F --> G[ADMIN nhập:\nusername, password]:::user
    G --> H[Chọn role:\nADMIN / POS User]:::user
    H --> I[Chọn warehouse + locations]:::user
    I --> J[INSERT users table\nhr_employee_id + credentials]:::db
    J --> K([User tạo thành công]):::terminal

    L([ADMIN vô hiệu hóa user]):::terminal --> M[PATCH /admin/users/id/status]:::user
    M --> N[Set is_active = false]:::db
    N --> O[Xóa tất cả refresh_tokens\ncủa user]:::db
    O --> P([User bị vô hiệu hóa\nToken invalidate ngay]):::terminal

    Q([ADMIN mở khóa user]):::terminal --> R[POST /admin/users/id/unlock]:::user
    R --> S[Reset login_attempts = 0\nis_locked = false]:::db
    S --> T([User đã mở khóa]):::terminal
```

**Ghi chú:**
- Thông tin cá nhân kế thừa từ `hr.employee` — không nhập thủ công
- 1 `hr.employee` chỉ liên kết 1 user POS
- Admin mặc định không cần `hr_employee_id`
- Vô hiệu hóa → invalidate token ngay lập tức (OQ-M02)

---

## B4. Luồng Reset Password

```mermaid
sequenceDiagram
    actor Admin as ADMIN
    actor User as POS User
    participant FE as Frontend
    participant BE as Backend (FastAPI)
    participant DB as PostgreSQL
    participant SES as AWS SES (SMTP)

    rect rgb(219, 234, 254)
        Note over Admin,FE: Kịch bản 1: ADMIN reset cho user
        Admin->>FE: Bấm "Reset Password" trên user
        FE->>BE: POST /admin/users/{id}/reset-password
    end
    rect rgb(241, 245, 249)
        Note over BE,SES: Xử lý hệ thống
        BE->>DB: Tạo reset_token + expires_at (1 giờ)
        BE->>SES: Gửi email chứa link reset
        Note over SES: Email gửi tới hr.employee.work_email
        SES-->>User: Email với link reset password
    end

    rect rgb(219, 234, 254)
        Note over User,FE: Kịch bản 2: User tự reset
        User->>FE: Bấm "Quên mật khẩu" tại trang login
        FE->>BE: POST /auth/forgot-password {email}
    end
    rect rgb(241, 245, 249)
        Note over BE,SES: Xử lý hệ thống
        BE->>DB: Tìm user theo email (từ hr.employee)
        BE->>DB: Tạo reset_token + expires_at (1 giờ)
        BE->>SES: Gửi email chứa link reset
        SES-->>User: Email với link reset password
    end

    rect rgb(219, 234, 254)
        Note over User,FE: Đặt mật khẩu mới
        User->>FE: Click link → nhập mật khẩu mới
        FE->>BE: POST /auth/reset-password {token, new_password}
    end
    rect rgb(241, 245, 249)
        Note over BE,DB: Xử lý hệ thống
        BE->>DB: Validate token + chưa hết hạn
        BE->>DB: Update password_hash
        BE->>DB: Xóa tất cả refresh_tokens của user
        BE-->>FE: Thành công → redirect login
    end
```

**Ghi chú:**
- Email lấy từ `hr.employee.work_email` trên Odoo
- Token reset hết hạn sau 1 giờ
- Password policy: min 8 ký tự, chữ hoa + số + ký tự đặc biệt (OQ-W01)
- Sau reset → xóa tất cả session cũ

---

## B5. Luồng Kiểm tra tồn kho

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

## B6. Luồng Tra cứu MST

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

## B7. Luồng In phiếu bán hàng (Generate PDF)

```mermaid
sequenceDiagram
    actor User as Nhân viên
    participant FE as Frontend
    participant BE as FastAPI (WeasyPrint + Jinja2)
    participant DB as PostgreSQL

    rect rgb(219, 234, 254)
        Note over User,FE: Thao tác người dùng
        User->>FE: Bấm "In phiếu" → Chọn loại + khổ giấy
        FE->>BE: GET /print/{receipt_type}?order_id=X&paper_size=A4
        Note over BE: receipt_type: order_confirm / deposit / payment
    end
    rect rgb(241, 245, 249)
        Note over BE,DB: Xử lý hệ thống
        BE->>DB: Lấy template HTML/CSS theo loại phiếu
        DB-->>BE: Template Jinja2
        BE->>BE: Render template với dữ liệu đơn hàng
        Note over BE: Bao gồm: số tiền bằng chữ (TV), vùng chữ ký
        BE->>BE: WeasyPrint: HTML/CSS → PDF binary
        BE-->>FE: PDF file (Content-Type: application/pdf)
    end
    rect rgb(219, 234, 254)
        Note over FE,User: Phản hồi người dùng
        FE-->>User: Mở PDF preview / download
    end
```

**Ghi chú:**
- User chọn A4 hoặc A5 khi in (OQ-Z01)
- Chỉ preview / download, không in trực tiếp (OQ-U02)

---

## Tham chiếu

| Tài liệu | Mô tả |
|---|---|
| [scope.md](scope.md) | Scope & Architecture |
| [api_contract.md](api_contract.md) | API Contract (40 endpoints) |
| [frontend_design.md](frontend_design.md) | Thiết kế giao diện |
| [open_questions.md](open_questions.md) | Câu hỏi đã xác nhận |
