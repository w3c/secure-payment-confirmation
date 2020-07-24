# Explainer: Secure Payment Confirmation

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

## Querying the credential

Only the creator of the credential can query it through the `navigator.credentials.get()` API.

```javascript
const credential = await navigator.credentials.get({
    publicKey: publicKeyCredentialRequestOptions
});
```

## Confirming payment

Any origin may invoke the [Payment Request API](https://w3c.github.io/payment-request/) to prompt the user to verify a credential from any origin.
