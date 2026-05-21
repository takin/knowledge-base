---
Title: Xendit xenPlatform
Topic: xendit
Subtopic: Platform Payments
Sources:
  - "Xendit Documentation, 2026-05-20"
  - "Xendit Indonesia Pricing, 2026-05-20"
Raw:
  - "[xenPlatform Overview](../../../raw/xendit/2026-05-20-overview.md)"
  - "[xenPlatform Fees](../../../raw/xendit/2026-05-20-xen-platform-fees.md)"
  - "[Biaya Transaksi](../../../raw/xendit/2026-05-20-biaya.md)"
Updated: 2026-05-20
---

# Xendit xenPlatform

xenPlatform is Xendit's multi-account payment solution for platforms that manage payments, splits, balances, and payouts across multiple merchants, partners, sellers, vendors, branches, or similar entities.

## Core Concepts

| Concept | Meaning |
| --- | --- |
| Master Account | The platform's Xendit account with xenPlatform activated. It can create and manage sub-accounts. |
| Sub-Account | The account where transactions are processed for an individual merchant or entity, with its own balance and transaction history. |
| Account ID | Identifier used in API requests to target a sub-account. The docs note synonyms such as `user-id`, `owner_id`, and `business_id`. |

xenPlatform enables the platform to create sub-accounts, accept payments on behalf of merchants, split payments between accounts, track transactions centrally, and pay out to merchants' bank accounts.

## Typical Platform Flow

The overview reduces sub-account payment management into four steps:

| Step | Goal |
| --- | --- |
| Create sub-account | Establish the merchant, partner, seller, vendor, branch, or other entity account. |
| Manage payments | Accept customer payments on behalf of the sub-account. |
| Monitor transactions | Track activity across sub-accounts in one place. |
| Pay out | Send funds to merchants' bank accounts or allow withdrawals, depending on setup. |

The documented business models include marketplaces, SaaS platforms, point-of-sale platforms, payment service providers, brick-and-mortar chains, and other partner networks with multiple recipients of funds.

## Managed Versus Owned Sub-Accounts

The fee source distinguishes Managed and Owned sub-accounts in billing and invoice responsibility.

| Area | Managed Sub-Accounts | Owned Sub-Accounts |
| --- | --- | --- |
| Direct billing | Fees are deducted from the sub-account. | Fees are deducted from the sub-account. |
| Indirect billing | Merchant pays after billing statement is issued. | Platform pays after billing statement is issued. |
| Billing statement recipient | Sub-account. | Master account. |
| Tax invoice recipient | Sub-account. | Master account. |

Under direct deduction, transaction fees are deducted from the account that receives the transaction. Xendit can also provide consolidated invoices that combine invoices into the platform regardless of sub-account type. Sub-accounts under Global accounts currently default to Managed Sub-Accounts.

## xenPlatform Fees

The fee source identifies two usage-based xenPlatform fees:

| Fee | Trigger | Indonesia public pricing source |
| --- | --- | --- |
| Sub-account activity fee | Charged per monthly active sub-account. A sub-account is active if it has at least one chargeable transaction in the month, excluding withdrawals and top-ups. | `Rp25,000` account activity fee. |
| In-house transaction fee | Charged for transfers and split fees between accounts, accumulated and charged to the platform via indirect billing. | `0.5%` in-house transaction fee. |

The in-house transaction fee applies to the amount transferred or split, not the full customer transaction amount. The example given is a `0.5%` fee on an `IDR5,000` split from an `IDR100,000` customer payment, producing an `IDR25` fee.

## See Also

- [Xendit QRIS](xendit-qris.md)
- [Xendit Indonesia Pricing](xendit-indonesia-pricing.md)
