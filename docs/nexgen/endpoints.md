---
id: endpoints
title: Endpoints
sidebar_label: Endpoints
sidebar_position: 2
---

## 4) Endpoints (Staging Base)

Staging host: `https://nexgen-stg.dbkl.gov.my`

Common query params: `?ApiSecret=...&CheckSum=...`

Header: `ApiKey: ...`

---

### 4.1 Collections

#### Create Collection — POST `/api/v1/collection/create`

Body (form-data): `fieldName`, `fieldDescription`, `fieldCurrency` (rm/usd), `fieldStatus` (active/inactive)

Response: `{ code, name, description, currency, status }`

Validations return `422` with field errors.

#### Get Collection List — GET `/api/v1/collection/get/list`

Returns an array of `{ code, name, description, currency, status }`.

#### Get Collection Data — GET `/api/v1/collection/get/data/{collection_code}`

Returns single collection.

#### Get Collection Data (with Bills) — GET `/api/v1/collection/get/data/{collection_code}/billing`

Returns collection plus `bill_list[]` with each bill’s `code`, `status`, `status_code`, `amount`, `due_date`, `payer_*`....

#### Switch Collection Status — PUT `/api/v1/collection/switch/status/data/{collection_code}`

Form field: `fieldStatus` (active/inactive). Toggles status.

---

### 4.2 Billing

#### Create Billing — POST `/api/v1/billing/create/{collection_code}`

Body (form-data)

- Required: `fieldName`, `fieldEmail`, `fieldPhone`, `fieldAmount`, `fieldPaymentDescription`, `fieldCallbackUrl`, `fieldMerchantTransactionID`
- Optional: `fieldDueDate`, `fieldRedirectUrl`, `fieldDreamsReceiptNumber`, `fieldMerchantBankAccount`, `fieldExternalReferenceLabel1/Value1`, `fieldExternalReferenceLabel2/Value2`

Returns bill info + `payment_url` to open for the payer.

#### Get Billing Data — GET `/api/v1billing/get/data/{collection_code}/{bill_code}`

Returns bill details; after payment includes `payment_method_accepted` and `payment_method_detail` (OBW: `transaction_id`, `customer_bank`, etc.; MPGS: `order_id`/`reference_id`).

#### Update Receipt Number (DREAMS) — PUT `/api/v1/billing/update/receipt/data/{collection_code}/{bill_code}`

Form/query: `fieldDreamsReceipt` — push DREAMS receipt number into NexGEN once confirmed.

---

### 6) Status & Error Handling

#### 6.1 Bill/Payment status

```
pending (0), completed (1), failed (2)
```

Used across list/detail/callback/redirect.

#### 6.2 HTTP status codes

- 200 OK success
- 401 Unauthorized invalid/missing keys
- 404 Not Found
- 422 Unprocessable Content (field validation errors)
- 500 Internal Server Error (server‑side)

---

### 8) Validation Rules (high‑impact fields)

- `fieldPhone`: starts with 6 then valid MY mobile pattern (01); 9–10 digits total.
- `fieldAmount`: 1.00–100,000.00; up to 2 decimals.
- `fieldPaymentDescription`: 5–40 chars; alnum/space/hyphen.
- `fieldRedirectUrl` / `fieldCallbackUrl`: must be valid URLs; callback cannot be localhost.

---

### 9) Error Responses — What to Expect

- `401` Unauthorized: keys/checksum/team not found.
- `422` Unprocessable Content: per‑field validation messages.
- `404/500`: resource missing / server error. Log raw gateway payload for audits.

