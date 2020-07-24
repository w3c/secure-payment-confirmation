# Explainer: Secure Payment Confirmation

## Creating a credential

A new credential type is introduced for `navigator.credentials` that stores the label and the icon of a payment instrument.

## Querying the credential

Only the creator of the credential can query it through the `navigator.credentials` API.
