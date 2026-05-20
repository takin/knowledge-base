---
Title: TikTok Shop Platform Commission Fee
Topic: tiktok
Subtopic: TikTok Shop Fees
Sources:
  - TikTok Shop Academy, 2026-04-28
  - TikTok Shop Academy, 2026-05-20
  - Tokopedia & TikTok Shop fee calculator, 2026-05-18
Raw:
  - "[Platform Commission Fee](../../raw/tiktok/shop/2026-05-20-biaya-komisi-platform.md)"
  - "[Platform Commission Savings](../../raw/tiktok/shop/2026-05-20-diskon-komisi-platform.md)"
  - "[Fee Calculator XLSX](../../raw/tiktok/shop/attachments/d9a9c4266c2b4fd7ab8f992248b98ca9.xlsx)"
  - "[Commission Rates PDF](../../raw/tiktok/shop/attachments/bb8d830d1cbe4ddeada588ca6e3f3085.pdf)"
  - "[GMV Max Fee Rates PDF](../../raw/tiktok/shop/attachments/2bec14002b494b3ab93f663e5a51a1c4.pdf)"
Updated: 2026-05-20
---

# TikTok Shop Platform Commission Fee

Platform Commission Fee is the fee charged to all sellers in Indonesia based on product price. It differs for Mall sellers and Marketplace sellers and is deducted directly from order settlement after a valid order is successfully delivered and accepted.

From `2026-05-18` Jakarta time, TikTok Shop by Tokopedia introduced new Platform Commission Savings to reduce seller operational cost and improve exposure and sales for dedicated sellers.

## Calculation Basis

The fee uses item price after seller discount. Shipping fees and platform-sponsored discounts are excluded from the calculation basis.

```text
Platform Commission Fee = (Item Price - Seller Discount) * Fee rate
```

The rate differs by product category and seller group. The attached PDF table contains category-level rates and final-rate scenarios, while the XLSX calculator provides indicative estimates for a seller's chosen category, price, discount, GMV Max status, and Growth Xtra status.

The calculator caveat should be preserved in any operational use: it is for indicative estimates only and final fees can vary.

## Example

For a Mall seller selling a guitar:

| Field | Amount |
| --- | --- |
| Original item price | `Rp10,000` |
| Seller discount | `Rp1,000` |
| Subtotal after discount | `Rp9,000` |

Example commission outcomes from the source:

| Scenario | Final Commission Rate | Fee |
| --- | --- | --- |
| Default rate | `9.20%` | `Rp828` tax inclusive |
| Joined Growth Xtra, GMV Max spend to GMV below `3%`, order without GMV Max | `7.80%` | `Rp702` tax inclusive |
| Shop Ads Spend to GMV at least `3%`, not joined Growth Xtra | `5.70%` | `Rp513` tax inclusive |
| GMV Max spend to GMV at least `3%` and joined Growth Xtra | `5.20%` | `Rp468` tax inclusive |

## Platform Commission Savings

Platform Commission Savings reduces commission rates for eligible sellers. Eligibility can come from GMV Max, Growth Xtra, or both.

The post-18 May rate tables split the logic into several columns:

| Rate Family | What It Represents |
| --- | --- |
| Default Platform Commission Rate | Baseline Marketplace and Mall platform commission before savings. |
| Platform Commission Savings from `2026-05-18` | Savings based on GMV Max spend-to-GMV and Growth Xtra participation. |
| Final Marketplace Rate After Savings | Marketplace final rate after applying the applicable savings scenario. |
| Final Mall Rate After Savings | Mall final rate after applying the applicable savings scenario. |

The category tables are large, so this article records the rules and examples. Exact category-level rates should be checked in the official PDF or XLSX.

### GMV Max Savings

From `2026-04-28`, sellers using GMV Max can receive two levels of savings.

Order-level savings apply when a product or LIVE has an active GMV Max campaign. Products without an active GMV Max campaign in the same shop follow the original standard fee rates.

Shop-level savings apply if GMV Max ad spend to GMV over the previous 30 days is at least `3%`.

```text
GMV Max percentage = GMV Max spending / shop GMV
```

Important notes:

- Active GMV Max means the Product GMV Max campaign is active and has sufficient balance.
- The spend calculation includes Product GMV Max and LIVE GMV Max.
- The spend calculation excludes Custom Ads and Brand Ads.
- GMV from ad-prohibited products under Indonesian law is excluded.

### Growth Xtra Savings

Growth Xtra, previously Xtra Boost, gives participating sellers whole-shop Platform Commission Savings. Orders sold via GMV Max by Growth Xtra sellers can receive additional order-specific savings.

The calculator labels the post-18 May Growth Xtra/GXP charge as `2.0% - 4.0%` with no cap. Before 18 May, the calculator labels XBP as `3.5% - 4.5%` with a `Rp60,000` cap. This is separate from Platform Commission Fee itself, but it matters for estimating the net effect of participating in Growth Xtra.

### Dual Stacked Savings

Sellers that qualify for both Growth Xtra and GMV Max spend-to-GMV of at least `3%` can receive Dual Stacked savings. The source states this can reach up to `8.18%` in Platform Commission Savings.

The attached calculator treats the highest-benefit scenario as GMV Max spend-to-GMV `>=3%` plus Growth Xtra participation. It applies to all orders while the shop satisfies both conditions, not only to orders with an active GMV Max campaign.

## Calculator Inputs

The XLSX calculator asks sellers to fill in:

| Input | Why It Matters |
| --- | --- |
| Seller type | Determines whether Marketplace or Mall rate columns apply. |
| GMV Max spend-to-GMV `>=3%` | Determines shop-level GMV Max eligibility. |
| Order using GMV Max | Determines order-level GMV Max eligibility. |
| Growth Xtra participation | Determines Growth Xtra savings and combined scenarios. |
| XBP/Growth Xtra charge day | Used by the calculator's fee comparison. |
| Category level 1-3 | Selects the category-specific commission rate. |
| Product price and seller discount | Determines the calculation base. |

The calculator compares estimated fees before and after 18 May in formula columns and simulates commission savings across default, GMV Max, Growth Xtra, and combined scenarios.

## Pre-order Service Fee

Pre-order goods incur an additional `3%` pre-order service fee per product sold on top of the Platform Commission Fee.

```text
Pre-order Service Fee = (Item Price - Seller Discount) * 3%
```

The source says this fee has applied since 2025.

## Valid Orders and Refunds

A valid order is a paid order that has been successfully delivered to and accepted by the buyer. Cancelled, fully returned, or refunded orders are not considered valid orders.

If an entire order is returned, the full commission fee is refunded to the seller. If part of the order is refunded, the commission fee charged on the refunded item or items is refunded to the seller.

## Tax, Invoice, and Finance Records

Platform Commission Fee rates in the source are tax inclusive.

Sellers can review commission details in Seller Center finance transactions:

- Orders not settled: `Finance > Transactions > To settle > View Details > Estimated Settlement Details`
- Settled orders: `Finance > Transactions > Settled > View Details > Earning Details`

Monthly invoices are available the following month for the previous billing period under `Finance > Invoice Center > Platform Service Fee`.

## GMV Max Qualification Rules

Product GMV Max and LIVE GMV Max provide the same saving amount, but qualification differs by sales context.

For non-LIVE orders, the order qualifies for order-level GMV Max savings only if the product has an active Product GMV Max campaign.

For LIVE orders, the order qualifies only if the LIVE room has an active LIVE GMV Max campaign. If the LIVE room does not have active LIVE GMV Max, orders sold through that LIVE do not qualify for order-level savings even when the product has active Product GMV Max.

If the shop's GMV Max spend to GMV percentage over the previous 30 days is at least `3%`, all orders qualify for shop-level GMV Max savings regardless of LIVE status or campaign status.

## Savings Timing

GMV Max order-level savings are assessed daily. A product qualifies on the specific day it is sold with an active Product or LIVE GMV Max campaign.

GMV Max shop-level savings are assessed on a rolling 30-day basis. If the previous 30 days meet the `3%` threshold, the shop qualifies on the 31st day.

Growth Xtra shop-level savings apply in real time after successful enrollment until successful opt-out.

Dual Stacked savings apply as long as the shop qualifies for both Growth Xtra and GMV Max `3%` shop-level savings.

## See Also

- [TikTok Shop Fees Overview](tiktok-shop-fees-overview.md)
- [TikTok Shop Dynamic Commission Fee](tiktok-shop-dynamic-commission-fee.md)
- [TikTok Shop Order Processing Fee](tiktok-shop-order-processing-fee.md)
- [TikTok Shop Affiliate Commission and Orders](tiktok-shop-affiliate-commission-and-orders.md)
- [TikTok Shop Logistics Service Fee](tiktok-shop-logistics-service-fee.md)
