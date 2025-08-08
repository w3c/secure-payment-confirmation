# Secure Payment Confirmation: Non-normative UX Guidelines

Status: This is a **draft** document without consensus.

A key step of the user journey for Secure Payment Confirmation (SPC) is the
display of and user interaction with the [transaction confirmation UX]. This
step communicates information from the caller of SPC (e.g., a merchant website,
perhaps on behalf of a payment service provider or issuing bank) to the user,
for the user to verify and ultimately sign via WebAuthn.

While web specifications do not generally make recommendations about how User
Agents implement UX, it can be useful for both integrators of SPC (e.g.,
financial organizations) and implementors of SPC (i.e., User Agents) to have
some general guidelines about the information that can be provided and will be
shown. This file aims to document those guidelines, based on implementor
experience and payment industry feedback.

## Information presented in the transaction confirmation UX

### Payee information

Relevant fields: [payeeName], [payeeOrigin]

The payee information fields are intended to communicate to the user who will
be the recipient of the funds. In most cases this would be the merchant (or
other entity) that the user has already been interacting with in the current
session. The `payeeName` field, if present, communicates a natural language
name, such as "Big Shoe Store Inc.", whilst the `payeeOrigin`, if present,
communicates the web URL at which the payee can be found, such as
"https://bigshoestore.example".

The specification allows for either one or both of the fields to be present.
Implementors may display them separately, or may combine them into a single
visual block such as `payeeName (payeeOrigin)`, for example "Big Shoe Store
Inc. (https://bigshoestore.example)".

Note: An implementor may truncate these fields in order to fit the text into
the available UX space. See issue #269 for discussions on setting normative
length limits on fields including payeeName and payeeOrigin.

### Payment instrument information

Relevant fields: [instrument.displayName], [instrument.icon], [instrument.details]

The payment instrument information fields are intended to communicate to the
user which payment instrument will be charged. Common payment instruments
include payment cards (credit, debit, or other), bank accounts, and a number of
country-specific payment methods (e.g., UPI, PIX, GiroPay, etc). SPC has no
requirement or limitation on the type of payment instrument being used or
displayed.

The `displayName` and `details` fields are used to convey textual information
about the payment instrument. As the names suggest, `displayName` should be set
to the primary name for the payment instrument, while `details` can be used to
add additional clarifying details to help the user differentiate or make
decisions.  For example, for a credit card payment instrument, `displayName`
may be set to the product name for the card (e.g., "FancyBank Infinity Card"),
whilst details might include the last four digits of the card and/or expiry
date (e.g., "1234 01/32).

Note: An implementor may truncate the `displayName` and `details` fields in
order to fit the text into the available UX space. See issue #269 for
discussions on setting normative length limits on fields including payeeName
and payeeOrigin.

The `icon` field is set to the URL of an image representing the payment
instrument. Implementors are expected to support common image formats such as
PNG or JPG. The payment instrument icon image is intended to be a quick visual
confirmation for the user, and is expected to be displayed fairly small.

Implementation specifics:

- Google Chrome's (and other Chromium-based browsers) implementation on both
  desktop and mobile platforms currently linearly scales the image (preserving
  the input aspect ratio) to fit in a 32dp x 20dp region, e.g. the display
  region has an aspect ratio of 1.6:1.
    - Given current average screen densities, we would recommend that
      integrators pass in a logo with physical pixel size at least 3x of that,
      e.g. minimal 96px by 60px.
    - The Chrome team have stated that they are interested in hearing from
      partners if a 32dp x 20dp region is too small, as it may not allow for
      detailed icons (such as full card art for credit cards).

### Payment entity logos

Relevant fields: [paymentEntitiesLogos], [paymentEntitiesLogos\[x\].url], [paymentEntitiesLogos\[x\].label]a

The payment entity logos are intended to communicate to the user the identity
of payment companies or organizations that are involved in the current
transaction, who might otherwise not be clear to the user. Examples include the
issuing bank and/or a card network in the case of credit/debit card payments,
the user's bank and a payment initiation services provider (PISP) in Open
Banking, etc.

The payment entity logos are expected to be displayed prominently within the
SPC transaction confirmation UX, but should not significantly overweight the
other information shown. Implementors are able to support any number of logos
passed within `paymentEntityLogos`, and may ignore 'extra' logos beyond what
they support displaying, but it is recommended that they support up to two
logos being shown.

Each payment entity logo has two fields.

The `label` field is used to convey a textual description of the payment entity
to the user. This field may be displayed by the implementor, or it may be used
only for accessibility purposes (e.g., for a screen reader to describe the
logo). It is recommended that this text be short but descriptive to the user,
e.g. "FancyBank logo".

The `url` field is set to the URL of a payment entity logo image. Implementors
are expected to support common image formats such as PNG or JPG. Logos should
generally be "wordmark" or combination logos, which often display more clearly
to users than 'simple' square logos. Logos should also have transparent
backgrounds, to allow them to be layered seamlessly onto implementor UX.

Implementation specifics:

- Google Chrome's (and other Chromium-based browsers) implementation currently
  supports 0-2 payment entity logos, inclusive. If more than two logos are
  provided, Chrome currently discards the third logo onwards. Logos are placed
  at the top of the SPC dialog.
    - For `label`; Chrome currently does not display this value, but does use
      it as the accessibility text for the logo.
    - For `url` (i.e., the logo itself), Chrome's implementation currently
      linearly scales the image (preserving the input aspect ratio) to fit in a
      104dp x 24dp region on mobile and a planned 188dp x 43dp on desktop, e.g.
      the display region has an aspect ratio of 4.333:1.
        - We recommend treating this as the upper-bound for aspect-ratio;
          images that have to be scaled to fit the height look better than
          images that have to be scaled to fit the width.
        - Given current average screen densities, we recommend that integrators
          pass in a logo with at least pixel size 3x of the device-pixel sizes
          mentioned above. For example, an image with ideal aspect ratio of
          4.333:1 should be at least 563px x 130px. If the image instead was
          narrower with an aspect ratio of 2.7:1, it should be at least
          350px x 130px.

### Transaction amount

Relevant fields: [details.total.amount.currency], [details.total.amount.value]

The transaction amount fields are intended to communicate to the user the value
of the purchase that they are verifying. This should be the full value,
inclusive of any shipping, taxes, etc. - such that it should match the value
that the user later observes charged against their payment instrument.

Note: The transaction amount is communicated via the top-level [Payment
Request](https://w3c.github.io/payment-request) API call, instead of as part of
the SPC method itself.

## UX differences across different screen sizes/layouts

## Appendix: Example transaction confirmation UX screens

This appendix contains some example transaction confirmation UX screens from
implementors. Where possible, fake entity logos are used, but any other images
or trademarks are used without permission and without intending to imply any
support and/or usage of SPC by those entities.

**Google Chrome on Android, with "wordmark"/combination payment entity logos**:

![Screenshot of the transaction confirmation UX for SPC in Google Chrome for Android. The transaction UX shows two wordmark logos for "Moon Bank" and "Chillipay", followed by the text "Verify its you". Below, the transaction details show that the payee is "The Design Store" at designstore.com, the payment instrument is a "Chase Freedom Flex Card" with last-four digits 1234, and the amount is $120 USD.](/images/ux-guidelines-google-chrome-android.png)

**Google Chrome on Android, with square payment entity logos**:

![Screenshot of the transaction confirmation UX for SPC in Google Chrome for Android. The image is identical to the one proceeding it, but has square logos for "Moon Bank" and "Chillipay" which are much harder to read.](/images/ux-guidelines-google-chrome-android-square-logos.png)

[details.total.amount.currency]: https://w3c.github.io/payment-request/#dom-paymentcurrencyamount-currency
[details.total.amount.value]: https://w3c.github.io/payment-request/#dom-paymentcurrencyamount-value
[instrument.details]: https://w3c.github.io/secure-payment-confirmation/#dom-paymentcredentialinstrument-details
[instrument.displayName]:https://w3c.github.io/secure-payment-confirmation/#dom-paymentcredentialinstrument-displayname
[instrument.icon]: https://w3c.github.io/secure-payment-confirmation/#dom-paymentcredentialinstrument-icon
[payeeName]: https://w3c.github.io/secure-payment-confirmation/#dom-securepaymentconfirmationrequest-payeename
[payeeOrigin]: https://w3c.github.io/secure-payment-confirmation/#dom-securepaymentconfirmationrequest-payeeorigin
[paymentEntitiesLogos]: https://w3c.github.io/secure-payment-confirmation/#dom-securepaymentconfirmationrequest-paymententitieslogos
[paymentEntitiesLogos\[x\].label]: https://w3c.github.io/secure-payment-confirmation/#dom-paymententitylogo-label
[paymentEntitiesLogos\[x\].url]: https://w3c.github.io/secure-payment-confirmation/#dom-paymententitylogo-url
[transaction confirmation UX]: https://w3c.github.io/secure-payment-confirmation/#sctn-transaction-confirmation-ux
