# Kính ngắm — canvas mockup (Claude Design)

Bản lưu của canvas Claude Design: https://claude.ai/artifact/LCYb31AqiycawH8Jhny6kw

23 màn hình × 3 thiết bị (Mobile 390×844, iPad 1180×820 ngang, Desktop 1440×900), giao diện sáng Kính ngắm.

- `project/canvas.json` — chỉ mục canvas: trang, vị trí artboard, tiêu đề nhóm.
- `project/Main.dc.html` — trang Tổng quan, link tới từng màn.
- `project/{M,T,D}NN-Slug.dc.html` — artboard theo thiết bị (M = Mobile, T = iPad, D = Desktop).
- `SPEC.md` — mô tả chung đã dùng để vẽ (token màu, font, khung giao diện, dữ liệu mẫu, luật nghiệp vụ).
- `vjshop-logo.svg` — logo; trên canvas nó là asset `/_blob/ca4bc8854f059ad7465e93434a0ee522`.

Các file `.dc.html` chỉ chạy trong trình sửa Claude Design (cần `support.js` của canvas); mở trực tiếp bằng trình duyệt sẽ không hiển thị đúng.
Canvas trên claude.ai là bản gốc để chỉnh; thư mục này được cập nhật lại mỗi khi canvas thay đổi.
