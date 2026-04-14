# VJ Mobile POS — API Contract

> Trạng thái: Draft  
> Cập nhật: 2026-04-14  
> Phiên bản: 0.1

Base URL: `https://{domain}/api/v1`

---

## Quy ước chung

### Authentication

- Tất cả endpoint (trừ Auth) yêu cầu header: `Authorization: Bearer {access_token}`
- Access token TTL: 15 phút
- Khi 401 → gọi `/auth/refresh` để lấy token mới

### Response format

```json
{
  "success": true,
  "data": { ... },
  "message": "optional message"
}
```

### Error format

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Mô tả lỗi"
  }
}
```

### HTTP Status Codes

| Code | Ý nghĩa |
|---|---|
| 200 | Thành công |
| 201 | Tạo mới thành công |
| 400 | Request không hợp lệ (validation) |
| 401 | Chưa xác thực / token hết hạn |
| 403 | Không có quyền (role) |
| 404 | Không tìm thấy resource |
| 409 | Conflict (vd: đơn đã hủy rồi) |
| 422 | Dữ liệu không xử lý được |
| 500 | Lỗi server |
| 502 | Odoo không phản hồi |

### Pagination

Các endpoint danh sách hỗ trợ:

```
?page=1&page_size=20
```

Response bao gồm:

```json
{
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total": 150,
    "total_pages": 8
  }
}
```

### Định dạng dữ liệu

- Tiền tệ: số nguyên (đơn vị: VND, không thập phân)
- Ngày giờ: ISO 8601 (`2026-04-14T10:30:00+07:00`)
- ID Odoo: số nguyên

---

## 1. Auth

### 1.1 Đăng nhập

```
POST /auth/login
```

**Public** — không cần token.

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `username` | string | Có | |
| `password` | string | Có | |

**Response 200:**

```json
{
  "access_token": "eyJ...",
  "refresh_token": "abc...",
  "expires_in": 900,
  "user": {
    "id": 1,
    "username": "nva",
    "hr_employee_id": 42,
    "full_name": "Nguyễn Văn A",
    "email": "nva@company.vn",
    "phone": "0901234567",
    "department": "Bán hàng",
    "job_title": "Nhân viên kinh doanh",
    "roles": ["POS_USER"],
    "warehouse": { "id": 5, "name": "Kho Quận 7" },
    "locations": [
      { "id": 10, "name": "Kệ A" },
      { "id": 11, "name": "Kệ B" }
    ]
  }
}
```

**Error codes:**
- `INVALID_CREDENTIALS` — sai username/password
- `ACCOUNT_LOCKED` — tài khoản bị khóa (sau 5 lần sai)
- `ACCOUNT_DISABLED` — tài khoản bị vô hiệu hóa

---

### 1.2 Refresh Token

```
POST /auth/refresh
```

**Public** — dùng refresh token thay vì access token.

| Field | Type | Required |
|---|---|---|
| `refresh_token` | string | Có |

**Response 200:**

```json
{
  "access_token": "eyJ...",
  "expires_in": 900
}
```

---

### 1.3 Đăng xuất

```
POST /auth/logout
```

**Auth required.**

| Field | Type | Required |
|---|---|---|
| `refresh_token` | string | Có |

**Response 200:** `{ "success": true }`

Backend: xóa refresh token khỏi DB.

---

### 1.4 Yêu cầu reset password

```
POST /auth/forgot-password
```

**Public.**

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `email` | string | Có | Email liên kết với tài khoản |

**Response 200:** `{ "success": true, "message": "Email đã được gửi" }`

Backend: gửi email chứa token reset qua AWS SES (SMTP).

---

### 1.5 Reset password

```
POST /auth/reset-password
```

**Public.**

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `token` | string | Có | Token từ email |
| `new_password` | string | Có | Min 8 chars, chữ hoa + số + ký tự đặc biệt |

---

## 2. Users (ADMIN)

### 2.1 Danh sách users

```
GET /admin/users
```

**Auth: ADMIN.**

Query params: `?page=1&page_size=20&status=active`

**Response 200:**

```json
{
  "data": [
    {
      "id": 1,
      "username": "nva",
      "hr_employee_id": 42,
      "full_name": "Nguyễn Văn A",
      "email": "nva@company.vn",
      "phone": "0901234567",
      "department": "Bán hàng",
      "job_title": "Nhân viên kinh doanh",
      "roles": ["POS_USER"],
      "warehouse": { "id": 5, "name": "Kho Quận 7" },
      "locations": [{ "id": 10, "name": "Kệ A" }],
      "is_active": true,
      "is_locked": false,
      "created_at": "2026-04-01T08:00:00+07:00"
    }
  ],
  "pagination": { ... }
}
```

Ghi chú: `full_name`, `email`, `phone`, `department`, `job_title` được lấy từ Odoo `hr.employee` theo `hr_employee_id`.

---

### 2.2 Tìm nhân viên trên Odoo (để chọn khi tạo user)

```
GET /admin/employees/search?q={keyword}
```

**Auth: ADMIN.**

Backend: gọi Odoo `hr.employee` search_read. Trả về danh sách nhân viên chưa có tài khoản trên hệ thống.

**Response 200:**

```json
{
  "data": [
    {
      "hr_employee_id": 42,
      "name": "Nguyễn Văn A",
      "email": "nva@company.vn",
      "phone": "0901234567",
      "department": "Bán hàng",
      "job_title": "Nhân viên kinh doanh",
      "already_linked": false
    }
  ]
}
```

`already_linked = true` nghĩa là employee đã có tài khoản POS → không cho tạo trùng.

---

### 2.3 Tạo user

```
POST /admin/users
```

**Auth: ADMIN.**

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `username` | string | Có | Unique |
| `password` | string | Có | Min 8, chữ hoa + số + ký tự đặc biệt |
| `hr_employee_id` | int | Có | Odoo `hr.employee` ID. Thông tin cá nhân (tên, email, SĐT...) kế thừa từ Odoo |
| `roles` | string[] | Có | `["ADMIN"]`, `["POS_USER"]`, hoặc cả hai |
| `warehouse_id` | int | Có | Odoo warehouse ID |
| `location_ids` | int[] | Có | Odoo location IDs |

Ghi chú: `full_name`, `email`, `phone` **không nhập thủ công** — backend tự lấy từ `hr.employee` trên Odoo theo `hr_employee_id`. Admin mặc định không cần `hr_employee_id`.

**Response 201:**

```json
{
  "data": {
    "id": 5,
    "username": "nva",
    "hr_employee_id": 42,
    "full_name": "Nguyễn Văn A",
    "email": "nva@company.vn",
    "roles": ["POS_USER"]
  }
}
```

**Error codes:**
- `EMPLOYEE_ALREADY_LINKED` — employee đã có tài khoản POS
- `EMPLOYEE_NOT_FOUND` — hr_employee_id không tồn tại trên Odoo

---

### 2.4 Cập nhật user

```
PUT /admin/users/{user_id}
```

**Auth: ADMIN.**

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `roles` | string[] | Không | |
| `warehouse_id` | int | Không | |
| `location_ids` | int[] | Không | |

Ghi chú: không cho đổi `hr_employee_id` sau khi tạo. Thông tin cá nhân cập nhật trên Odoo, hệ thống tự đồng bộ.

---

### 2.5 Vô hiệu hóa / Kích hoạt user

```
PATCH /admin/users/{user_id}/status
```

**Auth: ADMIN.**

| Field | Type | Required |
|---|---|---|
| `is_active` | boolean | Có |

Backend: nếu `is_active = false` → xóa tất cả refresh token của user → invalidate ngay.

---

### 2.6 Mở khóa user (bị khóa sau 5 lần sai)

```
POST /admin/users/{user_id}/unlock
```

**Auth: ADMIN.**

**Response 200:** `{ "success": true }`

---

### 2.7 Reset password cho user (ADMIN)

```
POST /admin/users/{user_id}/reset-password
```

**Auth: ADMIN.**

Backend: gửi email reset password token cho user qua AWS SES.

---

### 2.8 Activity Log

```
GET /admin/activity-log
```

**Auth: ADMIN.**

Query params: `?page=1&page_size=20&user_id=1&action=order_create&date_from=2026-04-01&date_to=2026-04-14`

**Response 200:**

```json
{
  "data": [
    {
      "id": 100,
      "user_id": 1,
      "user_name": "Nguyễn Văn A",
      "action": "order_create",
      "resource_type": "sale_order",
      "resource_id": "SO-001",
      "details": "Tạo đơn hàng SO-001, tổng 2.000.000 ₫",
      "ip_address": "192.168.1.10",
      "created_at": "2026-04-14T10:30:00+07:00"
    }
  ],
  "pagination": { ... }
}
```

**Action types:**
`login`, `logout`, `order_create`, `order_confirm`, `order_cancel`, `payment_record`, `deposit_record`, `picking_validate`, `backorder_validate`, `customer_create`, `customer_update`, `user_create`, `user_update`, `user_disable`, `user_unlock`, `cache_flush`, `template_update`

---

## 3. Customers

### 3.1 Tìm kiếm khách hàng

```
GET /customers/search?q={keyword}
```

**Auth required.**

Query params: `?q=nguyen` — tìm theo tên, SĐT, email, MST.

**Response 200:**

```json
{
  "data": [
    {
      "id": 42,
      "name": "Nguyễn Văn A",
      "phone": "0901234567",
      "email": "nva@example.com",
      "mst": "0312345678",
      "address": "123 Nguyễn Huệ, Q.1, TP.HCM"
    }
  ]
}
```

Min 3 ký tự để tìm (OQ-S01).

---

### 3.2 Tạo khách hàng

```
POST /customers
```

**Auth required.**

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `name` | string | Có | |
| `phone` | string | Có | |
| `email` | string | Có | |
| `mst` | string | Không | Nếu có → auto-fill từ VietQR |
| `address` | string | Không | |

Backend: sync ngay lên Odoo `res.partner` (OQ-X01).

**Response 201:**

```json
{
  "data": {
    "id": 43,
    "odoo_partner_id": 1205,
    "name": "Công ty ABC",
    "phone": "0909999888",
    "email": "abc@corp.vn",
    "mst": "0312345679",
    "address": "456 Lê Lợi, Q.1, TP.HCM"
  }
}
```

---

### 3.3 Cập nhật khách hàng

```
PUT /customers/{customer_id}
```

**Auth required.** POS User được phép cập nhật (OQ-J04).

Body tương tự 3.2.

Backend: gọi Odoo `res.partner.write()`.

---

### 3.4 Tra cứu MST

```
GET /customers/mst/{tax_code}
```

**Auth required.**

**Response 200:**

```json
{
  "data": {
    "mst": "0312345678",
    "name": "Công ty TNHH XYZ",
    "address": "789 Điện Biên Phủ, Q.3, TP.HCM",
    "status": "active"
  },
  "cached": true
}
```

Backend: TTLCache 10 phút → VietQR API fallback.

---

## 4. Products

### 4.1 Danh sách sản phẩm (batch loading)

```
GET /products?category_id={id}&cursor={last_id}&limit=30
```

**Auth required.**

Query params:
- `category_id` — lọc theo danh mục (optional)
- `cursor` — ID sản phẩm cuối trang trước (infinite scroll)
- `limit` — số SP mỗi batch (default: 30)
- `q` — tìm kiếm (tên, SKU, barcode, serial). Min 3 ký tự

**Response 200:**

```json
{
  "data": [
    {
      "id": 101,
      "name": "Sản phẩm A",
      "sku": "SP-A001",
      "barcode": "8934567890123",
      "price": 200000,
      "image_url": "/static/products/101_thumb.jpg",
      "has_serial": true,
      "stock_qty": 48,
      "category": { "id": 5, "name": "Điện tử" }
    }
  ],
  "has_more": true,
  "next_cursor": 130
}
```

Ghi chú: `stock_qty` theo location của user hiện tại (OQ-S05).

---

### 4.2 Danh mục sản phẩm

```
GET /products/categories
```

**Auth required.**

**Response 200:**

```json
{
  "data": [
    { "id": 1, "name": "Tất cả", "parent_id": null },
    { "id": 5, "name": "Điện tử", "parent_id": 1 },
    { "id": 6, "name": "Phụ kiện", "parent_id": 1 }
  ]
}
```

Backend: cache 8 tiếng (OQ-X03).

---

### 4.3 Gợi ý Serial Number

```
GET /products/{product_id}/serials?location_id={id}
```

**Auth required.**

**Response 200:**

```json
{
  "data": [
    { "id": 501, "name": "SN-20260414-001", "available": true },
    { "id": 502, "name": "SN-20260414-002", "available": true },
    { "id": 503, "name": "SN-20260414-003", "available": false }
  ]
}
```

Backend: Odoo `stock.lot` search_read theo product + location.

---

### 4.4 Upload ảnh sản phẩm (ADMIN)

```
POST /admin/products/{product_id}/image
```

**Auth: ADMIN.** Content-Type: `multipart/form-data`.

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `image` | file | Có | Max 5MB. JPG, PNG |

Backend: resize + crop → thumbnail, lưu local storage.

**Response 200:**

```json
{
  "data": {
    "image_url": "/static/products/101_thumb.jpg"
  }
}
```

---

### 4.5 Xóa ảnh sản phẩm (ADMIN)

```
DELETE /admin/products/{product_id}/image
```

**Auth: ADMIN.**

---

## 5. Orders

### 5.1 Tạo và xác nhận đơn hàng

```
POST /orders
```

**Auth required.**

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `customer_id` | int | Không | Odoo partner ID. Không bắt buộc khi tạo, bắt buộc khi thanh toán |
| `lines` | array | Có | Tối thiểu 1 dòng |
| `lines[].product_id` | int | Có | |
| `lines[].qty` | int | Có | > 0 |
| `lines[].price` | int | Có | VND |
| `lines[].serial_id` | int | Không | Nếu SP có serial → ID từ stock.lot |
| `lines[].serial_name` | string | Không | Nếu serial mới (chưa có trên Odoo) |
| `confirm` | boolean | Có | `true` → xác nhận ngay. `false` → chỉ tạo draft trên Odoo |

Backend:
1. Tạo `sale.order` + `sale.order.line` trên Odoo
2. Nếu `confirm=true` → gọi `action_confirm` → Odoo tự tạo `stock.picking`
3. Tạo `account.move` draft
4. Ghi audit log

**Response 201:**

```json
{
  "data": {
    "id": 1,
    "odoo_order_id": 5001,
    "name": "SO-5001",
    "state": "confirmed",
    "customer": { "id": 42, "name": "Nguyễn Văn A" },
    "lines": [ ... ],
    "total": 800000,
    "picking_id": 3001,
    "created_at": "2026-04-14T10:30:00+07:00",
    "created_by": "nva"
  }
}
```

---

### 5.2 Danh sách đơn hàng

```
GET /orders?page=1&page_size=5
```

**Auth required.**

Query params:
- `page`, `page_size` (default: 5, OQ-S03)
- `state` — filter: `draft`, `confirmed`, `deposited`, `paid`, `cancelled`
- `q` — tìm: tên KH, SĐT, MST, email

Backend:
- POS User: chỉ đơn của mình (OQ-L03)
- ADMIN: tất cả đơn
- Phạm vi: 1 tuần (OQ-L02)

**Response 200:**

```json
{
  "data": [
    {
      "id": 1,
      "name": "SO-5001",
      "state": "confirmed",
      "customer": { "id": 42, "name": "Nguyễn Văn A", "phone": "0901234567" },
      "total": 800000,
      "amount_paid": 200000,
      "amount_remaining": 600000,
      "created_at": "2026-04-14T10:30:00+07:00",
      "created_by": "nva"
    }
  ],
  "pagination": { ... }
}
```

---

### 5.3 Chi tiết đơn hàng

```
GET /orders/{order_id}
```

**Auth required.**

**Response 200:**

```json
{
  "data": {
    "id": 1,
    "odoo_order_id": 5001,
    "name": "SO-5001",
    "state": "confirmed",
    "customer": {
      "id": 42,
      "name": "Nguyễn Văn A",
      "phone": "0901234567",
      "email": "nva@example.com",
      "mst": "0312345678"
    },
    "lines": [
      {
        "id": 1,
        "product_id": 101,
        "product_name": "Sản phẩm A",
        "sku": "SP-A001",
        "qty": 2,
        "price": 200000,
        "subtotal": 400000,
        "serial": { "id": 501, "name": "SN-20260414-001" }
      }
    ],
    "total": 800000,
    "amount_paid": 200000,
    "amount_remaining": 600000,
    "payments": [
      {
        "id": 1,
        "method": "cash",
        "amount": 200000,
        "type": "deposit",
        "note": "",
        "created_at": "2026-04-14T10:35:00+07:00",
        "created_by": "nva"
      }
    ],
    "picking_id": 3001,
    "picking_state": "done",
    "backorder_ids": [3005],
    "created_at": "2026-04-14T10:30:00+07:00",
    "created_by": "nva"
  }
}
```

---

### 5.4 Hủy đơn hàng

```
POST /orders/{order_id}/cancel
```

**Auth required.**

Business rules:
- Draft → POS User hoặc ADMIN được phép
- Confirmed (chưa xuất kho) → chỉ ADMIN. Backend hủy `stock.picking` trước, rồi `sale.order.action_cancel`
- Đã xuất kho → trả 403, cần xử lý trên Odoo

**Response 200:**

```json
{
  "data": {
    "id": 1,
    "name": "SO-5001",
    "state": "cancelled"
  }
}
```

**Error codes:**
- `ORDER_ALREADY_CANCELLED`
- `ORDER_DELIVERED_CANNOT_CANCEL` — đã xuất kho
- `PERMISSION_DENIED` — POS User hủy đơn confirmed

---

## 6. Draft Orders

### 6.1 Lưu đơn nháp

```
POST /drafts
```

**Auth required.**

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `customer_id` | int | Không | Có thể chưa có KH |
| `lines` | array | Có | |
| `lines[].product_id` | int | Có | |
| `lines[].qty` | int | Có | |
| `lines[].price` | int | Có | |
| `lines[].serial_id` | int | Không | |

Backend: INSERT vào `draft_orders`, `expires_at = now + 8h`.

**Response 201:**

```json
{
  "data": {
    "draft_id": 10,
    "expires_at": "2026-04-14T18:30:00+07:00"
  }
}
```

---

### 6.2 Load đơn nháp

```
GET /drafts/{draft_id}
```

**Auth required.** Chỉ user tạo mới được load.

---

### 6.3 Danh sách đơn nháp của user

```
GET /drafts
```

**Auth required.**

---

### 6.4 Xóa đơn nháp

```
DELETE /drafts/{draft_id}
```

**Auth required.**

---

## 7. Payments

### 7.1 Ghi nhận thanh toán

```
POST /orders/{order_id}/payments
```

**Auth required.**

**Điều kiện**: Đơn hàng phải có `customer_id`. Nếu chưa → 400 `CUSTOMER_REQUIRED`.

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `payments` | array | Có | 1 hoặc nhiều phương thức |
| `payments[].method` | string | Có | `cash`, `transfer`, `vnpay`, `momo`, `credit_card` |
| `payments[].amount` | int | Có | VND |
| `payments[].note` | string | Không | Ghi chú (vd: số TK ngân hàng) |
| `type` | string | Có | `deposit` hoặc `full_payment` |

Backend:
1. Validate: tổng payments >= amount_remaining (nếu `full_payment`)
2. Tạo `account.payment` trên Odoo cho mỗi payment line
3. Cập nhật state đơn hàng (`deposited` hoặc `paid`)
4. Ghi audit log

**Response 200:**

```json
{
  "data": {
    "order_id": 1,
    "order_state": "paid",
    "amount_paid": 800000,
    "amount_remaining": 0,
    "change": 50000,
    "payments_recorded": [
      { "id": 5, "method": "cash", "amount": 500000 },
      { "id": 6, "method": "transfer", "amount": 350000 }
    ]
  }
}
```

**Error codes:**
- `CUSTOMER_REQUIRED` — đơn chưa có KH
- `INSUFFICIENT_AMOUNT` — số tiền chưa đủ (cho full_payment)
- `ORDER_ALREADY_PAID`
- `ORDER_CANCELLED`

---

### 7.2 Gắn khách hàng vào đơn (trước thanh toán)

```
PATCH /orders/{order_id}/customer
```

**Auth required.**

| Field | Type | Required |
|---|---|---|
| `customer_id` | int | Có |

Backend: cập nhật `partner_id` trên `sale.order` Odoo.

---

## 8. Inventory

### 8.1 Kiểm tra tồn kho

```
GET /inventory/stock?product_id={id}&location_id={id}
```

**Auth required.**

Query params:
- `product_id` — optional, filter theo SP
- `location_id` — optional (mặc định theo location của user)
- `q` — tìm theo tên, SKU, barcode

Backend: TTLCache 5 phút → Odoo `stock.quant` search_read.

**Response 200:**

```json
{
  "data": [
    {
      "product_id": 101,
      "product_name": "Sản phẩm A",
      "sku": "SP-A001",
      "location": { "id": 10, "name": "Kệ A" },
      "qty_on_hand": 50,
      "qty_available": 48,
      "serial_count": 48
    }
  ],
  "cached_at": "2026-04-14T10:25:00+07:00",
  "cache_ttl": 300
}
```

---

### 8.2 Làm mới cache tồn kho

```
POST /inventory/refresh
```

**Auth required.**

Backend: clear stock_cache → lần gọi tiếp sẽ fetch từ Odoo.

---

## 9. Backorders

### 9.1 Danh sách backorder

```
GET /backorders
```

**Auth required.**

**Response 200:**

```json
{
  "data": [
    {
      "id": 3005,
      "picking_name": "PICK-005/BACK",
      "origin_order": { "id": 1, "name": "SO-5001" },
      "state": "waiting",
      "lines": [
        {
          "product_id": 101,
          "product_name": "Sản phẩm A",
          "qty_remaining": 3,
          "has_serial": true,
          "stock_available": 5
        }
      ],
      "created_at": "2026-04-14T11:00:00+07:00"
    }
  ]
}
```

`state`: `waiting` (chờ hàng), `ready` (đủ hàng, sẵn sàng validate)

---

### 9.2 Validate backorder

```
POST /backorders/{backorder_id}/validate
```

**Auth required.**

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `lines` | array | Có | |
| `lines[].product_id` | int | Có | |
| `lines[].serial_id` | int | Không | Nếu SP có serial |
| `lines[].serial_name` | string | Không | Serial mới |

Backend: Odoo `button_validate` → invalidate stock_cache.

---

### 9.3 Hủy backorder (ADMIN)

```
POST /backorders/{backorder_id}/cancel
```

**Auth: ADMIN.**

---

## 10. Print

### 10.1 Generate PDF

```
GET /print/{receipt_type}?order_id={id}&paper_size={size}
```

**Auth required.**

Path params:
- `receipt_type`: `order_confirm`, `deposit`, `payment`

Query params:
- `order_id` — ID đơn hàng
- `paper_size` — `A4` hoặc `A5` (user chọn khi in, OQ-Z01)

**Response 200:** `Content-Type: application/pdf`

Backend: lấy template từ DB → Jinja2 render HTML + data → WeasyPrint → PDF binary.

Dữ liệu trên phiếu bao gồm:
- Thông tin công ty (hardcode trong template, OQ-I02)
- Logo (từ assets, OQ-I01)
- Thông tin đơn hàng + dòng SP
- Lịch sử thanh toán
- Số tiền bằng chữ (tiếng Việt, OQ-I03)
- Vùng chữ ký (OQ-I07)

---

## 11. Templates (ADMIN)

### 11.1 Danh sách templates

```
GET /admin/templates
```

**Auth: ADMIN.**

**Response 200:**

```json
{
  "data": [
    {
      "id": 1,
      "type": "order_confirm",
      "name": "Phiếu xác nhận đơn",
      "version": 3,
      "updated_at": "2026-04-14T09:00:00+07:00"
    },
    {
      "id": 2,
      "type": "deposit",
      "name": "Phiếu đặt cọc",
      "version": 1,
      "updated_at": "2026-04-10T08:00:00+07:00"
    },
    {
      "id": 3,
      "type": "payment",
      "name": "Phiếu thanh toán",
      "version": 2,
      "updated_at": "2026-04-12T14:00:00+07:00"
    }
  ]
}
```

---

### 11.2 Chi tiết template

```
GET /admin/templates/{template_id}
```

**Auth: ADMIN.**

**Response 200:**

```json
{
  "data": {
    "id": 1,
    "type": "order_confirm",
    "name": "Phiếu xác nhận đơn",
    "html_content": "<html>...</html>",
    "css_content": "body { ... }",
    "version": 3,
    "updated_at": "2026-04-14T09:00:00+07:00"
  }
}
```

---

### 11.3 Cập nhật template

```
PUT /admin/templates/{template_id}
```

**Auth: ADMIN.**

| Field | Type | Required |
|---|---|---|
| `html_content` | string | Có |
| `css_content` | string | Có |

Backend: lưu version mới (giữ bản cũ cho rollback, OQ-P03). Auto-increment version.

---

### 11.4 Preview template

```
POST /admin/templates/{template_id}/preview
```

**Auth: ADMIN.**

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `html_content` | string | Có | HTML chưa lưu (preview trước khi save) |
| `css_content` | string | Có | |
| `paper_size` | string | Có | `A4` hoặc `A5` |

**Response 200:** `Content-Type: application/pdf` — PDF preview với sample data.

**Error:** Nếu HTML/CSS lỗi → 422 + thông báo lỗi cụ thể từ WeasyPrint (OQ-P04).

---

### 11.5 Lịch sử version

```
GET /admin/templates/{template_id}/versions
```

**Auth: ADMIN.**

**Response 200:**

```json
{
  "data": [
    { "version": 3, "updated_at": "2026-04-14T09:00:00+07:00", "updated_by": "admin" },
    { "version": 2, "updated_at": "2026-04-12T14:00:00+07:00", "updated_by": "admin" },
    { "version": 1, "updated_at": "2026-04-10T08:00:00+07:00", "updated_by": "admin" }
  ]
}
```

---

### 11.6 Rollback version

```
POST /admin/templates/{template_id}/rollback
```

**Auth: ADMIN.**

| Field | Type | Required |
|---|---|---|
| `version` | int | Có |

---

## 12. Cache Management (ADMIN)

### 12.1 Flush cache

```
POST /admin/cache/flush
```

**Auth: ADMIN.**

| Field | Type | Required | Ghi chú |
|---|---|---|---|
| `scope` | string | Có | `products`, `pricelist`, `inventory`, `customers`, `categories`, `all` |

**Response 200:**

```json
{
  "data": {
    "scope": "products",
    "cleared_keys": 150
  }
}
```

---

## Tổng hợp Endpoints

| # | Method | Path | Auth | Role |
|---|---|---|---|---|
| 1.1 | POST | `/auth/login` | Public | — |
| 1.2 | POST | `/auth/refresh` | Public | — |
| 1.3 | POST | `/auth/logout` | Auth | All |
| 1.4 | POST | `/auth/forgot-password` | Public | — |
| 1.5 | POST | `/auth/reset-password` | Public | — |
| 2.1 | GET | `/admin/users` | Auth | ADMIN |
| 2.2 | GET | `/admin/employees/search` | Auth | ADMIN |
| 2.3 | POST | `/admin/users` | Auth | ADMIN |
| 2.4 | PUT | `/admin/users/{id}` | Auth | ADMIN |
| 2.5 | PATCH | `/admin/users/{id}/status` | Auth | ADMIN |
| 2.6 | POST | `/admin/users/{id}/unlock` | Auth | ADMIN |
| 2.7 | POST | `/admin/users/{id}/reset-password` | Auth | ADMIN |
| 2.8 | GET | `/admin/activity-log` | Auth | ADMIN |
| 3.1 | GET | `/customers/search` | Auth | All |
| 3.2 | POST | `/customers` | Auth | All |
| 3.3 | PUT | `/customers/{id}` | Auth | All |
| 3.4 | GET | `/customers/mst/{tax_code}` | Auth | All |
| 4.1 | GET | `/products` | Auth | All |
| 4.2 | GET | `/products/categories` | Auth | All |
| 4.3 | GET | `/products/{id}/serials` | Auth | All |
| 4.4 | POST | `/admin/products/{id}/image` | Auth | ADMIN |
| 4.5 | DELETE | `/admin/products/{id}/image` | Auth | ADMIN |
| 5.1 | POST | `/orders` | Auth | All |
| 5.2 | GET | `/orders` | Auth | All |
| 5.3 | GET | `/orders/{id}` | Auth | All |
| 5.4 | POST | `/orders/{id}/cancel` | Auth | All* |
| 6.1 | POST | `/drafts` | Auth | All |
| 6.2 | GET | `/drafts/{id}` | Auth | All |
| 6.3 | GET | `/drafts` | Auth | All |
| 6.4 | DELETE | `/drafts/{id}` | Auth | All |
| 7.1 | POST | `/orders/{id}/payments` | Auth | All |
| 7.2 | PATCH | `/orders/{id}/customer` | Auth | All |
| 8.1 | GET | `/inventory/stock` | Auth | All |
| 8.2 | POST | `/inventory/refresh` | Auth | All |
| 9.1 | GET | `/backorders` | Auth | All |
| 9.2 | POST | `/backorders/{id}/validate` | Auth | All |
| 9.3 | POST | `/backorders/{id}/cancel` | Auth | ADMIN |
| 10.1 | GET | `/print/{receipt_type}` | Auth | All |
| 11.1 | GET | `/admin/templates` | Auth | ADMIN |
| 11.2 | GET | `/admin/templates/{id}` | Auth | ADMIN |
| 11.3 | PUT | `/admin/templates/{id}` | Auth | ADMIN |
| 11.4 | POST | `/admin/templates/{id}/preview` | Auth | ADMIN |
| 11.5 | GET | `/admin/templates/{id}/versions` | Auth | ADMIN |
| 11.6 | POST | `/admin/templates/{id}/rollback` | Auth | ADMIN |
| 12.1 | POST | `/admin/cache/flush` | Auth | ADMIN |

**Tổng: 40 endpoints** (5 public + 21 auth all + 14 admin only)

---

## Tham chiếu

| Tài liệu | Mô tả |
|---|---|
| [scope.md](scope.md) | Scope & Architecture |
| [process_flow.md](process_flow.md) | Luồng nghiệp vụ |
| [open_questions.md](open_questions.md) | Câu hỏi đã xác nhận |
| [frontend_design.md](frontend_design.md) | Thiết kế giao diện |
