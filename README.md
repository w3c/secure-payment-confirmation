# Secure Payment Confirmation

Using FIDO-based authentication to securely confirm payments initiated via the Payment Request API.

This work is currently in incubation within [W3C Web Payments Working Group](https://www.w3.org/Payments/WG/).

![Screenshot](https://raw.githubusercontent.com/rsolomakhin/secure-payment-confirmation/master/payment.png)

The rest of the document is organized into these sections:

- [Problem](#problem)
- [Solution: Secure Payment Confirmation](#solution-secure-payment-confirmation)
- [Proposed APIs](#proposed-apis)
  - [Creating a credential](#creating-a-credential)
  - [Querying a credential](#querying-a-credential)
  - [Authenticating a payment](#authenticating-a-payment)
- [Security and Privacy Considerations](#security-and-privacy-considerations)
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
  required DOMString displayName;
  required USVString icon;
};
```

Example usage:
```javascript
const securePaymentConfirmationCredentialCreationOptions = {
  rp,
  instrument: {
    displayName: 'Mastercard····4444',
    icon: 'icon.png'
  },
  challenge,
  pubKeyCredParams,
  timeout,
};

// This returns a PaymentCredential, which is a subtype of PublicKeyCredential.
const credential = await navigator.credentials.create({
  payment: securePaymentCredentialCreationOptions
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
  payment: securePaymentCredentialCreationOptions
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
enum SecurePaymentConfirmationAction {
  "authenticate",
};

dictionary SecurePaymentConfirmationRequest {
  SecurePaymentConfirmationAction action;
  required FrozenArray<BufferSource> credentialIds;
  // Opaque data about the current transaction provided by the issuer. As the issuer is the RP
  // of the credential, `networkData` provides protection against replay attacks.
  required BufferSource networkData;
  unsigned long timeout;
  required USVString fallbackUrl;
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
  action: 'authenticate',
  credentialIds: [Uint8Array.from(atob('Q1J4AwSWD4Dx6q1DTo0MB21XDAV76'), c => c.charCodeAt(0))],
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
await response.complete('success');

// Merchant validates |response.challenge| and sends |response| to the issuer for authentication.
```

#### Transaction Binding and Web Payments Cryptogram

In a normal WebAuthn interaction, the Relying Party provides a `challenge` that is signed by the authenticator. When a `PaymentCredential` is exercised via Payment Request API, the browser does an additional step: it takes the standard challenge (`networkData` in the example above) and combines it with the transaction details (e.g. payee origin and amount) to create a new challenge for the authenticator to sign. The response is a Web Payments Cryptogram, which contains the browser-constructed challenge in plain text and the signature. This allows the issuer and any party with whom the issuer has chosen to share the credential's public key, to verify that the signature matches the expected transaction details.

For reference, see Chromium's [implementation of transaction binding](https://source.chromium.org/chromium/chromium/src/+/master:components/payments/content/secure_payment_confirmation_app.cc;l=53;drc=965551d5ef24466c83a4edc9fe13e9813443d9f1). 

The Web Payments Cryptogram is returned via payment method [details](https://w3c.github.io/payment-request/#dom-paymentresponse-details) of `PaymentResponse`.

For example, in the sample response below, `challenge` is the browser-constructed challenge that contains transaction details, and `signature` is the authenticator signature over a SHA-256 hash of `challenge`:

```
"details": {
    "appid_extension": false,
    "challenge": "{\"merchantData\":{\"merchantOrigin\":\"https://rsolomakhin.github.io\",\"total\":{\"currency\":\"USD\",\"value\":\"0.01\"}},\"networkData\":\"bmV0d29ya19kYXRh\"}",
    "signature": "MEYCIQDfx231pNz7DfhCoNl20uwqmJ304JtXJwyxo/jQzSLiwgIhAL2wFlSSSZR3tQ6D9MaMxhyT7NYB332FVhWAwCGh6OAs",
    "echo_appid_extension": false,
    "echo_prf": false,
    "info": {
      "authenticator_data": "yFnBU/b2Oli2DTHAX1WdPaZktogLFFJVu9ECZWWXxpMFYGHoqg==",
      "client_data_json": "eyJ0eXBlIjoid2ViYXV0aG4uZ2V0IiwiY2hhbGxlbmdlIjoiY0NzTnU1aTZVbldCcE45ZHNmTEpNMnU1NHlPX3pQaVMxaVdsVS1kajlHVSIsIm9yaWdpbiI6Imh0dHBzOi8vcnNvbG9tYWtoaW4uZ2l0aHViLmlvIiwiY3Jvc3NPcmlnaW4iOmZhbHNlfQ==",
      "id": "Aaz3T-2kASNRxxvP1AMH9WGWjtjH7aTlzVce_bW61aEKZgYKr3-d2F5h3c_0e1Tz-GRdHEgUjDA_MxWhy__H8TbAsYQyxo2WRGonnXJ_Z1MJPObHHgVqmL0mWGfG"
    },
    "prf_not_evaluated": false,
    "prf_results": {},
    "user_handle": "qR72lDpOmub67MWJ1E+9TCqV7MLuJXWO5EAIauG6DHs="
  },
```

#### Caveats

If no payment instrument specified by `credentialIds` is available or if the user failed to authenticate, the desired long-term solution is for the user agent to open `fallbackUrl` inside the Secure Modal Window and somehow extract a response from that interaction. 🚧 **The exact mechanism for support this flow still needs to be designed.**

🚨 As a hack for the [pilot], the user agent will simply resolve `request.show()` with an exception. The caller is responsible for constructing a second Payment Request to open `fallbackUrl` inside a Secure Modal Window by utilizing the Just-in-Time registration and skip-the-sheet features of Payment Handler API.

## Security and Privacy Considerations

### Security of PaymentCredential

#### Downgrading the Relying Party origin restriction for exercising a credential

A regular PublicKeyCredential can only be used to generate assertions by the origin that created them to avoid phishing risk. In contrast, the PaymentCredential can be used from any origin to generate an assertion, but only via the Payment Request API. When used this way, the browser shows a native UI to clearly inform the user that they are in a payments context and the parties involved in the payment transaction (i.e. payee, payment instrument - which usually identifies the issuing bank). An attacker that is not genuinely interested in facilitating a payment transaction cannot effectively use the new behavior in this proposal to phish a user credential.

This security downgrade in the payments context is neccessary because payments are initiated by a variety of origins (e.g. merchants) who do not have a direct relationship with the relying party (e.g. the user's bank that issued their payment card). An alternative is to redirect the user to the issuer's origin for authentication, but both merchants and issuing banks would like to avoid this increased friction due to impact on cart abandonment.

#### Probing

If there are no matching credentials for a request, the Web Payment service immediately rejects PaymentRequest.show() without showing any user interface. However, since PaymentRequest.show() consumes a user activation, a malicious website cannot efficiently probe available credentials on the user’s device without tricking the user to repeatedly provide user activations.

#### Issuer supplied display name and icon

The display name and icon are an arbitrary string and icon provided by the issuer (RP). Although a poor name may be confusing, we don’t believe this creates a new security threat because the credential will only be displayed to the user on a website that has a relationship with the issuer and the credential is only usable with the issuer. A confusing or misleading name cannot be used by a malicious party in a phishing attack.


### Privacy

The [privacy considerations for Web Authentication][webauthn-privacy] are also applicable to this proposal.
[webauthn-privacy]: https://www.w3.org/TR/webauthn/#sctn-privacy-considerations

#### 3rd party access to authenticator assertion

This proposal allows a 3rd party (i.e. a merchant) that is not the Relying Party of the credential (i.e. the issuing bank) to gain access to the authenticator assertion, which contains a credential ID and the authenticator signature. However, we do not believe this increases the privacy attack surface.

While the credential ID may be used as a tracker to correlate a user across origins, an origin (e.g. a merchant) needs to already have access to the credential ID in order to invoke the new API. A merchant also already has a more powerful identifier, i.e. the raw credit card number the user provided, that it used to exchange to the credential ID via trusted server side integration with the issuing bank.

The authenticator signature is emphemeral to each transaction and is not useful to any party unless it obtains the public key from the issuing bank.


[PublicKeyCredential]: https://www.w3.org/TR/webauthn/#iface-pkcredential
[Web Authentication]: https://www.w3.org/TR/webauthn/
[pilot]: https://bit.ly/webauthn-to-pay-2020h2-pilot

## FAQ

### Q. Who can validate the SPC response besides the actual Relying Party (RP)?

An SPC challenge bundles transaction details with transaction-specific dynamic data from the Relying Party. An SPC response includes a signature over that challenge. Validation in SPC refers to the verification of that signature using the credential public key. A Relying Party can choose to share the credential public key with another party (e.g., a card network or payment service provider) via out-of-band communication to enable that party to validate the SPC assertion.

## Acknowledgements

Contributors:

* Adrian Hope-Bailie (Coil)
* Benjamin Tidor (Stripe)
* Danyao Wang (Google)
* Christian Brand (Google)
* Rouslan Solomakhin (Google)
* Nick Burris (Google)
* Gerhard Oosthuizen (Entersekt)
