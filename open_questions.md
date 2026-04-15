# VJ Mobile POS — Open Questions

> Trạng thái: Draft  
> Cập nhật: 2026-04-15  
> Phiên bản: 0.9

Danh sách toàn bộ câu hỏi cần được làm rõ trước khi thiết kế / triển khai các phần liên quan.

**Trạng thái**: ⏳ `Chờ` / 🔄 `Đang xử lý` / ✅ `Đã xác nhận`

---

## Nhóm A — Nghiệp vụ Bán hàng

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-A01 | Invoice (account.move) được tạo ngay khi xác nhận đơn hay chỉ sau khi validate picking? | Luồng thanh toán, flow bán hàng | ✅ Đã xác nhận | Tạo ngay ở trạng thái draft. Kế toán post trên Odoo riêng. |
| OQ-A02 | Chiết khấu áp dụng theo từng dòng, theo tổng đơn, hay cả hai? | UI tạo đơn, tính giá | ✅ Đã xác nhận | Không có chiết khấu. |
| OQ-A03 | Có hỗ trợ đặt cọc (deposit) không? | Luồng thanh toán | ✅ Đã xác nhận | Có hỗ trợ. |
| OQ-A04 | Một đơn hàng có thể giao nhiều lần (partial delivery) không? | Luồng xuất kho | ✅ Đã xác nhận | Có — kết hợp với backorder. |
| OQ-A05 | Có cần tính năng tạm lưu đơn nháp (save for later) không? | UI, data model | ✅ Đã xác nhận | Có — lưu trong vòng 8 giờ (PostgreSQL). |
| OQ-A06 | (1) Số tiền cọc do thu ngân nhập tự do hay theo tỷ lệ cố định? (2) Đơn hàng ở trạng thái gì trên Odoo sau khi đặt cọc? (3) Thu phần còn lại qua flow nào trên Mobile POS? | Luồng đặt cọc, Odoo model | ✅ Đã xác nhận | (1) Nhân viên nhập tự do. (2) sale.order giữ trạng thái confirmed. (3) Nhân viên tìm đơn theo thông tin khách hàng rồi ghi nhận tiếp. |
| OQ-A07 | Đơn nháp 8 giờ lưu tại Redis hay PostgreSQL? | Data model, reliability | ✅ Đã xác nhận | PostgreSQL (bảng `draft_orders` + `expires_at`). Redis bị loại khỏi stack — dùng TTLCache in-memory cho hot data. |

---

## Nhóm B — Xuất kho & Kho

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-B01 | stock.picking validate tự động hay thủ công? | Luồng xuất kho | ✅ Đã xác nhận | Tự động validate phần có hàng. Tạo backorder cho phần thiếu. |
| OQ-B02 | Nếu thiếu hàng, hệ thống xử lý thế nào? | Validation tạo đơn | ✅ Đã xác nhận | Cảnh báo và tạo backorder. |
| OQ-B03 | Bao nhiêu kho và địa điểm? | Cache tồn kho, UI | ✅ Đã xác nhận | 14 warehouse, hơn 20 location. |
| OQ-B04 | Hiển thị tồn kho theo từng kho hay tổng hợp? | UI kiểm kho | ✅ Đã xác nhận | Theo từng location, user được phân quyền location cụ thể. |
| OQ-B05 | Mobile POS cần hiển thị và xử lý backorder không? | UI, scope | ✅ Đã xác nhận | Có — theo dõi và validate backorder trong Mobile POS. |
| OQ-B06 | Warehouse chọn tự động hay thủ công khi tạo đơn? | UX tạo đơn | ✅ Đã xác nhận | Tự động theo warehouse/location được gán cho user. |

---

## Nhóm C — Thanh toán

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-C01 | Các phương thức thanh toán cần hỗ trợ? | Luồng thanh toán, UI | ✅ Đã xác nhận | Tiền mặt, chuyển khoản, VNPay, MoMo, thẻ tín dụng, COD (giao hàng thu tiền hộ). |
| OQ-C02 | Payment gateway cụ thể? | Tích hợp external API | ✅ Đã xác nhận | VNPay, MoMo, VietQR Payment — giai đoạn 1 ghi nhận thủ công; tích hợp API theo planned (B10, B11, B12). |
| OQ-C03 | Thanh toán nhiều phương thức trong 1 đơn? | Logic thanh toán | ✅ Đã xác nhận | Có. |
| OQ-C04 | Webhook / callback gateway? | Backend, infra | ✅ Đã xác nhận | Có — IPN callback cho VNPay và MoMo; webhook ngân hàng hoặc fallback thủ công cho VietQR Payment. |
| OQ-C05 | Retry policy khi gateway thất bại? | Error handling | ✅ Đã xác nhận | Không áp dụng giai đoạn thủ công. Sẽ thiết kế khi tích hợp API. |
| OQ-C06 | Tích hợp gateway API có trong roadmap không? | Kế hoạch phát triển | ✅ Đã xác nhận | Có — VNPay, MoMo, VietQR Payment đều được thiết kế luồng (B10, B11, B12). |
| OQ-C07 | `account.payment` tạo từ Mobile POS hay kế toán tạo thủ công? | Tích hợp Odoo | ✅ Đã xác nhận | Tạo từ Mobile POS dựa trên thanh toán thực tế. |

---

## Nhóm D — Hủy đơn & Hoàn trả

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-D01 | Rule hủy đơn theo từng trạng thái? | Luồng hủy đơn | ✅ Đã xác nhận | Draft → cancel. Đã xác nhận/xuất kho → cần nhập hàng về, thực hiện trên Odoo. |
| OQ-D02 | Hoàn trả (return) có trong scope không? | Scope | ✅ Đã xác nhận | Chưa có giai đoạn này. |
| OQ-D03 | Hủy đơn đã xác nhận có cần reverse picking không? | Tích hợp Odoo | ✅ Đã xác nhận | Có — hủy picking trước khi cancel sale.order. |

---

## Nhóm E — Phân quyền & Người dùng

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-E01 | Danh sách role và permission? | Auth, UI | ✅ Đã xác nhận | Giai đoạn dev: ADMIN (toàn quyền) và POS User. |
| OQ-E02 | Cần quản lý ca làm việc (shift) không? | Auth, audit log | ✅ Đã xác nhận | Không. |
| OQ-E03 | 1 user có thể có nhiều role không? | Data model | ✅ Đã xác nhận | Có. |
| OQ-E04 | Quản lý user ở đâu? | Admin UI | ✅ Đã xác nhận | Trong app, ADMIN quản lý. |

---

## Nhóm F — Tích hợp Odoo

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-F01 | Odoo service account cần quyền gì? | Setup Odoo, bảo mật | ✅ Đã xác nhận | Cấp toàn quyền admin trên Odoo — kiểm soát phân quyền tại backend. |
| OQ-F02 | Pricelist loại nào? | Logic tính giá | ✅ Đã xác nhận | Fixed price, 1 pricelist duy nhất. |
| OQ-F03 | Có dùng UoM không? | Hiển thị sản phẩm | ✅ Đã xác nhận | 1 UoM duy nhất cho tất cả sản phẩm. |
| OQ-F04 | VAT áp dụng thế nào? | Tạo đơn, invoice | ✅ Đã xác nhận | Chưa áp dụng giai đoạn đầu. |
| OQ-F05 | Có dùng Serial / Lot Number không? | Xuất kho | ✅ Đã xác nhận | Có — nhiều sản phẩm có serial. |
| OQ-F06 | Ai nhập serial khi validate picking? | UX xuất kho | ✅ Đã xác nhận | POS user nhập / chọn. Hệ thống gợi ý từ `stock.lot` có sẵn theo sản phẩm. |
| OQ-F07 | Quyền chi tiết của service account? | Setup Odoo | ✅ Đã xác nhận | Cấp toàn quyền admin Odoo. |

---

## Nhóm G — External APIs

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-G01 | MST API provider? | Tích hợp MST | ✅ Đã xác nhận | VietQR API: `https://api.vietqr.io/v2/business/{mst}` |
| OQ-G02 | MST API rate limit / chi phí? | Cache strategy | ✅ Đã xác nhận | Hiện tại miễn phí — cần theo dõi. |
| OQ-G03 | Cache MST bao lâu? | Cache design | ✅ Đã xác nhận | TTLCache in-memory, TTL = 10 phút. |

---

## Nhóm H — Hạ tầng & Vận hành

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-H01 | Deploy environment? | Infrastructure | ✅ Đã xác nhận | Docker Compose, server độc lập với Odoo. |
| OQ-H02 | Availability requirement? | SLA, monitoring | ✅ Đã xác nhận | 24/7. |
| OQ-H03 | Audit log ghi gì? Lưu bao lâu? | Data model, storage | ✅ Đã xác nhận | File `*.log` — có thể stream vào Loki sau. |
| OQ-H04 | Backup database? | Ops, DR | ✅ Đã xác nhận | Tự backup PostgreSQL qua Docker. |
| OQ-H05 | SSL / domain? | Infrastructure | ✅ Đã xác nhận | NGINX làm reverse proxy + SSL termination. |

---

## Nhóm I — In phiếu & PDF

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-I01 | Logo công ty lấy từ đâu? Upload trong app hay từ Odoo (`res.company`)? | Template design, data source | ✅ Đã xác nhận | Lưu trong thư mục assets của backend. |
| OQ-I02 | Thông tin công ty (tên, địa chỉ, MST, SĐT) lấy từ đâu? Config trong app hay đọc từ Odoo `res.company`? | Template data, config | ✅ Đã xác nhận | Hardcode trong file HTML template. |
| OQ-I03 | Có cần in số tiền bằng chữ (tiếng Việt) trên phiếu không? | PDF content, backend logic | ✅ Đã xác nhận | Có. |
| OQ-I04 | PDF sau khi generate: mở tab mới để in hay download file? | UX, frontend | ✅ Đã xác nhận | Mở preview (tab mới hoặc inline viewer). |
| OQ-I05 | Admin chỉnh sửa template theo dạng nào? Soạn thảo HTML/CSS trực tiếp hay có visual editor? | Admin UI, độ phức tạp | ✅ Đã xác nhận | Có visual editor (WYSIWYG). |
| OQ-I06 | Mỗi loại phiếu có 1 template cố định hay có thể tạo nhiều template rồi chọn? | Data model template | ✅ Đã xác nhận | Mỗi loại chỉ có 1 template cố định. |
| OQ-I07 | Phiếu có cần chữ ký / vùng ký tên không? (ví dụ: "Người mua" / "Nhân viên bán hàng") | Template design | ✅ Đã xác nhận | Có vùng ký tên để ký sau khi in. |

---

## Nhóm J — Khách hàng

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-J01 | Có cho phép tạo đơn không có thông tin khách hàng (walk-in customer) không? | Luồng tạo đơn, data model | ✅ Đã xác nhận | Cho phép tạo đơn không cần KH. Chỉ bắt buộc chọn/tạo KH trước khi ghi nhận thanh toán. |
| OQ-J02 | Khi tạo khách hàng mới từ Mobile POS, những trường nào bắt buộc? (tên, SĐT, MST, địa chỉ?) | UI form khách hàng | ✅ Đã xác nhận | Tên, số điện thoại, email. |
| OQ-J03 | Tìm kiếm khách hàng theo tiêu chí nào? (tên, SĐT, MST — 1 hay nhiều tiêu chí?) | UI tìm kiếm | ✅ Đã xác nhận | Tên, số điện thoại, email, MST. |
| OQ-J04 | POS User có thể chỉnh sửa thông tin khách hàng (res.partner) không, hay chỉ ADMIN? | Phân quyền, Odoo write | ✅ Đã xác nhận | POS User được phép cập nhật trực tiếp. |

---

## Nhóm K — Sản phẩm

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-K01 | Sản phẩm có variant (màu sắc, kích cỡ...) không? Nếu có, hiển thị chọn variant thế nào? | UI tạo đơn, Odoo model | ✅ Đã xác nhận | Không có biến thể sản phẩm trên Odoo. |
| OQ-K02 | Tìm kiếm sản phẩm theo tiêu chí nào? (tên, mã SKU, barcode?) | UI tạo đơn | ✅ Đã xác nhận | Tên, SKU, barcode, số serial. |
| OQ-K03 | Khi sản phẩm hết hàng tại location, có cho phép thêm vào đơn không? (cảnh báo hay chặn?) | Logic validate | ✅ Đã xác nhận | Cho phép thêm vào đơn kèm cảnh báo. Nếu có serial thì để trống serial. |
| OQ-K04 | Sản phẩm có ảnh không? Cần hiển thị ảnh trong danh sách sản phẩm không? | UI, cache size | ✅ Đã xác nhận | Có thể bật/tắt tính năng hiển thị ảnh. |
| OQ-K05 | Có cần nhóm/danh mục sản phẩm để lọc khi tạo đơn không? | UI tạo đơn | ✅ Đã xác nhận | Sử dụng danh mục kế thừa từ Odoo (`product.category`). |

---

## Nhóm L — Danh sách & Tìm kiếm đơn hàng

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-L01 | Danh sách đơn hàng filter theo tiêu chí nào? (ngày, trạng thái, nhân viên, khách hàng?) | UI, API query | ✅ Đã xác nhận | Load cache toàn bộ đơn (1 tuần) lên frontend, filter theo tên KH, SĐT, MST, email. |
| OQ-L02 | Lịch sử đơn hiển thị bao xa? (hôm nay, 30 ngày, tất cả?) | Performance, UX | ✅ Đã xác nhận | 1 tuần. |
| OQ-L03 | POS User chỉ thấy đơn của mình hay tất cả đơn trong location? | Phân quyền, query | ✅ Đã xác nhận | Chỉ thấy đơn của mình. |
| OQ-L04 | Khi thu phần còn lại (đặt cọc), tìm đơn theo tiêu chí gì? (tên KH, SĐT, mã đơn?) | Luồng đặt cọc | ✅ Đã xác nhận | Tên KH, SĐT, mã đơn, email, MST. |

---

## Nhóm M — Session & Bảo mật chi tiết

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-M01 | Access token TTL bao lâu? Refresh token TTL bao lâu? | Auth, UX | ✅ Đã xác nhận | Access token: 15 phút. Refresh token: cần xác nhận thêm. |
| OQ-M02 | Khi user bị ADMIN vô hiệu hóa, access token hiện tại có bị invalidate ngay không? | Auth security | ✅ Đã xác nhận | Invalidate ngay lập tức — backend kiểm tra trạng thái user trong DB mỗi request. |
| OQ-M03 | Sau bao lâu không hoạt động thì tự đăng xuất (idle timeout)? | UX, security | ✅ Đã xác nhận | 5 phút. |
| OQ-M04 | Có cần giới hạn số lần đăng nhập sai (brute force protection) không? | Security | ✅ Đã xác nhận | Khóa tài khoản sau 5 lần sai. |

---

## Nhóm N — Xử lý lỗi & Edge Cases

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-N01 | Khi Odoo không phản hồi (timeout/down), hiển thị gì và user làm gì tiếp? | UX, error handling | ✅ Đã xác nhận | Hiển thị thông báo lỗi, đề nghị user chờ và thử lại. |
| OQ-N02 | Khi XML-RPC call thất bại giữa chừng (đã tạo sale.order nhưng picking chưa tạo), xử lý thế nào? | Data integrity | ✅ Đã xác nhận | Báo lỗi, user tạo lại. Dữ liệu lỗi để kế toán xử lý trên Odoo. |
| OQ-N03 | Khi tồn kho cache (5 phút) không khớp với Odoo thực tế lúc validate, flow xử lý thế nào? | Stock validation | ✅ Đã xác nhận | Báo lỗi tại bước validate, user cần kiểm tra lại tồn kho. |
| OQ-N04 | Odoo XML-RPC timeout được set bao lâu? | Performance, infra | ✅ Đã xác nhận | Tối đa 20 giây. |

---

## Nhóm O — Đặt cọc (chi tiết bổ sung)

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-O01 | 1 đơn có thể đặt cọc nhiều lần không? (vd: cọc lần 1 rồi cọc thêm lần 2?) | Luồng cọc, data model | ✅ Đã xác nhận | Có — cho phép nhiều lần thanh toán một phần. |
| OQ-O02 | Đơn đặt cọc có thời hạn hết hiệu lực không? (vd: giữ cọc 30 ngày?) | Business rule | ✅ Đã xác nhận | Không hết hiệu lực, nhưng theo dõi được trạng thái các đơn. |

---

## Nhóm P — Template & Visual Editor

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-P01 | Visual editor là loại gì? WYSIWYG đơn giản (GrapesJS, TinyMCE) hay chỉ là code editor với preview? | Độ phức tạp, frontend | ✅ Đã xác nhận | WYSIWYG. |
| OQ-P02 | Admin có thể preview phiếu trước khi lưu template không? | UX admin | ✅ Đã xác nhận | Preview realtime, có thể bật/tắt. |
| OQ-P03 | Có cần version history cho template (lưu bản trước để rollback) không? | Data model | ✅ Đã xác nhận | Có. |
| OQ-P04 | Khi template bị lỗi HTML/CSS, WeasyPrint fail — cần hiển thị lỗi cụ thể cho Admin không? | Error handling | ✅ Đã xác nhận | Có hiển thị lỗi cụ thể. |

---

## Nhóm Q — UX & Tương tác

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-Q01 | Hành động nào cần dialog xác nhận trước khi thực hiện? (hủy đơn, xóa SP khỏi giỏ, logout?) | UX, an toàn dữ liệu | ✅ Đã xác nhận | Hủy đơn, xóa SP khỏi giỏ, logout, lưu/cập nhật thông tin KH, xác nhận đơn hàng, xác nhận thanh toán. |
| OQ-Q02 | Có cần keyboard shortcuts trên desktop không? (vd: F2 tìm SP, F4 thanh toán, Esc hủy) | UX desktop | ✅ Đã xác nhận | Có. |
| OQ-Q03 | Trên iPad khi nhập số lượng / số tiền, dùng numpad trên màn hình hay bàn phím ảo hệ thống? | UX tablet, component | ✅ Đã xác nhận | Numpad trên màn hình. |
| OQ-Q04 | Khi thêm sản phẩm có serial vào đơn, chọn serial ngay lúc đó hay chọn sau (lúc xác nhận đơn)? | Luồng tạo đơn, UX | ✅ Đã xác nhận | Chọn serial ngay khi thêm SP. Nếu SP không tồn kho → ghi chú `**` trong tên SP. |
| OQ-Q05 | Có cần swipe gesture trên iPad không? (vd: swipe trái để xóa SP khỏi giỏ) | UX tablet | ✅ Đã xác nhận | Có. |

---

## Nhóm R — Hiển thị & Định dạng

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-R01 | Định dạng tiền tệ: `1.000.000 ₫` hay `1,000,000 VND` hay dạng khác? | Toàn bộ UI | ✅ Đã xác nhận | `1.000.000 ₫` |
| OQ-R02 | Định dạng ngày giờ: `dd/MM/yyyy HH:mm` hay dạng khác? | Bảng, chi tiết đơn | ✅ Đã xác nhận | `dd/MM/yyyy HH:mm` |
| OQ-R03 | Product grid: bao nhiêu cột trên Desktop (3, 4, 5?) và trên iPad (3, 4?)? | Layout tạo đơn | ✅ Đã xác nhận | 3 cột (cả Desktop và iPad). |
| OQ-R04 | Tên sản phẩm dài quá thì cắt bao nhiêu ký tự? Có tooltip hiện đầy đủ không? | Product card | ✅ Đã xác nhận | Giới hạn 10 ký tự đầu, có tooltip hiện đầy đủ. |
| OQ-R05 | Font chữ: dùng font hệ thống hay custom font? Kích thước tối thiểu trên iPad? | Typography, accessibility | ✅ Đã xác nhận | Font hệ thống, kích thước theo common practice. |

---

## Nhóm S — Tìm kiếm & Hiệu năng

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-S01 | Tìm kiếm sản phẩm: gõ tối thiểu bao nhiêu ký tự mới bắt đầu tìm? (1, 2, 3?) | UX, performance | ✅ Đã xác nhận | 3 ký tự. |
| OQ-S02 | Ảnh sản phẩm: lấy từ Odoo (`product.product` image field) hay upload riêng? Kích thước tối đa? | Cache, bandwidth | ✅ Đã xác nhận | Upload riêng, lưu trong S3-compatible storage hoặc Azure Blob Storage. |
| OQ-S03 | Danh sách đơn hàng: phân trang (pagination) hay cuộn vô hạn (infinite scroll)? Bao nhiêu dòng/trang? | UX, performance | ✅ Đã xác nhận | Phân trang, 5 đơn hàng/trang. |
| OQ-S04 | Product grid: phân trang hay cuộn vô hạn? Load tất cả SP hay phân batch? | UX, memory | ✅ Đã xác nhận | Cuộn vô hạn (infinite scroll), load theo batch. |
| OQ-S05 | Có cần hiển thị số lượng tồn kho trên product card (tại màn tạo đơn) không? | UI tạo đơn, API call | ✅ Đã xác nhận | Có. |

---

## Nhóm T — Thông báo & Phản hồi

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-T01 | Toast notification hiển thị bao lâu? (2 giây, 3 giây, cho đến khi đóng?) | UX | ✅ Đã xác nhận | 2 giây. |
| OQ-T02 | Có cần âm thanh khi thao tác thành công / thất bại không? | UX, accessibility | ✅ Đã xác nhận | Không. |
| OQ-T03 | Dữ liệu đơn hàng (cache 1 tuần) auto-refresh theo interval hay chỉ khi user vào trang? | Data freshness | ✅ Đã xác nhận | Auto-refresh mỗi 20 phút. |
| OQ-T04 | Khi mất kết nối mạng, hiển thị thông báo dạng nào? (banner cố định, toast, full-screen?) | UX, error handling | ✅ Đã xác nhận | Thông báo mất kết nối toàn màn hình. |

---

## Nhóm U — In ấn & PDF chi tiết

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-U01 | Khổ giấy phiếu in: A4, A5, hay khổ hóa đơn 80mm? | Template design, WeasyPrint | ✅ Đã xác nhận | A4 và A5. Không có máy in receipt. |
| OQ-U02 | Có cần nút "In trực tiếp" (gọi window.print) từ preview, hay chỉ preview/download? | UX frontend | ✅ Đã xác nhận | Không cần in trực tiếp — chỉ preview/download. |
| OQ-U03 | Có cần in nhiều bản (copies) cùng lúc không? | UX, backend | ✅ Đã xác nhận | Không.  |

---

## Nhóm V — Object Storage & Ảnh sản phẩm

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-V01 | Dùng provider nào? (MinIO self-hosted, AWS S3, Azure Blob?) | Infra, chi phí, Docker | ✅ Đã xác nhận | Local storage (lưu trên server, không dùng S3/Azure). |
| OQ-V02 | Ai upload ảnh sản phẩm? ADMIN trong app hay quy trình riêng? | Admin UI, scope | ✅ Đã xác nhận | ADMIN upload trong app. |
| OQ-V03 | Có cần tự động tạo thumbnail cho product card (vd: 150x150px) không? | Performance, storage | ✅ Đã xác nhận | Có — tất cả ảnh upload sẽ được backend resize + crop thành thumbnail phù hợp. |
| OQ-V04 | Kích thước ảnh gốc tối đa cho phép upload? (vd: 2MB, 5MB?) | Validation, UX | ✅ Đã xác nhận | Tối đa 5MB. Backend tự resize + crop sau khi upload. |

---

## Nhóm W — Admin & Quản trị chi tiết

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-W01 | Password policy: độ dài tối thiểu? Có yêu cầu chữ hoa, số, ký tự đặc biệt không? | Security, UX | ✅ Đã xác nhận | Tối thiểu 8 ký tự, bắt buộc chữ hoa + số + ký tự đặc biệt. |
| OQ-W02 | ADMIN có thể reset password cho POS User không? Flow nào? | Admin UI | ✅ Đã xác nhận | ADMIN reset qua email chứa token reset password. |
| OQ-W03 | User bị khóa sau 5 lần sai: tự mở sau bao lâu hay ADMIN phải mở thủ công? | Security, UX | ✅ Đã xác nhận | ADMIN mở thủ công. |
| OQ-W04 | ADMIN có cần xem activity log không? (ai tạo đơn, ai hủy, ai thanh toán, khi nào?) | Admin UI, audit | ✅ Đã xác nhận | Có. |

---

## Nhóm X — Đồng bộ dữ liệu

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-X01 | Khi tạo KH mới từ Mobile POS, sync ngay lên Odoo `res.partner` hay lưu local trước? | Data flow, UX | ✅ Đã xác nhận | Sync ngay lên Odoo khi bấm lưu. KH có MST → dùng VietQR API auto-fill trước khi lưu. |
| OQ-X02 | Khi Odoo cập nhật sản phẩm (giá, tên, trạng thái), Mobile POS biết qua cách nào? (chỉ chờ TTL hết hay ADMIN flush thủ công?) | Cache strategy | ✅ Đã xác nhận | Chờ TTL hết hạn (không cần flush thủ công cho SP). |
| OQ-X03 | Danh mục sản phẩm (`product.category`) cache bao lâu? Có cần flush riêng không? | Cache, UI danh mục | ✅ Đã xác nhận | Cache 8 tiếng, có thể flush riêng. |

---

## Nhóm Y — Keyboard Shortcuts & Phím tắt

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-Y01 | Danh sách keyboard shortcuts cụ thể cần hỗ trợ? (vd: F2 tìm SP, F4 thanh toán, F8 lưu nháp, Esc hủy...) | Frontend, UX | ✅ Đã xác nhận | Gợi ý: `F2` tìm SP, `F3` tìm KH, `F4` thanh toán, `F8` lưu nháp, `F9` xác nhận đơn, `Esc` hủy/đóng dialog, `Del` xóa dòng SP đang chọn. |
| OQ-Y02 | Có cần hiển thị shortcut hints trên UI không? (vd: tooltip hiện "(F2)" trên nút tìm kiếm) | UX | ✅ Đã xác nhận | Có — hiển thị shortcut hint trên tooltip của nút. |

---

## Nhóm Z — Phiếu in bổ sung

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-Z01 | Mỗi loại phiếu chỉ 1 khổ giấy cố định hay user chọn A4/A5 khi in? | Template, UX | ✅ Đã xác nhận | User chọn khi in. Khổ giấy phụ thuộc vào độ dài đơn (số lượng SP). |
| OQ-Z02 | 3 loại phiếu (xác nhận, cọc, thanh toán): mỗi loại dùng A4 hay A5 mặc định? | Template design | ✅ Đã xác nhận | Tùy thuộc vào độ dài đơn hàng — user tự chọn A4/A5 khi in. |

---

## Nhóm AA — Hoá đơn điện tử (Misa)

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-AA01 | Misa API credentials cụ thể: `client_id`, `client_secret`, `company_tax_code`, và base URL cho môi trường staging / production? | B8 — tích hợp Misa, config | ⏳ Chờ | |
| OQ-AA02 | Đơn COD: thời điểm tạo hoá đơn Misa là lúc xác nhận đơn hay sau khi đơn vị vận chuyển xác nhận đã thu tiền? | A6, luồng COD, thứ tự sự kiện | ⏳ Chờ | |
| OQ-AA03 | Nếu khách hàng không có MST, có vẫn tạo hoá đơn Misa không? Dùng thông tin gì cho trường người mua? | A6, data model hoá đơn | ⏳ Chờ | |
| OQ-AA04 | Email kế toán thuế nhận thông báo là địa chỉ cố định (config) hay một role user trong hệ thống? Có thể cấu hình nhiều người nhận không? | B8, admin config, phân quyền | ⏳ Chờ | |
| OQ-AA05 | Hoá đơn Misa có cần ghi reference đến mã đơn hàng (`sale.order` name) để đối chiếu không? | Data model hoá đơn, kế toán | ⏳ Chờ | |
| OQ-AA06 | Thuế VAT trên hoá đơn Misa ghi gì khi POS hiện tại chưa tính VAT? (0%, để trống, hay một giá trị mặc định?) | Nội dung hoá đơn, tuân thủ thuế | ⏳ Chờ | |
| OQ-AA07 | Sau khi kế toán phát hành hoá đơn chính thức trên Misa, Mobile POS có cần cập nhật trạng thái đơn hàng tương ứng không? | Đồng bộ trạng thái, UX | ⏳ Chờ | |

---

## Nhóm AB — Giao hàng & COD

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-AB01 | Đơn vị vận chuyển cụ thể nào sẽ tích hợp? (GHN, GHTK, Viettel Post, J&T, hay đơn vị nội bộ?) | B9 — tích hợp carrier, API design | ⏳ Chờ | |
| OQ-AB02 | Phí COD (phí thu hộ) do công ty hay khách hàng chịu? Có cộng vào tổng tiền đơn hiển thị trên POS không? | Tính tiền, UI thanh toán | ⏳ Chờ | |
| OQ-AB03 | Các trường bắt buộc khi tạo vận đơn COD? (tên người nhận, SĐT, địa chỉ đầy đủ, quận/huyện, tỉnh/thành?) | Form COD trên UI, API payload | ⏳ Chờ | |
| OQ-AB04 | Giao hàng thất bại (khách vắng nhà, từ chối nhận hàng) — Mobile POS xử lý thế nào? Có cần hiển thị trạng thái và cho phép tạo lại vận đơn không? | Luồng COD, UX, edge case | ⏳ Chờ | |
| OQ-AB05 | Đơn COD có thể kết hợp với đặt cọc không? (vd: cọc 30% trước tại cửa hàng, 70% còn lại thu hộ qua COD) | Luồng thanh toán A2, data model | ⏳ Chờ | |
| OQ-AB06 | Carrier có hỗ trợ webhook xác nhận thu tiền không, hay toàn bộ sẽ dùng fallback xác nhận thủ công? | B9, độ phức tạp tích hợp | ⏳ Chờ | |
| OQ-AB07 | Quy trình và tần suất đối soát tiền thu hộ giữa công ty và đơn vị vận chuyển? Mobile POS có cần hỗ trợ màn hình đối soát không? | Nghiệp vụ kế toán, scope | ⏳ Chờ | |

---

## Nhóm AC — Payment Gateway API (VNPay / MoMo / VietQR Payment)

| ID | Câu hỏi | Ảnh hưởng đến | Trạng thái | Câu trả lời |
|---|---|---|---|---|
| OQ-AC01 | VNPay credentials: `vnp_TmnCode`, `vnp_HashSecret`, endpoint môi trường sandbox và production? | B10 — tích hợp VNPay | ⏳ Chờ | |
| OQ-AC02 | MoMo credentials: `partnerCode`, `accessKey`, `secretKey`, endpoint môi trường sandbox và production? | B11 — tích hợp MoMo | ⏳ Chờ | |
| OQ-AC03 | VietQR Payment: ngân hàng đối tác cụ thể nào? Có dùng API ngân hàng để sinh QR hay sinh tĩnh theo chuẩn NAPAS? | B12 — tích hợp VietQR | ⏳ Chờ | |
| OQ-AC04 | VietQR Payment: ngân hàng đối tác có hỗ trợ webhook xác nhận giao dịch không? Nếu không, fallback thủ công có được chấp nhận? | B12, UX xác nhận thanh toán | ⏳ Chờ | |
| OQ-AC05 | Khi thanh toán VNPay / MoMo thất bại giữa chừng (user đóng trình duyệt, hết timeout), đơn hàng xử lý thế nào? Cho phép thử lại hay phải tạo lại đơn? | Luồng thanh toán, UX, data model | ⏳ Chờ | |
| OQ-AC06 | Có cần hiển thị trạng thái real-time cho user khi chờ IPN callback không? (vd: polling hoặc WebSocket để biết thanh toán thành công) | FE UX, backend | ⏳ Chờ | |
| OQ-AC07 | Một đơn hàng có thể kết hợp thanh toán VNPay/MoMo/VietQR với tiền mặt (multi-method) không, hay mỗi gateway chỉ dùng cho toàn bộ số tiền? | Logic multi-method, data model | ⏳ Chờ | |
