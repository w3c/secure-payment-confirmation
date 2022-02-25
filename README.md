# Secure Payment Confirmation

Secure Payment Confirmation (SPC) is a Web API to support streamlined
authentication during a payment transaction. It is designed to scale
authentication across merchants, to be used within a wide range of
authentication protocols, and to produce cryptographic evidence that the user
has confirmed transaction details. The [W3C Web Payments Working
Group](https://www.w3.org/Payments/WG/) is developing SPC.

Links:

- [Explainer](explainer.md)
- [Specification](https://w3c.github.io/secure-payment-confirmation/) ([spec.bs](spec.bs))
- [Use Cases](scope.md#user-stories)
- [Requirements](requirements.md)
- [Tests](https://wpt.fyi/results/secure-payment-confirmation?label=master&label=experimental&aligned)

![Screenshot](payment.png)

## FAQ

### Q. Who can validate the SPC response besides the actual Relying Party (RP)?

An SPC challenge bundles transaction details with transaction-specific dynamic data from the Relying Party. An SPC response includes a signature over that challenge. Validation in SPC refers to the verification of that signature using the credential public key. A Relying Party can choose to share the credential public key with another party (e.g., a card network or payment service provider) via out-of-band communication to enable that party to validate the SPC assertion.

## Acknowledgements

Contributors:

* Adrian Hope-Bailie (Coil)
* Benjamin Tidor (Stripe)
* Danyao Wang (Google)
* Christiaan Brand (Google)
* Rouslan Solomakhin (Google)
* Nick Burris (Google)
* Gerhard Oosthuizen (Entersekt)
