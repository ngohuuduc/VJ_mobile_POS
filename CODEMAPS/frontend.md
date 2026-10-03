<!-- Generated: 2026-10-03 | Files scanned: frontend/src (~185) | Token estimate: ~800 -->
# Frontend (Vue 3.5 · Quasar 2 · Pinia · `frontend/src`)

Boot: `boot/http.ts` (apiFetch, token refresh) · `auth-guard.ts` · `idle-watcher.ts` · `posthog.ts` · `theme.ts`.

## Page tree (`router/routes.ts`)
```
/login            LoginPage (Microsoft button + "Quản trị dự phòng")
/auth/callback    AuthCallbackPage (exchange one-time code)
/no-store         NoStorePage            /prefetch PrefetchLoader
/ MainLayout (sidebar ≥1280 · bottom tabs + More on mobile)
  orders/new      OrderNewPage (cart, serials, customer, PaymentDialog)
  orders          OrderListPage          orders/:id OrderDetailPage (pay, deliver, VietQR pending, photos)
  inventory       InventoryPage          backorders BackorderPage
  customer-deposit CustomerDepositPage (Thu tiền: held orders, payments)
  settings        SettingsPage
  admin/* (desktopOnly): users UserListPage+UserEditDialog · templates TemplateEditorPage ·
                  cache CachePage · payment-methods PaymentMethodsPage · store-configs StoreConfigPage
```

## Key components
PaymentDialog → VietQrPanel / VietQrWatchDialog (polling `useVietQrWatch` + `utils/vietqrPolling`) ·
PaymentProofUploader · SerialPickerDialog · BarcodeScanDialog (BarcodeDetector / @zxing) ·
CustomerDialog / CustomerProfileDialog · DeliverDialog · PickupDateDialog · ReleaseDepositDialog ·
CartPanel · ProductGrid/ProductCard · OrderStatusBadge · CommissionEmployeePicker.

## State (Pinia `stores/`)
auth (tokens, user, active location) · cart · products · customers · orders · drafts · inventory ·
backorders · sources · store-setup · admin-users · admin-templates · admin-store-configs.

## API layer (`services/`)
ordersApi · customersApi · productsApi · storeConfigsApi (vietqr) · adminApi · authApi · printApi ·
inventoryApi · backordersApi · employeesApi · sourcesApi. Types in `types/api.ts`.

## Utils of note
analytics.ts (PII rules) · authFlow/authErrors · orderFlows · customerLedger · paymentDialog · storeSetup ·
printPaper · reorder · vietqrAccount · paymentProof. Tests: `frontend/test` (vitest, ~570).
