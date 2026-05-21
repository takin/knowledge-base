---
Title: TikTok Shop Affiliate Commission and Orders
Topic: tiktok
Subtopic: TikTok Shop Affiliate
Sources:
  - "TikTok Shop Academy, 2026-05-20"
Raw:
  - "[Affiliate Orders](../../../raw/tiktok/affiliate/2026-05-20-pesanan-afiliasi.md)"
  - "[Affiliate Commission Settings](../../../raw/tiktok/affiliate/2026-05-20-pengaturan-komisi-afiliasi-untuk-seller.md)"
  - "[Affiliate Collaboration](../../../raw/tiktok/affiliate/2026-05-20-kolaborasi-afiliasi.md)"
  - "[External Traffic Program](../../../raw/tiktok/affiliate/2026-05-20-panduan-program-trafik-eksternal-untuk-penjual.md)"
  - "[Tax FAQ](../../../raw/tiktok/2026-05-20-faq-perpajakan.md)"
Updated: 2026-05-20
---

# TikTok Shop Affiliate Commission and Orders

Affiliate commission is paid to creators or affiliate partners for eligible sales generated through affiliate promotion. For sellers, it is an additional cost layer on top of core TikTok Shop fees such as Platform Commission Fee, Dynamic Commission Fee, Order Processing Fee, and logistics-related fees.

## Commission Formula

The affiliate order source states that actual commission can differ from estimated commission because of returns or partial refunds.

```text
Commission = (Revenue - Refund) * Commission Percentage
```

If an order is cancelled or fully refunded, no commission is received by the creator. If only part of an order is refunded, the remaining item value can still generate commission.

## Seller-Controlled Rate

In Open Collaboration, sellers set the commission percentage for products added to affiliate. TikTok may show suggested commission percentages based on category data, but the seller decides the actual rate.

In Targeted Collaboration, sellers set commission per product when creating the invitation. A targeted rate takes priority over Open Collaboration when both apply.

## Commission Change Lockdown

When a seller lowers commission for a product, the lower rate does not immediately apply to existing creators who already had the product in their content or showcase.

Example from the source:

| Creator Type | Rate Behavior |
| --- | --- |
| Existing creator before change | Keeps old `15%` rate for 30 days, then receives new `12%` rate. |
| New creator after change | Receives new `12%` rate immediately. |

This means one product can temporarily have multiple active commission rates for different creators.

## Affiliate Order Definitions

The source distinguishes SKU-level orders from affiliate order count.

| Metric | Meaning |
| --- | --- |
| SKU orders | Number of affiliate product SKU units ordered because of creator promotion, including refunded orders. |
| Affiliate order count | Number of buyer orders generated through creator promotion. |

Example: if two buyers place orders and the orders contain four affiliate SKUs total, the display can show `4` SKU orders but `2` affiliate orders.

## Order Statuses

Affiliate orders can appear under several statuses:

| Status | Meaning |
| --- | --- |
| Belum Dibayar | Buyer ordered but has not completed payment; no commission yet. |
| Masih Diproses | Order is being processed; commission can be withdrawn after delivery and payment completion. |
| Pembayaran Selesai | Order is complete and commission can be withdrawn. |
| Tidak Memenuhi Syarat | Order was cancelled or returned and does not generate commission. |
| Dibekukan | Order is under risk-control review. |

Reports usually update within minutes after buyer order creation, though major promotions can create delays.

## Excluded Orders

The affiliate order details page does not show these order types:

| Excluded Type |
| --- |
| Returnable orders / free samples |
| Seller-fulfilled orders |
| Gift orders |

Exports support up to `15,000` orders per download.

## External Traffic Commission

External traffic orders use affiliate links outside ShopTokopedia. Commission generally follows Open Collaboration commission, but Targeted Collaboration or TAP campaign rates apply when the purchase came through those contexts.

For ShopTokopedia recommendation traffic, commission is `50%` of the relevant Open Collaboration, Targeted Collaboration, or TAP campaign rate.

## Tax Responsibility

The tax FAQ states that creators are independent contractors providing services to the seller under the seller-creator agreement, and TikTok is not a party to that agreement. Sellers are responsible for reporting and/or withholding applicable income tax on creator commission payments.

## See Also

- [TikTok Shop Affiliate Program](tiktok-shop-affiliate-program.md)
- [TikTok Shop Free Samples for Affiliates](tiktok-shop-free-samples-for-affiliates.md)
- [TikTok Shop Tax FAQ](tiktok-shop-tax-faq.md)
- [TikTok Shop Fees Overview](tiktok-shop-fees-overview.md)
