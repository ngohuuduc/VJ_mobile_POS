# VJ Mobile POS — Open Questions

> Trạng thái: Draft  
> Cập nhật: 2026-04-14  
> Phiên bản: 0.3

Danh sách toàn bộ câu hỏi cần được làm rõ trước khi thiết kế / triển khai các phần liên quan.

**Trạng thái**: `Chờ` / `Đang xử lý` / `Đã xác nhận`

---

## Nhóm A — Nghiệp vụ Bán hàng

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-A01 | Invoice (account.move) được tạo ngay khi xác nhận đơn hay chỉ sau khi validate picking? | Luồng thanh toán, flow bán hàng | Đã xác nhận | Tạo ngay ở trạng thái draft. Kế toán post trên Odoo riêng. |
| OQ-A02 | Chiết khấu áp dụng theo từng dòng, theo tổng đơn, hay cả hai? | UI tạo đơn, tính giá | Đã xác nhận | Không có chiết khấu. |
| OQ-A03 | Có hỗ trợ đặt cọc (deposit) không? | Luồng thanh toán | Đã xác nhận | Có hỗ trợ. |
| OQ-A04 | Một đơn hàng có thể giao nhiều lần (partial delivery) không? | Luồng xuất kho | Đã xác nhận | Có — kết hợp với backorder. |
| OQ-A05 | Có cần tính năng tạm lưu đơn nháp (save for later) không? | UI, data model | Đã xác nhận | Có — lưu trong vòng 8 giờ tại Redis. |
| OQ-A06 | (1) Số tiền cọc do thu ngân nhập tự do hay theo tỷ lệ cố định? (2) Đơn hàng ở trạng thái gì trên Odoo sau khi đặt cọc? (3) Thu phần còn lại qua flow nào trên Mobile POS? | Luồng đặt cọc, Odoo model | Đã xác nhận | (1) Nhân viên nhập tự do. (2) sale.order giữ trạng thái confirmed. (3) Nhân viên tìm đơn theo thông tin khách hàng rồi ghi nhận tiếp. |
| OQ-A07 | Đơn nháp 8 giờ lưu tại Redis hay PostgreSQL? | Data model, reliability | Đã xác nhận | PostgreSQL (bảng `draft_orders` + `expires_at`). Redis bị loại khỏi stack — dùng TTLCache in-memory cho hot data. |

---

## Nhóm B — Xuất kho & Kho

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-B01 | stock.picking validate tự động hay thủ công? | Luồng xuất kho | Đã xác nhận | Tự động validate phần có hàng. Tạo backorder cho phần thiếu. |
| OQ-B02 | Nếu thiếu hàng, hệ thống xử lý thế nào? | Validation tạo đơn | Đã xác nhận | Cảnh báo và tạo backorder. |
| OQ-B03 | Bao nhiêu kho và địa điểm? | Cache tồn kho, UI | Đã xác nhận | 14 warehouse, hơn 20 location. |
| OQ-B04 | Hiển thị tồn kho theo từng kho hay tổng hợp? | UI kiểm kho | Đã xác nhận | Theo từng location, user được phân quyền location cụ thể. |
| OQ-B05 | Mobile POS cần hiển thị và xử lý backorder không? | UI, scope | Đã xác nhận | Có — theo dõi và validate backorder trong Mobile POS. |
| OQ-B06 | Warehouse chọn tự động hay thủ công khi tạo đơn? | UX tạo đơn | Đã xác nhận | Tự động theo warehouse/location được gán cho user. |

---

## Nhóm C — Thanh toán

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-C01 | Các phương thức thanh toán cần hỗ trợ? | Luồng thanh toán, UI | Đã xác nhận | Tiền mặt, chuyển khoản, VNPay, MoMo, thẻ tín dụng. |
| OQ-C02 | Payment gateway cụ thể? | Tích hợp external API | Đã xác nhận | VNPay, MoMo — ghi nhận thủ công, chưa tích hợp API. |
| OQ-C03 | Thanh toán nhiều phương thức trong 1 đơn? | Logic thanh toán | Đã xác nhận | Có. |
| OQ-C04 | Webhook / callback gateway? | Backend, infra | Đã xác nhận | Không áp dụng giai đoạn này. |
| OQ-C05 | Retry policy khi gateway thất bại? | Error handling | Đã xác nhận | Không áp dụng giai đoạn này. |
| OQ-C06 | Tích hợp gateway API có trong roadmap không? | Kế hoạch phát triển | Đã xác nhận | Không phải giai đoạn này. |
| OQ-C07 | `account.payment` tạo từ Mobile POS hay kế toán tạo thủ công? | Tích hợp Odoo | Đã xác nhận | Tạo từ Mobile POS dựa trên thanh toán thực tế. |

---

## Nhóm D — Hủy đơn & Hoàn trả

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-D01 | Rule hủy đơn theo từng trạng thái? | Luồng hủy đơn | Đã xác nhận | Draft → cancel. Đã xác nhận/xuất kho → cần nhập hàng về, thực hiện trên Odoo. |
| OQ-D02 | Hoàn trả (return) có trong scope không? | Scope | Đã xác nhận | Chưa có giai đoạn này. |
| OQ-D03 | Hủy đơn đã xác nhận có cần reverse picking không? | Tích hợp Odoo | Đã xác nhận | Có — hủy picking trước khi cancel sale.order. |

---

## Nhóm E — Phân quyền & Người dùng

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-E01 | Danh sách role và permission? | Auth, UI | Đã xác nhận | Giai đoạn dev: ADMIN (toàn quyền) và POS User. |
| OQ-E02 | Cần quản lý ca làm việc (shift) không? | Auth, audit log | Đã xác nhận | Không. |
| OQ-E03 | 1 user có thể có nhiều role không? | Data model | Đã xác nhận | Có. |
| OQ-E04 | Quản lý user ở đâu? | Admin UI | Đã xác nhận | Trong app, ADMIN quản lý. |

---

## Nhóm F — Tích hợp Odoo

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-F01 | Odoo service account cần quyền gì? | Setup Odoo, bảo mật | Đã xác nhận | Cấp toàn quyền admin trên Odoo — kiểm soát phân quyền tại backend. |
| OQ-F02 | Pricelist loại nào? | Logic tính giá | Đã xác nhận | Fixed price, 1 pricelist duy nhất. |
| OQ-F03 | Có dùng UoM không? | Hiển thị sản phẩm | Đã xác nhận | 1 UoM duy nhất cho tất cả sản phẩm. |
| OQ-F04 | VAT áp dụng thế nào? | Tạo đơn, invoice | Đã xác nhận | Chưa áp dụng giai đoạn đầu. |
| OQ-F05 | Có dùng Serial / Lot Number không? | Xuất kho | Đã xác nhận | Có — nhiều sản phẩm có serial. |
| OQ-F06 | Ai nhập serial khi validate picking? | UX xuất kho | Đã xác nhận | POS user nhập / chọn. Hệ thống gợi ý từ `stock.lot` có sẵn theo sản phẩm. |
| OQ-F07 | Quyền chi tiết của service account? | Setup Odoo | Đã xác nhận | Cấp toàn quyền admin Odoo. |

---

## Nhóm G — External APIs

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-G01 | MST API provider? | Tích hợp MST | Đã xác nhận | VietQR API: `https://api.vietqr.io/v2/business/{mst}` |
| OQ-G02 | MST API rate limit / chi phí? | Cache strategy | Đã xác nhận | Hiện tại miễn phí — cần theo dõi. |
| OQ-G03 | Cache MST bao lâu? | Cache design | Đã xác nhận | TTLCache in-memory, TTL = 10 phút. |

---

## Nhóm H — Hạ tầng & Vận hành

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-H01 | Deploy environment? | Infrastructure | Đã xác nhận | Docker Compose, server độc lập với Odoo. |
| OQ-H02 | Availability requirement? | SLA, monitoring | Đã xác nhận | 24/7. |
| OQ-H03 | Audit log ghi gì? Lưu bao lâu? | Data model, storage | Đã xác nhận | File `*.log` — có thể stream vào Loki sau. |
| OQ-H04 | Backup database? | Ops, DR | Đã xác nhận | Tự backup PostgreSQL qua Docker. |
| OQ-H05 | SSL / domain? | Infrastructure | Đã xác nhận | NGINX làm reverse proxy + SSL termination. |

---

## Nhóm I — In phiếu & PDF

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-I01 | Logo công ty lấy từ đâu? Upload trong app hay từ Odoo (`res.company`)? | Template design, data source | Chờ | |
| OQ-I02 | Thông tin công ty (tên, địa chỉ, MST, SĐT) lấy từ đâu? Config trong app hay đọc từ Odoo `res.company`? | Template data, config | Chờ | |
| OQ-I03 | Có cần in số tiền bằng chữ (tiếng Việt) trên phiếu không? | PDF content, backend logic | Chờ | |
| OQ-I04 | PDF sau khi generate: mở tab mới để in hay download file? | UX, frontend | Chờ | |
| OQ-I05 | Admin chỉnh sửa template theo dạng nào? Soạn thảo HTML/CSS trực tiếp hay có visual editor? | Admin UI, độ phức tạp | Chờ | |
| OQ-I06 | Mỗi loại phiếu có 1 template cố định hay có thể tạo nhiều template rồi chọn? | Data model template | Chờ | |
| OQ-I07 | Phiếu có cần chữ ký / vùng ký tên không? (ví dụ: "Người mua" / "Nhân viên bán hàng") | Template design | Chờ | |
