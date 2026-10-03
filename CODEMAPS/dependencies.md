<!-- Generated: 2026-10-03 | Files scanned: pyproject.toml, package.json | Token estimate: ~400 -->
# Dependencies

## External services
- Odoo 14 (JSON-RPC, user `posdev`) — system of record.
- Microsoft Entra ID + Graph (login, group gate, group sync; app perms GroupMember.Read.All, User.Read.All).
- SePay (bank-transfer webhooks) · Odoo module `parainsight_sepay_webhook` creates account.payment.
- api.vietqr.io (bank BIN list, bundled fallback) · escodata.net (MST lookup, VietQR fallback).
- PostHog US cloud (analytics + session recording, PII masked).
- Container registry registry.parainsight.com (CI images).

## Backend (Python, uv)
fastapi · uvicorn · sqlalchemy[asyncio] + asyncpg · alembic · pydantic(-settings) · python-jose · passlib[bcrypt] ·
python-multipart · apscheduler · weasyprint + jinja2 (PDF) · pillow · httpx · qrcode · posthog.

## Frontend (pnpm)
vue · vue-router · pinia · quasar + @quasar/extras · @zxing/browser/@zxing/library (barcode) · posthog-js.
Dev: vite (@quasar/app-vite) · vitest · vue-tsc · eslint.
