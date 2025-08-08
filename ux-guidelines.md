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
`https://bigshoestore.example`.

The specification allows for either one or both of the fields to be present.
Implementors may display them separately, or may combine them into a single
visual block such as `payeeName (payeeOrigin)`, for example "Big Shoe Store
Inc. (https://bigshoestore.example)".

Note: An implementor may truncate these fields in order to fit the text into
the available UX space. See issue #269 for discussions on setting normative
length limits on fields including payeeName and payeeOrigin.

### Payment instrument information

Relevant fields: [instrument.displayName], [instrument.icon], [instrument.details]

### Payment entity logos

Relevant fields: [paymentEntitiesLogos], [paymentEntitiesLogos\[x\].url], [paymentEntitiesLogos\[x\].label]

## UX differences across different screen sizes/layouts

[instrument.details]: https://w3c.github.io/secure-payment-confirmation/#dom-paymentcredentialinstrument-details
[instrument.displayName]:https://w3c.github.io/secure-payment-confirmation/#dom-paymentcredentialinstrument-displayname
[instrument.icon]: https://w3c.github.io/secure-payment-confirmation/#dom-paymentcredentialinstrument-icon
[payeeName]: https://w3c.github.io/secure-payment-confirmation/#dom-securepaymentconfirmationrequest-payeename
[payeeOrigin]: https://w3c.github.io/secure-payment-confirmation/#dom-securepaymentconfirmationrequest-payeeorigin
[paymentEntitiesLogos]: https://w3c.github.io/secure-payment-confirmation/#dom-securepaymentconfirmationrequest-paymententitieslogos
[paymentEntitiesLogos\[x\].label]: https://w3c.github.io/secure-payment-confirmation/#dom-paymententitylogo-label
[paymentEntitiesLogos\[x\].url]: https://w3c.github.io/secure-payment-confirmation/#dom-paymententitylogo-url
[transaction confirmation UX]: https://w3c.github.io/secure-payment-confirmation/#sctn-transaction-confirmation-ux
