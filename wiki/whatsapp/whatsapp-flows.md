---
Title: WhatsApp Flows
Topic: whatsapp
Subtopic: WhatsApp Business Platform
Sources:
  - "Meta Developer Documentation, 2026-04-15"
  - "Meta Developer Documentation, 2024-07-29"
Raw:
  - "[WhatsApp Flows Developer Documentation](../../raw/whatsapp/flow/2026-05-20-whats-app-flows--developer-documentation.md)"
  - "[Collect Purchase Interest Use Case](../../raw/whatsapp/flow/2026-05-20-use-case-guide-collect-purchase-interest.md)"
Updated: 2026-05-20
---

# WhatsApp Flows

WhatsApp Flows lets businesses create structured, in-chat interactions for lead generation, product discovery, sales qualification, onboarding, and other journeys where guided input is better than a free-form chat with an agent.

## Core Model

Flows are configurable WhatsApp experiences with rich interaction screens. A business defines the flow structure, customizes it for a use case, tests it in WhatsApp Manager, publishes it, sends it through supported message types, and receives completed responses through the normal message webhook path.

Common packaged use cases include:

| Use Case | What The Flow Collects Or Enables |
| --- | --- |
| Pre-approved loans | Loan amount, repayment period, disbursement method, payment method, and identity confirmation. |
| Insurance quote | Personal preferences, coverage details, quote selection, and payment frequency. |
| Purchase intent | Customer contact data and product or service interests for later marketing and sales campaigns. |
| Personalised offer | Product interests, budget, and recommended purchase follow-up. |

## Collect Purchase Interest Pattern

The purchase-interest template is a reusable lead-generation pattern. It collects personal information and lets users choose the products or services they care about, giving the business structured preference data before a campaign.

This pattern can be adapted beyond commerce, including webinar registration, event signup, newsletter opt-in, profile completion, or any journey where the business needs customer attributes and preferences.

## Build And Test Workflow

For a template-based Flow, the source guide describes this operating sequence:

| Step | Detail |
| --- | --- |
| Create | In WhatsApp Manager, open Flows, create a new Flow, choose a category such as Lead generation, and start from a template such as Collect purchase intent. |
| Edit | Customize the Flow JSON and preview the screens in the Builder UI. The Flow remains in Draft while editable. |
| Test interactively | Enable interactive mode in the preview, select the first screen, walk through the Flow, and inspect generated actions such as `navigate`. |
| Test on device | Send the draft Flow to a device for testing before publishing. Draft sends are for testing only. |
| Publish | Resolve validation errors and publishing checks, confirm the Flow follows WhatsApp Flows design principles and relevant WhatsApp policies, then publish. |

Publishing is a hard lifecycle boundary: once published, the Flow can be sent broadly but can no longer be modified.

## Sending And Response Handling

Published Flows can be sent as template messages or as interactive Flow messages.

| Send Mode | Constraint |
| --- | --- |
| Template message | Can be sent without an open 24-hour customer service window. |
| Interactive Flow message | Requires an open customer service window between the business and the user. |

When the user completes the Flow, WhatsApp sends the response into the chat, and the business receives it through the same message webhook path used for other incoming user messages.

## Operational Constraints

Flow monitoring applies only to Flows with an endpoint. Before publishing, the business should treat validation, publishing checks, design principles, WhatsApp Terms of Service, WhatsApp Business Messaging Policy, and any applicable Commerce Policy as release gates.

## See Also

- [WhatsApp Business Platform Pricing](whatsapp-business-platform-pricing.md)
