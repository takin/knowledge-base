---
"Source URL": "https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/gettingstarted/purchase-intent"
Collected: "20/05/2026"
Published:
Title: "Use Case Guide: Collect Purchase Interest"
---
WhatsApp Flows

Updated: Jul 29, 2024

## Intro and Overview

![[651706993_1459945709197412_5831719837776262328_n.png|Image]]

It’s easier than ever to collect information from your customers, to understand their preferences and collect opt in for promotions ahead of the sales season. With WhatsApp Flows, your customers can provide their details and interests in a fast and simple way, without the need to speak with an agent.A business can leverage this information to drive targeted promotions and purchases.

In this guide, we will walk through the entire process to build a Flow for ‘Collect Purchase Interest’ use case. The templates here can be adapted to suit your use case.

Flows we will build will demonstrate how you can:

- Collect relevant personal information from a user
- Allow users to select products or services they are interested in, which can be leveraged in future promotional campaigns.

This template can be further adapted for any use case where you want to collect information from your customers to better understand their attributes and preferences (i.e. registering for a webinar, event, newsletter etc.).

## Getting Started

To follow this guide, ensure you have:

## Flows JSON Template

Flow JSON

Preview

Run

Settings

JOIN\_NOW

![[Ei8b9RGc2VT.svg]]

---

Learn more

### Create new flow from a template

1. In the [Flows section of WhatsApp Manager⁠](https://business.facebook.com/wa/manage/flows/) click on the **Create Flow** button in the top right corner.
2. In the Create page, fill in the details for the pre-approved loan Flow:
	- **Name** - Type *Collect Purchase Intent*, or choose another name you like.
		- **Categories** - Select *Lead generation*.
		- **Template** - Choose *Collect purchase intent*. You can further customize the template to suit your use case.
3. Click **Create** to create flow.

You can preview the Flow on the right of the Builder UI.

The Flow remains in the draft state as you edit it. You can share it with your team for testing purposes only. To share it with a large audience, you’ll need to publish it. However, you can’t edit the Flow once you [publish](#publishing).

**See also**

- [Flow JSON Overview](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/flowjson)

## Testing and Debugging

### Debug flow using the interactive preview

After you complete the configurations, toggle the interactive preview in the WhatsApp Builder UI to test the Flow.

1. Trigger the interactive preview by clicking on settings menu in the **Preview** section of the Flow Builder and enabling **Interactive mode** toggle.
2. In the modal that appears, select **JOIN\_NOW** as the **First Screen**.

Now, click on **Actions** tab at the bottom of the code editor in Builder. You’ll see an `navigate` action in the list. Click on it to see the details of the action.

Return back to **Preview** and proceed to complete the first screen and then click on *Continue* button to navigate to next screen. Back in **Actions** tab notice the new `navigate` action logged and details contains data passed to next screen.

Keep testing out the Flow and observe the data changes in the **Actions** tab. Similar logs will be generated when users interact with the Flow from their mobile devices.

### Send draft flow to your device

Before you publish your flow you can also send it and test it on an actual device. To send draft flow to your device, follow [instructions here](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/testingdebugging#send-draft-flow-to-your-device).

**See also**

- [Flow Testing and debugging guide](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/testingdebugging)

## Publishing

When you first created your Flow, it entered the Draft state. And as you edited and saved the modified Flow JSON content, it remained in the Draft state. You are able to send the Flow while it’s in the Draft state, but only for testing purposes. If you want to send the Flow to a larger audience, you’ll need to Publish the Flow.

You can publish your Flow once you have ensured that:

- All validation errors and [publishing checks](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/healthmonitoring#publishing-checks) have been resolved.
- The Flow meets the [design principles](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/bestpractices) of WhatsApp Flows
- The Flow complies with [WhatsApp Terms of Service⁠](https://l.facebook.com/l.php?u=https%3A%2F%2Fwww.whatsapp.com%2Flegal%2Fterms-of-service%2F%3Flang%3Den%26fbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExQVFpOW9FTkRBbko5SHFHR3NydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR5uZidkksXxShONFwLl7OBsjuX4S7zZ_UohqtM6L4hHa0u4wFFGaUCrhuhQMA_aem_KdYLqIsMvIRCZaAM7k4aHg&h=AUBtyv6iP78KPw9xpxl1WqHr-7ORNT5fyE7vHTMXxXe92Ukjwwie0BrPp0CFMuVIXeH2MNROvF0mO80pW8QQdjz0-3mviYX3icDyq2W1_xhzo8Xk-rh2nvRfB2NPNEmJ7OryhqKj), the [WhatsApp Business Messaging Policy⁠](https://l.facebook.com/l.php?u=https%3A%2F%2Ffaq.whatsapp.com%2F933578044281252%3Ffbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExQVFpOW9FTkRBbko5SHFHR3NydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR7yxr2JuMUAufcSI_-qWd6Rz2t7EzIF9IichFrfMWMCpjJuVlbzajtmJ9H-0Q_aem_iLA6Q9uKUq2p4e0Xn22pSw&h=AUBtyv6iP78KPw9xpxl1WqHr-7ORNT5fyE7vHTMXxXe92Ukjwwie0BrPp0CFMuVIXeH2MNROvF0mO80pW8QQdjz0-3mviYX3icDyq2W1_xhzo8Xk-rh2nvRfB2NPNEmJ7OryhqKj) and, if applicable, the [WhatsApp Commerce Policy⁠](https://l.facebook.com/l.php?u=https%3A%2F%2Fwww.whatsapp.com%2Flegal%2Fcommerce-policy%2F%3Flang%3Den%26fbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExQVFpOW9FTkRBbko5SHFHR3NydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR4Jx-LBc2DDnPtwpq9zNhpRjVdsoHqfp331bBTf2tV8dj9a8G-ozuAiECBBcQ_aem_18_cLjmoH2QbBk6HaXSoMw&h=AUBtyv6iP78KPw9xpxl1WqHr-7ORNT5fyE7vHTMXxXe92Ukjwwie0BrPp0CFMuVIXeH2MNROvF0mO80pW8QQdjz0-3mviYX3icDyq2W1_xhzo8Xk-rh2nvRfB2NPNEmJ7OryhqKj)

Remember, once a Flow has been published it can no longer be modified. See [Flow Status Lifecycle](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/lifecycle) for more information on the different Flow states.

To publish your Flow, open the **three dot** menu to the right of the **Save** button and click **Publish**. Once published, the Flow can be sent to anyone!

## Sending

You can send your WhatsApp Flow as:

- **[Template messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/sendingaflow#templatemessages)** - these do not require a 24-hour customer service window to be open between you and the message recipient before the message can be sent.
- **[Interactive Flow messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/sendingaflow#userinitiated)** - these can only be sent to a user when a customer service window is open between you and the user.

[Learn more about sending your Flow](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/sendingaflow)

## Receiving flow response

Upon flow completion a response message will be sent to the WhatsApp chat. You will receive it in the same way as you receive all other messages from the user - via message webhook.

## Monitoring

Flow monitoring is only applicable to Flows with endpoint.

Now that you have successfully completed this guide, learn more about what you can do with this Flows in our [Guides](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides) and [Reference](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/reference) sections.

Did you find this page helpful?