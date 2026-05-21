---
Title: WhatsApp Business Platform Pricing
Topic: whatsapp
Subtopic: WhatsApp Business Platform Pricing
Sources:
  - "Meta Developer Documentation, 2026-03-30"
  - "WhatsApp Business, 2026-05-20"
Raw:
  - "[Pricing on the WhatsApp Business Platform](../../../raw/whatsapp/pricing/2026-05-20-pricing-on-the-whats-app-business-platform.md)"
  - "[Marketing API Pricing](../../../raw/whatsapp/pricing/2026-05-20-harga-api-marketing.md)"
  - "[Utility API Pricing](../../../raw/whatsapp/pricing/2026-05-20-harga-api-utility.md)"
Updated: 2026-05-20
---

# WhatsApp Business Platform Pricing

WhatsApp Business Platform pricing charges businesses per delivered template message, with the amount determined by destination market, message category, and where applicable volume tier. Meta's pricing doc says the platform moved to per-message pricing on July 1, 2025, replacing conversation-based pricing.

## Pricing Basis

The pricing sources state these key mechanics:

| Mechanic | Detail |
| --- | --- |
| Charge event | WhatsApp charges when a message is delivered, not merely sent. |
| Classification | Pricing depends on who receives the message and which message category applies. |
| Rate publication | Rates vary by market-category pair and are published for transparency. |
| Charge scope | Template messages are billable when delivered; non-template service messages are free inside an open customer service window. |
| Template categories | Marketing, utility, and authentication templates are the template categories called out in Meta's pricing doc. |
| Rate updates | Meta may update pricing on the first day of a quarter: January 1, April 1, July 1, or October 1. |

The WhatsApp Business pricing pages also describe service as a category in the customer-journey explanation, but Meta's pricing doc frames service messages as free non-template messages within a customer service window rather than pre-approved templates sent outside the window.

## Free Messaging Cases

| Case | Billing treatment |
| --- | --- |
| Non-template service messages | Free, but only sendable during an open customer service window. Webhooks use `type: free_customer_service` and `category: service`. |
| Utility templates inside an open customer service window | Free. Webhooks use `type: free_customer_service` and `category: utility`. |
| Free entry point window | If a user enters via Click to WhatsApp Ad or Facebook Page CTA on Android/iOS and the business responds within 24 hours, a 72-hour free entry point window opens. While it is open, any message type is free. |

The free entry point window is independent from the customer service window. If the customer service window closes while the free entry point window remains open, the business can only send templates, but those template sends remain free during the free entry point window.

## Indonesia Rates Captured

The May 2026 raw pages captured these list rates for Indonesia with currency set to IDR:

| Category | Captured Rate |
| --- | ---: |
| Marketing | `IDR586.3300` per delivered message |
| Utility | `IDR356.6500` per delivered message |

These are source-page snapshots, not a complete pricing table for every category or market. For exact billing, use the current WhatsApp Business pricing page or Meta rate cards with the selected market, currency, and category.

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

Meta's main pricing doc adds that volume is aggregated at the business portfolio level across all WABAs owned by the portfolio. Only charged messages count toward tiers; free utility templates inside a customer service window and utility templates inside a free entry point window do not count. Tiers are market-category specific and reset monthly at the start of the next month by WABA timezone.

Starting October 1, 2025, Meta sends an `account_update` webhook with `event` set to `VOLUME_BASED_PRICING_TIER_UPDATE` when a WABA reaches a new volume tier in a market for a given month. If multiple webhooks describe the same tier switch, the webhook with the smaller `tier_update_time` is treated as official.

## Category Intent

The pricing pages describe categories through customer journey examples:

| Category | Example Intent |
| --- | --- |
| Marketing | Promotional messages such as product campaigns. |
| Utility | Transactional or informational updates such as boarding passes or reminders. |
| Authentication | Verification codes and security-sensitive login flows. |
| Service | User-initiated support conversations. |

## Billing And Webhooks

Billable messages appear in status `messages` webhooks with `billable: true`, `pricing_model: PMP`, `type: regular`, and a pricing category such as `marketing`. Free customer-service cases use `billable: false` and `type: free_customer_service`.

The API's send response does not prove delivery. Pricing and delivery should be reconciled using status webhooks and pricing analytics. The pricing doc notes that tiering information is not currently included in status webhooks; use `pricing_analytics` for delivered-message tiering information.

## See Also

- [WhatsApp Flows](whatsapp-flows.md)
- [WhatsApp Service Messages](../../engineering/whatsapp/whatsapp-service-messages.md)
