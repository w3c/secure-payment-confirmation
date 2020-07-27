# Secure Payment Confirmation

The goal is to improve payment confirmation experience with the help of WebAuthn.

![Screenshot](https://raw.githubusercontent.com/rsolomakhin/secure-payment-confirmation/master/payment.png)

The rest of the document is organized into these sections:

- [Creating a credential](#creating-a-credential)
- [Querying a credential](#querying-a-credential)
- [Authenticating a payment](#authentication-a-payment)

## Creating a credential

A new `SecurePaymentCredential` credential type is introduced for `navigator.credentials.create()` to bind descriptions of a payment instrument, i.e. a name and an icon, with a vanilla [PublicKeyCredential].

Proposed new spec that extends [Web Authentication]:
```webidl
[SecureContext, Exposed=Window]
interface SecurePaymentCredential : PublicKeyCredential {
};

partial dictionary CredentialCreationOptions {
  SecurePaymentCredentialCreationOptions securePayment;
};

dictionary SecurePaymentCredentialCreationOptions {
  required PublicKeyCredentialRpEntity rp;
  required SecurePaymentCredentialInstrument instrument;
  required BufferSource challenge;
  required sequece<PublicKeyCredentialParameters> pubKeyCredParams;
  unsigned long timeout;
  
  // PublicKeyCredentialCreationOption attributes that are intentionall omitted:
  // user: For a SecurePaymentCredential, |instrument| is analogous to |user|.
  // excludeCredentials: No payment use case has been proposed for this field.
  // attestation: Authenticator attestation is considered an anti-pattern for adoption so will not be supported.
  // extensions: No payment use case has been proposed for this field.
};

dictionary SecurePaymentCredentialInstrument {
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

// This returns a SecurePaymentCredential, which is a subtype of PublicKeyCredential.
const credential = await navigator.credentials.create({
  securePayment: securePaymentCredentialCreationOptions
});
```

See the [Guide to Web Authentication](https://webauthn.guide/) for mode details about the `navigator.credentials` API.

### [Future] Register an existing PublicKeyCredential for Secure Payment Confirmation

The relying party of an existing PublicKeyCredential can bind it for use in Secure Payment Confirmation.

Proposed new spec that extends [Web Authentication]:
```webidl
partial dictionary SecurePaymentCredentialCreationOptions {
  PublicKeyCredentialDescriptor existingCredential;
};

partial interface SecurePaymentCredential {
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


## Querying a credential

The creator of a SecurePaymentCredential can query it through the `navigator.credentials.get()` API, as if it is a vanilla PublicKeyCredential.

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

## Authenticating a payment

Any origin may invoke the [Payment Request API](https://w3c.github.io/payment-request/) with the `secure-payment-confirmation` payment method to prompt the user to verify a SecurePaymentCredential created by any other origin. The `PaymentRequest.show()` method requires a user gesture. The browser will display a native user interface with the payment amount and the payee origin, which is taken to be the origin of the top-level context where the `PaymentRequest` API was invoked.

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
  // The WebAuthn assertion created by signing over |paymentData|.
  required AuthenticatorAssertionResponse assertion;
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
  challenge,
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

[PublicKeyCredential]: https://www.w3.org/TR/webauthn/#iface-pkcredential
[Web Authentication]: https://www.w3.org/TR/webauthn/
[pilot]: bit.ly/webauthn-to-pay-2020h2-pilot
