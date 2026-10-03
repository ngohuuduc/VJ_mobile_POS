<!-- Generated: 2026-10-03 | Files scanned: 331 code | Token estimate: ~550 -->
# Architecture

Monorepo `VJ_POS_Platform` (git submodules): `backend/` FastAPI · `frontend/` Quasar PWA · `docs/` · `planning/` (main repo) · `odoo_modules/`.

```
 Browser/PWA (Vue3+Quasar, :9000)
     │  /api/v1 (JWT bearer, refresh rotation)
     ▼
 FastAPI (:8008, uvicorn, APScheduler) ──► Postgres (POS state: users, configs, links, cache)
     │ JSON-RPC (posdev, MAIL_QUIET_CTX)        ▲
     ▼                                          │
 Odoo 14 (sale/stock/account/hr) ◄── SePay webhook module (creates account.payment)
     ▲                                          │
 Microsoft Entra ID (OIDC+PKCE, Graph)    SePay ──► POST /api/v1/webhooks/sepay
 PostHog (US) analytics · api.vietqr.io bank list · escodata MST lookup
```

## Boundaries
- Odoo = system of record (orders, pickings, invoices, payments, partners, employees).
- POS Postgres = auth, store/payment config, VietQR requests, payment↔order links, attachments index, audit, pg_cache.
- Backend never stores card data; VietQR payload built locally (EMVCo, CRC16).

## Key flows
- Login: `/auth/microsoft/start → Entra → /callback → one-time code → /exchange → JWT`. Group gate (POS group / Admin group). Break-glass admin = password.
- Sale: OrderNew → `POST /orders` (sale.order confirm, draft invoice) → payments → deliver → `invoice_finalize` job posts invoice when delivered + fully paid (`INVOICE_POST_MODE=on_delivery`).
- VietQR: `GET /orders/{id}/vietqr` (pending request) → bank transfer → SePay webhook marks `bank_confirmed` → sweep links Odoo payment → finalize.
- Every POS write → Odoo internal note (`odoo_audit_note`).

## Deploy
`.deploy_staging/docker-compose.yml` (backend+frontend images from registry.parainsight.com, host NGINX, host Postgres). CI: `.github/workflows/deploy.yml` builds on push to `main`.
