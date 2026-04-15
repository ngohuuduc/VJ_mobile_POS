# VJ Mobile POS — Ước tính Khối lượng Công việc

> Trạng thái: Draft  
> Cập nhật: 2026-04-15  
> Phiên bản: 0.1

**Giả định:**
- Đơn vị: giờ làm việc của 1 developer có kinh nghiệm (senior)
- Bao gồm: thiết kế, code, unit test cơ bản, self-review
- Không bao gồm: QA độc lập, deployment production, họp, viết tài liệu kỹ thuật chi tiết
- Phần **Planned** (Misa, COD Carrier) được ước tính riêng ở cuối

---

## 1. Backend (FastAPI)

| # | Hạng mục | Chi tiết | Giờ |
|---|---|---|---|
| 1 | Setup & Infrastructure — DEV | Docker Compose dev, NGINX HTTP, uvicorn reload, `.env.dev`, Alembic, cấu trúc project | 12 |
| 2 | Setup & Infrastructure — PRD | Docker Compose prod, NGINX HTTPS + SSL, gunicorn multi-worker, `.env.prod`, log rotation, restart policy | 10 |
| 3 | Authentication | JWT login/logout/refresh token, kiểm tra `is_active` / `is_locked` mỗi request, idle timeout (backend), brute force 5 lần | 20 |
| 4 | Password Reset | Token reset (1h TTL), gửi email qua AWS SES, validate & cập nhật password hash, xóa refresh tokens | 10 |
| 5 | User Management | CRUD users, role ADMIN/POS User, liên kết `hr_employee_id`, disable/unlock, phân quyền location/warehouse | 16 |
| 6 | Odoo XML-RPC Layer | Connection wrapper, service account, xử lý timeout (20s), retry, error mapping cho tất cả models | 20 |
| 7 | Odoo — Bán hàng | `sale.order` create/confirm/cancel, `sale.order.line`, draft order (PostgreSQL `draft_orders`) | 18 |
| 8 | Odoo — Xuất kho | `stock.picking` validate, `stock.move.line`, serial/lot (`stock.lot`), backorder (`stock.backorder.confirmation`) | 20 |
| 9 | Odoo — Thanh toán | `account.move` create draft, `account.payment` create, ghi nhận multi-method + deposit | 16 |
| 10 | Odoo — Sản phẩm & Danh mục | `product.product` search, `product.category`, `product.pricelist` get_product_price | 12 |
| 11 | Odoo — Khách hàng | `res.partner` search/create/write | 8 |
| 12 | Odoo — Nhân viên | `hr.employee` search_read (lấy thông tin cá nhân khi tạo user) | 4 |
| 13 | Product API | Endpoint tìm kiếm (tên/SKU/barcode/serial), danh mục, hiển thị tồn kho theo location | 12 |
| 14 | Customer API | Tìm kiếm (tên/SĐT/email/MST), tạo mới, cập nhật | 8 |
| 15 | Order API | Tạo/xác nhận/hủy đơn, lưu nháp, danh sách 1 tuần, tìm kiếm | 16 |
| 16 | Payment API | Ghi nhận thanh toán (multi-method), deposit, tìm đơn còn lại | 16 |
| 17 | Inventory API | Tra cứu `stock.quant` theo location, TTLCache 5 phút | 8 |
| 18 | Backorder API | Danh sách, validate, hủy (ADMIN only) | 10 |
| 19 | MST Lookup API | Gọi VietQR API, TTLCache 10 phút, error handling | 6 |
| 20 | Image Upload | Upload ảnh sản phẩm, resize + crop thumbnail (backend), serve qua NGINX `/media/*` | 10 |
| 21 | Print / PDF | WeasyPrint + Jinja2, 3 loại phiếu (xác nhận/cọc/thanh toán), A4 + A5, số tiền bằng chữ TV | 20 |
| 22 | Template Management | CRUD template HTML/CSS, version history, lưu PostgreSQL | 14 |
| 23 | TTLCache Management | Cache cho products, pricelist, customers, categories, stock; flush API (theo loại / flush all) | 10 |
| 24 | Audit Log | Ghi log toàn bộ hành động nghiệp vụ (tạo đơn, thanh toán, hủy, xuất kho...) | 10 |
| 25 | Admin APIs | Activity log, user management endpoints, cache flush endpoints | 10 |
| | | **Tổng Backend** | **316** |

---

## 2. Frontend (Vue 3 + Quasar v2)

| # | Hạng mục | Chi tiết | Giờ |
|---|---|---|---|
| 1 | Setup & Project Structure | Vue 3 + Quasar v2, Pinia stores, Vue Router, Axios interceptors, env config | 12 |
| 2 | Auth | Trang login, lưu token (memory + localStorage), refresh token tự động, idle watcher 5 phút, trang đăng xuất | 14 |
| 3 | Layout & Navigation | Desktop layout + tablet layout (iPad), top bar, sidebar/drawer, responsive breakpoints | 16 |
| 4 | Product Grid | Danh sách SP (infinite scroll), tìm kiếm (3 ký tự), lọc theo danh mục, hiển thị tồn kho + ảnh (toggle) | 22 |
| 5 | Cart & Order Creation | Thêm/xóa SP (swipe trên iPad), chọn serial, nhập số lượng (numpad), xem lại đơn, lưu nháp, xác nhận | 24 |
| 6 | Customer Form | Tìm kiếm KH, tạo mới (tên/SĐT/email), chỉnh sửa, MST auto-fill từ VietQR | 14 |
| 7 | Payment Flow | Form thanh toán multi-method (tiền mặt/chuyển khoản/VNPay/MoMo/thẻ/COD), deposit, tính tiền thừa, numpad | 28 |
| 8 | COD Form | Chọn carrier, nhập địa chỉ giao hàng, mã vận đơn, trạng thái thu hộ | 12 |
| 9 | Order List & Search | Danh sách đơn (5/trang), tìm kiếm (tên KH/SĐT/email/MST/mã đơn), filter trạng thái, auto-refresh 20 phút | 16 |
| 10 | Backorder Management | Danh sách backorder, xem chi tiết, validate (chọn serial), hủy (ADMIN) | 12 |
| 11 | Stock Check | Tìm kiếm SP, hiển thị tồn kho theo location được phân quyền | 8 |
| 12 | PDF Preview | Modal preview PDF (inline viewer), nút download, chọn khổ giấy A4/A5 | 8 |
| 13 | Template Editor | Tích hợp WYSIWYG editor (GrapesJS hoặc TinyMCE), preview realtime, lưu/rollback version | 24 |
| 14 | Admin — User Management | Danh sách user, tạo mới (chọn hr.employee), phân quyền, disable, unlock, reset password | 18 |
| 15 | Admin — Cache Management | Nút flush theo loại (sản phẩm/pricelist/tồn kho/KH/tất cả), hiển thị TTL còn lại | 8 |
| 16 | Admin — Activity Log | Bảng log (ai/hành động/thời gian/đơn hàng), filter, xem chi tiết | 12 |
| 17 | UX Polish | Keyboard shortcuts (F2–F9, Esc, Del), tooltip shortcut hints, dialog xác nhận, toast 2 giây, offline banner toàn màn hình | 18 |
| 18 | Numpad Component | Component numpad dùng chung (số lượng, số tiền), tích hợp trên iPad | 8 |
| | | **Tổng Frontend** | **264** |

---

## 3. Tích hợp API — Giai đoạn hiện tại

| # | Tích hợp | Chi tiết | Giờ |
|---|---|---|---|
| 1 | Odoo 14 CE (XML-RPC) | Đã tính trong Backend mục 5–11 | — |
| 2 | VietQR API (MST) | Đã tính trong Backend mục 18 | — |
| 3 | AWS SES (Email) | Config SMTP, template email (reset password, thông báo hệ thống), gửi qua `smtplib` | 8 |
| | | **Tổng tích hợp hiện tại** | **8** |

---

## 4. Tích hợp API — Planned

| # | Tích hợp | Chi tiết | Backend | Frontend | Tổng |
|---|---|---|---|---|---|
| 1 | Misa eInvoice | OAuth token + cache, tạo hoá đơn nháp, retry (2 lần), audit log lỗi, email thông báo kế toán | 20 | 6 | **26** |
| 2 | COD Carrier | Tạo vận đơn, lưu tracking code, webhook HMAC verify, cập nhật trạng thái, fallback thủ công | 24 | 0 | **24** |
| 3 | VNPay API | Tạo payment URL, ký HMAC-SHA512, IPN callback handler, xác thực hash, cập nhật trạng thái, audit log | 16 | 8 | **24** |
| 4 | MoMo API | Payment request, ký HMAC-SHA256, IPN callback handler, xác thực signature, redirect + QR deeplink | 16 | 8 | **24** |
| 5 | VietQR Payment | Sinh QR chuẩn NAPAS, webhook ngân hàng hoặc fallback thủ công, đối chiếu description | 14 | 6 | **20** |
| | | **Tổng Planned** | **90** | **28** | **118** |

> Frontend COD (12h) đã được tính trong mục 2.8 — COD Form

---

## 5. Tổng hợp

| Hạng mục | Giờ |
|---|---|
| Backend (core) | 316 |
| Frontend | 264 |
| Tích hợp hiện tại (AWS SES) | 8 |
| **Subtotal — Giai đoạn 1** | **588** |
| Misa eInvoice | 26 |
| COD Carrier | 24 |
| VNPay API | 24 |
| MoMo API | 24 |
| VietQR Payment | 20 |
| **Subtotal — Tất cả Planned** | **118** |
| **Subtotal — Giai đoạn 1 + Planned** | **706** |
| Buffer dự phòng ~15% (bug fix, điều chỉnh scope, review) | ~106 |
| **Tổng ước tính toàn bộ** | **~812** |

---

## 6. Phân bổ theo thứ tự ưu tiên triển khai

| Sprint | Hạng mục | Backend | Frontend | Tổng |
|---|---|---|---|---|
| 1 | Setup, Auth, User Management | 46 | 42 | **88** |
| 2 | Odoo Layer + Product/Customer API + UI tương ứng | 64 | 36 | **100** |
| 3 | Bán hàng, Xuất kho, Backorder | 54 | 36 | **90** |
| 4 | Thanh toán, Deposit, Order List | 38 | 56 | **94** |
| 5 | Print/PDF, Template Editor, Admin Panel | 54 | 62 | **116** |
| 6 | Cache, Audit Log, UX Polish, MST | 24 | 26 | **50** |
| 7 | AWS SES + Buffer Sprint 1–6 | 30 | 18 | **48** |
| 8 *(Planned)* | Misa eInvoice | 20 | 6 | **26** |
| 9 *(Planned)* | COD Carrier Integration | 24 | 12 | **36** |
| 10 *(Planned)* | VNPay API | 16 | 8 | **24** |
| 11 *(Planned)* | MoMo API | 16 | 8 | **24** |
| 12 *(Planned)* | VietQR Payment | 14 | 6 | **20** |

---

## Tham chiếu

| Tài liệu | Mô tả |
|---|---|
| [scope.md](scope.md) | Scope & Architecture |
| [process_flow.md](process_flow.md) | Process Flows |
| [open_questions.md](open_questions.md) | Câu hỏi còn mở |
