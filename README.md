# Secure Payment Confirmation

The goal is to improve payment confirmation experience with the help of WebAuthn.

![Screenshot](https://raw.githubusercontent.com/rsolomakhin/secure-payment-confirmation/master/payment.png)

## Creating a credential

A new credential type is introduced for `navigator.credentials.create()` that stores the name and the icons of a payment instrument.

```javascript
const publicKeyCredentialCreationOptions = {
  paymentInstrument: {
    name: 'Mastercard****4444',
    icons: [{
  	  'src': 'icon.png',
  	  'sizes': '48x48',
  	  'type': 'image/png',
    }],
  },
  challenge,
  rp,
  user,
  pubKeyCredParams,
  authenticatorSelection,
  timeout,
  attestation,
};

const credential = await navigator.credentials.create({
  publicKey: publicKeyCredentialCreationOptions
});
```

See the [Guide to Web Authentication](https://webauthn.guide/) for mode details about the `navigator.credentials` API.

## Querying the credential

Only the creator of the credential can query it through the `navigator.credentials.get()` API.

```javascript
const publicKeyCredentialRequestOptions = {
  challenge,
  allowCredentials: [{
    id: ['ADSUllKQmbqdGtpu4sjseh4cg2TxSvrbcHDTBsv4NSSX9...'],
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

```javascript
const publicKeyCredentialRequestOptions = {
  challenge,
  allowCredentials: [{
    id: ['ADSUllKQmbqdGtpu4sjseh4cg2TxSvrbcHDTBsv4NSSX9...'],
    type,
    transports,
  }],
  timeout,
};

const request = new PaymentRequest(
  [{supportedMethods: 'secure-payment-confirmation',
    data: {
      publicKey: publicKeyCredentialRequestOptions,
      fallback: 'https://fallback.example/url',
    },
  }],
  {total: {label: 'total', amount: {currency: 'USD', value: '20.00'}}});
const response = await request.show();
```

If none of the credentials from `allowCredentials.id` are available, then the user agent will invoke the payment handler for the payment method specified in the `fallback` method. It is recommended that the `fallback` payment method supports just-in-time installation of a payment handler, so the user agent may install it just-in-time. This enables the payment handler to open a fallback URL in the payment handler window.
