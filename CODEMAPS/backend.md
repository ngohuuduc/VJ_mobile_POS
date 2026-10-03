<!-- Generated: 2026-10-03 | Files scanned: backend/app (~60) | Token estimate: ~950 -->
# Backend (FastAPI · `backend/app`)

Entry: `app/main.py` (routers under `/api/v1`, global rate limit, startup: migrations, Odoo auth check, seed admin, scheduler).

## Routes → module (`app/api/*`)
```
auth            POST login(break-glass) refresh logout change/forgot/reset · GET me prefetch-status
auth_microsoft  GET start callback · POST exchange
customers       GET search {id}/profile|credits|held-orders|payments mst/{tax} · POST · PUT {id}
products        GET / categories brands {id}/serials
orders          POST / · GET / {id} · POST {id}/warehouse apply-credit release-deposit deliver cancel
                PATCH {id}/pickup-date {id}/customer
payments        POST /orders/{id}/payments · GET {id}/vietqr · {id}/vietqr/{rid}/status · POST …/cancel
                GET /payment-methods/active
order_attachments POST|GET /orders/{id}/attachments   (photo → ir.attachment on invoice)
customer_deposits POST /customer-deposits
inventory       GET stock · POST refresh       backorders GET / · POST {id}/validate|cancel
drafts          CRUD /drafts                   print GET /print/{receipt_type} (WeasyPrint PDF)
sources         GET /sources                   employees GET /employees/search
webhooks        POST /webhooks/sepay (API key, dedupe)
admin           users(list/get/put/status/sync-group) employees warehouses locations activity-log
                templates(CRUD, preview, versions, rollback, seed) cache(stats, flush)
                products/{id}/image payment-methods(CRUD, order, journals) vietqr/banks
store_configs   GET|PUT /admin/store-configs/{warehouse_id} · journals
```

## Services (`app/services/*`, lines)
order_fulfillment 748 (invoice create/finalize, credits, links) · vietqr_matching 489 · entra 454 ·
promotions 444 · odoo_client 442 (JSON-RPC, retries, company scope) · store_config 418 ·
customer_ledger 404 · tax_lookup 365 · sepay_webhook 345 · vietqr 295 · pg_cache 263 ·
entra_users 221 · posthog_client 192 · odoo_audit_note 169 · employees 162 · order_attachments 155 · lots 154

## Cross-cutting (`app/core`)
config.py (Settings) · dependencies.py (CurrentPosUser/Admin/AnyUser, NO_STORE_ASSIGNED gate) ·
security.py (JWT) · authorization.py · audit.py (`write_audit`) · rate_limit.py · errors.py

## Jobs (`app/jobs/scheduler.py`)
draft_cleanup · cache_cleanup · invoice_finalize (5 min) · vietqr_match (2 min)

## Tests
`backend/tests` (~700 pytest, fake Odoo: `_fake_odoo_ledger.py`, `_entra.py`). Run with private DB via `TEST_DATABASE_URL`.
