---
id: examples
title: Implementation Examples
sidebar_label: Examples (Laravel & Node)
sidebar_position: 4
---

## 7) Implementation — Code Examples

### 7.1 Laravel 12 (GuzzleHttp)

Config (`.env`)

```dotenv
NEXGEN_BASE=https://nexgen-stg.dbkl.gov.my
NEXGEN_API_KEY=your_api_key
NEXGEN_API_SECRET=your_api_secret
NEXGEN_MERCHANT_CODE=YOUR_MERCHANT_CODE
NEXGEN_MERCHANT_NAME=YOUR_MERCHANT_NAME
CALLBACK_URL=https://yourapp.com/api/nexgen/callback
REDIRECT_URL=https://yourapp.com/payments/result
```

Helper: checksum

```php
function nexgenChecksum(): string {
    $str = implode('|', [
        env('NEXGEN_MERCHANT_CODE'),
        env('NEXGEN_MERCHANT_NAME'),
        env('NEXGEN_API_KEY'),
        env('NEXGEN_API_SECRET'),
    ]);
    return hash('sha256', $str);
}
```

Create Collection

```php
use GuzzleHttp\Client;

$client = new Client(['base_uri' => env('NEXGEN_BASE')]);

$res = $client->post('/api/v1/collection/create', [
  'headers' => ['ApiKey' => env('NEXGEN_API_KEY')],
  'query'   => ['ApiSecret' => env('NEXGEN_API_SECRET'), 'CheckSum' => nexgenChecksum()],
  'multipart' => [
    ['name'=>'fieldName','contents'=>'Parking 2025'],
    ['name'=>'fieldDescription','contents'=>'City parking'],
    ['name'=>'fieldCurrency','contents'=>'rm'],
    ['name'=>'fieldStatus','contents'=>'active'],
  ],
]);
$collection = json_decode($res->getBody(), true);
```

Create Billing

```php
$collectionCode = $collection['code'];
$res = $client->post("/api/v1/billing/create/{$collectionCode}", [
  'headers' => ['ApiKey' => env('NEXGEN_API_KEY')],
  'query'   => ['ApiSecret'=>env('NEXGEN_API_SECRET'),'CheckSum'=>nexgenChecksum()],
  'multipart' => [
    ['name'=>'fieldName','contents'=>'Ali Bin Abu'],
    ['name'=>'fieldEmail','contents'=>'ali@example.com'],
    ['name'=>'fieldPhone','contents'=>'60123456789'],
    ['name'=>'fieldAmount','contents'=>'25.00'],
    ['name'=>'fieldPaymentDescription','contents'=>'Parking compound #8877'],
    ['name'=>'fieldCallbackUrl','contents'=>env('CALLBACK_URL')],
    ['name'=>'fieldRedirectUrl','contents'=>env('REDIRECT_URL')],
    ['name'=>'fieldMerchantTransactionID','contents'=>'CM8877-2025-08-15'],
  ],
]);
$bill = json_decode($res->getBody(), true);
return redirect()->away($bill['payment_url']);
```

Handle Callback (`routes/api.php`)

```php
Route::post('/nexgen/callback', [NexgenController::class, 'callback']);

public function callback(Request $req) {
  $data = $req->all();
  Bill::where('code', $data['code'])->update([
    'status' => $data['status'],
    'status_code' => $data['status_code'],
    'gateway_payload' => json_encode($data),
  ]);
  return response()->json(['ok'=>true], 200);
}
```

Redirect page (`routes/web.php`)

```php
Route::get('/payments/result', function (Request $req) {
  return view('payments.result', [
    'code' => $req->query('code'),
    'status' => $req->query('status'),
    'status_code' => $req->query('status_code'),
    'payer_name' => $req->query('payer_name'),
  ]);
});
```

Re-query Billing Data

```php
$res = $client->get("/api/v1billing/get/data/{$collectionCode}/{$billCode}", [
  'headers' => ['ApiKey' => env('NEXGEN_API_KEY')],
  'query'   => ['ApiSecret'=>env('NEXGEN_API_SECRET'),'CheckSum'=>nexgenChecksum()],
]);
$bill = json_decode($res->getBody(), true);
```

Update DREAMS Receipt

```php
$receipt = 'DRM-2025-000123';
$client->put("/api/v1/billing/update/receipt/data/{$collectionCode}/{$billCode}", [
  'headers' => ['ApiKey' => env('NEXGEN_API_KEY')],
  'query'   => [
    'ApiSecret'=>env('NEXGEN_API_SECRET'),
    'CheckSum'=>nexgenChecksum(),
    'fieldDreamsReceipt' => $receipt
  ],
]);
```

Notes:

- OBW includes `transaction_id`/`customer_bank`; MPGS includes `order_id`/`reference_id`.
- DREAMS receipt is mandatory for successful transactions when aligned with DREAMS; max 40 alphanumeric.

### 7.2 Node.js (axios) — Create Billing (quick start)

```js
import axios from 'axios';
import crypto from 'crypto';

const base = 'https://nexgen-stg.dbkl.gov.my';
const ApiKey = process.env.NEXGEN_API_KEY;
const ApiSecret = process.env.NEXGEN_API_SECRET;

function checksum() {
  const s = [
    process.env.NEXGEN_MERCHANT_CODE,
    process.env.NEXGEN_MERCHANT_NAME,
    ApiKey,
    ApiSecret,
  ].join('|');
  return crypto.createHash('sha256').update(s).digest('hex');
}

export async function createBilling(collectionCode) {
  const url = `${base}/api/v1/billing/create/${collectionCode}`;
  const { data } = await axios.post(
    url,
    {
      fieldName: 'Ali',
      fieldEmail: 'ali@example.com',
      fieldPhone: '60123456789',
      fieldAmount: '25.00',
      fieldPaymentDescription: 'Parking #8877',
      fieldCallbackUrl: 'https://yourapp.com/api/nexgen/callback',
      fieldRedirectUrl: 'https://yourapp.com/payments/result',
      fieldMerchantTransactionID: 'CM8877-2025-08-15',
    },
    {
      headers: { ApiKey },
      params: { ApiSecret, CheckSum: checksum() },
    }
  );
  return data.payment_url;
}
```

---

## 11) Appendix

Official Postman collection link is provided in the SDS (Lampiran A).

