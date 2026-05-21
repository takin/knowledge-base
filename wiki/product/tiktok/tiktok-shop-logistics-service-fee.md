---
Title: TikTok Shop Logistics Service Fee
Topic: tiktok
Subtopic: TikTok Shop Logistics
Sources:
  - "TikTok Shop Academy, 2026-05-20"
  - "Tokopedia & TikTok Shop logistics fee table, 2026-05-01"
Raw:
  - "[Logistics Service Fee](../../../raw/tiktok/order-delivery/2026-05-20-biaya-layanan-logistik-berlaku-mulai-1-mei-2026.md)"
  - "[Logistics Fee XLSX](../../../raw/tiktok/order-delivery/attachments/2e9078a0bcae4c1e8820683b5d6466de.xlsx)"
Updated: 2026-05-20
---

# TikTok Shop Logistics Service Fee

Logistics Service Fee is a seller-side fee for logistics-related services, including processing, coordination, shipping issue handling, and basic shipping incentives. It applies to all new orders created from `2026-05-01 10:00 WIB`.

The buyer is not directly affected at checkout. The fee is handled on the seller side and can be reviewed in finance transactions.

## Calculation Basis

The fee is a flat amount per order. It varies by:

| Dimension | Meaning |
| --- | --- |
| Logistics service | Standard, Economy, Cargo, Instant, or Sameday when available. |
| Shipping route | Zone A to Zone B route, applied in both directions. |
| Parcel weight tier | Based on chargeable weight, using the tier that matches or exceeds the parcel weight. |

The XLSX tariff table states that rates are in IDR and VAT inclusive.

## Chargeable Weight

Parcel tier is based on chargeable weight: the higher value between actual parcel weight and volumetric weight.

```text
Volumetric weight = Length * Width * Height (cm) / 6000
Chargeable weight = max(actual weight, volumetric weight)
```

See also: [TikTok Shop Shipping Weight and Package Rules](tiktok-shop-shipping-weight-and-package-rules.md).

## Standard Service Examples

The official XLSX contains the full route table. Representative standard-service rows are:

| Route | 0-1 kg | 1.01-2 kg | 2.01-3 kg | 3.01-4 kg | 4.01-5 kg | >5 kg |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Jawa to Jawa (Jakarta) | `Rp690` | `Rp890` | `Rp1,620` | `Rp2,220` | `Rp2,730` | `Rp4,350` |
| Jawa to Jawa (Selain Jakarta) | `Rp990` | `Rp1,090` | `Rp2,220` | `Rp3,030` | `Rp3,540` | `Rp5,060` |
| Jawa to Bali | `Rp1,720` | `Rp2,220` | `Rp4,150` | `Rp5,060` | `Rp5,060` | `Rp5,060` |
| Jawa to Sumatera | `Rp2,830` | `Rp3,330` | `Rp5,060` | `Rp5,060` | `Rp5,060` | `Rp5,060` |
| Jawa to Papua & Maluku | `Rp5,060` | `Rp5,060` | `Rp5,060` | `Rp5,060` | `Rp5,060` | `Rp5,060` |
| Luar Jawa to Luar Jawa | `Rp2,020` | `Rp2,530` | `Rp3,840` | `Rp4,950` | `Rp5,060` | `Rp5,060` |

Use the official XLSX for exact service, route, and tier combinations.

## Service Availability

The source summarizes service availability by region and chargeable weight:

| Region | Chargeable Weight | Standard | Economy | Cargo |
| --- | --- | --- | --- | --- |
| Jawa | `0-2 kg` | Available | Not available | Not available |
| Jawa | `>2 kg` | Available | Not available | Available |
| Luar Jawa | `0-2 kg` | Available | Available | Not available |
| Luar Jawa | `>2 kg` | Available | Available | Available |

Availability can change with external market conditions. The final tariff for a specific order depends on the option selected by the buyer at checkout; by default, the lowest tariff after shipping incentives is selected, but final choice remains buyer-dependent.

## Finance Visibility

Before settlement, sellers can see estimated Logistics Service Fee under `Finance > Transactions > Unsettled`.

After settlement, sellers can see the exact fee charged under `Finance > Transactions > Settled`. Differences can occur because estimates use seller-provided listing dimensions and standard route assumptions, while exact charges use logistics-provider package measurements and actual route.

## Refund Treatment

Logistics Service Fee is charged when initial delivery is successfully received by the buyer. If the buyer later requests return and refund after delivery, the fee is not refunded to the seller.

The source also states that Logistics Service Fee applies to free samples sent by sellers to affiliates.

## See Also

- [TikTok Shop Fees Overview](tiktok-shop-fees-overview.md)
- [TikTok Shop Shipping Weight and Package Rules](tiktok-shop-shipping-weight-and-package-rules.md)
- [TikTok Shop Free Samples for Affiliates](tiktok-shop-free-samples-for-affiliates.md)
