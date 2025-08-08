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
