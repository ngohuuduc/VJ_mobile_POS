<!-- Generated: 2026-10-03 | Files scanned: backend/app/models, alembic | Token estimate: ~650 -->
# Data

## POS Postgres tables (`backend/app/models`)
```
pos_users            Entra users (entra_oid, roles, warehouses, location_ids, hr_employee_id/auto)
admin_users          break-glass admin (password)
refresh_tokens       rotation, remember-me
auth_flow_states     OIDC state/nonce/PKCE + one-time exchange codes
password_reset_tokens legacy
payment_method_configs code, label, sequence, vietqr_enabled, vietqr_bank_bin/account_no/name, memo_suffix
store_configs        per warehouse: invoice journal
store_payment_methods warehouse × method → payment journal (cash/bank)
order_payment_links  POS payment ↔ sale.order (draft-invoice credits; source=payment|vietqr_webhook…)
vietqr_requests      pending|bank_confirmed|matched|cancelled|expired per (order, method, amount)
sepay_transactions   webhook rows (sepay_id unique)
order_attachments    photo index → ir.attachment (rename on invoice post)
print_templates (+_versions) Jinja/WeasyPrint templates, paper_size
draft_orders         offline/held carts
audit_logs           write_audit trail
pg_cache (raw table) key/value cache with expires_at, updated_at
```

## Migrations (`alembic/versions`)
0001 initial · 0002 payment_method_configs · 0003 user allowed methods · 0004 username ci ·
0005 order_payment_links · 0006 store_configs · 0007 vietqr flag · 0008 template paper/cache ts ·
0009 vietqr account · 0010 vietqr requests + attachments · 0011 Entra login · 0012 SePay webhook · 0013 memo prefix → suffix.
Applied automatically at backend startup.

## Odoo models used (by frequency)
sale.order · stock.picking · account.payment · account.move(.line) · res.partner · stock.warehouse ·
product.product · stock.location · stock.quant/move/move.line · hr.employee · stock.production.lot ·
mail.activity · account.journal · product.pricelist(.item) · ir.attachment · account.payment.register ·
account.partial.reconcile · coupon.program/coupon · utm.source · product.brand.
