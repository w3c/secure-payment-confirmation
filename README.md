# Secure Payment Confirmation

The goal is to improve payment confirmation experience with the help of WebAuthn.

![Screenshot](https://raw.githubusercontent.com/rsolomakhin/secure-payment-confirmation/master/payment.png)

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
    displayName: 'Mastercard****4444',
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
    displayName: 'Mastercard****4444',
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


## Querying the credential

The creator of the credential can query it through the `navigator.credentials.get()` API, as if it is a vanilla PublicKeyCredential.

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

## Confirming payment

Any origin may invoke the [Payment Request API](https://w3c.github.io/payment-request/) to prompt the user to verify a credential created by any other origin. The `PaymentRequest.show()` method must require a user gesture and display user interface with the amount of payment and the hostname of the top-level context where the `PaymentRequest` API was invoked.

Proposed new `secure-payment-confirmation` payment method:

```webidl
dictionary SecurePaymentConfirmationRequest {
  SecurePaymentConfirmationAction action;
  required DOMString instrumentId;
  required BufferSource networkData;
  unsigned long timeout;
  required USVString fallbackUrl;
};

enum SecurePaymentConfirmationAction {
  "authenticate",
};

dictionary SecurePaymentConfirmationResponse {
  required SecurePaymentConfirmationMerchantData merchantData;
  required BufferSource networkData;
  required AuthenticatorAssertionResponse assertion;
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

If the payment instrument specified by `instrumentId` is not available or if the user failed to authenticate, then the
user agent will open `fallbackUrl` inside the Secure Modal Window.

[PublicKeyCredential]: https://www.w3.org/TR/webauthn/#iface-pkcredential
[Web Authentication]: https://www.w3.org/TR/webauthn/
