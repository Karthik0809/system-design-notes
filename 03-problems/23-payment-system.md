# 23 · Design a Payment System (e-commerce checkout)

> **Category:** Money movement · **Difficulty:** ⭐⭐⭐⭐⭐ · **Patterns:** PSP integration via hosted page, idempotency keys, double-entry ledger, async processing, retries/DLQ, reconciliation

---

## 1. TL;DR
The **payment service** receives a payment event (an order), creates a payment order per seller, and the **payment executor** charges through a **PSP** (Stripe/Adyen) using a **hosted payment page** so card data never touches us (PCI scope). Successful charges update the **wallet** (balances) and the immutable **double-entry ledger**. Every call carries an **idempotency key**; failures go to retry/dead-letter queues; nightly **reconciliation** with PSP settlement files catches inconsistencies.

---

## 2. Requirements
| Functional | Non-functional |
|---|---|
| Pay-in: customer pays (on behalf of sellers) | **Correctness & exactly-once** charging |
| Pay-out: transfer to sellers monthly | Reliability, fault tolerance |
| Refunds, status tracking | Reconciliation between internal & external records |
| | Low QPS (~10 TPS for 1M txns/day) but zero tolerance for errors |

---

## 3. Pay-in architecture

```mermaid
flowchart LR
    U["Customer"] --> CO["Checkout"] --> PS["Payment service"]
    PS --> RISK["Risk check (fraud, AML)"]
    PS --> PE["Payment executor"] --> PSP["PSP (Stripe/Adyen)"] --> CS["Card schemes (Visa/MC)"]
    PS --> PDB[("Payment DB: payment_event, payment_order")]
    PS --> W["Wallet service: seller balances"]
    PS --> L["Ledger service: double-entry records"]
    PSP -->|"webhook"| PS
```

### Hosted payment page flow
```mermaid
sequenceDiagram
    participant U as Customer
    participant P as Payment service
    participant S as PSP
    U->>P: checkout (order 123)
    P->>S: register payment (amount, nonce = payment_order_id)
    S-->>P: token
    P-->>U: redirect to PSP hosted page (token)
    U->>S: enter card details
    S->>S: charge card
    S-->>U: redirect to our result page
    S->>P: webhook: payment status
    P->>P: update status, wallet, ledger (idempotent)
```

---

## 4. Data model
```
payment_event(checkout_id PK, buyer_info, seller_info, credit_card_info_token, is_payment_done)
payment_order(payment_order_id PK, checkout_id, amount, currency, status, ledger_updated, wallet_updated)
```
Status machine: `NOT_STARTED → EXECUTING → SUCCESS / FAILED` (+ `REFUNDED`).
Store amounts as **strings/decimals or integer minor units** — never floats.

---

## 5. Double-entry ledger
Every movement has equal debit and credit entries; the sum of all entries is always zero → easy auditing.

| Account | Debit | Credit |
|---|---|---|
| Buyer | $10 | |
| Seller | | $10 |

Ledger is **append-only**; corrections are new entries, never updates.

---

## 6. Reliability toolkit
- **Idempotency:** the same `payment_order_id` / `Idempotency-Key` header on every PSP and internal call → retries never double-charge; DB unique constraint guards inserts.
- **Retries:** exponential backoff for transient failures; **retry queue** → **dead letter queue** after N attempts for manual investigation.
```mermaid
flowchart LR
    E["Payment attempt"] --> R{"Result"}
    R -->|"success"| OK["Update wallet + ledger"]
    R -->|"retryable error / timeout"| RQ["Retry queue (backoff)"] --> E
    R -->|"non-retryable"| F["Mark failed, notify"]
    RQ -->|"too many attempts"| DLQ["Dead letter queue"]
```
- **Timeouts are "unknown"**, not failures → query PSP for status before retrying.
- **Async communication** (Kafka) between payment, wallet, ledger, analytics; at-least-once + idempotent consumers.
- **Reconciliation:** nightly compare PSP settlement file vs ledger vs wallet → classify mismatches (auto-fixable / manual / unknown).
- **Distributed transaction across wallet + ledger:** use outbox/saga patterns or keep them consistent via event sourcing.

---

## 7. Pay-out
Accumulated seller balances → payout provider (e.g., Tipalti) → bank transfers; same idempotency and reconciliation rules.

## 8. Security
HTTPS, hosted payment pages/tokenization (PCI DSS), fraud checks (3-D Secure, ML risk scoring), rate limiting, audit logs, encryption at rest.

---

## 9. Interview Q&A
**Q: How do you ensure exactly-once payment?** At-least-once retries + idempotency keys on both sides = effectively once.

**Q: User double-clicks "Pay"?** Same checkout ID → idempotency key → second request returns the first result.

**Q: PSP says success but our DB update failed?** PSP webhook + periodic status polling + reconciliation repair it.

**Q: Why a ledger if we have balances?** Balances are derived; the ledger is the auditable source of truth.

---

**Related:** [Digital wallet](24-digital-wallet.md) · [Distributed transactions](../01-fundamentals/10-distributed-transactions-2pc-saga-outbox.md) · [Fraud detection](55-fraud-detection.md) · [Security](../01-fundamentals/15-security-authn-authz-encryption.md)
