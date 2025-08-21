---
id: overview
title: Overview & Concepts
sidebar_label: Overview
sidebar_position: 1
---

## 1) Introduction

NexGEN is DBKL’s modern e‑payment platform, replacing EPIC. It supports:

- DuitNow Online Banking/Wallet (OBW) for bank/wallet payments.
- Mastercard Payment Gateway Services (MPGS) for card payments.

The API is RESTful over HTTPS with JSON responses. Transaction limits and PDPA consent are enforced.

## 2) Overview

### 2.1 Terminology

- **Collection**: logical group of bills (e.g., Parking, Service Fees).
- **Bill**: a payable item under a Collection.
- **Callback URL**: your server endpoint to receive POST payment notifications.
- **Redirect URL**: your frontend page to show the final result after server‑side processing.

### 2.2 Payment methods & limits

- **OBW (BIMB)**: RM1.00–RM50,000 (Individual), up to RM1,000,000 (Corporate).
- **MPGS (BIMB)**: RM1.00–RM1,000,000.

For larger totals, split into multiple transactions or use corporate channels.

### 2.3 Security & authentication

Requests must include `ApiKey` (header), `ApiSecret` (query), and `CheckSum` (query).

Checksum formula:

```
SHA256(MerchantCode | MerchantName | ApiKey | ApiSecret)
```

All calls use HTTPS.

### 2.4 PDPA consent

Your portal must present a checkbox consent before storing/updating personal data and include PDPA notice content (Malay & English) as per appendices.

## 3) API Architecture & Flow

1. Create Collection (once per program/type).
2. Create Billing under a collection → get `payment_url`.
3. Payer completes payment via OBW or MPGS.
4. NexGEN POSTs Callback to your server with final status.
5. After successful server processing, NexGEN triggers Redirect back to your UI with concise GET params for display.

Status mapping across endpoints:

```
status: pending | completed | failed
status_code: 0 | 1 | 2
```

