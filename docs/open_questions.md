# VJ Mobile POS — Open Questions

> Trạng thái: Draft  
> Cập nhật: 2026-04-14  
> Phiên bản: 0.1

Danh sách toàn bộ câu hỏi cần được làm rõ trước khi thiết kế / triển khai các phần liên quan.

**Trạng thái**: `Chờ` / `Đang xử lý` / `Đã xác nhận`

---

## Nhóm A — Nghiệp vụ Bán hàng

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-A01 | Invoice (account.move) được tạo ngay khi xác nhận đơn hay chỉ sau khi validate picking (giao hàng)? | Luồng thanh toán, flow bán hàng | Chờ | |
| OQ-A02 | Chiết khấu áp dụng theo từng dòng sản phẩm, theo tổng đơn, hay cả hai? | UI tạo đơn, tính giá | Chờ | |
| OQ-A03 | Có hỗ trợ đặt cọc (deposit) trước khi giao hàng không? | Luồng thanh toán | Chờ | |
| OQ-A04 | Một đơn hàng có thể giao nhiều lần (partial delivery) không? | Luồng xuất kho | Chờ | |
| OQ-A05 | Có cần tính năng tạm lưu đơn nháp (save for later) không? | UI, data model | Chờ | |

---

## Nhóm B — Xuất kho & Kho

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-B01 | Sau khi xác nhận sale.order, stock.picking được validate **tự động** hay thủ công bởi thủ kho? | Luồng xuất kho, phân quyền | Chờ | |
| OQ-B02 | Nếu thiếu hàng tại thời điểm xác nhận đơn, hệ thống xử lý thế nào? (Chặn / Cảnh báo / Cho phép backorder?) | Validation tạo đơn | Chờ | |
| OQ-B03 | Hệ thống Odoo đang có bao nhiêu kho (warehouse) và địa điểm (location)? | Cache tồn kho, UI chọn kho | Chờ | |
| OQ-B04 | Mobile POS cần hiển thị tồn kho theo từng kho riêng lẻ hay tổng hợp? | UI kiểm kho | Chờ | |

---

## Nhóm C — Thanh toán

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-C01 | Các phương thức thanh toán cần hỗ trợ? (Tiền mặt / Chuyển khoản / QR / Thẻ) | Luồng thanh toán, UI | Chờ | |
| OQ-C02 | Payment gateway cụ thể là gì? (VNPay, MoMo, ngân hàng nào?) | Tích hợp external API | Chờ | |
| OQ-C03 | Một đơn hàng có thể thanh toán bằng nhiều phương thức không? (VD: một phần tiền mặt, một phần chuyển khoản) | Logic thanh toán | Chờ | |
| OQ-C04 | Webhook / callback từ payment gateway được xử lý như thế nào? Cần expose endpoint public không? | Backend, infrastructure | Chờ | |
| OQ-C05 | Nếu giao dịch thanh toán gateway timeout / thất bại, retry policy là gì? | Error handling | Chờ | |

---

## Nhóm D — Hủy đơn & Hoàn trả

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-D01 | Rule hủy đơn theo từng trạng thái: Draft / Đã xác nhận / Đã xuất kho? | Luồng hủy đơn, phân quyền | Chờ | |
| OQ-D02 | Hoàn trả hàng (return) có nằm trong scope của dự án này không? | Scope, luồng hoàn trả | Chờ | |
| OQ-D03 | Khi hủy đơn đã xác nhận, có cần reverse stock.picking không? | Tích hợp Odoo | Chờ | |

---

## Nhóm E — Phân quyền & Người dùng

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-E01 | Danh sách đầy đủ các role và permission cụ thể của từng role? | Auth, UI, toàn bộ luồng | Chờ | |
| OQ-E02 | Có cần quản lý ca làm việc (shift) cho thu ngân không? | Auth, audit log | Chờ | |
| OQ-E03 | Một user có thể có nhiều role không? | Data model user | Chờ | |
| OQ-E04 | Quản lý user (tạo, vô hiệu hóa) thực hiện ở đâu — trong app hay ngoài? | Admin UI | Chờ | |

---

## Nhóm F — Tích hợp Odoo

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-F01 | Odoo service account cần những quyền (access rights) gì? | Setup Odoo, bảo mật | Chờ | |
| OQ-F02 | Pricelist trên Odoo đang dùng loại nào? (Fixed price / Percentage / Formula) | Logic tính giá, cache | Chờ | |
| OQ-F03 | Có sử dụng Unit of Measure (UoM) không? | Hiển thị sản phẩm | Chờ | |
| OQ-F04 | Thuế (VAT) áp dụng như thế nào? Tính tự động theo sản phẩm hay cần chọn thủ công? | Tạo đơn, invoice | Chờ | |
| OQ-F05 | Có sử dụng tính năng Lot / Serial Number tracking không? | Xuất kho | Chờ | |

---

## Nhóm G — External APIs

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-G01 | MST API: dùng provider nào? (Tổng cục Thuế trực tiếp hay bên thứ ba như VNPT, MISA?) | Tích hợp MST | Chờ | |
| OQ-G02 | MST API có rate limit hoặc chi phí không? | Cache strategy, cost | Chờ | |
| OQ-G03 | Kết quả tra cứu MST có cần cache lâu dài (persistent) không hay chỉ Redis TTL? | Cache design | Chờ | |

---

## Nhóm H — Hạ tầng & Vận hành

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-H01 | Deploy trên server riêng hay cùng server với Odoo? Dùng Docker / Docker Compose không? | Infrastructure | Chờ đánh giá |Docker compose trên 1 server độc lập với Odoo.  |
| OQ-H02 | Availability requirement: chỉ giờ hành chính hay 24/7? | SLA, monitoring | Chờ | 24/7 |
| OQ-H03 | Audit log cần ghi những thao tác nào? Lưu trữ bao lâu? | Data model, storage | Chờ | sẽ tạm thời hiển thị các giao dich trong *.log, có thể sau này stream vào loki |
| OQ-H04 | Có yêu cầu backup database backend (PostgreSQL) không? Tần suất? | Ops, DR plan | Chờ | Sẽ tự backup database docker  |
| OQ-H05 | Domain / SSL certificate do ai quản lý? | Infrastructure | Chờ | dùng NGINX |
