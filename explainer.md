# Secure Payment Confirmation explained

## tl;dr

**Secure Payment Confirmation** is a proposed Web API that enables the use of
biometric authentication in payment flows on the web, building on top of
[WebAuthn]. The goal is to provide strong, low-friction authentication of a
customer in a payment context, whilst still preserving user privacy better than
current fingerprinting-based methods.

See also:
  - [Specification](https://w3c.github.io/secure-payment-confirmation/), the formal draft spec.
  - [Scope document](https://github.com/w3c/secure-payment-confirmation/blob/main/scope.md)
  - [Requirements document](https://github.com/w3c/secure-payment-confirmation/blob/main/requirements.md)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Contents

- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Proposed Solution: Secure Payment Confirmation](#proposed-solution-secure-payment-confirmation)
  - [Proposed APIs](#proposed-apis)
    - [Creating a credential](#creating-a-credential)
      - [Creating a SPC credential in a cross-origin iframe](#creating-a-spc-credential-in-a-cross-origin-iframe)
    - [Authenticating a payment](#authenticating-a-payment)
- [Other Considerations](#other-considerations)
  - [Initial Experimentation with Stripe](#initial-experimentation-with-stripe)
  - [Are SPC credentials identical to WebAuthn credentials?](#are-spc-credentials-identical-to-webauthn-credentials)
  - [Why use the PaymentRequest API](#why-use-the-paymentrequest-api)
- [Alternatives Considered](#alternatives-considered)
  - [Traditional WebAuthn](#traditional-webauthn)
  - [Delegated Authentication](#delegated-authentication)
- [Security Considerations](#security-considerations)
  - [Enrollment in cross-origin iframes](#enrollment-in-cross-origin-iframes)
  - [Cross-origin authentication ceremony](#cross-origin-authentication-ceremony)
  - [Merchant-supplied data](#merchant-supplied-data)
- [Privacy Considerations](#privacy-considerations)
  - [Probing](#probing)
  - [Credential-sharing](#credential-sharing)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Motivation

Online payments are usually a 3 party interaction:

* The Merchant (e.g. an online web store)
* The Customer (e.g. the user buying from an online web store)
* The Account Provider (e.g. the bank that issued the payment instrument being used)

> **NOTE**: It is not uncommon for there to be a fourth party: the Payment
> Service Provider, whom the Merchant may delegate the task of accepting the
> payment method to. This is usually done by the Merchant either redirecting the
> User to the Payment Service Provider's website, or by embedding the Payment
> Service Provider in an iframe. Any following mention of the term Merchant
> should be taken to mean either a Merchant or a Payment Service Provider;
> whichever is handling payment.

Traditionally, the Customer only interacts with the Merchant, and the Merchant
communicates with the Account Provider via a back-channel protocol. However,
Account Providers are increasingly looking to authenticate Customers during
online payments. This is done for both regulatory reasons (e.g. [SCA] in the
European Union) or for fraud prevention reasons - either fradulent Customers or
fradulent Merchants.

Existing methods of authenticating a Customer during an online payment are
either high friction (e.g. embedding challenge iframes from the Account
Provider) or have poor user privacy (e.g. fingerprinting or tracking the user
for risk analysis, to provide a 'frictionless flow'). The payments industry
needs a consistent, low friction, strong authentication flow for online
payments.

The [Web Authentication (WebAuthn) API][webauthn] makes FIDO-based
authentication available on the web, which provides a strong, low-friction
method for a user to prove a pre-established identity with a given site.
However, WebAuthn is not immediately suitable to solve the payments
authentication problem, as:

1. It requires the Account Provider to be present in the transaction (e.g. via
   an embedded iframe), which increases user friction and lowers conversion
   rates.
1. The generated assertion contains no payments-related information, so it
   cannot be used as-is to fulfill regulatory requirements to provide evidence
   of user content (e.g. [Dynamic Linking] requirements).
1. WebAuthn does not allow credential creation in a cross-origin iframe, thus
   excluding a useful onboarding flow - enrolling a Customer after they have
   completed a traditional authentication flow (e.g. via SMS OTP), without
   the friction of redirecting them to the bank's site or app.

This proposal attempts to provide an authentication solution for online
payments that is both as strong and low-friction as WebAuthn, whilst also
solving these three issues.

### Goals

Find a solution that (in no particular order):

* Has as-strong or stronger authentication than current challenge flows.
* Is less friction for the Customer than current challenge flows.
* Is more protective of user-privacy than current frictionless flows.
* Improves the ability for online payments to meet regulations such as [Dynamic
  Linking].

> **NOTE**: It is out-of-scope for the specification to **prove** adherence
> to e.g. Dynamic Linking, but we aim to produce a solution that could be
> vetted as such.

* Allows for in-flow enrollment of Customers during a traditional challenge
  flow, as well as outside of a transaction.

### Non-Goals

* Selection of a Payment Instrument by the Customer; it is presumed that the
  Customer has already done so (e.g. by typing in their credit card number).
* ID & V to establish real world identity during enrollment; it is up to the
  Account Provider to determine the Customer's identity to their satisfaction.
* Providing authentication for peer-to-peer or business-to-business
  transactions.
    * We expect that Secure Payment Confirmation may be useful in these
      cases, but are concentrating on consumer-to-business cases for now.

## Proposed Solution: Secure Payment Confirmation

Secure Payment Confirmation (SPC) builds on top of [WebAuthn] to add
payment-specific datastructures to the signed assertion, and to relax
assumptions to allow the API to be called in payment contexts.

The intention for Secure Payment Confirmation is that a Customer would enroll
once on a given device for a given account with an Account Provider, either on
the Account Provider's website or during a traditionally-authenticated online
payment (e.g. after completing a challenge in a Account Provider iframe). Then,
in subsequent transactions on **any Merchant** that wishes to use Secure
Payment Confirmation:

1. The Customer identifies themselves to the Merchant as per normal (e.g.
   by selecting a payment instrument such as a credit card or bank account.)
1. Using a back-channel (e.g. the EMV® 3-D Secure protocol), the Merchant asks
   for, and receives, a list of credentials for the identified Customer from
   the Account Provider.
1. The Merchant calls the SPC API with the list of
   [webauthn-credentials|credentials].
1. The User Agent displays a UX to the Customer, informing them of the
   transaction details and asking if they wish to authenticate their identity to
   the Account Provider.
1. The Customer consents, and the User Agent and Customer perform a [WebAuthn]
   signing ceremony. Payment details are included in the returned assertion.
1. The Merchant receives the assertion, and using the existing back-channel
   sends the assertion to the Account Provider.
1. The Account Provider verifies the signature on the assertion, and verifies
   that the data in the assertion (e.g. transaction amount, payee) is as
   expected.
1. The Account Provider informs the Merchant of transaction success, and the
   payment concludes successfully.

> **NOTE**: Most of the above flow happens in the background. The user
> experience consists only of examining and agreeing to the transaction
> details, and performing a [WebAuthn] interaction.

**TODO**: Color-coded image here showing the authentication flow from the user
perspective. Merchant site with 'pay' button, then overlaid payment transaction
details, then WebAuthn interface, then success.

For the handling of cases when no returned credentials match the current
device, see [the Privacy section](#privacy-considerations).

### Proposed APIs

Secure Payment Confirmation introduces a new [WebAuthn extension], `payment`,
which adds three payments-specific capabilities on top of traditional WebAuthn:

1. Allows calling `navigator.credentials.create` in a cross-origin iframe, as long
   as a ["payment" permission policy] is set on the iframe.
1. Allows a third-party (the Merchant) to initiate an authentication ceremony
   **on behalf of** the RP (the Account Provider), by passing in credentials
   provided to the Merchant by the Account Provider.
1. Enforces that the User Agent appropriately communicates to the user that they
   are authenticating a transaction and the transaction details. Those details
   are then included in the assertion signed by the authenticator.

> **NOTE**: Allowing `navigator.credentials.create` in a cross-origin iframe is
> [currently a topic of discussion](https://github.com/w3c/webauthn/issues/1656)
> in the WebAuthn WG too.

> **NOTE**: It is currently undecided whether an SPC credential should be
> explicitly different from a WebAuthn credential (e.g. by adding a new 'SPC
> bit' of some form), or whether existing WebAuthn credentials should be able
> to be used in SPC authentication. See **TODO**: link or file issue.

#### Creating a credential

Creating a credential in Secure Payment Confirmation is done by the same
`navigator.credentials.create` call as with [WebAuthn], but with a `payment`
extension specified.

```javascript
const publicKey = {
  challenge: Uint8Array.from(
      randomStringFromServer, c => c.charCodeAt(0)),

  rp: {
    name: "Fancy Bank",
  },

  user: {
    id: Uint8Array.from(userId, c => c.charCodeAt(0)),
    name: "jane.doe@example.org",
    displayName: "Jane Doe",
  },

  pubKeyCredParams: [
    {
      type: "public-key",
      alg: -7 // "ES256"
    },
    {
      type: "public-key",
      alg: -257 // "RS256"
    }
  ],

  authenticatorSelection: {
    userVerification: "required",
    residentKey: "required",
    authenticatorAttachment: "platform",
  },

  timeout: 60000,  // 1 minute

  extensions: {
    payment: {
      isPayment: true,
    },
  },
};

navigator.credentials.create({ publicKey })
  .then(function (newCredentialInfo) {
    // Send new credential info to server for verification and registration.
  }).catch(function (err) {
    // No acceptable authenticator or user refused consent. Handle appropriately.
  });
```

As per the above note, the need to have a special enrollment flow for SPC will
depend on whether:

* [WebAuthn] starts to allow credential creation in a cross-origin iframe.
* If it is considered reasonable for 'vanilla' WebAuthn credentials to be used
   in SPC authentication.

If both of these become true, then the extension would not be needed during
enrollment.

##### Creating a SPC credential in a cross-origin iframe

Unlike normal WebAuthn credentials, SPC allows a credential to be created in a
cross-origin iframe (e.g. if `merchant.com` embeds an iframe from `bank.com`). This is
intended to support the common enrollment flow of a bank enrolling the user during a
step-up challenge (e.g. after proving their identity via OTP).

To allow this, the cross-origin iframe must have the ["payment" permission
policy] set. For example:

```html
<!-- Assume parent origin is merchant.com -->
<!-- Inside this cross-origin iframe, script would be allowed to create a SPC credential for example.org -->
<iframe src="https://example.org" allow="payment">
```

#### Authenticating a payment

An origin may invoke the [Payment Request API] with the
`secure-payment-confirmation` payment method to prompt the user to verify a
Secure Payment Confirmation credential created by any other origin. The User
Agent will display a native user interface with transaction details (e.g. the
payment amount and the payee origin).

> **NOTE**: The `PaymentRequest.show()` method requires a user gesture. The User
> Agent will display a native user interface with the payment amount and the
> payee origin.

> **NOTE**: [Per the Payment Request specification][pr-cross-origin], if
> `PaymentRequest` is used within a cross-origin iframe (e.g. if `merchant.com`
> embeds an iframe from `psp.com`, and `psp.com` wishes to use
> `PaymentRequest`), that iframe must have the ["payment" permission policy]
> set.

Proposed new `secure-payment-confirmation` payment method:

```javascript
const request = new PaymentRequest([{
  supportedMethods: "secure-payment-confirmation",
  data: {
    // List of credential IDs obtained from the Account Provider.
    credentialIds,

    // The challenge is also obtained from the Account Provider.
    challenge: new Uint8Array(
        randomStringFromServer, c => c.charCodeAt(0)),

    instrument: {
      displayName: "Fancy Card ****1234",
      icon: "https://fancybank.com/card-art.png",
    },

    payeeOrigin: "https://merchant.com",

    timeout: 60000,  // 1 minute
  }], {
    total: {
      label: "Total",
      amount: {
        currency: "USD",
        value: "5.00",
      },
    },
  });

try {
  // NOTE: canMakePayment() checks only public information for whether the SPC
  // call is valid. To preserve user privacy, it does not check whether any
  // passed credentials match the current device.
  const canMakePayment = await request.canMakePayment();
  if (!canMakePayment) { throw new Error('Cannot make payment'); }

  const response = await request.show();
  await response.complete('success');

  // response.details is a PublicKeyCredential, with a clientDataJSON that
  // contains the transaction data for verification by the issuing bank.

  /* send response.details to the issuing bank for verification */
} catch (err) {
  /* SPC cannot be used; merchant should fallback to traditional flows */
}
```
## Other Considerations

### Initial Experimentation with Stripe

### Are SPC credentials identical to WebAuthn credentials?

### Why use the PaymentRequest API

## Alternatives Considered

### Traditional WebAuthn

### Delegated Authentication

A previous attempt that explored [delegated authentication] from the bank to
*specific* 3rd parties did not scale to the tens of thousands of online
merchants that accept credit cards.

## Security Considerations

### Enrollment in cross-origin iframes

### Cross-origin authentication ceremony

### Merchant-supplied data

## Privacy Considerations

### Probing

### Credential-sharing

[SCA]: https://en.wikipedia.org/wiki/Strong_customer_authentication
[webauthn]: https://www.w3.org/TR/webauthn
[webauthn-credentials]: https://www.w3.org/TR/webauthn/#credential-id
[Dynamic Linking]: https://www.twilio.com/blog/dynamic-linking-psd2#:~:text=What%20is%20Dynamic%20Linking
[webauthn extension]: https://www.w3.org/TR/webauthn/#sctn-extensions
[3D Secure]: https://en.wikipedia.org/wiki/3-D_Secure
[delegated authentication]: https://www.w3.org/2020/02/3p-creds-20200219.pdf
[Payment Request API]: https://www.w3.org/TR/payment-request
[pr-cross-origin]: https://www.w3.org/TR/payment-request/#using-with-cross-origin-iframes
["payment" permission policy]: https://w3c.github.io/payment-request/#permissions-policy


