---
"Source URL": "https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing"
Collected: "20/05/2026"
Published:
Title: "Pricing on the Whatsapp Business Platform"
---
WhatsApp Business Platform

Updated: Mar 30, 2026

This document explains how pricing works on the WhatsApp Business Platform.

To align with industry-standards, effective July 1, 2025, Meta now charges on a **per-message basis**:

- You are only charged when a [template message](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/overview) is delivered (`"type":"template"`).
- Rates vary based on the template’s [category](#message-template-categories) and the recipient WhatsApp phone number’s [country calling code](#country-calling-codes).

Meta provides value to businesses in several ways:

- All non-template messages are free (`"type":"text"`, `"type":"image"`, and so on). These can only be sent within an open [customer service window](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages#customer-service-windows). See [Sending messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages#sending-messages) for a list of message types.
- Utility templates delivered within an open customer service window are free.
- You can unlock [lower rates](#volume-tiers) for utility and authentication template messages, based on messaging volume.
- All messages are free for 72 hours, including template messages, if sent within an open [free entry point window](#free-entry-point-windows).

## Pricing explainer

Our pricing explainer PDF outlines how Meta charges and the various ways Meta provides value to businesses, in PDF form:

[Pricing Explainer PDF](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk25-2.fna.fbcdn.net%2Fv%2Ft39.2365-6%2F506409115_515804291560768_5477144239594007982_n.pdf%3F_nc_cat%3D111%26ccb%3D1-7%26_nc_sid%3De280be%26_nc_ohc%3DnU1DC594bGoQ7kNvwFZFhDg%26_nc_oc%3DAdprv2Q-xdRGK9mgePVV2ZM7bfbxdOYaLUJQ4JlNpAf4NpkgcOXAfNZ1PKQ9k-w9KiI%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk25-2.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af66jsXiKqNa7VdxU9ElYI3od2kk12pBLW_PeD6V4aX4Iw%26oe%3D6A27E5FA&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ)

## Message template categories

Unlike non-template messages, template messages are the only message type that can be sent outside of a customer service window. Templates can be categorized as:

- Marketing
- Utility
- Authentication

See [Template categorization](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/template-categorization) to learn how template categorization works.

### Template messages vs. non-template messages

![[571069280_1541820816822527_6328052606712176022_n.png|Diagram showing template messages vs non-template messages pricing]]
- CSW = [Customer service window](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages#customer-service-windows)
- FEP = [Free entry point window](#free-entry-point-windows)

Businesses are responsible for reviewing the category assigned to their approved templates. Whenever a template is used, a business accepts the charges associated with the category applied to the template at time of use.

## Charge example

In the example below, a business sends 4 messages to a WhatsApp user but is only charged for 2 (1 marketing charge, 1 utility charge).

| Hour | Action | Rate | Reason |
| --- | --- | --- | --- |
| 0 | You send a marketing template message to a WhatsApp user, promoting your new product. | Marketing | All marketing template messages are charged. |
| 2 | The user messages you about the product.  This opens a 24 hour [customer service window](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages#customer-service-windows) (“CSW”). | \- | Messages sent from a WhatsApp user to a business are not charged. |
| 3 | You send a [text message](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/text-messages) to the user (`"type":"text"`), describing the product in more detail. | None | All non-template messages are free within an open customer service window. |
| 4 | The user purchases the product and you send them a utility template confirming their order. | None | The CSW is still open, and utility templates sent within an open CSW are free. |
| 26 | The CSW closes, which means you can no longer send non-template messages. | \- | 24 hours have passed since the user last messaged you. |
| 30 | You send a utility template message to the user, updating them on their order. | Utility | Utility template messages sent outside of a CSW are charged, and no open CSW exists between you and the user. |

## Pricing calendar

To better enable our customers to plan and prepare for pricing updates, the following pricing calendar applies for messaging and voice on the WhatsApp Business Platform:

- Meta may update pricing only *on the 1st day of each quarter*, thus up to 4 times per year: January 1, April 1, July 1, and/or October 1.
- Meta will provide advanced notice that is better aligned to the effort required to implement different types of pricing updates, per below:

| Type of pricing update | Examples | Minimum advance notice |
| --- | --- | --- |
| **Rate card update** | Updating the [rate](#rates) for a given market–product  Updating the volume tiers for a given market–product (utility and authentication only)  Moving a market from one [pricing region](#country-calling-codes) (e.g. “Other”) to another or to be standalone on the rate card | 1 month |
| **Pricing model add-on** | Our July 1, 2025, introduction of new [volume tiers](#volume-tiers) for utility and authentication messages | 3 months |
| **Pricing model change** | Our July 1, 2025 update to our pricing model, from conversation-based pricing to per-message pricing | 6 months |

## Rates

Rates vary based on [template category](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/template-categorization), [volume tier](#volume-tiers), and [country/region](#country-calling-codes) rate.

### Rate cards and volume tiers

| Currency | Rates(CSV) | Volume tiers(CSV) | Rates and Volume tiers(PDF) |
| --- | --- | --- | --- |
| USD | [USD rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk25-2.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F660193787_1259283089081048_6880423588502598896_n.csv%3F_nc_cat%3D102%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3DpLIBgR_TQscQ7kNvwGgzSf3%26_nc_oc%3DAdpYiZNlAjQsr9KtSotuAbXXR5_YVaTdEgP7wSYiUnCeYLamsq-NDbCSUU9Kf2PMRQY%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk25-2.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af49Lj_RvoX2Chqcyu86EDqfPDHbLX5lqnPyvKcd8NlB5A%26oe%3D6A137AD6&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| AED | [AED rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk25-2.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F658872504_1454892143039475_7897061783074927263_n.csv%3F_nc_cat%3D109%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3DyhKLniIhsP4Q7kNvwEp6jzh%26_nc_oc%3DAdrwj0ZzXQntgn6RKSLTxMQlInAgYaHb6wCHBsAVS6MSerfGdQPn0VH2sH9_IsO0X2E%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk25-2.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af7eGCOWovg91miagcwTkJT4wPhLuTo_wS_tppF2AT9eYw%26oe%3D6A135473&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| ARS | [ARS rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk40-1.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F661003297_919166927798549_7511716460360617532_n.csv%3F_nc_cat%3D106%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3Do9IrPlXclDEQ7kNvwF5cYTW%26_nc_oc%3DAdpYYz2NA4agyK1zpyCKbZ23ZqA1rTdpWkYtU6P7IzDjwr3BPNBolW2JQNPbC6TH0eY%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk40-1.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af5-JkdgoZKEdIFS_jixbJWVAxaUM0Gb9QA_O-Uk3SL3Jg%26oe%3D6A1360D4&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| AUD | [AUD rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk40-1.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F658370781_2385139235338652_3521489198118718824_n.csv%3F_nc_cat%3D100%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3DANZUcGx5cj8Q7kNvwE6tJhc%26_nc_oc%3DAdq6OVTyJcOjFoeahET4sXsfP0iew_MFlqe4LHUKqLQeUg9H9NlItUhB-6Zz4BilUcc%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk40-1.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af5Hx61zuGfPrGufPlK3juzU8Ha4Odp1otFFOuvOrrqqAQ%26oe%3D6A136A2F&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| CLP | [CLP rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk40-1.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F659430130_1168908098577566_1088673212819614411_n.csv%3F_nc_cat%3D107%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3Dq9shHyqLB0sQ7kNvwFklynm%26_nc_oc%3DAdqKcsUcwpA_f3rqoOZh3epolgqi-RFgR7W4LA_6GDe3ZQ1NSikIJs02-E9coLJGbro%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk40-1.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af57Dvu3sotHYAEK8S3B1zcL2lrVLjFCCoCI2wNk6lnsnA%26oe%3D6A13843C&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| COP | [COP rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk25-2.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F658977438_3239912412843347_549063928521883057_n.csv%3F_nc_cat%3D105%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3Duv19JMeVPhQQ7kNvwHaQjr7%26_nc_oc%3DAdoobcKUoSdqt8G4Ud1eoSlEaMNpeJwM9u4wQLZXHkWO5J4CiJjRvqfWgovgOEqs0AQ%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk25-2.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af6M66R_9NK4hdbTeWoO2WvOiGMdOF49sKQOV5PuqbOWcw%26oe%3D6A137C78&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| EUR | [EUR rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk40-1.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F662338109_756710277374713_1187618423821131958_n.csv%3F_nc_cat%3D100%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3DVbTefhReptoQ7kNvwF316en%26_nc_oc%3DAdriZ23kH_lqQqcanNObZMQzULhEDdO4a3H_Kwmf5gDTzlubg-6bMYO9jtFzPGdog1A%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk40-1.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af5ICceuVMblU-j774aI_Tdt3fF0GgLbTjKs8GPh-q7KsA%26oe%3D6A1357C0&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| GBP | [GBP rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk40-1.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F659068780_1265335938476075_2412997692082178180_n.csv%3F_nc_cat%3D107%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3DNdmI0annNbwQ7kNvwHRJbn4%26_nc_oc%3DAdoFm9ZmLuYeKMkto3M1qzu9a6lOi61bi5no2PBNwjf_JqFHGqHQi8zuNMLEwQT989k%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk40-1.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af4bYIX4yoOU8wNilbJ6QMF3fnz9I2ZhMgFHm8yTyTkrKg%26oe%3D6A1362F4&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| IDR | [IDR rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk40-1.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F660036844_979850801088657_7934439379303390406_n.csv%3F_nc_cat%3D100%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3DJ3dhNtu07VcQ7kNvwHHDSNr%26_nc_oc%3DAdomaFR97cIxzvlH8kEm1mS-S5UUzZkUm2VMOkxiN_-fpnQl6JtElqaGc0zSxHPuOhA%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk40-1.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af4CK7cDb3FftVDtCsq-7_xrQWBmuxenssj4FBQvBEEh9Q%26oe%3D6A137E01&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| INR | [INR rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk40-1.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F658200936_1281806530574956_1771789765840398787_n.csv%3F_nc_cat%3D107%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3DxPz38SAFLi0Q7kNvwE5e6Ae%26_nc_oc%3DAdrCDTfl_GqQldeDOlGyWEdTfNWaiuzCo1GqnjnpjTInMWOvrA0TxT1n3z9CGnlRJMM%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk40-1.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af7IfQG6DLJiszeTJf-s2eHiygvErfA7r-ByJqk4IxTXCQ%26oe%3D6A13678B&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| MXN | [MXN rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk25-2.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F658386158_1278060070319428_6885687788221735893_n.csv%3F_nc_cat%3D102%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3DnpnDOg4h7HwQ7kNvwEegi4L%26_nc_oc%3DAdrfAvjDwdaKLqbxon3-f1YgEw3hpJfiUAzd3TNCgO_BLmfCG9DnD3gZgAxMAdw-y8k%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk25-2.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af5JxPkdD_jIHFfdR_e-5pVGS_FSnfJnZ6JyiLp8vqD9yA%26oe%3D6A136CBE&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| MYR | [MYR rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk40-1.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F658376025_1003284798690933_380775706448839675_n.csv%3F_nc_cat%3D110%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3DNgFqyf3jtHwQ7kNvwEEHGer%26_nc_oc%3DAdrhuMpjDg-36-gbsw7V_zsHJOEbgT7OePs1dY_o5FfsG3VUnf25p07h3YDHqWbM8SQ%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk40-1.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af6cDSIVZYVIPZzXMq4xwj0E6mvX40GhSp170F1F_Napzw%26oe%3D6A138B99&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| PEN | [PEN rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk40-1.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F657709960_929399026516735_8296747819736583770_n.csv%3F_nc_cat%3D108%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3D1h2n7B4sXgUQ7kNvwFRuA4t%26_nc_oc%3DAdpBq6sg-kldxMMfEtPBlwfYAMrk6vBNlttBuJ5vzr-73PDDMESHx7A7nWngfvltw_4%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk40-1.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af5SnHxRwZ_e2eQlJnrPArHowjdu9hc7TxxUWsqAHBtPjg%26oe%3D6A1368DF&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| SAR | [SAR rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk40-1.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F660043621_2237704266761118_7421728136795961522_n.csv%3F_nc_cat%3D110%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3D4Ok0o02XTpUQ7kNvwFgKIbk%26_nc_oc%3DAdocjj_LU0Hm1b9utFQDjmE_cc2hsXn68wZevjOwj7Jz0wdph_I3u6SeEuO5AiVhuf8%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk40-1.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af7Jzmw3x50aFONfgTaUwVL4hI5zjefre0qSAcHgJfcVYg%26oe%3D6A136CE8&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |
| SGD | [SGD rates](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk40-1.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F658854530_802348715803555_6244498368048976092_n.csv%3F_nc_cat%3D106%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3DC6bUKXpWPYwQ7kNvwH44xjG%26_nc_oc%3DAdrxHz2lPuuF47O_DNPhGIKknbc0y9xZ3EqVbDOILVuh-H2goQ5nnzJnuFM3RsKLFys%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk40-1.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af7hw0_qZQDFMKw6kO7_W3FdumoGwDkdhxKWX8FM6Bl5Og%26oe%3D6A1376F2&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ) |  |  |

### Updates to rate cards

Below represents future updates to our rates. See our [rate cards](#rate-cards-and-volume-tiers) above for current rates.

**Rate cards effective July 1, 2026**

**Billing localization for India and Brazil**

Meta is introducing billing localization to help eligible customers to better manage costs of messaging amidst currency fluctuations. This will apply to the markets below, and specifically to Solution Partners and directly-integrated clients whose Sold-To country in Billing Hub is a market below:

- [India⁠](https://www.facebook.com/business/help/2301408543603167) – As of January 1, 2026.
- [Brazil⁠](https://www.facebook.com/business/help/4344414845795884) – As of July 1, 2026.

**Previous updates**

- Effective April 1, 2026 at 12am by WhatsApp Business Account timezone, the rate updates below applied:
	- Saudi Arabia – Higher marketing message rate.
		- India – Higher authentication-international rate.
		- Pakistan – Higher utility and authentication rates. No change to the authentication-international rate.
		- Turkey – Lower utility and authentication rates.
		- 8 new billing currencies introduced: ARS (Argentina), CLP (Chile), COP (Colombia), MYR (Malaysia), PEN (Peru), SAR (Saudi Arabia), SGD (Singapore), AED (United Arab Emirates).
- Effective January 1, 2026 at 12am by WhatsApp Business Account timezone, the rate updates below applied:
	- India - Higher marketing rate.
		- France, Egypt - Lower marketing rates.
		- North America - Lower utility and authentication rates.
- Effective October 1, 2025 at 12am by WhatsApp Business Account timezone, the rate updates below applied:
	- Colombia – Higher utility and authentication rates.
		- Mexico – Lower marketing rates.
		- United Arab Emirates – Higher marketing message rate.
		- Argentina, Egypt, Saudi Arabia – Lower utility and authentication rates.
		- Zimbabwe is mapped to our “Rest of Africa” region vs. “Other”. Messages delivered to WhatsApp users with a +263 country calling code (Zimbabwe) will be charged “Rest of Africa” rates.
- Effective July 1, 2025 – Lower utility and authentication message rates across several markets, to ensure pricing is on-par to alternate channels for these use cases. Marketing conversation rates became marketing message rates.
- Effective April 1, 2025 – Lowered [authentication-international conversation rates](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/authentication-international-rates) for Egypt, Nigeria, Pakistan, and South Africa.
- Effective February 1, 2025 – Lowered [authentication conversation rates](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing#rates) for Egypt, Malaysia, Nigeria, Pakistan, Saudi Arabia, South Africa, and the United Arab Emirates.
- Effective November 1, 2024 – [Service conversations](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/conversation-based-pricing#service-conversations) are now free for all businesses.
- Effective October 1, 2024 – Updated [marketing conversation rates](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing#rates) in India, Saudi Arabia, the United Arab Emirates, and the United Kingdom.
- Effective August 1, 2024 – Lowered [utility conversation rates](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing#rates).

### Authentication-international rates

Specific countries have an authentication-international rate. Our rate cards reflect these rates. See [Authentication-International rates](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/authentication-international-rates) to learn about these rates and if they apply to you.

### Country calling codes

Charges for conversations are based on the country calling code of the recipient WhatsApp phone number. The table below shows how Meta maps country calling codes to countries or regions. If a country is not listed below, it maps to **Other**.

This information is also available in a CSV file:

[Country Calling Codes and Regional Rate Mapping CSV](https://l.facebook.com/l.php?u=https%3A%2F%2Fscontent.fcgk25-2.fna.fbcdn.net%2Fv%2Ft39.8562-6%2F559604326_1510649803514615_3972087685039081235_n.csv%3F_nc_cat%3D111%26ccb%3D1-7%26_nc_sid%3Db8d81d%26_nc_ohc%3DF3NCovl_4VYQ7kNvwEescUg%26_nc_oc%3DAdodSnbUQdKog28khi6muHkPbSAcjFzz1Q3auWw_kj-ExoDHvlqMWBmai4e2lh0APLE%26_nc_zt%3D14%26_nc_ht%3Dscontent.fcgk25-2.fna%26_nc_gid%3Dc2eHU4EzBTa9fahJyFZ7Aw%26_nc_ss%3D7b289%26oh%3D00_Af4q8b4QSO3U2SrIFy0p9yhkIkldlzpgJ08sXY0lN3z7JA%26oe%3D6A137A53&h=AUBGxq7NdaQC5EerVGdyh8ArQUwN_0SwSFtzH8QMEZneayxyPDkXRBoiPVRM9FZjEYhlaDhbz-TrHn5h7EynC1hZv2WhOjP6R0wfctT-EpOLAQ2AuEh6ODDO927RdZy0qdvWdHRJ7_qK38y8GIcjdLwVPmQ)

| Markets | Calling Code   (and network prefix if applicable) |
| --- | --- |
| Countries  Argentina      Brazil      Chile      Colombia      Egypt      France      Germany      India      Indonesia      Israel      Italy      Malaysia      Mexico      Netherlands      Nigeria      Pakistan      Peru      Russia      Saudi Arabia      South Africa      Spain      Turkey      United Arab Emirates      United Kingdom | 54      55      56      57      20      33      49      91      62      972      39      60      52      31      234      92      51      7      966      27      34      90      971      44 |
| North America  Canada      United States | 1      1 |
| Rest of Africa  Algeria      Angola      Benin      Botswana      Burkina Faso      Burundi      Cameroon      Chad      Republic of the Congo (Brazzaville)      Eritrea      Ethiopia      Gabon      Gambia      Ghana      Guinea-Bissau      Ivory Coast      Kenya      Lesotho      Liberia      Libya      Madagascar      Malawi      Mali      Mauritania      Morocco      Mozambique      Namibia      Niger      Rwanda      Senegal      Sierra Leone      Somalia      South Sudan      Sudan      Swaziland      Tanzania      Togo      Tunisia      Uganda      Zambia      Zimbabwe | 213      244      229      267      226      257      237      235      242      291      251      241      220      233      245      225      254      266      231      218      261      265      223      222      212      258      264      227      250      221      232      252      211      249      268      255      228      216      256      260      263 |
| Rest of Asia Pacific  Afghanistan      Australia      Bangladesh      Cambodia      China      Hong Kong      Japan      Laos      Mongolia      Nepal      New Zealand      Papua New Guinea      Philippines      Singapore      Sri Lanka      Taiwan      Tajikistan      Thailand      Turkmenistan      Uzbekistan      Vietnam | 93      61      880      855      86      852      81      856      976      977      64      675      63      65      94      886      992      66      993      998      84 |
| Rest of Central and Eastern Europe  Albania      Armenia      Azerbaijan      Belarus      Bulgaria      Croatia      Czech Republic      Georgia      Greece      Hungary      Latvia      Lithuania      Moldova      North Macedonia      Poland      Romania      Serbia      Slovakia      Slovenia      Ukraine | 355      374      994      375      359      385      420      995      30      36      371      370      373      389      48      40      381      421      386      380 |
| Rest of Western Europe  Austria      Belgium      Denmark      Finland      Ireland      Norway      Portugal      Sweden      Switzerland | 43      32      45      358      353      47      351      46      41 |
| Rest of Latin America  Bolivia      Costa Rica      Dominican Republic      Ecuador      El Salvador      Guatemala      Haiti      Honduras      Jamaica      Nicaragua      Panama      Paraguay      Puerto Rico      Uruguay      Venezuela | 591      506      1 (809, 829, 849)      593      503      502      509      504      1 (658, 876)      505      507      595      1 (787, 939)      598      58 |
| Rest of Middle East  Bahrain      Iraq      Jordan      Kuwait      Lebanon      Oman      Qatar      Yemen | 973      964      962      965      961      968      974      967 |
| Other  All other countries | Varies by country |

## Volume tiers

You can unlock lower utility and authentication rates based on the number of messages you send in a month.

### Tiering accrual

- **Messages are aggregated at the business portfolio level, across all WhatsApp Business Accounts (WABAs) owned by the portfolio** — To determine what tier rates may apply in a given month for a given market–category pair, Meta aggregates messages across all of a business portfolio’s WABAs for each market-category pair (e.g., Brazil–authentication, Brazil–utility, India–authentication, and so on).
- **Only messages that are charged count toward the tiers** — Thus, the following messages do not count:
	- Utility templates delivered to WhatsApp users within an open customer service window.
		- Utility templates delivered within a [free entry point window](#free-entry-point-windows).
- **Volume tiers will be determined solely by Meta** — All insights data is approximate due to small variations in data processing. Undue reliance should not be placed on insights data.

### Key dynamics

- **Tiers are market–category specific** — Volume tiers are aligned to our rate cards and differ by market (e.g., Brazil or Rest of Latin America) and category (utility, authentication).
- **Rates are tier-specific** — When a business sends enough messages at a given market–category pair to reach the next tier, they unlock the rate of the next tier, specifically for messages in that tier. This rate applies across all of their WABAs.
- **Tiers reset monthly** — At the start of the next month (12am WABA timezone), message count resets to 0 and businesses begin to accrue messages toward that month.

### Volume tiers examples

The table below is illustrative and only highlights the dynamics of volume tiers. Please refer to our [rate cards](#rate-cards-and-volume-tiers) to see the rates charged.

![[502514970_1202956304344062_4629097874159039633_n.png|Table showing volume tier rate examples]]

Below are several examples to highlight how the tiers work and what is charged in a given month, for a given market–category. These examples refer to the illustrative table above:

Example 1: A business that sends a total of B authentication messages in a month to India is charged:

- List rate for the first A messages.
- Tier rate 1 for messages A+1 to B.
- Total charges for that month = Rate per tier 𝗑 messages in each tier.

Example 2: A business that starts to be charged our authentication-international rates on the 15th day of the month:

- Day 1 to 14 of that month: Volume tiers apply on the authentication rate.
- Day 15 onward of that month: Volume tiers apply on the authentication-international rate, with messages continuing to accrue in that month. For example, if a business has already reached the Tier 2, the business would be charged Tier 2’s authentication-international rate:

Example 3: A business has 3 WABAs sending authentication messages to India. For WABA A, it is still July 31 based on their timezone. For WABAs B and C, it is already August 1 based on their timezone. For July, the business is already being charged Tier Rate 1.

- The business portfolio will be accruing toward tiers for both July (via WABA A) and August (via WABAs B, C) for a period of time.
- The business can reach the next tier for July, via WABA A. If that happens, messages for the remainder of July for WABA A will be charged Tier Rate 2.

Example 4: A business has 3 WABAs, integrated across 2 solution providers. Provider 1 sends the first B messages in a given month, and provider 2 starts sending messages as of when the business is in the 3rd tier. The business does not send enough messages that month to reach the next tier. What we would charge each provider:

- Provider 1: List rate for A messages, then Tier Rate 1 from A+1 to B, and Tier Rate 2 for B+1 to C.
- Provider 2: Tier Rate 2 across all of their messages.

### Tiering webhooks

Starting October 1, 2025, an [account\_update](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/reference/account_update) webhook with `event` set to `VOLUME_BASED_PRICING_TIER_UPDATE` will be triggered when your WhatsApp Business Account reaches a new volume tier, in any market, in a given month. This complements our [pricing\_analytics](https://developers.facebook.com/documentation/business-messaging/whatsapp/analytics#pricing-analytics) endpoint, which will continue to provide intra-month tiering progress and tiering information for delivered messages.

Example webhook:

```
{
  "object": "whatsapp_business_account",
  "entry": [
    {
      "id": "102290129340398",
      "time": 1743451903,
      "changes": [
        {
          "value": {
            "volume_tier_info": {
                "tier_update_time": 1743451903,
                "pricing_category": "UTILITY",
                "tier": "25000001:50000000",
                "effective_month": "2025-11",
                "region": "India"
            },
            "event": "VOLUME_BASED_PRICING_TIER_UPDATE"
          },
          "field": "account_update"
        }
      ]
    }
  ]
}
```

- `tier_update_time` tells when your WABA reached a higher volume tier (Unix timestamp).
- `pricing_category` tells you the template category for which your new volume tier rate applies.
- `tier` tells you the new volume tier’s lower and upper bounds.
- `effective_month` tells you the month in which your new volume tier rate is in effect.
- `region` tells you the WhatsApp user country/region for which your new volume tier rate applies.

Note that it’s possible for multiple [account\_update](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/reference/account_update) webhooks to be triggered that describe the same tier switch event. In these cases, use the webhook with the smaller `tier_update_time` Unix timestamp as the official webhook.

### Tiering analytics

You can get [volume tier information](https://developers.facebook.com/documentation/business-messaging/whatsapp/analytics#volume-tier-information) via [template analytics](https://developers.facebook.com/documentation/business-messaging/whatsapp/analytics#template-analytics).

## Free non-template messages

Non-template messages, which can only be sent within an open [customer service window](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages#customer-service-windows), are free. These messages will have `type` set to `free_customer_service` in the `pricing` object of status [messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/reference/messages/status) webhooks:

```
"pricing": {
  "billable": false,
  "pricing_model": "PMP",
  "type": "free_customer_service",
  "category": "service"
}
```

## Free utility template messages

Utility template messages sent within an open [customer service window](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages#customer-service-windows) are free. These messages will have `type` set to `free_customer_service` and `category` set to `utility` in the `pricing` object of status [messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/reference/messages/status) webhooks:

```
"pricing": {
  "billable": false,
  "pricing_model": "PMP",
  "type": "free_customer_service",
  "category": "utility"
}
```

### Edge case

If you send a message to a WhatsApp user prior to July 1, 2025 (which is when Meta switched from conversation-based pricing to per-message pricing), a utility conversation is opened between you and a user that spans the switch to per-message pricing (the conversation was opened before the switch but won’t close until after the switch). In this case, utility templates sent to the user after the switch while the conversation is open will be free, but attributed to the open conversation. In status [messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/reference/messages/status) webhooks, these messages will have a `pricing_model` of `CBP` and the utility conversation ID will be assigned to `conversation.id`. Once the conversation closes, subsequent utility messages will use per-message pricing, which will be reflected in new webhooks.

## Free Entry Point windows

If a WhatsApp user messages you via a Click to WhatsApp Ad or Facebook Page Call-to-Action button using a device running our Android or iOS app (our desktop and web apps are not supported):

- A 24-hour [customer service window](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages#customer-service-windows) is opened (as normal).
- If you respond within 24 hours using any type of message, the message will be free, and a Free Entry Point (“FEP”) window will be opened, starting from the time when you responded.

FEP windows remain open for 72 hours. While open, you can send any type of message to the user at no charge. Note, however, that the customer service window is independent of the FEP window, so if the customer service window closes, you will only be able to send template messages.

Starting in 2026, businesses integrated into Marketing Messages API for WhatsApp can choose to set a [max-price](https://developers.facebook.com/documentation/business-messaging/whatsapp/marketing-messages/pricing) per marketing message delivery; when a max-price is set, Meta will charge that max-price or lower for delivery.

## New pricing policy for AI Providers leveraging WhatsApp Business Platform

Click [here](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/ai-providers) to learn more about our new pricing policy for “AI Providers” leveraging WhatsApp Business Platform, which is effective February 16, 2026 and updated as of May 12, 2026.

## Analytics

Use the [pricing\_analytics field](https://developers.facebook.com/documentation/business-messaging/whatsapp/analytics#pricing-analytics) to get per-message pricing breakdowns and tiering information for delivered messages.

## Webhooks

Billable messages have `type` set to `regular` in the `pricing` object of status [messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/reference/messages/status) webhooks:

```
"pricing": {
  "billable": true,
  "pricing_model": "PMP",
  "type": "regular",
  "category": "<PRICING_CATEGORY>"
}
```

The `<PRICING_CATEGORY>` tells you what rate was applied (e.g. `marketing`). See the status messages webhook reference for a list of possible values.

Note that currently, tiering information is not included in any webhooks. Use the [pricing\_analytics field](https://developers.facebook.com/documentation/business-messaging/whatsapp/analytics#pricing-analytics) to get tiering information for delivered messages.

## Billing

## WhatsApp Business Calling API pricing

The WhatsApp Business Calling API has different pricing. See our [Calling API pricing document](https://developers.facebook.com/documentation/business-messaging/whatsapp/calling/pricing) to learn more.

## Conversation-based pricing

[Conversation-based pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/conversation-based-pricing) is deprecated. It was replaced with per-message pricing on July 1, 2025.

Did you find this page helpful?