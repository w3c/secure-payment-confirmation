# Secure Payment Confirmation

Using FIDO-based authentication to securely confirm payments initiated via the Payment Request API.

![Screenshot](https://raw.githubusercontent.com/rsolomakhin/secure-payment-confirmation/master/payment.png)

The rest of the document is organized into these sections:

- [Problem](#problem)
- [Solution: Secure Payment Confirmation](#solution-secure-payment-confirmation)
- [Proposed APIs](#proposed-apis)
  - [Creating a credential](#creating-a-credential)
  - [Querying a credential](#querying-a-credential)
  - [Authenticating a payment](#authenticating-a-payment)
- [Acknowledgements](#acknowledgements)

## Problem

Strong customer authentication during online payment is becoming a requirement in many regions, including the European Union. For card payments, 3D Secure (3DS) is the most widely deployed solution. However, 3DS1 has high user friction that led to significant cart abandonment, while improvements upon 3DS1, such as risk-based authentication and 3DS2, rely on fingerprinting techniques that are also frequently abused by trackers. The payments industry needs a consistent, low friction, strong authentication flow for online payments.

Recently, cross-browser support for [Web Authentication API][webauthn] makes FIDO-based authentication available on the web. However, it is not immediately suitable to solve the payments authentication problem because in the payments context, the party that needs to authenticate the user (e.g. the bank) is not usually in a user visible browsing context during an online checkout. Past solutions that try to create a browsing context with the bank via redirect, iframe or popup suffer from poor user experience. A more recent solution that explores [delegated authentication] from the bank to specific 3rd parties does not scale to the tens of thousands of online merchants that accept credit cards.

[webauthn]: https://www.w3.org/TR/webauthn
[delegated authentication]: https://www.w3.org/2020/02/3p-creds-20200219.pdf

## Solution: Secure Payment Confirmation

Secure Payment Confirmation defines a new `PaymentCredential` credential type to the [Credential Management][cm] spec. A `PaymentCredential` is a [PublicKeyCredential] with the special priviledge that it can be queried by any origin via the Payment Request API. A bank can register a `PaymentCredential` on the user's device after an initial ID&V process. Merchants can exercise this credential to sign over transaction data during an online payment and the bank can assert the user's identity by verifying the signature.

See [UX mocks] for a complete user journey.

[cm]: https://w3c.github.io/webappsec-credential-management/
[PublicKeyCredential]: https://www.w3.org/TR/webauthn/#iface-pkcredential
[UX mocks]: https://bit.ly/webauthn-to-pay-2020h2-pilot-ux


### Benefits

#### For browsers

- Offer a secure, privacy-preserving payment authentication primitive as part of the web platform
- Phase out banks’ use of browser fingerprinting to secure online payments
- Accelerate adoption of Web Authentication (FIDO) for payments use cases by streamlining enrollment flows
- Enroll payment instruments with structured metadata (supported networks, user-recognizable descriptor) for improved UX

#### For the payments ecosystem

- Accelerate adoption of FIDO (Web Authentication) to enable two-factor, biometric-enabled, hardware-backed, phishing-resistant checkout
- Support enrollment of secure EMV/DSRP-like tokens in web browsers
- Increase consumer confidence with biometric payment confirmation in trusted UI
- Introduce a new FIDO-based authentication value (AV) format, allowing AVs to be generated in secure hardware on the cardholder’s device and routed directly on the authorization network
- Reduce latency and increase availability compared to vanilla 3D Secure

#### For online merchants

- Provide a reliable, reliably low-friction payment authentication mechanism that significantly improves conversion compared to vanilla 3D Secure
- Increase consumer confidence with biometric payment confirmation in trusted UI
- Improve security boundary between the merchant and issuing bank, avoiding the need to poke holes in Content Security Policy

#### For customers

- More security and lower friction during online payment

## Proposed APIs

### Creating a credential

A new `PaymentCredential` credential type is introduced for `navigator.credentials.create()` to bind descriptions of a payment instrument, i.e. a name and an icon, with a vanilla [PublicKeyCredential].

Proposed new spec that extends [Web Authentication]:
```webidl
[SecureContext, Exposed=Window]
interface PaymentCredential : PublicKeyCredential {
};

partial dictionary CredentialCreationOptions {
  PaymentCredentialCreationOptions securePayment;
};

dictionary PaymentCredentialCreationOptions {
  required PublicKeyCredentialRpEntity rp;
  required PaymentCredentialInstrument instrument;
  required BufferSource challenge;
  required sequence<PublicKeyCredentialParameters> pubKeyCredParams;
  unsigned long timeout;
  
  // PublicKeyCredentialCreationOption attributes that are intentionally omitted:
  // user: For a PaymentCredential, |instrument| is analogous to |user|.
  // excludeCredentials: No payment use case has been proposed for this field.
  // attestation: Authenticator attestation is considered an anti-pattern for adoption so will not be supported.
  // extensions: No payment use case has been proposed for this field.
};

dictionary PaymentCredentialInstrument {
  // |instrumentId| is a caller provided ID for the payment instrument to which the new PaymentCredential should
  // be bound. It should be an opaque string generated using a payment network specific algorithm that allows the
  // network to identify the issuer of the instrument and the issuer to identify the account associated with this
  // instrument.
  required DOMString instrumentId;
  required DOMString displayName;
  required USVString icon;
};
```

Example usage:
```javascript
const securePaymentConfirmationCredentialCreationOptions = {
  instrument: {
    instrumentId: "Q1J4AwSWD4Dx6q1DTo0MB21XDAV76",
    displayName: 'Mastercard····4444',
    icon: 'icon.png'
  },
  challenge,
  rp,
  pubKeyCredParams,
  timeout,
};

// This returns a PaymentCredential, which is a subtype of PublicKeyCredential.
const credential = await navigator.credentials.create({
  securePayment: securePaymentCredentialCreationOptions
});
```

See the [Guide to Web Authentication](https://webauthn.guide/) for mode details about the `navigator.credentials` API.

#### [Future] Register an existing PublicKeyCredential for Secure Payment Confirmation

The relying party of an existing `PublicKeyCredential` can bind it for use in Secure Payment Confirmation.

Proposed new spec that extends [Web Authentication]:
```webidl
partial dictionary PaymentCredentialCreationOptions {
  PublicKeyCredentialDescriptor existingCredential;
};

partial interface PaymentCredential {
  readonly attribute boolean createdCredential;
};
```

Example usage:
```javascript
const securePaymentConfirmationCredentialCreationOptions = {
  instrument: {
    instrumentId: "Q1J4AwSWD4Dx6q1DTo0MB21XDAV76",
    displayName: 'Mastercard····4444',
    icon: 'icon.png'
  },
  existingCredential: {
    type: "public-key",
    id: Uint8Array.from(credentialId, c => c.charCodeAt(0))
  },
  challenge,
  rp,
  pubKeyCredParams,
  timeout
};

// Bind |instrument| to |credentialId|, or create a new credential if |credentialId| doesn't exist.
const credential = await navigator.credentials.create({
  securePayment: securePaymentCredentialCreationOptions
});

// |credential.createdCredential| is true if the specified credential does not exist and a new one is created instead.
```


### Querying a credential

The creator of a `PaymentCredential` can query it through the `navigator.credentials.get()` API, as if it is a vanilla `PublicKeyCredential`.

```javascript
const publicKeyCredentialRequestOptions = {
  challenge,
  allowCredentials: [{
    id: Uint8Array.from(credentialId, c => c.charCodeAt(0)),
    type,
    transports,
  }],
  timeout,
};

const credential = await navigator.credentials.get({
  publicKey: publicKeyCredentialRequestOptions
});
```

### Authenticating a payment

Any origin may invoke the [Payment Request API](https://w3c.github.io/payment-request/) with the `secure-payment-confirmation` payment method to prompt the user to verify a `PaymentCredential` created by any other origin. The `PaymentRequest.show()` method requires a user gesture. The browser will display a native user interface with the payment amount and the payee origin, which is taken to be the origin of the top-level context where the `PaymentRequest` API was invoked.

Proposed new `secure-payment-confirmation` payment method:

```webidl
dictionary SecurePaymentConfirmationRequest {
  SecurePaymentConfirmationAction action;
  required DOMString instrumentId;
  // Opaque data about the current transaction provided by the issuer. As the issuer is the RP
  // of the credential, |networkData| provides protection against replay attacks.
  required BufferSource networkData;
  unsigned long timeout;
  required USVString fallbackUrl;
};

enum SecurePaymentConfirmationAction {
  "authenticate",
};

dictionary SecurePaymentConfirmationResponse {
  // A bundle of key attributes about the transaction.
  required SecurePaymentConfirmationPaymentData paymentData;
  // The public key credential used to sign over |paymentData|.
  // |credential.response| is an AuthenticatorAssertionResponse and contains
  // the signature.
  required PublicKeyCredential credential;
};

dictionary SecurePaymentConfirmationPaymentData {
  required SecurePaymentConfirmationMerchantData merchantData;
  required BufferSource networkData;
};

dictionary SecurePaymentConfirmationMerchantData {
  required USVString merchantOrigin;
  required PaymentCurrencyAmount total;
};
```

Example usage:

```javascript
const securePaymentConfirmationRequest = {
  action: "authenticate",
  instrumentId: "Q1J4AwSWD4Dx6q1DTo0MB21XDAV76",
  networkData,
  timeout,
  fallbackUrl: "https://fallback.example/url"
};

const request = new PaymentRequest(
  [{supportedMethods: 'secure-payment-confirmation',
    data: securePaymentConfirmationRequest
  }],
  {total: {label: 'total', amount: {currency: 'USD', value: '20.00'}}});
const response = await request.show();

// Merchant validates |response.merchantData| and sends |response| to issuer for authentication.
```

If the payment instrument specified by `instrumentId` is not available or if the user failed to authenticate, the desired long-term solution is for the user agent to open `fallbackUrl` inside the Secure Modal Window and somehow extract a response from that interaction. 🚧 **The exact mechanism for support this flow still needs to be designed.**

🚨 As a hack for the [pilot], the user agent will simply resolve `request.show()` with an exception. The caller is responsible for constructing a second Payment Request to open `fallbackUrl` inside a Secure Modal Window by abusing the Just-in-Time registration and skip-the-sheet features of Payment Handler API.

## Privacy Considerations

Regular PublicKeyCredentials can only be used to generate assertions by the origin that created them, preventing the credential from being used to track the user across origins. In contrast, the PaymentCredential can be used from any origin to generate an assertion.

This behaviour is neccessary because payments are initiated by a variety of origins (e.g. merchants) who do not have a direct relationship with the relying party (e.g. the user's bank that issued their payment card). To protect the abuse of this API as a tracking tool it is only possible to invoke the Secure Payment Confirmation via the Payment Request API.

[PublicKeyCredential]: https://www.w3.org/TR/webauthn/#iface-pkcredential
[Web Authentication]: https://www.w3.org/TR/webauthn/
[pilot]: https://bit.ly/webauthn-to-pay-2020h2-pilot


## Acknowledgements

Contributors:

* Adrian Hope-Bailie (Coil)
* Benjamin Tidor (Stripe)
* Danyao Wang (Google)
* Christian Brand (Google)
* Rouslan Solomakhin (Google)
* Gerhard Oosthuizen (Entersekt)
