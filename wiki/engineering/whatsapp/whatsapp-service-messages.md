---
Title: WhatsApp Service Messages
Topic: whatsapp
Subtopic: WhatsApp Business Platform Messaging
Sources:
  - "Meta Developer Documentation, 2026-04-23"
Raw:
  - "[Service Messages Developer Documentation](../../../raw/whatsapp/2026-05-20-service-messages--developer-documentation.md)"
Updated: 2026-05-20
---

# WhatsApp Service Messages

Service messages are free-form WhatsApp Business Platform messages sent through the Messages API during an open 24-hour customer service window. They do not require template pre-approval, but they can only be sent to opted-in users while the customer service window is open.

## Customer Service Window

| Rule | Detail |
| --- | --- |
| Window trigger | A WhatsApp user messages or calls the business. |
| Window length | `24` hours. |
| Reset behavior | If the user messages or calls again before expiry, the timer resets to `24` hours. |
| After expiry | The business can only send pre-approved template messages. |
| Opt-in requirement | The user must have opted in to receive messages from the business. |

The source notes a rare known issue where a business may receive a user message but still be unable to respond within the customer service window.

## Supported Message Types

During an open customer service window, the business can send multiple service message types through the Messages API:

| Type | Use |
| --- | --- |
| Address | Request a delivery address. |
| Audio | Send an audio file. |
| Contacts | Send rich contact details. |
| Document | Send a downloadable document. |
| Image | Send an image with optional caption. |
| Interactive CTA URL | Attach a URL to a button instead of putting a long raw URL in message text. |
| Interactive voice call | Trigger a WhatsApp call from the user. |
| Interactive Flow | Send a structured WhatsApp Flow for appointments, product browsing, feedback, leads, or other guided input. |
| Interactive list | Let the user choose from a list of options. |
| Interactive location request | Ask the user to share a location. |
| Interactive reply buttons | Present up to three predefined replies. |
| Location | Send latitude and longitude coordinates. |
| Sticker | Send static or animated stickers. |
| Text | Send a text body with optional link preview. |
| Video | Send a video with optional caption. |
| Reaction | React to a previous user message with an emoji. |

Commerce messages are a separate interactive pattern used with product catalogs.

## API Shape

All service-message sends use the Messages API endpoint:

```text
POST /<WHATSAPP_BUSINESS_PHONE_NUMBER_ID>/messages
```

The request body uses a common envelope where `type` selects the message type and the matching object contains that message's contents:

```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "<WHATSAPP_USER_PHONE_NUMBER>",
  "type": "<MESSAGE_TYPE>",
  "<MESSAGE_TYPE>": {}
}
```

The API response only confirms that Meta accepted the send request. Delivery success must be tracked through `messages` webhooks.

## Operational Notes

| Area | Rule |
| --- | --- |
| Message quality | Based on the last seven days of user feedback, weighted by recency. Signals include blocks, reports, mutes, archives, and block reasons. |
| Phone number formatting | Include plus sign and country code. Omitting them can cause the business phone number's country code to be prepended, which may misdeliver the message. |
| Media caching | Media sent by `link` is cached by Cloud API for `10` minutes. Add a random query string to force a new fetch. |
| Multiple-message ordering | Delivery order is not guaranteed to match API request order. Wait for a `delivered` status webhook before sending the next message when sequence matters. |
| Default TTL | All messages except authentication templates use `30` days; authentication templates use `10` minutes. |

## Message Quality Guidance

The source recommends sending only opted-in, useful, personalized messages; following WhatsApp Business Messaging Policy; avoiding open-ended welcome or introductory messages; avoiding excessive daily volume; and optimizing message content and length. High-traffic numbers can see quality changes within short intervals.

## See Also

- [WhatsApp Business Platform Pricing](../../product/whatsapp/whatsapp-business-platform-pricing.md)
- [WhatsApp Flows](../../product/whatsapp/whatsapp-flows.md)
