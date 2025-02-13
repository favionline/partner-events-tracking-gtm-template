# FAVI Partner Events Tracking

## Implementation guide - Google Tag Manager

### Integration types

There are three main ways how to integrate *FAVI Partner Events Tracking* into your e-shop:

1. [direct client-side (frontend) integration of FAVI script (called C2S)](https://github.com/favionline/partner-events-tracking/blob/master/doc/implementation-guide/client-direct.md),
2. client-side (frontend) integration using Google Tag Manager (called GTM),
3. [server-to-server integration using API (called S2S)](https://github.com/favionline/partner-events-tracking/blob/master/doc/implementation-guide/server-to-server.md).

One of the client side solutions is needed to measure campaign performance - such as attribution.

The S2S solution is most reliable and secure, but requires direct integration into your backend code, which is not feasible for everyone.

You can pick any solution, or even combination of them - for example it is best to use a client side solution to create an order, but you can send more (or update) information about it later using the S2S solution.

This guide is for the GTM integration, but you can have a look at the other ones using appropriate links.

### Adding *FAVI Partner Events Tracking* into your GTM

We have prepared a GTM tag template, which is published in the GTM *Community Template Gallery*, so it is ready to be used - you will just provide the needed data.

Use the template by following these steps:

1. add new tag to your GTM container and select `Discover more tag types in the Community Template Gallery`,

![Add new tag -> Discover more tag types in the Community Template Gallery](1-community-template-gallery.png)

2. find the `FAVI Partner Events Tracking` template and select it,

![Import Tag Template -> `FAVI Partner Events Tracking`](2-find-template.png)

3. add the template to your workspace,

![Add to workspace](3-add-to-workspace.png)

4. configure the tag based on the event type (see below).

![Tag configuration](tag-configuration-page-view.png)

You will need to go through steps 1-3. only the first time, then you can find the tag template in the `Custom` section.

![Add tag](add-tag.png)

### User consent

To measure campaign performance (e.g., attribution), we store a cookie in the user's browser. Consent to store this cookie is given (or declined) on the FAVI website, and this choice is passed to this tracking code. If consent is not given, no cookie is created.

This means that no further consent is required on your page, and triggering tags based on this template should not be conditional on any other consent.

It's best to set `Advanced Settings` / `Consent Settings` to `No additional consent required` for each tag based on this template. If you leave the default value (`Not set`), GTM might make changes in the future that could prevent correct event tracking.

![Consent Settings](tag-configuration-consent.png)

### How to verify that the events are triggering correctly

The best way to verify that the events are triggering correctly is to turn on the `Debug Mode` in the tag configuration.

Then you will see all the events that are triggered and registered by the tracking script in the browser console / developer tools (usually F12 in browser).

If GTM tags are firing, but you see no events being logged, you should see error messages in web console.

You should also see network requests being sent to `https://partner-events.favi.{XX}`, where `{XX}` is the country where your e-shop is registered. But not all triggered events are sent to the backend (see details in the description of particular events), so using the `debug` mode is more reliable.

Chrome and Chrome-based browsers may not show responses for failed requests, so in order to see helpful error messages from the API, please try using Firefox or the most reliable way is using a "copy as cURL" feature on the request.

If your e-shop is using Content Security Policy, see the dedicated section below.

### `pageView` event

This event should be triggered for every page shown on your web (except for pages that have a more specific view event type).

By default, the event sends current URL, so it is also important to trigger it before the URL is manipulated (for example removing analytical/marketing parameters).

If you need to trigger the event after the URL is manipulated, please store the original URL and pass it to the event explicitly (see event properties below).

If you show multiple pages that have their own URL using client side code, then you should send a view event for each of them.

This data (alongside the other events) is used to measure and optimize campaign performance.

Note that not all triggered events result in the event being sent to the server - if there is no information related to the campaign performance evaluation, the request is not sent. But if you turn on the `debug` mode, you will see even these events being logged in the console, to be able to check that the implementation is correct.

![Tag configuration](tag-configuration-page-view.png)

List of configurable fields for the event:

* `URL`
    * optional, string
    * URL must include analytical/marketing query parameters
    * if URL is not set, current URL will be used

Fields are configured using a GTM variable with appropriate data. In GTM there are multiple ways how to pass this data, typically it can be based on a *Data Layer Variable*, a *Custom JavaScript* variable or any other option. You might even already have a variable with appropriate data (used for a different service).

### `productDetailView` event

This event is a specialized version of the `pageView` event, so everything from the `pageView` event also applies here. This event should be triggered instead of (not in addition to) the `pageView` event on a product detail page, which allows for more precise campaign evaluation.

![Tag configuration](tag-configuration-product-detail-view.png)

List of configurable fields for the event:

* `URL`
    * optional, string
    * URL must include analytical/marketing query parameters
    * if URL is not set, current URL will be used
* `Product ID`
    * required, string
    * your internal product ID, the same you are sending to FAVI through XML feed

Fields are configured using a GTM variable with appropriate data. In GTM there are multiple ways how to pass this data, typically it can be based on a *Data Layer Variable*, a *Custom JavaScript* variable or any other option. You might even already have a variable with appropriate data (used for a different service).

### `createOrder` event

Use this event after an order is created on your e-shop with products that you advertise on FAVI.

Also use this endpoint to update the same order (for example when expected delivery date changes) or if you want to provide additional data (for example add customer e-mail for review). Always send complete information for the particular time, not only changed properties.

If you send `customer` information (see below), FAVI will send a review request to the customer.

![Tag configuration](tag-configuration-create-order.png)

In `createOrder Event Data` field, you need to provide a GTM variable with order data. In GTM there are multiple ways how to pass this data, typically it can be based on a *Data Layer Variable*, a *Custom JavaScript* variable or any other option. You might even already have an order variable ready used for a different service.

**In any case please make sure the format of the object returned by the selected variable matches the format described below.**

`order` is an object with the following format:

* `orderId`
  * required, string
  * your internal order ID
* `orderItems`
  * required, array of `orderItem` objects (see below)
* `totalAmountWithoutVat`
  * optional, `totalAmountWithoutVat` object (see below)
  * total value of the order, including prices of all items, services, discounts, delivery, etc.
    * without VAT
* `customer`
  * optional, `customer` object (see below)
* `expectedDeliveryDate`
  * optional, string
  * format conforming to `Y-m-d` according to [PHP date format](https://www.php.net/manual/en/datetime.format.php#refsect1-datetime.format-parameters)
  * will be used to decide when to send review request to customer

`orderItem`:

* `product` - required, object:
  * `id`
    * required, string
    * your internal product ID, the same you are sending to FAVI through XML feed
  * `name`
    * required, string
    * your product name, must match the name you are using in XML feed provided to FAVI

`totalAmountWithoutVat`:
  * `value`
    * required, string, format of string is either integer or decimal number
      * `.` used as decimal separator
      * no spaces or other characters
      * there can be no leading zeroes for the whole number part and no trailing zeroes for the fraction part (except for `0.1`, `1.0`, etc.)
    * number of decimal spaces is limited up to the number of digits the currency supports according to [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217)
  * `currency`
    * required, string
    * 3-letter code according to [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217)

`customer`:

* `email`
  * required, string
  * will be used to send review request to customer
* `name`
  * optional, string
  * will be used to show name in the review, customer will have to confirm sharing it publicly, but this eases the review filling process

If you have this data already in GTM, but the format does not match, you can always create for example a new GTM *Custom JavaScript* variable and map the data to the required format. You can even compose multiple existing variables this way.

For example if you were using a flatter structure for the order object, you can convert it to the required format by this *Custom JavaScript* variable:

```js
function() {
    var order = {{Order}};

    return {
        orderId: order.id,
        orderItems: order.items.map(function (item) {
            return {
                product: {
                    id: item.id,
                    name: item.name,
                },
            };
        }),
        totalAmountWithoutVat: {
            value: item.price,
            currency: 'CZK',
        },
        customer: {
            email: order.email,
            name: order.name,
        },
        expectedDeliveryDate: order.expectedDeliveryDate,
    };
}
```

### Content Security Policy (CSP)

If your e-shop is using CSP, you will have to allow *FAVI Partner Events Tracking* to do its job by allowing sending tracking data to `https://partner-events.favi.{XX}` (`connect-src`), where `{XX}` is the country where your e-shop is registered.

Also, if you are not using the recommended setup of GTM with CSP using nonce - https://developers.google.com/tag-manager/web/csp - you might need to also allow loading tracking script from `https://partner-events.favicdn.net` (`script-src`).

If you did not use *Custom JavaScript* variables before and are using now, read https://developers.google.com/tag-manager/web/csp#custom_javascript_variables .

### Contact

If you have any questions regarding the implementation or need any help, please ask our account managers (you can find your contact in your partner dashboard after logging in on FAVI).
