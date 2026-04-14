# VJ Mobile POS — Scope & Architecture Document

> Trạng thái: Draft  
> Cập nhật: 2026-04-14  
> Phiên bản: 0.1

---

## 1. Tổng quan dự án

VJ Mobile POS là một ứng dụng web POS nhẹ, chạy trên trình duyệt (iOS & Android), tích hợp với hệ thống Odoo 14 CE thông qua XML-RPC. Hệ thống **không thay thế module POS của Odoo** mà đóng vai trò là một UI nghiệp vụ riêng, gọi vào Odoo để thực hiện các thao tác bán hàng và quản lý kho.

---

## 2. Phạm vi nghiệp vụ (In Scope)

| # | Nghiệp vụ | Mô tả |
|---|---|---|
| 1 | Bán hàng | Tạo và xác nhận `sale.order` |
| 2 | Nhận thanh toán | Ghi nhận thanh toán, tích hợp payment gateway |
| 3 | Xuất tồn kho | Validate `stock.picking` sau khi xác nhận đơn |
| 4 | Kiểm tra tồn kho | Tra cứu `stock.quant` theo sản phẩm / kho |
| 5 | Tra cứu MST | Tra cứu mã số thuế khách hàng qua API ngoài |

### Ngoài phạm vi (Out of Scope)

- Module POS (`pos.order`) của Odoo
- Offline mode / PWA sync
- Tích hợp phần cứng (máy in, barcode scanner, màn hình phụ)
- Quản lý sản phẩm / danh mục (thực hiện trực tiếp trên Odoo)
- Báo cáo / BI

---

## 3. Người dùng & Phân quyền

> **Cần làm rõ**: Các role cụ thể và permission tương ứng.

Sơ bộ các role dự kiến:

| Role | Mô tả |
|---|---|
| Thu ngân (Cashier) | Tạo đơn, nhận thanh toán |
| Thủ kho (Warehouse) | Xác nhận xuất kho, kiểm tra tồn kho |
| Quản lý (Manager) | Xem báo cáo, hủy đơn, điều chỉnh |

**Số lượng người dùng đồng thời**: ~30 người, không có giao dịch trùng lúc.

---

## 4. Kiến trúc hệ thống

```
[Browser SPA - Vue 3 + Quasar]
          ↕ HTTPS REST + JWT
┌─────────────────────────────────┐
│        FastAPI Backend          │
│  ┌──────────┐  ┌─────────────┐  │
│  │  Redis   │  │ PostgreSQL  │  │
│  │  Cache   │  │  (users,    │  │
│  │          │  │  audit log) │  │
│  └──────────┘  └─────────────┘  │
└────────────┬────────────────────┘
       ↕ XML-RPC        ↕ REST
  [Odoo 14 CE]    [External APIs]
                  - Payment Gateway
                  - MST (Tổng cục Thuế)
```

### 4.1 Tech Stack

| Layer | Công nghệ | Ghi chú |
|---|---|---|
| Frontend | Vue 3 + Quasar Framework | Mobile-first, SPA |
| Backend | Python FastAPI | Async, Swagger auto-doc |
| Cache | Redis | TTL linh hoạt, JWT blacklist |
| Database | PostgreSQL | Users, audit log, config |
| Auth | JWT (access + refresh token) | Stateless |
| Odoo connector | `xmlrpc.client` (Python built-in) | Service account model |

### 4.2 Mô hình Auth — Service Account

- Backend duy trì **1 Odoo service account** dùng chung cho tất cả lời gọi XML-RPC
- Người dùng đăng nhập vào **hệ thống backend riêng** (PostgreSQL)
- Phân quyền được kiểm soát tại tầng backend, không phụ thuộc Odoo user permission
- JWT access token (TTL ngắn) + refresh token (TTL dài, lưu Redis)

### 4.3 Chiến lược Cache (Redis)

| Dữ liệu | TTL | Ghi chú |
|---|---|---|
| Danh mục sản phẩm, giá | 30 phút | Flush thủ công khi cập nhật trên Odoo |
| Thông tin khách hàng | 15 phút | On-demand |
| Tồn kho (`stock.quant`) | 5 phút | Chấp nhận được vì không giao dịch đồng thời |
| Pricelist / chiết khấu | 1 giờ | Flush thủ công |
| JWT blacklist (logout) | = token expiry | Tự động expire |

---

## 5. Tích hợp Odoo 14 CE

### 5.1 Odoo Models sử dụng

| Nghiệp vụ | Model | Phương thức |
|---|---|---|
| Bán hàng | `sale.order`, `sale.order.line` | `create`, `action_confirm` |
| Xuất kho | `stock.picking`, `stock.move` | `button_validate` |
| Thanh toán | `account.move`, `account.payment` | `action_post`, `action_register_payment` |
| Tồn kho | `stock.quant` | `search_read` |
| Khách hàng | `res.partner` | `search_read`, `create`, `write` |
| Sản phẩm | `product.product`, `product.template` | `search_read` |

### 5.2 Ghi chú tích hợp Odoo

- Service account dùng chung, gọi XML-RPC qua `xmlrpc.client`
- Các vấn đề chi tiết xem bảng Open Questions (mục 9), nhóm C

---

## 6. Tích hợp External APIs

### 6.1 Tra cứu MST (Mã số thuế)

- **Mục đích**: Tự động điền thông tin doanh nghiệp khi nhập MST
- **Provider**: Chưa xác định — xem OQ-15, OQ-16

### 6.2 Payment Gateway

- **Mục đích**: Nhận thanh toán điện tử từ khách hàng
- **Provider**: Chưa xác định — xem OQ-17, OQ-18, OQ-19

---

## 7. Cấu trúc thư mục dự kiến

```
vj-mobile-pos/
├── backend/
│   ├── app/
│   │   ├── api/          # routes: auth, orders, products, inventory, payment
│   │   ├── core/         # config, security (JWT), cache (Redis)
│   │   ├── odoo/         # XML-RPC client wrapper + model helpers
│   │   └── external/     # MST API, payment gateway
│   ├── .env
│   └── requirements.txt
├── frontend/
│   └── (Vue 3 + Quasar project)
└── docs/
    ├── scope.md
    └── process_flow.md
```

---

## 8. Non-functional Requirements

| Tiêu chí | Yêu cầu hiện tại | Trạng thái |
|---|---|---|
| Concurrent users | ~30, không giao dịch trùng lúc | Đã xác nhận |
| Response time | < 2s cho các thao tác thường | Cần xác nhận |
| Availability | Chưa xác định | Xem OQ-21 |
| Backup Odoo data | Do Odoo quản lý | Đã xác nhận |
| Audit log | Chưa xác định | Xem OQ-22 |
| HTTPS | Bắt buộc | Đã xác nhận |
| Ngôn ngữ UI | Tiếng Việt | Đã xác nhận |

---

## 9. Các vấn đề còn mở

Xem file [open_questions.md](open_questions.md) để theo dõi toàn bộ câu hỏi cần làm rõ.
