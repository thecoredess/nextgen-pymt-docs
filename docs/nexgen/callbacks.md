---
id: callbacks
title: Callbacks & Redirects
sidebar_label: Callbacks & Redirects
sidebar_position: 3
---

## 5) Callbacks & Redirects

### 5.1 OBW Callback (server‑to‑server, POST JSON)

NexGEN POSTs to your Callback URL (cannot be localhost) with bill, payer, amounts, and method details (e.g., OBW `transaction_id`, `customer_bank`, `customer_bank_type`). Your server should validate/process, update status, and return `200 OK`.

Sample OBW payload fields: `code`, `status`, `status_code`, `amount`, `payment_description`, `payer_*`, `payment_method_accepted`, `payment_method_detail{ transaction_date, transaction_id, reference_id, customer_bank, customer_bank_type }`.

### 5.2 MPGS Callback (server‑to‑server, POST JSON)

Similar to OBW, but `payment_method_detail` returns `order_id` and `reference_id`.

### 5.3 Redirect (browser, GET)

After callback is processed successfully, the user is redirected to your Redirect URL with minimal GET params for UI display: `code`, `status`, `status_code`, `payer_name/email/phone`.

Amounts are not included — render amounts from your own stored data if needed.

---

### 10) Best Practices & Compliance

- Always trust the Callback (server‑to‑server) as source of truth. Use Redirect only for UX.
- Idempotency: guard your callback handler to avoid double‑posting receipts.
- Security: verify your own bill record (amount, payer) before marking paid; compute and log checksum per request.
- PDPA: display consent & include PDPA notice content in your portal (Malay & English).

