---
Title: Xendit QRIS
Topic: xendit
Subtopic: Indonesian Payments
Sources:
  - "Xendit Documentation, 2026-05-20"
Raw:
  - "[QRIS](../../raw/xendit/api/2026-05-20-qris.md)"
Updated: 2026-05-20
---

# Xendit QRIS

Xendit's QRIS channel supports Indonesian QR code payments using QRIS, the national QR code standard developed by Bank Indonesia and the Indonesian Payment System Association for cashless payments in Indonesia.

## Channel Characteristics

| Property | Value |
| --- | --- |
| Channel code | `QRIS` |
| Display name | `QRIS` |
| Currency | `IDR` |
| Country | `ID` |
| Type | QR code |
| Minimum amount | `1` |
| Maximum amount | `10,000,000.00` |
| User approval flow | Present to customer |
| Reusable payment code | Supported |

The source table indicates QRIS does not support save, merchant-initiated transaction, auth and capture, partial capture, multiple partial capture, or desktop support in the captured feature matrix.

## Payment Flow

The user-facing payment flow is straightforward:

| Step | Action |
| --- | --- |
| 1 | Customer selects QRIS at checkout. |
| 2 | A QR code appears on screen. |
| 3 | Customer opens a mobile banking or e-wallet app and chooses scan QR code. |
| 4 | Customer scans the QR code. |
| 5 | Customer checks that the amount and merchant are correct. |
| 6 | Customer confirms payment. |

## Refund Limitations

The QRIS source notes refund capability varies by issuer and refund type. DANA, ShopeePay, OVO, Gopay, Permata, Jenius/SMBC, and BSI support full refunds within 24 hours, full refunds after 24 hours, and partial refunds in the captured table. CIMB differs: the captured table shows it does not support full refunds within 24 hours, but does support full refunds after 24 hours and partial refunds.

## See Also

- [Xendit xenPlatform](xendit-xenplatform.md)
