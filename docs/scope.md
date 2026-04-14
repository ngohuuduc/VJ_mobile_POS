# VJ Mobile POS — Scope & Architecture Document

> Trạng thái: Draft  
> Cập nhật: 2026-04-14  
> Phiên bản: 0.3

---

## 1. Tổng quan dự án

VJ Mobile POS là một ứng dụng web POS nhẹ, chạy trên trình duyệt, tích hợp với hệ thống Odoo 14 CE thông qua XML-RPC.

**Thiết bị & trình duyệt mục tiêu:**
- Máy tính — Chrome (desktop layout)
- iPad — Safari (tablet layout, touch-friendly) Hệ thống **không thay thế module POS của Odoo** mà đóng vai trò là một UI nghiệp vụ riêng, gọi vào Odoo để thực hiện các thao tác bán hàng và quản lý kho.

---

## 2. Phạm vi nghiệp vụ (In Scope)

| # | Nghiệp vụ | Mô tả | Trạng thái |
|---|---|---|---|
| 1 | Bán hàng | Tạo và xác nhận `sale.order`, lưu đơn nháp tối đa 8 giờ (PostgreSQL) | Đã xác nhận |
| 2 | Nhận thanh toán | Ghi nhận thủ công: tiền mặt, chuyển khoản, VNPay, MoMo, thẻ | Đã xác nhận |
| 3 | Đặt cọc | Nhân viên nhập số tiền cọc tự do. sale.order giữ trạng thái confirmed. Thu phần còn lại bằng cách tìm đơn theo thông tin KH | Đã xác nhận |
| 4 | Xuất tồn kho | Auto validate `stock.picking` + gán serial, tạo backorder nếu thiếu hàng | Đã xác nhận |
| 5 | Quản lý backorder | Hiển thị, theo dõi và xử lý backorder trong Mobile POS | Đã xác nhận |
| 6 | Kiểm tra tồn kho | Tra cứu `stock.quant` theo địa điểm được phân quyền | Đã xác nhận |
| 7 | Tra cứu MST | Tra cứu mã số thuế khách hàng qua VietQR API | Đã xác nhận |
| 8 | Quản lý user | Tạo, vô hiệu hóa user trong app (ADMIN) | Đã xác nhận |
| 9 | In phiếu bán hàng | Generate PDF từ template HTML/CSS. 3 loại phiếu: xác nhận đơn, đặt cọc, thanh toán | Đã xác nhận |
| 10 | Quản lý template in | ADMIN chỉnh sửa template HTML/CSS trong app, lưu PostgreSQL | Đã xác nhận |

### Ngoài phạm vi (Out of Scope)

- Module POS (`pos.order`) của Odoo
- Offline mode / PWA sync
- Tích hợp phần cứng (máy in, barcode scanner, màn hình phụ)
- Quản lý sản phẩm / danh mục (thực hiện trực tiếp trên Odoo)
- Hoàn trả hàng (return) — giai đoạn đầu
- Báo cáo / BI
- Tích hợp trực tiếp payment gateway API (VNPay, MoMo) — giai đoạn này
- Chiết khấu (discount)
- Nhập hàng (stock receipt) — thực hiện trên Odoo

---

## 3. Người dùng & Phân quyền

Giai đoạn phát triển ban đầu, hệ thống có **2 role**:

| Role | Mô tả | Quyền chính |
|---|---|---|
| ADMIN | Quản trị hệ thống | Toàn quyền: quản lý user, cấu hình, hủy đơn đã xác nhận |
| POS User | Nhân viên sử dụng POS | Tạo đơn, ghi nhận thanh toán, kiểm tra tồn kho, xử lý backorder |

**Ghi chú:**
- 1 user có thể được gán nhiều role
- Không quản lý ca làm việc (shift)
- Phân quyền theo địa điểm kho (location): user chỉ thấy tồn kho tại location được gán
- Warehouse tự động chọn dựa trên warehouse/location được gán cho user
- Quản lý user (tạo, vô hiệu hóa) thực hiện trong app bởi ADMIN

**Số lượng người dùng đồng thời**: ~30 người, không có giao dịch trùng lúc.

---

## 4. Kiến trúc hệ thống

```mermaid
graph TD
    subgraph Client["Client"]
        Browser["Browser SPA\nVue 3 + Quasar\niOS / Android"]
    end

    subgraph Docker["Docker Compose — Server độc lập"]
        NGINX["NGINX\nSSL Termination + Reverse Proxy"]
        FastAPI["FastAPI Backend\n+ TTLCache + WeasyPrint"]
        PG["PostgreSQL\nusers / logs / drafts / tokens"]
    end

    subgraph External["External Services"]
        Odoo["Odoo 14 CE\nServer riêng"]
        VietQR["VietQR API\nMST Lookup"]
    end

    Browser -->|HTTPS| NGINX
    NGINX -->|HTTP internal| FastAPI
    FastAPI <-->|SQL| PG
    FastAPI -->|XML-RPC| Odoo
    FastAPI -->|HTTPS REST| VietQR
```

### 4.1 Tech Stack

| Layer | Công nghệ | Ghi chú |
|---|---|---|
| Frontend | Vue 3 + Quasar Framework | Mobile-first, SPA |
| State Management | Pinia | Shared state: user session, cart, draft order |
| Backend | Python FastAPI | Async, Swagger auto-doc |
| Cache | `cachetools.TTLCache` (in-process) | Hot data: sản phẩm, tồn kho, MST, pricelist |
| Database | PostgreSQL | Users, audit log, draft orders, refresh tokens, print templates, config |
| Auth | JWT (access + refresh token) | Stateless |
| Reverse Proxy | NGINX | SSL termination, static serving |
| Container | Docker Compose | Server riêng với Odoo |
| Odoo connector | `xmlrpc.client` (Python built-in) | Service account model |
| PDF generation | `WeasyPrint` + `Jinja2` (Python, backend) | FastAPI render HTML/CSS template → PDF binary |

### 4.2 Mô hình Auth — Service Account

- Backend duy trì **1 Odoo service account** dùng chung cho tất cả lời gọi XML-RPC
- **Service account được cấp toàn quyền (admin) trên Odoo** — phân quyền kiểm soát hoàn toàn tại tầng backend
- Người dùng đăng nhập vào hệ thống backend riêng (PostgreSQL)
- JWT access token (TTL ngắn) + refresh token (TTL dài, lưu PostgreSQL)
- Logout: xóa refresh token khỏi DB — không cần JWT blacklist

### 4.3 Chiến lược Cache

**In-memory TTLCache** (`cachetools`, trong FastAPI process) — hot data, tự expire, không cần service ngoài:

| Dữ liệu | TTL | Ghi chú |
|---|---|---|
| Danh mục sản phẩm, giá | 30 phút | Flush thủ công khi cập nhật trên Odoo |
| Thông tin khách hàng | 15 phút | On-demand |
| Tồn kho (`stock.quant`) | 5 phút | Chấp nhận được vì không giao dịch đồng thời |
| Pricelist (fixed price) | 1 giờ | 1 pricelist duy nhất, flush thủ công |
| Kết quả tra cứu MST | 10 phút | VietQR API |

**PostgreSQL** — dữ liệu cần persistence qua restart:

| Dữ liệu | Bảng | Ghi chú |
|---|---|---|
| Đơn nháp (draft order) | `draft_orders` + cột `expires_at` | Cleanup job định kỳ xóa bản ghi hết hạn |
| Refresh token | `refresh_tokens` + cột `expires_at` | Xóa khi logout hoặc hết hạn |

---

## 5. Tích hợp Odoo 14 CE

### 5.1 Odoo Models sử dụng

| Nghiệp vụ | Model | Phương thức |
|---|---|---|
| Bán hàng | `sale.order`, `sale.order.line` | `create`, `action_confirm`, `action_cancel` |
| Xuất kho | `stock.picking`, `stock.move.line` | `button_validate` |
| Backorder | `stock.backorder.confirmation` | `process` |
| Serial / Lot | `stock.lot` | `search_read` (gợi ý), `create` (nếu mới) |
| Invoice | `account.move` | `create` (draft — kế toán post trên Odoo) |
| Thanh toán | `account.payment` | `create` (từ Mobile POS, dựa trên thanh toán thực tế) |
| Tồn kho | `stock.quant` | `search_read` |
| Khách hàng | `res.partner` | `search_read`, `create`, `write` |
| Sản phẩm | `product.product` | `search_read` |
| Pricelist | `product.pricelist` | `get_product_price` |

### 5.2 Ghi chú thiết kế quan trọng

- **Service account**: Cấp toàn quyền Odoo admin — kiểm soát truy cập qua backend
- **Pricelist**: Fixed price, 1 pricelist duy nhất, không chiết khấu
- **UoM**: 1 đơn vị tính duy nhất cho tất cả sản phẩm
- **VAT**: Chưa áp dụng giai đoạn đầu
- **Invoice**: Tạo ở trạng thái **draft** — kế toán post trên Odoo riêng
- **account.payment**: Tạo từ Mobile POS dựa trên thanh toán thực tế của khách
- **Serial Number**: POS user nhập / chọn serial, hệ thống **gợi ý từ `stock.lot` có sẵn** theo sản phẩm
- **Warehouse**: Tự động chọn dựa trên warehouse/location được gán cho user (14 warehouse, 20+ location)
- **Backorder**: Được hiển thị và xử lý trong Mobile POS

---

## 6. Tích hợp External APIs

### 6.1 Tra cứu MST (Mã số thuế)

- **Provider**: VietQR API (miễn phí)
- **Endpoint**: `GET https://api.vietqr.io/v2/business/{mst}`
- **Trả về**: Tên doanh nghiệp, địa chỉ, trạng thái hoạt động
- **Cache**: TTLCache in-memory, TTL = 10 phút
- **Rate limit**: Miễn phí — cần theo dõi giới hạn khi traffic tăng

### 6.2 Thanh toán

- **Phương thức**: Tiền mặt, Chuyển khoản, VNPay, MoMo, Thẻ tín dụng
- **Mô hình**: Ghi nhận **thủ công** — thu ngân xác nhận sau khi khách thanh toán
- **Gateway API**: Ngoài phạm vi giai đoạn này
- **Đặc điểm**: Hỗ trợ nhiều phương thức trong 1 đơn
- **Đặt cọc**: Hỗ trợ — nhân viên nhập tự do, tìm đơn theo thông tin KH khi thu phần còn lại

---

## 7. Cấu trúc thư mục dự kiến

```
vj-mobile-pos/
├── backend/
│   ├── app/
│   │   ├── api/          # routes: auth, orders, products, inventory, payment, templates, print
│   │   ├── core/         # config, security (JWT), cache (TTLCache)
│   │   ├── odoo/         # XML-RPC client wrapper + model helpers
│   │   ├── external/     # MST (VietQR)
│   │   └── print/        # WeasyPrint + Jinja2: render template → PDF
│   ├── .env
│   └── requirements.txt
├── frontend/
│   └── (Vue 3 + Quasar project)
├── nginx/
│   └── nginx.conf
├── docker-compose.yml
└── docs/
    ├── scope.md
    ├── process_flow.md
    └── open_questions.md
```

---

## 8. Non-functional Requirements

| Tiêu chí | Yêu cầu | Trạng thái |
|---|---|---|
| Concurrent users | ~30, không giao dịch trùng lúc | Đã xác nhận |
| Response time | < 2s cho các thao tác thường | Cần xác nhận |
| Availability | 24/7 | Đã xác nhận |
| Deploy | Docker Compose, server riêng với Odoo | Đã xác nhận |
| SSL / Reverse Proxy | NGINX | Đã xác nhận |
| Backup | Tự backup PostgreSQL qua Docker | Đã xác nhận |
| Audit log | File `*.log`, stream Loki (roadmap) | Đã xác nhận |
| HTTPS | Bắt buộc qua NGINX | Đã xác nhận |
| Ngôn ngữ UI | Tiếng Việt | Đã xác nhận |
| Thiết bị | Máy tính (Chrome) + iPad (Safari) | Đã xác nhận |
| Responsive | Desktop layout + Tablet layout | Đã xác nhận |
| Browser support | Chrome latest, Safari (iOS/iPadOS) | Đã xác nhận |

---

## 9. Các vấn đề còn mở

Xem file [open_questions.md](open_questions.md) để theo dõi toàn bộ câu hỏi cần làm rõ.

---

## 10. Backlog — Future Development

Các hạng mục chưa triển khai trong giai đoạn này, để dành cho các phiên bản tiếp theo.

**Mức độ ưu tiên**: `Cao` / `Trung bình` / `Thấp`

### 10.1 Nghiệp vụ

| # | Hạng mục | Mô tả | Ưu tiên |
|---|---|---|---|
| B-01 | Hoàn trả hàng (Return) | Tạo phiếu nhập hàng trả về từ Mobile POS, reverse stock.picking | Cao |
| B-02 | Tích hợp payment gateway | Kết nối trực tiếp VNPay / MoMo API, xử lý callback tự động | Cao |
| B-03 | Chiết khấu (Discount) | Chiết khấu theo dòng sản phẩm hoặc theo tổng đơn | Trung bình |
| B-04 | Thuế VAT | Tự động tính thuế theo cấu hình sản phẩm trên Odoo | Trung bình |
| B-05 | Hóa đơn điện tử | Kết nối nhà cung cấp HĐDT (VNPT, MISA...) sau khi invoice được post | Trung bình |
| B-06 | Báo cáo / Dashboard | Doanh thu theo ngày, theo nhân viên, tồn kho tổng hợp cho Manager | Trung bình |
| B-07 | Quản lý ca làm việc | Mở / đóng ca thu ngân, tổng kết doanh thu theo ca | Thấp |
| B-08 | Role chi tiết hơn | Thêm role: Thủ kho, Kế toán, Manager với permission riêng | Thấp |

### 10.2 Kỹ thuật & Infrastructure

| # | Hạng mục | Mô tả | Ưu tiên |
|---|---|---|---|
| B-09 | Audit log stream Loki | Chuyển từ file `*.log` sang Loki + Grafana để query và alert | Trung bình |
| B-10 | Webhook từ Odoo | Nhận sự kiện cập nhật sản phẩm / giá từ Odoo để invalidate cache tự động | Trung bình |
| B-11 | Push notification | Thông báo cho nhân viên khi backorder sẵn sàng validate | Thấp |
| B-12 | Scale multi-worker | Nếu cần scale FastAPI lên nhiều worker → thêm Redis cho shared cache | Thấp |
| B-13 | Monitoring / Alerting | Tích hợp Prometheus + Grafana để theo dõi uptime và performance | Thấp |

### 10.3 Thông báo & Messaging

| # | Hạng mục | Mô tả | Ưu tiên |
|---|---|---|---|
| B-14 | Email Service | Gửi email xác nhận đơn hàng, biên lai thanh toán, thông báo đặt cọc cho khách hàng. Provider: SMTP nội bộ hoặc SendGrid / AWS SES | Trung bình |
| B-15 | Zalo Messaging | Gửi thông báo tự động qua Zalo OA (Official Account): xác nhận đơn, trạng thái giao hàng, nhắc thanh toán phần còn lại | Trung bình |

### 10.4 Ngoài phạm vi dài hạn

| # | Hạng mục | Lý do chưa xem xét |
|---|---|---|
| B-16 | Offline mode / PWA sync | Phức tạp, nghiệp vụ yêu cầu live network |
| B-17 | Native mobile app (iOS / Android) | Không cần — chỉ dùng Chrome (PC) và Safari (iPad) |
| B-18 | Tích hợp phần cứng | Máy in hóa đơn, barcode scanner, customer display |
| B-19 | BI / Analytics nâng cao | Ngoài phạm vi hệ thống POS — dùng Odoo reporting |
