---
Title: WhatsApp Business Platform Pricing
Topic: whatsapp
Subtopic: WhatsApp Business Platform Pricing
Sources:
  - "WhatsApp Business, 2026-05-20"
Raw:
  - "[Marketing API Pricing](../../raw/whatsapp/pricing/2026-05-20-harga-api-marketing.md)"
  - "[Utility API Pricing](../../raw/whatsapp/pricing/2026-05-20-harga-api-utility.md)"
Updated: 2026-05-20
---

# WhatsApp Business Platform Pricing

WhatsApp Business Platform pricing charges businesses per delivered message, with the amount determined by destination market and message category. For Indonesia in Indonesian Rupiah, the collected source pages show separate list rates for marketing and utility messages.

## Pricing Basis

The pricing source states three key mechanics:

| Mechanic | Detail |
| --- | --- |
| Charge event | WhatsApp charges when a message is delivered, not merely sent. |
| Classification | Pricing depends on who receives the message and which message category applies. |
| Rate publication | Rates vary by market-category pair and are published for transparency. |

The platform categories referenced in the source are marketing, utility, authentication, and service.

## Indonesia Rates Captured

The May 2026 raw pages captured these list rates for Indonesia with currency set to IDR:

| Category | Captured Rate |
| --- | ---: |
| Marketing | `IDR586.3300` per delivered message |
| Utility | `IDR356.6500` per delivered message |

These are source-page snapshots, not a complete pricing table for every category or market. For exact billing, use the current WhatsApp Business pricing page with the selected market, currency, and category.

## Utility Volume Tiers

The utility pricing page includes volume tiers for utility and authentication messages. The page says tier rates apply only to messages within that tier.

| Monthly Messages | Utility/Auth Rate | Versus List |
| --- | ---: | ---: |
| `0 - 750,000` | `IDR356.6500` | `0%` |
| `750,001 - 4,000,000` | `IDR338.8200` | `-5%` |
| `4,000,001 - 25,000,000` | `IDR320.9900` | `-10%` |
| `25,000,001 - 75,000,000` | `IDR303.1500` | `-15%` |
| `75,000,001 - 150,000,000` | `IDR285.3200` | `-20%` |
| `150,000,001+` | `IDR267.4900` | `-25%` |

## Category Intent

The pricing pages describe categories through customer journey examples:

| Category | Example Intent |
| --- | --- |
| Marketing | Promotional messages such as product campaigns. |
| Utility | Transactional or informational updates such as boarding passes or reminders. |
| Authentication | Verification codes and security-sensitive login flows. |
| Service | User-initiated support conversations. |

## See Also

- [WhatsApp Flows](whatsapp-flows.md)
