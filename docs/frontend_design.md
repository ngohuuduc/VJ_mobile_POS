# VJ Mobile POS — Frontend Design

> Trạng thái: Draft  
> Cập nhật: 2026-04-14  
> Phiên bản: 0.2

---

## 0. Tham chiếu giao diện — Odoo 19 POS

VJ Mobile POS tham khảo layout và UX pattern của **Odoo 19 Point of Sale** để tạo trải nghiệm quen thuộc cho người dùng đã sử dụng Odoo.

### 0.1 Layout chính của Odoo 19 POS

Odoo 19 POS chia màn hình thành **3 vùng chính**:

```
┌──────────────────────────────────────────────────────────────────┐
│  [≡] [Đơn #1] [Đơn #2] [+]        [🔍 Tìm SP] [📷] [👤] [⚙] │  ← Header
├────────────────────────────┬─────────────────────────────────────┤
│                            │                                     │
│   ORDER LINES              │     PRODUCT CATALOG                 │
│   ─────────────            │     ──────────────                  │
│   SP A      x2   200,000  │     [Danh mục 1] [DM 2] [DM 3]     │
│   SP B      x1   150,000  │                                     │
│   SP C      x3   450,000  │     ┌─────┐ ┌─────┐ ┌─────┐        │
│                            │     │ img │ │ img │ │ img │        │
│                            │     │ SP1 │ │ SP2 │ │ SP3 │        │
│   ─────────────            │     │ giá │ │ giá │ │ giá │        │
│   Tổng:       800,000     │     └─────┘ └─────┘ └─────┘        │
│                            │                                     │
│   ┌─────────────────────┐  │     ┌─────┐ ┌─────┐ ┌─────┐        │
│   │ [Qty] [%] [Price]  │  │     │ img │ │ img │ │ img │        │
│   │     NUMPAD          │  │     │ SP4 │ │ SP5 │ │ SP6 │        │
│   │  [1] [2] [3]       │  │     │ giá │ │ giá │ │ giá │        │
│   │  [4] [5] [6]       │  │     └─────┘ └─────┘ └─────┘        │
│   │  [7] [8] [9]       │  │                                     │
│   │  [.] [0] [⌫]       │  │                                     │
│   └─────────────────────┘  │                                     │
│                            │                                     │
│  [👤 Khách hàng] [💳 Thanh toán]                                │
└────────────────────────────┴─────────────────────────────────────┘
```

### 0.2 Các UI pattern chính từ Odoo 19

| Pattern | Odoo 19 POS | Áp dụng cho VJ Mobile POS |
|---|---|---|
| **2-panel layout** | Trái: order lines + numpad. Phải: product grid | Giữ nguyên — layout quen thuộc cho POS |
| **Category bar** | Tab danh mục ngang phía trên product grid | Dùng danh mục kế thừa từ Odoo `product.category` |
| **Product grid** | Card với ảnh + tên + giá, click để thêm | Tương tự, bật/tắt ảnh (OQ-K04) |
| **Numpad** | Nằm dưới order lines, dùng cho Qty/Discount/Price | Giữ lại cho Qty. Bỏ Discount (ngoài scope) |
| **Header bar** | Search, tabs đơn hàng, user, menu | Tương tự + thêm warehouse badge |
| **Payment screen** | Full-screen, chọn PT → nhập số tiền → Validate | Dialog thay vì full-screen (nhiều PT trong 1 đơn) |
| **Customer assign** | Nút trên order panel, mở popup tìm/tạo KH | Không bắt buộc khi thêm SP. Bắt buộc trước khi thanh toán |
| **Dark mode** | Có (mới trong v19) | Chưa cần giai đoạn đầu (backlog) |
| **Long-press info** | Nhấn giữ SP → popup thông tin chi tiết | Áp dụng — hiện tồn kho, serial count |
| **Multi-tab orders** | Tab chuyển đổi giữa các đơn đang mở | Không cần — VJ chỉ 1 đơn tại 1 thời điểm + draft |
| **Offline mode** | Full offline + auto sync | Ngoài scope |

### 0.3 Điểm khác biệt VJ Mobile POS so với Odoo POS

| # | Odoo 19 POS | VJ Mobile POS | Lý do |
|---|---|---|---|
| 1 | Walk-in customer (không bắt buộc KH) | Cho phép tạo đơn không cần KH. **Bắt buộc KH trước khi thanh toán** | Yêu cầu nghiệp vụ (OQ-J01) |
| 2 | Discount (%) trên numpad | **Không có** discount | Ngoài scope (OQ-A02) |
| 3 | Thanh toán: full-screen riêng | **Dialog** overlay trên chi tiết đơn | Hỗ trợ multi-payment + đặt cọc nhiều lần |
| 4 | pos.order | **sale.order** | Dùng Sale module, không dùng POS module |
| 5 | Multi-tab đơn hàng | 1 đơn + lưu nháp (8h) | Đơn giản hóa |
| 6 | Serial nhập khi validate picking | Serial nhập **ngay khi thêm SP** (nếu có) | UX: chọn serial sớm, biết hàng còn không |
| 7 | Invoice tự động | Invoice **draft** — kế toán post riêng | Quy trình 2 bước |
| 8 | POS-specific reports | Không có reports | Ngoài scope |
| 9 | Offline + sync | **Online only** | Yêu cầu live network |

### 0.4 Tham khảo thêm

- [Odoo 19 POS Documentation](https://www.odoo.com/documentation/19.0/applications/sales/point_of_sale.html)
- [Odoo 19 POS — Use Guide](https://www.odoo.com/documentation/19.0/applications/sales/point_of_sale/use.html)
- [Odoo 19 POS New Features — SerpentCS](https://www.serpentcs.com/blog/odoo-pos-390/odoo-19-pos-features-whats-new-in-point-of-sale-for-retail-restaurants-661)
- [Odoo 19 New Features — ECOSIRE](https://ecosire.com/blog/odoo-19-new-features)

---

## 1. Tech Stack

| Thành phần | Công nghệ | Ghi chú |
|---|---|---|
| Framework | Vue 3 (Composition API) | SPA |
| UI Library | Quasar Framework | Component-rich, responsive, touch-friendly |
| State Management | Pinia | Shared state: auth, cart, orders, products, customers |
| HTTP Client | `fetch` (built-in) | Wrapper tự viết cho JWT, refresh token, error handling |
| Router | Vue Router 4 | Tích hợp sẵn với Quasar |
| Ngôn ngữ UI | Tiếng Việt | Không cần i18n đa ngôn ngữ |

---

## 2. Thiết bị & Trình duyệt mục tiêu

| Thiết bị | Trình duyệt | Layout | Ghi chú |
|---|---|---|---|
| Máy tính | Chrome (latest) | Desktop — sidebar + main content | Bàn phím + chuột |
| iPad | Safari (iOS/iPadOS) | Tablet — bottom navigation + full-screen | Touch-friendly, 768px+ |

---

## 3. Hệ thống Layout

### 3.1 Desktop Layout (>= 1024px)

```
┌─────────────────────────────────────────────────────┐
│  Header: Logo | Tên user | Role | Warehouse | Logout│
├────────────┬────────────────────────────────────────┤
│            │                                        │
│  Sidebar   │         Main Content                   │
│  Navigation│         (Router View)                  │
│            │                                        │
│  - Tạo đơn │                                        │
│  - Đơn hàng│                                        │
│  - Tồn kho │                                        │
│  - Backorder│                                        │
│  ──────────│                                        │
│  ADMIN     │                                        │
│  - Users   │                                        │
│  - Template│                                        │
│            │                                        │
└────────────┴────────────────────────────────────────┘
```

**Quasar components**: `QLayout`, `QHeader`, `QDrawer` (fixed, mini-to-full), `QPageContainer`

### 3.2 Tablet Layout (768px — 1023px)

```
┌─────────────────────────────────────────────────────┐
│  Header: Logo | Tên user | Warehouse               │
├─────────────────────────────────────────────────────┤
│                                                     │
│                 Main Content                        │
│                 (Router View, full-width)            │
│                                                     │
│                                                     │
│                                                     │
├─────────┬─────────┬─────────┬─────────┬─────────────┤
│  Tạo đơn│ Đơn hàng│ Tồn kho │Backorder│    ...      │
└─────────┴─────────┴─────────┴─────────┴─────────────┘
```

**Quasar components**: `QLayout`, `QHeader`, `QFooter` chứa `QTabs` hoặc `QBtnGroup`

### 3.3 Breakpoint Strategy

| Breakpoint | Layout | Sidebar | Navigation |
|---|---|---|---|
| >= 1024px | Desktop | `QDrawer` cố định bên trái | Sidebar menu |
| 768px — 1023px | Tablet | Ẩn sidebar | Bottom navigation |
| < 768px | Không hỗ trợ chính thức | — | — |

Sử dụng Quasar `$q.screen` API để chuyển đổi layout theo breakpoint.

---

## 4. Điều hướng (Navigation)

### 4.1 Menu chính — POS User

| # | Menu item | Icon | Route | Mô tả |
|---|---|---|---|---|
| 1 | Tạo đơn hàng | `shopping_cart` | `/orders/new` | Màn hình chính: chọn KH, SP, tạo đơn |
| 2 | Đơn hàng | `receipt_long` | `/orders` | Danh sách đơn (1 tuần, chỉ đơn của mình) |
| 3 | Kiểm tra tồn kho | `inventory_2` | `/inventory` | Tra cứu stock theo location |
| 4 | Backorder | `local_shipping` | `/backorders` | Danh sách backorder chờ xử lý |

### 4.2 Menu bổ sung — ADMIN

| # | Menu item | Icon | Route | Mô tả |
|---|---|---|---|---|
| 5 | Quản lý User | `people` | `/admin/users` | CRUD user, gán role/location |
| 6 | Quản lý Template | `description` | `/admin/templates` | WYSIWYG editor + preview |
| 7 | Cache Management | `cached` | `/admin/cache` | Flush cache thủ công |

### 4.3 Route Guards

```
/login              → Public (không cần auth)
/orders/*           → Require auth
/inventory          → Require auth
/backorders         → Require auth
/admin/*            → Require auth + role ADMIN
```

---

## 5. Danh sách màn hình

### 5.1 Đăng nhập (`/login`)

**Mục đích**: Xác thực người dùng.

**Layout**: Full-screen centered form, không sidebar/nav.

```
┌─────────────────────────────────────┐
│                                     │
│          [Logo công ty]             │
│                                     │
│   ┌─────────────────────────────┐   │
│   │  Username                   │   │
│   └─────────────────────────────┘   │
│   ┌─────────────────────────────┐   │
│   │  Password          [👁]    │   │
│   └─────────────────────────────┘   │
│                                     │
│   [ Đăng nhập ]                     │
│                                     │
│   Sai 3/5 lần — cảnh báo           │
│                                     │
└─────────────────────────────────────┘
```

| Thành phần | Quasar Component |
|---|---|
| Form | `QForm` + validation rules |
| Input | `QInput` (username, password) |
| Nút đăng nhập | `QBtn` (primary) |
| Thông báo lỗi | `QBanner` hoặc `Notify` |
| Toggle password | `QInput` icon slot |

**Hành vi đặc biệt:**
- Khóa tài khoản sau 5 lần sai (OQ-M04)
- Hiển thị số lần còn lại khi sai >= 3 lần

---

### 5.2 Tạo đơn hàng (`/orders/new`)

**Mục đích**: Chọn khách hàng, thêm sản phẩm, xem giỏ hàng, xác nhận hoặc lưu nháp.

**Layout**: Tham khảo Odoo 19 POS — 2-panel (trái: order, phải: products).

#### Desktop Layout (theo pattern Odoo 19)

```
┌──────────────────────────────────────────────────────────────────┐
│  [≡]  VJ Mobile POS    [🔍 Tìm SP: tên/SKU/barcode/serial]  👤 │  ← Header
├───────────────────────────────┬──────────────────────────────────┤
│                               │                                  │
│  [👤 Chọn KH]  (tùy chọn)    │  [Tất cả] [Điện tử] [Phụ kiện]  │  ← Danh mục
│  ─────────────────────────    │                                  │
│                               │  ┌─────┐ ┌─────┐ ┌─────┐        │
│  ORDER LINES                  │  │ img │ │ img │ │ img │        │
│  ─────────────                │  │ SP1 │ │ SP2 │ │ SP3 │        │
│  SP A    SN:123   x2  200k   │  │ giá │ │ giá │ │ giá │        │
│  SP B             x1  150k   │  │ tồn │ │ tồn │ │ tồn │        │
│  ⚠ SP C** (hết)   x3  450k   │  └─────┘ └─────┘ └─────┘        │
│  ─────────────────────────    │                                  │
│  Tổng cộng:        800,000   │  ┌─────┐ ┌─────┐ ┌─────┐        │
│                               │  │ img │ │ img │ │ img │        │
│  ┌──────────────────────────┐ │  │ SP4 │ │ SP5 │ │ SP6 │        │
│  │ [Qty]           [Price]  │ │  │ giá │ │ giá │ │ giá │        │
│  │  [1] [2] [3]             │ │  │ tồn │ │ tồn │ │ tồn │        │
│  │  [4] [5] [6]   NUMPAD   │ │  └─────┘ └─────┘ └─────┘        │
│  │  [7] [8] [9]             │ │                                  │
│  │  [.] [0] [⌫]             │ │  ↓ cuộn vô hạn (infinite scroll)│
│  └──────────────────────────┘ │                                  │
│                               │                                  │
│  [💾 Lưu nháp]  [💳 Thanh toán]  [✓ Xác nhận đơn]              │
└───────────────────────────────┴──────────────────────────────────┘
```

**Ghi chú layout:**
- **KH không bắt buộc** khi thêm SP. Nút [👤 Chọn KH] ở đầu panel trái, hiển thị tên KH nếu đã chọn
- **Bắt buộc chọn KH trước khi bấm [💳 Thanh toán]** — nếu chưa có KH, mở dialog chọn/tạo KH
- Product card hiển thị **tồn kho** (OQ-S05). SP hết hàng có ghi chú `**` (OQ-Q04)
- Serial chọn ngay khi thêm SP (OQ-Q04)
- Product grid: **3 cột**, cuộn vô hạn + load batch (OQ-R03, OQ-S04)
- Không có nút Discount (%)
- Swipe trái trên order line để xóa SP (iPad, OQ-Q05)

#### Tablet Layout (iPad)

```
┌──────────────────────────────────────────────────────────────────┐
│  [≡]  VJ Mobile POS    [🔍 Tìm SP]                          👤 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  [👤 Chọn KH]  (tùy chọn — bắt buộc trước khi thanh toán)      │
│                                                                  │
│  [Tất cả] [Điện tử] [Phụ kiện] [Linh kiện]    ← scroll ngang  │
│                                                                  │
│  ┌─────┐ ┌─────┐ ┌─────┐                                        │
│  │ img │ │ img │ │ img │               ← 3 cột, cuộn vô hạn   │
│  │ SP1 │ │ SP2 │ │ SP3 │                                        │
│  │ giá │ │ giá │ │ giá │                                        │
│  │ tồn │ │ tồn │ │ tồn │                                        │
│  └─────┘ └─────┘ └─────┘                                        │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│  🛒 Giỏ hàng (3 SP)                         Tổng: 800,000 ▲   │
│  [💾 Nháp]  [💳 Thanh toán]  [✓ Xác nhận]                      │
└──────────────────────────────────────────────────────────────────┘
```

**Tablet**: Product grid full-width 3 cột, giỏ hàng thu gọn ở dưới (expand ▲), swipe xóa SP

| Thành phần | Quasar Component | Ghi chú |
|---|---|---|
| Tìm kiếm KH | `QSelect` (filter, async) | Tìm theo tên, SĐT, email, MST |
| Tạo KH mới | `QDialog` chứa `QForm` | Bắt buộc: tên, SĐT, email |
| Tra cứu MST | `QInput` + nút tra cứu | Auto-fill từ VietQR API |
| Lọc danh mục SP | `QSelect` | `product.category` từ Odoo |
| Tìm kiếm SP | `QInput` (debounce 300ms) | Tên, SKU, barcode, serial |
| Danh sách SP | `QCard` grid hoặc `QTable` | Bật/tắt ảnh SP (OQ-K04) |
| Giỏ hàng | `QTable` (editable qty) | Số lượng, giá, tổng dòng |
| Cảnh báo hết hàng | `QBanner` (type warning) | Cho phép thêm kèm cảnh báo (OQ-K03) |
| Nút hành động | `QBtn` | Lưu nháp / Xác nhận / Hủy |

**Hành vi đặc biệt:**
- Warehouse tự động gán theo user (không hiển thị chọn)
- Sản phẩm hết hàng: cho phép thêm + hiển thị cảnh báo vàng, serial để trống
- Không có biến thể sản phẩm (OQ-K01)
- Lưu nháp: tối đa 8 giờ, lưu PostgreSQL

---

### 5.3 Danh sách đơn hàng (`/orders`)

**Mục đích**: Xem, tìm kiếm, lọc đơn hàng. Truy cập chi tiết đơn.

```
┌─────────────────────────────────────────────────────┐
│  Đơn hàng của tôi                                   │
│                                                     │
│  [Tìm: tên KH / SĐT / MST / email]                │
│  [Trạng thái ▼] [Ngày ▼]                           │
│                                                     │
│  ┌─────────────────────────────────────────────────┐│
│  │ Mã đơn  │ Khách hàng │ Tổng tiền │ Trạng thái ││
│  │─────────│────────────│───────────│────────────││
│  │ SO-001  │ Nguyễn A   │ 2,000,000 │ Đã xác nhận││
│  │ SO-002  │ Trần B     │   500,000 │ Đặt cọc    ││
│  │ SO-003  │ Lê C       │ 1,200,000 │ Nháp       ││
│  └─────────────────────────────────────────────────┘│
│                                                     │
│  Trang 1/5  [<] [1] [2] [3] [>]                    │
└─────────────────────────────────────────────────────┘
```

| Thành phần | Quasar Component | Ghi chú |
|---|---|---|
| Tìm kiếm | `QInput` (debounce) | Filter trên dữ liệu đã cache |
| Bộ lọc trạng thái | `QSelect` (multiple) | Draft, Confirmed, Deposited, Paid, Cancelled |
| Bảng đơn hàng | `QTable` (sortable, pagination) | Kéo từ cache (1 tuần) |
| Badge trạng thái | `QBadge` hoặc `QChip` | Màu sắc theo trạng thái |

**Hành vi đặc biệt:**
- Dữ liệu: load toàn bộ đơn hàng (1 tuần) vào cache frontend, filter client-side (OQ-L01)
- POS User chỉ thấy đơn của mình (OQ-L03)
- ADMIN thấy tất cả đơn
- Click vào row → đi đến chi tiết đơn

---

### 5.4 Chi tiết đơn hàng (`/orders/:id`)

**Mục đích**: Xem thông tin đơn, thực hiện các hành động: thanh toán, đặt cọc, hủy, in phiếu.

```
┌─────────────────────────────────────────────────────┐
│  ← Quay lại         Đơn hàng SO-001                │
│                                                     │
│  ┌─ Thông tin KH ──────┐  ┌─ Trạng thái ──────────┐│
│  │ Tên: Nguyễn Văn A   │  │ Confirmed              ││
│  │ SĐT: 0901 234 567   │  │ Tạo: 14/04/2026 10:30 ││
│  │ MST: 0312345678      │  │ NV: Trần Thị B        ││
│  └──────────────────────┘  └────────────────────────┘│
│                                                     │
│  ┌─ Sản phẩm ──────────────────────────────────────┐│
│  │ Tên SP     │ SKU   │ SL │ Đơn giá  │ Thành tiền ││
│  │ Sản phẩm A │ A-001 │  2 │ 100,000  │   200,000 ││
│  │ Sản phẩm B │ B-002 │  1 │ 150,000  │   150,000 ││
│  │────────────│───────│────│──────────│───────────││
│  │                          Tổng:       350,000     ││
│  └──────────────────────────────────────────────────┘│
│                                                     │
│  ┌─ Lịch sử thanh toán ────────────────────────────┐│
│  │ Ngày       │ PT thanh toán │ Số tiền  │ Loại    ││
│  │ 14/04 10:35│ Tiền mặt      │ 100,000  │ Đặt cọc ││
│  │                    Còn lại:   250,000            ││
│  └──────────────────────────────────────────────────┘│
│                                                     │
│  [Thanh toán] [In phiếu ▼] [Hủy đơn]              │
└─────────────────────────────────────────────────────┘
```

| Thành phần | Quasar Component | Ghi chú |
|---|---|---|
| Breadcrumb / Back | `QBtn` flat | Quay lại danh sách |
| Card thông tin | `QCard` | KH, trạng thái, nhân viên |
| Bảng SP | `QTable` (read-only) | Danh sách line items |
| Lịch sử thanh toán | `QTable` hoặc `QTimeline` | Hiển thị các lần thanh toán |
| Nút thanh toán | `QBtn` primary | → Mở dialog thanh toán |
| Nút in phiếu | `QBtnDropdown` | Chọn loại: xác nhận / cọc / thanh toán |
| Nút hủy | `QBtn` negative | ADMIN only (nếu đã xác nhận) |

**Hành vi đặc biệt:**
- Đặt cọc nhiều lần được phép (OQ-O01)
- Đơn đặt cọc không hết hạn (OQ-O02)
- Hủy đơn: Draft → POS User được phép. Confirmed → chỉ ADMIN (OQ-D01)
- In phiếu: gọi API → mở PDF preview (OQ-I04)

---

### 5.5 Dialog Thanh toán

**Mục đích**: Ghi nhận thanh toán thủ công, hỗ trợ nhiều phương thức.  
**Điều kiện**: Bắt buộc đã chọn KH trước khi mở dialog. Nếu chưa có → mở dialog chọn/tạo KH trước.

```
┌─────────────────────────────────────┐
│  Thanh toán — SO-001                │
│  👤 Nguyễn Văn A  📞 0901 234 567  │
│                                     │
│  Tổng đơn:          350,000        │
│  Đã thanh toán:     100,000        │
│  Còn lại:           250,000        │
│                                     │
│  ┌─ Ghi nhận mới ──────────────┐   │
│  │ Phương thức: [Tiền mặt ▼]  │   │
│  │ Số tiền:     [250,000    ]  │   │
│  │ Ghi chú:     [           ]  │   │
│  │              [+ Thêm PT]    │   │
│  └─────────────────────────────┘   │
│                                     │
│  PT đã thêm:                        │
│  ✓ Tiền mặt: 150,000               │
│  ✓ Chuyển khoản: 100,000           │
│  Tổng: 250,000 ✓                   │
│                                     │
│  [Hủy]              [Xác nhận]     │
└─────────────────────────────────────┘
```

| Thành phần | Quasar Component | Ghi chú |
|---|---|---|
| Dialog | `QDialog` (persistent) | Ngăn đóng khi click ngoài |
| Chọn PT | `QSelect` | Tiền mặt, CK, VNPay, MoMo, Thẻ |
| Nhập số tiền | `QInput` (type number, format VND) | Auto-suggest số còn lại |
| Danh sách PT đã thêm | `QList` | Có nút xóa từng dòng |
| Tiền thừa | Hiển thị khi tiền mặt > còn lại | Tính tự động |
| Validate | Tổng >= Còn lại | Disable nút xác nhận nếu chưa đủ |

---

### 5.6 Kiểm tra tồn kho (`/inventory`)

**Mục đích**: Tra cứu tồn kho sản phẩm theo location được gán cho user.

```
┌─────────────────────────────────────────────────────┐
│  Kiểm tra tồn kho                                  │
│                                                     │
│  Location: Kho Quận 7 — Kệ A                       │
│  [Tìm sản phẩm: tên / SKU / barcode]               │
│                                                     │
│  ┌─────────────────────────────────────────────────┐│
│  │ SKU   │ Tên SP       │ Tồn kho │ SL khả dụng  ││
│  │ A-001 │ Sản phẩm A   │      50 │          48  ││
│  │ B-002 │ Sản phẩm B   │      20 │          20  ││
│  │ C-003 │ Sản phẩm C   │       0 │           0  ││
│  └─────────────────────────────────────────────────┘│
│                                                     │
│  Cache cập nhật: 14/04/2026 10:25 (TTL: 5 phút)    │
│  [Làm mới]                                          │
└─────────────────────────────────────────────────────┘
```

| Thành phần | Quasar Component | Ghi chú |
|---|---|---|
| Location label | `QChip` (read-only) | Tự động theo user, không cho chọn |
| Tìm kiếm | `QInput` (debounce) | Tên, SKU, barcode |
| Bảng tồn kho | `QTable` (sortable) | Tồn kho, khả dụng |
| Thời gian cache | Text muted | Hiển thị lần cache cuối |
| Nút refresh | `QBtn` flat | Force re-fetch từ Odoo |

---

### 5.7 Quản lý Backorder (`/backorders`)

**Mục đích**: Danh sách backorder chờ xử lý, validate khi có hàng.

```
┌─────────────────────────────────────────────────────┐
│  Backorder                                          │
│                                                     │
│  ┌─────────────────────────────────────────────────┐│
│  │ Mã picking │ Đơn gốc │ SP thiếu │ Trạng thái  ││
│  │ PICK-005   │ SO-001  │ SP-A (3) │ Chờ hàng    ││
│  │ PICK-008   │ SO-004  │ SP-B (1) │ Sẵn sàng    ││
│  └─────────────────────────────────────────────────┘│
│                                                     │
│  [Chi tiết]  [Validate]  [Hủy — ADMIN]             │
└─────────────────────────────────────────────────────┘
```

**Chi tiết backorder (expand hoặc dialog)**:
- Danh sách sản phẩm thiếu + số lượng
- Tồn kho hiện tại tại location
- Nếu có serial: gợi ý từ `stock.lot`, user chọn/nhập
- Validate: gọi `button_validate` trên Odoo

| Thành phần | Quasar Component | Ghi chú |
|---|---|---|
| Bảng backorder | `QTable` | Expandable rows |
| Badge trạng thái | `QBadge` | Chờ hàng (vàng), Sẵn sàng (xanh) |
| Chọn serial | `QSelect` (filterable) | Gợi ý từ stock.lot |
| Nhập serial mới | `QInput` trong dialog | Khi serial chưa có trên Odoo |
| Nút validate | `QBtn` positive | Chỉ khi đủ hàng |
| Nút hủy | `QBtn` negative | Chỉ ADMIN |

---

### 5.8 Quản lý User (`/admin/users`) — ADMIN only

**Mục đích**: Tạo, chỉnh sửa, vô hiệu hóa user.

```
┌─────────────────────────────────────────────────────┐
│  Quản lý người dùng                  [+ Tạo mới]   │
│                                                     │
│  ┌─────────────────────────────────────────────────┐│
│  │ Username │ Họ tên    │ Phòng ban │ Role      │ Location │ TT││
│  │ nva      │ Nguyễn A  │ Bán hàng  │ POS User  │ Kho Q7   │ ✓ ││
│  │ ttb      │ Trần B    │ IT        │ ADMIN     │ Tất cả   │ ✓ ││
│  │ lvc      │ Lê C      │ Bán hàng  │ POS User  │ Kho Q1   │ ✗ ││
│  └─────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────┘
```

**Dialog tạo user:**

| Bước | Field | Bắt buộc | Ghi chú |
|---|---|---|---|
| 1 | Chọn nhân viên Odoo | Có | `QSelect` async search `hr.employee` — hiện tên, phòng ban, chức danh |
| 2 | Username | Có | Unique, admin nhập |
| 2 | Password | Có | Min 8, chữ hoa + số + ký tự đặc biệt |
| 2 | Role | Có | Multi-select: ADMIN, POS User |
| 2 | Warehouse / Location | Có | Multi-select từ Odoo |

Ghi chú: **Họ tên, email, SĐT, phòng ban, chức danh** tự động lấy từ `hr.employee` sau khi chọn — không nhập thủ công.

**Dialog sửa user:**

| Field | Ghi chú |
|---|---|
| Role | Có thể đổi |
| Warehouse / Location | Có thể đổi |
| Nhân viên Odoo | Không cho đổi (read-only) |
| Thông tin cá nhân | Read-only, đồng bộ từ Odoo |

**Hành vi đặc biệt:**
- Vô hiệu hóa user → invalidate token ngay lập tức (OQ-M02)
- Không xóa user, chỉ inactive
- 1 `hr.employee` chỉ liên kết 1 user POS (không trùng)
- Admin mặc định không cần liên kết `hr.employee`

---

### 5.9 Quản lý Template in (`/admin/templates`) — ADMIN only

**Mục đích**: Chỉnh sửa template HTML/CSS cho 3 loại phiếu bằng WYSIWYG editor.

```
┌─────────────────────────────────────────────────────┐
│  Quản lý Template                                   │
│                                                     │
│  [Phiếu xác nhận đơn] [Phiếu đặt cọc] [Phiếu TT]  │
│                                                     │
│  ┌──────────────────────┬──────────────────────────┐│
│  │                      │                          ││
│  │   WYSIWYG Editor     │   Preview (realtime)     ││
│  │                      │                          ││
│  │   [B] [I] [Table]    │   ┌──────────────────┐   ││
│  │   [Image] [Variable] │   │  PHIẾU XÁC NHẬN │   ││
│  │                      │   │  Đơn hàng SO-001 │   ││
│  │                      │   │  ...              │   ││
│  │                      │   │                   │   ││
│  │                      │   │  Người mua  NVBH │   ││
│  │                      │   │  ________  ______ │   ││
│  │                      │   └──────────────────┘   ││
│  └──────────────────────┴──────────────────────────┘│
│                                                     │
│  Version: v3 (14/04/2026)  [Lịch sử] [Lưu]        │
└─────────────────────────────────────────────────────┘
```

| Thành phần | Ghi chú |
|---|---|
| Tab chọn loại phiếu | `QTabs` — 3 tab cố định (OQ-I06) |
| WYSIWYG Editor | GrapesJS hoặc tương đương, output HTML/CSS |
| Preview panel | Realtime, bật/tắt được (OQ-P02) |
| Biến template | Dropdown chèn biến: `{{order_id}}`, `{{customer_name}}`, `{{total}}`, `{{amount_in_words}}` |
| Logo | Lấy từ thư mục assets backend (OQ-I01) |
| Thông tin công ty | Hardcode trong template (OQ-I02) |
| Vùng chữ ký | Block cố định: "Người mua" / "Nhân viên bán hàng" (OQ-I07) |
| Số tiền bằng chữ | Biến `{{amount_in_words}}` — backend xử lý (OQ-I03) |
| Version history | Danh sách các phiên bản trước, cho phép rollback (OQ-P03) |
| Báo lỗi template | Hiển thị lỗi cụ thể từ WeasyPrint (OQ-P04) |

---

## 6. State Management — Pinia Stores

### 6.1 Danh sách Stores

| Store | Dữ liệu chính | Persistence |
|---|---|---|
| `useAuthStore` | user, token, roles, permissions, locations | `localStorage` (token) |
| `useCartStore` | customer, items[], draft state | Session (mất khi đóng tab) |
| `useOrderStore` | orders[] (cache 1 tuần), filters | Pinia (in-memory) |
| `useProductStore` | products[], categories[] | Pinia (in-memory, từ backend cache) |
| `useCustomerStore` | customers[] (search results) | Pinia (in-memory) |
| `useInventoryStore` | stock by location | Pinia (in-memory) |
| `useBackorderStore` | backorders[] | Pinia (in-memory) |

### 6.2 Auth Store — Chi tiết

```
useAuthStore
├── state
│   ├── user: { id, username, fullName, roles[], locations[] }
│   ├── accessToken: string
│   ├── refreshToken: string
│   ├── warehouse: { id, name }
│   └── idleTimer: number
├── getters
│   ├── isAuthenticated: boolean
│   ├── isAdmin: boolean
│   └── primaryLocation: object
├── actions
│   ├── login(username, password)
│   ├── logout()
│   ├── refreshAccessToken()
│   ├── resetIdleTimer()
│   └── startIdleWatcher()        → auto logout sau 5 phút
```

### 6.3 Cart Store — Chi tiết

```
useCartStore
├── state
│   ├── customer: { id, name, phone, email, mst } | null   ← tùy chọn, bắt buộc trước thanh toán
│   ├── items: [{ productId, name, sku, qty, price, serialNumber, hasSerial, outOfStock }]
│   ├── isDraft: boolean
│   └── draftId: number | null
├── getters
│   ├── totalAmount: number
│   ├── itemCount: number
│   ├── hasCustomer: boolean       ← kiểm tra trước khi cho thanh toán
│   └── hasOutOfStockItems: boolean
├── actions
│   ├── setCustomer(customer)     ← gọi khi user chọn KH (trước thanh toán)
│   ├── addItem(product)
│   ├── updateQty(index, qty)
│   ├── removeItem(index)
│   ├── saveDraft()               → POST /drafts (không cần KH)
│   ├── loadDraft(draftId)        → GET /drafts/:id
│   ├── confirmOrder()            → POST /orders/confirm
│   ├── proceedToPayment()        → kiểm tra hasCustomer, nếu chưa → mở dialog KH
│   └── clearCart()
```

---

## 7. Authentication & Session — Frontend

### 7.1 Token Management

```
Login thành công
  → Lưu accessToken vào memory (Pinia)
  → Lưu refreshToken vào localStorage

Mỗi API call (qua fetch wrapper)
  → Tự động gắn header: Authorization: Bearer {accessToken}
  → Kiểm tra response status
  → Nếu 401 → gọi /auth/refresh với refreshToken
  → Nếu refresh thành công → retry request gốc
  → Nếu refresh thất bại → logout + redirect /login
```

### 7.2 Idle Timeout (5 phút)

```
Mỗi tương tác (click, keypress, scroll, touch)
  → resetIdleTimer()

Khi timer hết 5 phút
  → Hiển thị QDialog cảnh báo (30 giây countdown)
  → Nếu user tương tác → reset
  → Nếu hết countdown → logout + redirect /login
```

### 7.3 Access Token TTL

- Access token: **15 phút** (OQ-M01)
- Refresh token: cần xác nhận TTL (gợi ý: 7 ngày)
- Frontend tự động refresh khi access token sắp hết hạn (trước 1 phút)

---

## 8. Xử lý lỗi — UX Patterns

### 8.1 Phân loại lỗi

| Loại | Ví dụ | UX Pattern |
|---|---|---|
| Validation | Thiếu field bắt buộc, sai format | Inline error dưới input (`QInput` rules) |
| Business | Hết hàng, không đủ quyền | `QBanner` warning trong page |
| Network | Mất mạng | `QBar` sticky top: "Mất kết nối mạng" |
| Odoo timeout | Odoo không phản hồi | `QDialog` thông báo + nút thử lại (OQ-N01) |
| Server error | 500 | `Notify` negative: "Lỗi hệ thống, vui lòng thử lại" |
| Auth error | Token expired, user disabled | Redirect → /login |

### 8.2 Loading States

| Thao tác | UX |
|---|---|
| Tải danh sách | `QTable` loading skeleton |
| Gọi Odoo (tạo đơn, validate...) | `QBtn` loading + disable |
| Tra cứu MST | `QInput` loading icon |
| Generate PDF | `QSpinner` overlay |

### 8.3 Odoo Timeout (tối đa 20 giây)

```
Khi gọi Odoo API:
  → Hiển thị loading overlay
  → Timeout 20 giây
  → Nếu thất bại → QDialog:
      "Không thể kết nối hệ thống Odoo.
       Vui lòng thử lại sau."
      [Thử lại] [Đóng]
```

---

## 9. Color Palette & Hệ thống màu sắc

### 9.1 Brand Palette

| Tên | Hex | Vai trò | Sử dụng |
|---|---|---|---|
| **Sky Blue** | `#a3d9ff` | Primary | Header, sidebar, nút chính, link, focus state |
| **Plum** | `#7e6b8f` | Secondary | Nút phụ, badge, icon inactive, border, text muted |
| **Mint Green** | `#96e6b3` | Success | Trạng thái thành công, xác nhận, tồn kho đủ |
| **Coral Red** | `#da3e52` | Danger | Lỗi, cảnh báo nghiêm trọng, nút hủy, hết hàng |
| **Lemon Yellow** | `#f2e94e` | Warning | Cảnh báo nhẹ, chờ xử lý, backorder |

### 9.2 Extended Palette (light / dark variants)

| Tên | Light (nền) | Base | Dark (text/border) |
|---|---|---|---|
| Sky Blue | `#d6eeff` | `#a3d9ff` | `#4a90b8` |
| Plum | `#c4b8cf` | `#7e6b8f` | `#4d3d5e` |
| Mint Green | `#d4f5e2` | `#96e6b3` | `#3d8a5c` |
| Coral Red | `#f5c6cc` | `#da3e52` | `#a12535` |
| Lemon Yellow | `#faf6b8` | `#f2e94e` | `#b0a820` |

### 9.3 Neutral Colors

| Tên | Hex | Sử dụng |
|---|---|---|
| White | `#ffffff` | Nền chính |
| Background | `#f8f9fa` | Nền phụ (sidebar, card) |
| Border | `#e0e0e0` | Viền, divider |
| Text Primary | `#212121` | Tiêu đề, nội dung chính |
| Text Secondary | `#757575` | Label, ghi chú, placeholder |
| Text Disabled | `#bdbdbd` | Text vô hiệu hóa |

### 9.4 Quasar CSS Variables

Cấu hình trong `quasar.config.js`:

```js
framework: {
  config: {
    brand: {
      primary:   '#a3d9ff',
      secondary: '#7e6b8f',
      positive:  '#96e6b3',
      negative:  '#da3e52',
      warning:   '#f2e94e',
      info:      '#a3d9ff',
      dark:      '#212121',
    }
  }
}
```

### 9.5 Áp dụng vào UI

| Thành phần UI | Màu | Ghi chú |
|---|---|---|
| Header bar | `#a3d9ff` (Sky Blue) | Nền header, text `#212121` |
| Sidebar menu (desktop) | `#f8f9fa` nền, `#a3d9ff` active item | Item active có left-border Sky Blue |
| Bottom nav (tablet) | `#ffffff` nền, `#a3d9ff` icon active | Icon inactive dùng `#7e6b8f` |
| Nút chính (CTA) | `#a3d9ff` nền, `#4a90b8` hover | Xác nhận đơn, Thanh toán |
| Nút phụ | `#7e6b8f` outline | Lưu nháp, Đổi KH |
| Nút nguy hiểm | `#da3e52` | Hủy đơn, Xóa |
| Badge thành công | `#96e6b3` nền, `#3d8a5c` text | Đã thanh toán, Đã xuất kho |
| Badge cảnh báo | `#f2e94e` nền, `#b0a820` text | Backorder, Chờ xử lý |
| Badge lỗi | `#da3e52` nền, `#ffffff` text | Đã hủy, Hết hàng |
| Card sản phẩm | `#ffffff` nền, `#e0e0e0` border | Hover: shadow + `#a3d9ff` border |
| Numpad | `#f8f9fa` nền, `#a3d9ff` nút active | Qty/Price active state |
| Order line | `#ffffff` | Hàng hết hàng: nền `#f5c6cc` nhạt |
| Tìm kiếm | `#ffffff` input, `#a3d9ff` focus border | Placeholder dùng `#757575` |

### 9.6 Trạng thái đơn hàng — Badge Color

| Trạng thái | Nền | Text | Ý nghĩa |
|---|---|---|---|
| Nháp (Draft) | `#e0e0e0` | `#757575` | Chưa gửi |
| Đã xác nhận (Confirmed) | `#d6eeff` | `#4a90b8` | Đơn đã xác nhận trên Odoo |
| Đã đặt cọc (Deposited) | `#faf6b8` | `#b0a820` | Đã nhận một phần tiền |
| Đã thanh toán đủ (Paid) | `#d4f5e2` | `#3d8a5c` | Hoàn tất thanh toán |
| Đã xuất kho (Delivered) | `#c4b8cf` | `#4d3d5e` | Hàng đã xuất |
| Đã hủy (Cancelled) | `#f5c6cc` | `#a12535` | Đơn bị hủy |
| Backorder (Chờ hàng) | `#faf6b8` | `#b0a820` | Chờ nhập thêm hàng |

---

## 10. Cấu trúc thư mục Frontend

```
frontend/
├── src/
│   ├── assets/              # Logo, icons, static images
│   ├── boot/                # Quasar boot files
│   │   ├── http.js          # fetch wrapper + interceptors
│   │   └── auth.js          # Route guard + idle watcher
│   ├── components/          # Shared components
│   │   ├── CustomerSearch.vue
│   │   ├── ProductSearch.vue
│   │   ├── CartPanel.vue
│   │   ├── PaymentDialog.vue
│   │   ├── SerialPicker.vue
│   │   ├── OrderStatusBadge.vue
│   │   └── PdfPreview.vue
│   ├── layouts/
│   │   ├── MainLayout.vue   # Desktop: sidebar + header
│   │   ├── TabletLayout.vue # Tablet: bottom nav + header
│   │   └── AuthLayout.vue   # Login page (no nav)
│   ├── pages/
│   │   ├── LoginPage.vue
│   │   ├── OrderNewPage.vue
│   │   ├── OrderListPage.vue
│   │   ├── OrderDetailPage.vue
│   │   ├── InventoryPage.vue
│   │   ├── BackorderPage.vue
│   │   ├── admin/
│   │   │   ├── UserListPage.vue
│   │   │   ├── UserFormDialog.vue
│   │   │   ├── TemplateEditorPage.vue
│   │   │   └── CachePage.vue
│   │   └── ErrorNotFound.vue
│   ├── router/
│   │   └── routes.js        # Route definitions + guards
│   ├── stores/              # Pinia stores
│   │   ├── auth.js
│   │   ├── cart.js
│   │   ├── orders.js
│   │   ├── products.js
│   │   ├── customers.js
│   │   ├── inventory.js
│   │   └── backorders.js
│   ├── services/            # API call wrappers
│   │   ├── authApi.js
│   │   ├── orderApi.js
│   │   ├── productApi.js
│   │   ├── customerApi.js
│   │   ├── inventoryApi.js
│   │   ├── paymentApi.js
│   │   ├── printApi.js
│   │   └── adminApi.js
│   └── utils/
│       ├── formatCurrency.js   # VND formatting
│       └── constants.js        # Order status, payment methods
├── quasar.config.js
└── package.json
```

---

## 11. Tham chiếu

| Tài liệu | Mô tả |
|---|---|
| [scope.md](scope.md) | Scope & Architecture |
| [process_flow.md](process_flow.md) | Luồng nghiệp vụ (Mermaid) |
| [open_questions.md](open_questions.md) | Câu hỏi và câu trả lời đã xác nhận |
