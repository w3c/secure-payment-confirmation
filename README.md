# Secure Payment Confirmation

The goal is to improve payment confirmation experience with the help of WebAuthn.

![Screenshot](https://raw.githubusercontent.com/rsolomakhin/secure-payment-confirmation/master/payment.png)

The rest of the document is organized into these sections:

- [Creating a credential](#creating-a-credential)
- [Querying a credential](#querying-a-credential)
- [Confirming a payment](#confirming-a-payment)

## Creating a credential

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
  required sequece<PublicKeyCredentialParameters> pubKeyCredParams;
  unsigned long timeout;
  
  // PublicKeyCredentialCreationOption attributes that are intentionall omitted:
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

### [Future] Register an existing PublicKeyCredential for Secure Payment Confirmation

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


## Querying a credential

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

## Confirming a payment

Any origin may invoke the [Payment Request API] in order to use a PaymentCredential to confirm a payment. The caller will either:
 - invoke the API using a known `instrumentId` (using the `secure-payment-confirmation` payment method identifier), or
 - invoke the API using one or more payment methods that support a Secure Payment Confirmation response and have a supporting Payment Handler installed

**Note:** The `PaymentRequest.show()` method requires a user gesture.

### Using the `secure-payment-confirmation` payment method

Any origin can use the [Payment Request API] with the `secure-payment-confirmation` payment method to immediately prompt the user to use a `PaymentCredential` to confirm a payment.

In this case the caller must provide:
 - an `instrumentId` provided by the RP for a payment instrument that was already selected by the user
 - an opaque `challenge` provided by the RP
 
These data elements will be sourced directly via the payment network by the caller before calling the API. This flow is well suited to payment methods such as card payments with 3D Secure (3DS) where the user has entered a payment card number into a form or selected a card on file (i.e. selected a payment instrument) and the caller has used the 3DS infrastructure to get a valid `instrumentId` and dynamic `networkData` from the card issuer.
 
After calling `PaymentRequest.show()` the browser will display a native user interface with the payment amount and the payee origin, which is taken to be the origin of the top-level context where the `PaymentRequest` API was invoked.

Proposed new `secure-payment-confirmation` payment method:

```webidl
dictionary SecurePaymentConfirmationRequest {
  SecurePaymentConfirmationAction action;
  required DOMString instrumentId;
  // Opaque data about the current transaction provided by the issuer. As the issuer is the RP
  // of the credential, |challenge| provides protection against replay attacks.
  required BufferSource challenge;
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
  required BufferSource challenge;
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

### [Future] Using an RP provided Payment Handler

The caller can invoke [Payment Request API] before the user selects a payment instrument and still leverage Secure Payment Confirmation. In this case the browser will prompt the user to select a payment instrument and then invoke the RP's Payment Handler which will invoke Secure Payment Confirmation.

> **NOTE:** There is not a great developer experience when using SPC and Payment Handlers as yet so the correlation is hacky. During the installation flow for a Payment Handler it is possible for the RP to [store](https://w3c.github.io/payment-handler/#set-method) the details of one or more payment instruments linked to the Payment Handler. The following behaviour is backwards compatible with existing [Payment Handler API] behaviour but could be improved in future.

 1. For each of the `supportedMethods` provided in the call to [Payment Request API] the browser will gather the set of Payment Handlers that support the given method.
 2. For each of the instruments linked to a Payment Handler in the returned set the browser will extract the `instrumentKey` provided when adding the instrument to the Payment Handler and look for a matching `instrumentId` among all of the instruments linked to a PaymentCredential. If it finds a match the instrument is added to the set of user choices.
 3. If the set of user choices is not empty the user is prompted to select a payment instrument, else the browser reverts to the usual Payment Handler selection flow.
 4. If the user selects an instrument the corresponding Payment Handler is invoked and the `instrumentId` is passed to it as the [`instrumentKey`](https://w3c.github.io/payment-handler/#dom-paymentrequestevent-instrumentkey) property of the `PaymentRequestEvent`.
 
 If the user's chosen instrument can be used for Secure Payment Confirmation (i.e. is linked to a credential) `PaymentRequestEvent.isSecureConfirmationAvailable` will be `true`.
 
 The Payment Handler can can query its backend systems to get the neccessary challenge (`challenge`) and verify that the `instrumentId` is still valid before calling `PaymentRequestEvent.securePaymentConfirmation(challenge, timeout)` to invoke the Secure Payment Confirmation flow.
 
The browser will display a native user interface with the payment amount and the payee origin, which is taken to be the origin of the top-level context where the `PaymentRequest` API was invoked.

Proposed new `PaymentRequestEvent`:

```webidl
[Exposed=ServiceWorker]
interface PaymentRequestEvent : ExtendableEvent {
  constructor(DOMString type, optional PaymentRequestEventInit eventInitDict = {});
  readonly attribute USVString topOrigin;
  readonly attribute USVString paymentRequestOrigin;
  readonly attribute DOMString paymentRequestId;
  readonly attribute FrozenArray<PaymentMethodData> methodData;
  readonly attribute object total;
  readonly attribute FrozenArray<PaymentDetailsModifier> modifiers;
  readonly attribute DOMString instrumentKey;
  readonly attribute boolean requestBillingAddress;
  readonly attribute object? paymentOptions;
  readonly attribute FrozenArray<PaymentShippingOption>? shippingOptions;
  Promise<WindowClient?> openWindow(USVString url);
  Promise<PaymentRequestDetailsUpdate?> changePaymentMethod(DOMString methodName, optional object? methodDetails = null);
  Promise<PaymentRequestDetailsUpdate?> changeShippingAddress(optional AddressInit shippingAddress = {});
  Promise<PaymentRequestDetailsUpdate?> changeShippingOption(DOMString shippingOption);
  void respondWith(Promise<PaymentHandlerResponse> handlerResponsePromise);
  // New members
  readonly attribute boolean isSecurePaymentConfirmationAvailable;
  Promise<SecurePaymentConfirmationResponse?> securePaymentConfirmation(BufferSource challenge, long timeout);
};
```

[Payment Request API]: https://w3c.github.io/payment-handler/
[Payment Handler API]: https://w3c.github.io/payment-request/
[PublicKeyCredential]: https://www.w3.org/TR/webauthn/#iface-pkcredential
[Web Authentication]: https://www.w3.org/TR/webauthn/
[pilot]: bit.ly/webauthn-to-pay-2020h2-pilot
