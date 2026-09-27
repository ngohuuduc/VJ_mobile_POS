# VJ POS "Kính ngắm" — full-screen mockup spec (shared by all artboard agents)

Canvas: Claude Design artifact. You only WRITE files; the coordinator publishes. Never call the Artifact tool.
Root folder: /tmp/claude-1000/-home-duc-github-VJ-POS-Platform/29ad48e0-9bcf-435d-8f2b-12a3fb794ee1/scratchpad/design
Write each artboard to `<root>/project/<File>.dc.html`. Do NOT touch canvas.json, Main.dc.html or other agents' files.

## Format (strict — each rule fails silently)
Read `/tmp/claude-1000/-home-duc-github-VJ-POS-Platform/29ad48e0-9bcf-435d-8f2b-12a3fb794ee1/scratchpad/artifact-files/9b7a4835-a1a3-41b4-bd55-40bd2fc60cf8/artifact-type/reference/format.md` first, and copy the head/helmet/script structure of `<root>/project/Main.dc.html`.
- `<html lang="vi">`, own `<title>`, head line `<script src="./support.js"></script>` EXACTLY.
- Everything inside `<x-dc>`; `<helmet>` holds ONLY the Google Fonts `<link>` (same as Main) and a `<style>` with `body{margin:0;...}` + `a`/`a:hover` colours + at most tiny resets. All design styling INLINE `style="…"`.
- Root element: FIXED size = board size, and the same `$preview` in `data-props`. Phone 390×844, iPad 1180×820 (landscape), Desktop 1440×900. Root gets `overflow: hidden`.
- Close every element, quote every attribute. Flex/grid + gap for layout; grids as `repeat(N, minmax(0, 1fr))`.
- `{{hole}}` = dotted lookup only. Always include `<script type="text/x-dc" data-dc-script data-props='…'>` with `class Component extends DCLogic { renderVals(){…} }` (classic JS). No innerHTML/appendChild. Copy text literal in markup (repeated rows may use `<sc-for>` over data in renderVals; set hint-placeholder-count).
- data-props JSON is inside a single-quoted attribute: escape `&` as `&amp;`, `'` as `&#39;`.
- No network except the fonts link and the logo `/_blob/ca4bc8854f059ad7465e93434a0ee522` (VJShop.vn logo: `<img src="/_blob/ca4bc8854f059ad7465e93434a0ee522" alt="VJShop.vn">`, ~24px tall in headers, 44px on login). Icons: inline stroke SVG (24 viewBox, stroke currentColor, width 1.75–2), never emoji. No iframe/object/embed, no global keydown.
- Real `<button>`, `<a href>`, `<input>` + `<label>`; aria-label on icon-only buttons; touch targets ≥44px; text contrast ≥4.5:1.
- Prototype nav: `<a href="M10-Orders.dc.html">` jumps to that artboard in Play. Link nav items (bottom bar / sidebar / tabs / back / close / main CTAs) to the SAME device's artboard (M… Mobile, T… iPad, D… Desktop). Style the `<a>` itself as the button (no button inside a link).
- Tweaks: at most 1–2 real levers per board (e.g. a `"state"` enum like normal/empty/error), or none. No copy tweaks.

## File names (NN-Slug)
01-Login 02-Prefetch 03-OrderNew 04-Cart 05-Serial 06-CustomerPick 07-CustomerProfile 08-Payment 09-Success 10-Orders 11-OrderDetail 12-Deliver 13-ReleaseDeposit 14-PickupDate 15-Deposit 16-Backorder 17-Inventory 18-Settings 19-Users 20-UserForm 21-Templates 22-Cache 23-PaymentMethods — prefixed M/T/D, e.g. `T11-OrderDetail.dc.html`.
Nav targets: Tạo đơn → 03-OrderNew, Đơn hàng → 10-Orders, Tồn kho → 17-Inventory, Backorder → 16-Backorder, Thu cọc → 15-Deposit, Cài đặt → 18-Settings, Quản lý User → 19-Users, Template → 21-Templates, Cache → 22-Cache, Phương thức TT → 23-PaymentMethods. Overview board: `Main.dc.html`.

## Tokens (Kính ngắm, light only — exact values)
paper #EEF0EE (page bg, inputs, pressed rows) · surface #FFFFFF · ink #1A1F24 (text, selection fill, focus) · ink-2 #4E5A55 · ink-3 #66716C (white bg only) · line #D6DBD8 · line-strong #9AA5A0 (control borders) · amber #F0A500 (MAIN ACTION FILL ONLY; text on it = ink) · amber-ink #8A5A00 (amber-toned text) · strip #1A1F24 / strip-text #F2F4F2 / strip-muted #A9B3AE (checkout strip, toasts) · ok #17663F on #E1F0E6 · warn #7A4B00 on #FCEFCF · bad #A1231B on #FBE4E1 · info #35607F · scrim rgba(26,31,36,.48) · shadow-pop 0 10px 28px rgba(26,31,36,.16)
Radii: 8 chips/small buttons, 10 buttons/fields, 14 cards/dialogs. Fonts: "Barlow" text (400/500/600/700); "Barlow Semi Condensed" (500/600/700) for numbers, prices, order codes, SKUs, titles. Sentence case, no uppercase labels. Base text 15–16px, labels 13–14px (never below 12).
Money `32.190.000 ₫` (dot thousands, ₫ after). Dates `26/09/2026`, times `14:05`.

## Shell per device (current app)
- Header (all): white, 1px bottom line #D6DBD8, 56px. Left: menu button (hamburger, 44px) + VJShop logo. Right: location button "23CTO/Stock ▾" (outlined 36px, radius 8, Semi Condensed 600, warehouse icon), online indicator (green #17663F dot + "Online" on desktop/iPad; dot only on phone), avatar circle ink fill white "H" (user huongnt · Nguyễn Thị Hường, role Quản trị).
- Desktop 1440: left sidebar 220px white, border-right line, under header. Group label "Bán hàng" (13px ink-3): Tạo đơn hàng, Đơn hàng, Kiểm tra tồn kho, Backorder & Nháp, Thu cọc trước. Group "Quản trị": Quản lý User, Quản lý Template, Cache Management, Phương thức thanh toán. Then "Cài đặt". Items 44px with stroke icon; active = paper bg + 3px ink bar left + 600 weight. Footer caption "VJShop POS v0.0.1".
- iPad 1180: no permanent sidebar; under header a 48px tab row (white, bottom line): Tạo đơn · Đơn hàng · Tồn kho · Backorder · Thu cọc (active = ink 600 + 2px ink underline, others ink-2). Menu button = overlay drawer with admin items + Cài đặt.
- Phone 390: header, content, bottom nav 64px white top line, 5 items (icon + 12px label): Tạo đơn, Đơn hàng, Tồn kho, Backorder, Thu cọc; active ink, others ink-3. Drawer holds only admin items + Cài đặt.
- Dialogs: phone = FULL SCREEN (56px header: close ✕ 44px + title, content scroll, sticky footer actions). iPad/desktop = centred card (radius 14, shadow-pop, white header with title + close) over the page dimmed by scrim — draw the underlying page (simplified) behind the scrim.

## Order screen reference (03)
Catalog: search field 44px on paper bg "Tìm SKU / Tên SP / Quét mã vạch..." with search icon; brand chips row (36px, radius 8, 1px line-strong; selected = ink fill, white text): Tất cả, DJI, Sony, Fujifilm, Tamron, Canon, K&F Concept, Sigma, Khác (exact order). Product rows flat with hairlines: name 16px 500, SKU (Semi Condensed ink-3), stock pill ("Còn 12" ok / "Còn 2" warn / "Đặt trước" bad text + bad border on white), price right (Semi Condensed 600). Desktop list has column header row: Sản phẩm · SKU · Tồn kho · Giá.
Cart pane (desktop right 400px, iPad right 380px; phone = full-screen sheet, board 04): header "Giỏ hàng · 2 món", customer button ("Chọn khách hàng" or selected name + small "5 KM" button), "Nguồn" select, lines (name, serial line e.g. "Serial: S01-5521A7IV", qty stepper, unit price with dashed underline = editable, line total, delete icon button), totals "Tổng cộng, 2 món" + big Semi Condensed total, "Hiện Numpad" toggle, bottom row ghost "Lưu nháp" + amber "Xác nhận". Phone order screen: dark strip above bottom nav: "2 món, xem giỏ" + total (strip-text) + amber button "Giỏ hàng".

## Sample data (use these; no lorem ipsum)
Products: "Sony Alpha A7 IV (Body)" SP006401 Còn 4 52.990.000 ₫ · "DJI Osmo Pocket 3 Creator Combo" SP006312 Còn 12 14.490.000 ₫ · "Fujifilm X-T5 Kit XF 16-80mm" SP006118 Còn 2 51.990.000 ₫ · "Tamron 28-75mm F2.8 Di III VXD G2 for Sony E" SP005920 Còn 7 21.490.000 ₫ · "Canon EOS R6 Mark II (Body)" SP006050 Đặt trước 55.990.000 ₫ · "Sigma 24-70mm F2.8 DG DN II Art" SP006233 Còn 3 28.990.000 ₫ · "K&F Concept Nano-X Black Mist 1/4 67mm" SP004410 Còn 25 690.000 ₫ · "Aputure LS 600d Pro (V-mount)" SP006222 Đặt trước 48.890.000 ₫.
Serials: S01-5521A7IV, S01-5522A7IV, S01-5530A7IV. Customers: "Vũ Thị Lệ Phương" 0350 978 833 · "Trần Minh Khoa" 0903 112 457 · "Công ty TNHH Ảnh Việt" MST 0312345678 · "Lê Hoàng Anh" 0912 345 678. Orders: S04512, S04511, S04498. Warehouses: "23CTO/Stock" (VJS Hệ thống), "BMT/Stock" (VJS Ban Mê Thuột). Promotions: "Giảm giá đơn hàng 200k" GG200 · "Khách hàng review giảm 100K" RV100K · "Giảm giá 2%" GG2%VJS · personal coupon 627145344932627426 (Mã riêng). Payment methods: Tiền mặt, Chuyển khoản, Quẹt thẻ (POS), Trả góp.

## Business rules to reflect
- Invoice stays DRAFT at confirm; posted only when delivered AND fully paid (badge "Hóa đơn nháp" warn; "Đã hạch toán" ok).
- Order detail: status, customer, warehouse, lines with serials, payments ("Đặt cọc 10.000.000 ₫ · Chuyển khoản · 20/09"), Còn phải thu, invoice state, delivery state (Chờ giao / Đã giao), card "Hoạt động cần làm" (Hẹn lấy hàng 28/09/2026 + "Đổi ngày"), actions: amber "Giao hàng", "Thu tiền", "In", "Xem profile", and "Chuyển cọc" (only when the order was cancelled in Odoo and has a deposit).
- Deliver: pickings, per-line serial (prefilled from sold serial, editable), note "Hóa đơn sẽ hạch toán theo ngày giao khi đã thu đủ"; result variants posted / "Đã giao, còn thu 12.000.000 ₫ — hóa đơn vẫn nháp".
- Chuyển cọc: cancelled order with deposit; deposit payments + total; choice "Áp vào đơn mới của cùng khách" (pick S04512) or "Giữ làm công nợ của khách"; note "Hoàn tiền do kế toán thực hiện trên Odoo".
- Đổi ngày lấy hàng: current activity (Hẹn khách lấy hàng, người phụ trách, hạn 28/09/2026), date picker (from today, max 365 ngày), lý do, Lưu.
- Order list: date range (Từ ngày – Đến ngày; default 7 ngày gần nhất; POS tối đa 92 ngày), search, status chips; columns Mã đơn, Ngày, Khách, Tổng, Đã thu, Trạng thái, Hóa đơn (Nháp / Đã hạch toán), Hẹn lấy. Phone = cards.
- Success: big code "S04512"; heading variants "Đã thanh toán đủ" / "Đã ghi nhận đặt cọc" / "Đã tạo đơn, chưa thu tiền" / warning "Đã tạo đơn, cần kiểm tra thanh toán"; rows Tổng đơn, Đã thu, Còn phải thu, Tiền thối, Kho; ghost "In hóa đơn" + amber "Tạo đơn mới"; link "Xem chi tiết đơn".
- Customer profile: info grid (SĐT, Email, MST, Ngày sinh, Bảng giá, Địa chỉ); "Khuyến mãi đang áp dụng (6)" cards: name, reward in amber-ink ("Giảm 200.000 ₫ trên tổng đơn"), condition bullets, validity ("Không giới hạn thời gian" / "Đến 30/09/2026" + warn tag "Sắp hết hạn"), code chip + copy button, company caption, badge "Mã riêng" for coupons. Read-only (from Odoo).
- Payment: amount due big; method segmented (Tiền mặt / Chuyển khoản / Quẹt thẻ / Trả góp); amount input + quick amounts; split payments list; "Dùng cọc/công nợ của khách" (khả dụng 10.000.000 ₫); change due; ghost "Xác nhận đơn (chưa thu)" + amber "Thu 42.990.000 ₫".
- Delete cart line: no confirm dialog; dark toast "Đã xoá 1 dòng · Hoàn tác".
- Light theme only; Settings has no theme option (show product images toggle, default print size A4/A5/K80, account info, đổi mật khẩu, đăng xuất, version).

Quality bar: looks like the real shipping POS — dense but calm, flat, hairline dividers, ink + one amber action per view. Each device layout genuinely adapted (not scaled). Same shell across all boards.
