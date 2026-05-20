---
Title: TikTok Shop Shipping Weight and Package Rules
Topic: tiktok
Subtopic: TikTok Shop Logistics
Sources:
  - "TikTok Shop Academy, 2026-05-20"
Raw:
  - "[Platform Shipping Fee Instructions](../../raw/tiktok/order-delivery/2026-05-20-instruksi-biaya-pengiriman-platform-indonesia.md)"
  - "[Chargeable Weight](../../raw/tiktok/order-delivery/2026-05-20-menghitung-berat-yang-ditagihkan.md)"
  - "[Split Orders](../../raw/tiktok/order-delivery/2026-05-20-memisahkan-pesanan-menjadi-beberapa-paket.md)"
  - "[Merge Orders](../../raw/tiktok/order-delivery/2026-05-20-menggabungkan-pesanan.md)"
Updated: 2026-05-20
---

# TikTok Shop Shipping Weight and Package Rules

TikTok Shop by Tokopedia shipping charges depend on chargeable weight, route, pickup address handling, and whether orders are split or merged before handoff to logistics.

## Chargeable Weight

Chargeable weight is the higher of actual weight and volumetric weight.

```text
Volumetric weight (kg) = Length * Width * Height (cm) / 6000
Chargeable weight = max(actual weight, volumetric weight)
```

Actual weight is the full packed parcel weight, including product and packaging. Volumetric weight matters for bulky or irregular products such as plush toys, diapers, pillows, and similar items.

Example from the source:

| Measurement | Value |
| --- | ---: |
| Actual weight | `4.5 kg` |
| Dimensions | `30 x 20 x 10 cm` |
| Volumetric weight | `1 kg` |
| Chargeable weight | `4.5 kg` |

## Seller vs LSP Measurement

From `2025-02-16`, platform shipping fee settlement uses the higher value between seller-declared chargeable weight and Logistics Service Provider measured chargeable weight.

This means under-declaring product weight or dimensions can make actual shipping settlement higher than expected. Sellers should list the packed product's accurate weight and dimensions.

## Pickup Address Rule

From `2025-04-01`, seller-side shipping fee calculation uses the actual pickup address recorded by the Logistics Service Provider at pickup, not only the pickup address saved by the seller in Seller Center.

Buyer checkout shipping fee still uses the pickup address saved by the seller in the system. Seller settlement can therefore differ if the actual pickup location differs from the configured address.

## Splitting Orders

Order splitting lets a seller split one multi-SKU order into multiple packages while the order is still in `Menunggu Pengiriman` status and before it moves to `Menunggu Pengambilan`.

Key limits:

| Rule | Detail |
| --- | --- |
| Platform scope | Only applies to TikTok Shop by Tokopedia orders. |
| SKU condition | Order must contain multiple SKU types. |
| Same-SKU limit | The same SKU cannot be split into multiple packages even if quantity is more than one. |
| Edit window | Split can be changed only before `Menunggu Pengambilan`. |

Example: an order with SKU A quantity 2 and SKU B quantity 1 can become package 1 containing both SKU A units and package 2 containing SKU B. SKU A cannot be split into two separate packages.

## Merging Orders

Order merging lets sellers combine multiple orders into one package for more efficient fulfillment. Seller Center can recommend mergeable orders when conditions are met.

Merging is optional. After merging, customers can track the combined package in the TikTok app. The combined package can still be changed before it reaches `Menunggu Pengambilan`.

If a customer requests refund for one order inside a combined package while the combined package is still waiting for pickup, approved refund processing can break the package back into separate orders. Seller Center can then recommend merging again where applicable.

## See Also

- [TikTok Shop Logistics Service Fee](tiktok-shop-logistics-service-fee.md)
- [TikTok Shop Fees Overview](tiktok-shop-fees-overview.md)
