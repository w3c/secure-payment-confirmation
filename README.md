# Explainer: Secure Payment Confirmation

## Creating a credential

A new credential type is introduced for `navigator.credentials` that stores the label and the icon of a payment instrument.

```javascript
const credential = await navigator.credentials.create({
    publicKey: publicKeyCredentialCreationOptions,
    paymentInstrument: {
      name: 'Mastercard****4444',
      icons: [{
  	    'src': 'icon.png',
  	    'sizes': '48x48',
  	    'type': 'image/png'
      }]
    }
});
```

## Querying the credential

Only the creator of the credential can query it through the `navigator.credentials` API.

## Confirming payment

Any origin may invoke the [Payment Request API](https://w3c.github.io/payment-request/) to prompt the user to verify a credential from any origin.
