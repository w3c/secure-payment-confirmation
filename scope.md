# Secure Payment Confirmation: Scope

Status: This is a draft document without consensus.

Secure Payment Confirmation (SPC) is a Web API to support streamlined
authentication during a payment transaction. It is designed to scale
authentication across merchants, to be used within a wide range of
authentication protocols, and to produce cryptographic evidence that
the user has confirmed transaction details.

Though this document we seek to build consensus around the scope of SPC.

See also: [SPC Requirements and Design Considerations](requirements.md) for discussion about concrete requirements.

We have been discussing three steps in a payment flow where browser
capabilities could help streamline the user experience:

* Establish user identity (e.g., cookie, session, payment app). See also [issue 62](https://github.com/w3c/secure-payment-confirmation/issues/62).
* Determine payment instruments for the user. When more than one is available, prompt the user to select one.
* Authenticate the user for that instrument for this transaction.

SPC is a tool for this last step.

## Benefits and Unique Features of SPC

### Benefits

* **Authentication Streamlined for Payment**. FIDO authentication
provides a very good user experience. We expect SPC to build on that
experience in several ways, by further accelerating authentication
(compared to one-time passcodes), requiring fewer user gestures,
offering a predictable user experience across sites, and avoiding
redirects.

* **Scalable and Ubiquitous**. SPC supports streamlined authentication
across merchant sites without additional enrollment.

* **Designed to Meet Regulatory Requirements**. The standardized
payment confirmation user experience is designed to help entities
fulfill regulatory requirements (e.g., strong customer authentication
and dynamic linking under PSD2) and other customer
authentication use cases.

* **Simpler and more Secure Front-end Deployment**. The browser (or
secure hardware) manages the display of the payment confirmation experience,
removing the need for other parties (e.g., issuing banks or payment
apps) to do so. In addition, enabling payment service providers or
others to authenticate the user can reduce the need to embed code
provided by a Relying Party in a Web page, reducing security risks.
Reducing the need for redirects should also simplify solutions.

See also [more SPC benefits](https://github.com/w3c/webpayments/wiki/Secure-Payment-Confirmation#benefits).

### Unique Features

The above benefits are grounded in a small number of unique API features:

* **Browser-native UX for payment confirmation**. The browser (or
secure hardware) provides a consistent and efficient authentication UX
across merchant sites.

* **Cryptographic evidence**. Payment confirmation generates
cryptographic evidence of the user's confirmation of payment
details.

* **Cross-origin authentication**. With FIDO, the Relying Party that
creates FIDO credentials is the only origin that can authenticate the
user with those credentials. With SPC, any origin can authenticate the
user during a transaction by leveraging another Relying Party's
credentials.

## Protocols and Systems Helping to Guide Requirements

This list is the result of people joining the SPC task force:

* EMV&reg; 3-D Secure (3DS)
* EMV&reg; Secure Remote Commerce (SRC)
* Open Banking APIs, with a focus on PISP use cases.
* Real time credit (see [issue 42](https://github.com/w3c/secure-payment-confirmation/issues/42)).

The following use cases may be of interest but are of distinctly lower
priority:

* Delegation to the merchant as Relying Party.
* QR-code triggered payment. User points phone at QR code which represents a Web Page. In vanilla mode, Web Page includes "Buy" button to trigger Payment Request. But in streamlined mode, SPC payment confirmation dialog is immediately displayed to the user for a default instrument associated with the payment method.
* Tap-to-pay (NFC)

## User Stories

* Our initial use cases focus consumer-to-business transactions, including eCommerce and bill pay.
* SPC should be usable with a variety of payment methods, including card pull, card push, and account push.

### Enrollment of multiple instruments with one authentication

* While mobile banking, Alice enroll her authenticator to be used with all her credit cards issued by this bank.
* A few days later during checkout, Alice is prompted to confirm a transaction with one of those cards via her authenticator.

### In-transaction enrollment, later authentication same merchant

* While checking out, Alice selects an instrument and is successfully authenticated via one-time password.
* She is then prompted with the opportunity to speed up future checkouts by
enrolling her authenticator with the bank in association with the same instrument.
* A few days later during checkout on the same merchant site, Alice is prompted (e.g., during an EMV&reg; 3-D Secure step up) to confirm a payment with the same instrument by using the enrolled authenticator.

### Authentication different merchant

* Having enrolled an authenticator previously (either out-of-band or during a transaction on any merchant site), Alice is shopping on an unrelated merchant
site.
* During checkout, Alice selects the same instrument and is prompted (e.g., during an EMV&reg; 3-D Secure step up) to authenticate by using the enrolled authenticator.

### Authentication with out-of-band authenticator

* While mobile banking, Alice enrolls the authenticator in her phone to be used with all her credit cards issued by this bank.
* While shopping on a desktop browser, Alice initiates a checkout.
* She receives a push notification on her phone to authenticate with that device for the transaction. At the same time, on her desktop she sees a message informing her to authenticate via her phone.
* She opens up the bank Web site on the browser on her phone, which displays the payment confirmation user experience.
* Upon successful authentication, the checkout experience on her desktop successfully completes.

### Enrollment for both payment authentication and account login

* During a guest checkout experience, Alice selects an instrument. As part of authenticating, she enrolls her authenticator with the bank in association with the instrument.
* In addition, Alice is prompted to set up a user account with this particular merchant, leveraging the same authentication credentials.
* The goal is thus to enable Alice to use the same authentication credential to (1) make payments on multiple sites, and (2) log into this site.

Notes:

* It has been pointed out that "sharing authentication credentials" may be closer to a FIDO topic than an SPC topic, but we include this use case to support that discussion.

### Express Checkout (no user presence check)

This use cases is like the previous authentication use cases (same merchant or different merchant) but removes the user presence check. Because doing so removes a security feature, we suggest native browser UX as a way to help ensure that the user has not been tricked into agreeing to less friction.

* Alice is prompted to enroll her authenticator during a transaction on a merchant site.
* In browser-native UX, Alice selects the "express checkout" option for this merchant. The browser does not share this user preference with any party.
* During a future checkout on the same merchant site, the user clicks the "Buy" button.
* The merchant communicates a preference for express checkout for this transaction. The Payment Service Provider conveys this preference to the Relying Party
when seeking SPC Payment Identifiers.
* The Relying Party makes a decision whether to accept express checkout
authentication for this transaction and communicates that along
with any SPC Payment Identifiers. ***Note*** The SPC Payment Identifiers might themselves be the mechanism for communicating the preference for express checkout authentication.
* The Payment Service Provider triggers SPC with this information.
* The browser prompts the user confirm the payment details (e.g., by clicking the "Verify" button).
* Having confirmed the user's preference for express checkout on
this merchant, the browser does not require a user presence check. The authenticator is used to sign the payment details, with additional information
that the confirmation did not include a user presence check.

Notes:

* This is a "2 click" flow: buy button, then payment confirmation dialog "verify".
* Web Authentication requires the user presence check, but CTAP does not.
* The Relying Party may make the decision for (or against) an express checkout flow based on a variety of information, including relationship to the
customer, payment amount, regulatory requirements, etc.
* The Relying Party should be sure to provide enough information so
  that if the user does not accept express checkout, there is a path
  to complete authentication.

### Frictionless Checkout (no user presence check or payment confirmation dialog)

This use cases is like the Express Checkout use case except that, in
addition, there is no browser-supplied confirmation dialog.

* Alice is prompted to enroll her authenticator during a transaction on a merchant site.
* In browser-native UX, Alice selects the "frictionless checkout" option for this merchant. The browser does not share this user preference with any party.
* During a future checkout on the same merchant site, the user clicks the "Buy" button.
* The merchant communicates a preference for frictionless checkout for this transaction. The Payment Service Provider conveys this preference to the Relying Party when seeking SPC Payment Identifiers.
* The Relying Party makes a decision whether to accept frictionless checkout
authentication for this transaction and communicates that along
with any SPC Payment Identifiers.
* The Payment Service Provider triggers SPC with this information.
* Having confirmed the user's preference for frictionless checkout on
this merchant, the authenticator is used to sign the payment details, with additional information that the transaction did not include a payment
confirmation dialog or user presence check.

Notes:

* This is a "1 click" flow: the "Buy" button.

### Authenticator unenrollment

* Alice drops her phone in the river.
* For housekeeping, she logs into her bank site and removes information about the authenticator. This causes the bank to remove any bindings between that authenticator and any instruments.
* Through her operating system or browser settings, Alice removes references to her authenticator. This causes the browser to remove any SPC-related information related to that authenticator.

### Instrument detail update

* TBD

### Authentication by bank after redirect

* Alice has enrolled her authenticator with her bank.
* While Alice is shopping on a merchant site, the merchant redirects her to her bank site to authenticate.
* The bank invokes SPC using the previously enrolled authenticator.

### Web Authentication enrollment

* Alice has received a roaming FIDO authenticator from her bank which has pre-enrolled credentials for her online banking account.
* Alice visits the bank with her desktop browser and authenticates with her roaming FIDO authenticator. The bank prompts Alice to use her platform authenticator for re-authentication and to use her platform authenticator for streamlined checkout.
* Alice thus uses her roaming FIDO authenticator to provision her platform authenticator for SPC.
* A month later Alice drops her phone in the river. She uses her roaming authenticator to log into the bank from her phone and repeats the same process to provision the platform authenticator in her phone.

Note: See related FIDO discussions on session binding assurances.

## Mockups of some user stories

* [Stripe pilot mockups](https://docs.google.com/presentation/d/1kZEv2Cf9W5kqG1fCGtP2CNQc9sVeZNuSlbRJvx_irHo/edit#slide=id.gc60d028daa_0_19)
* [Visa SRC (Oct 2020)](http://www.w3.org/2020/Talks/visa-spc-20201020.pdf)
* [Entersekt example re: tracking](https://github.com/w3c/secure-payment-confirmation/issues/49#issuecomment-817552203)

## Out of Scope

* ID & V to establish real world identity during enrollment.
* Use cases for peer-to-peer payments or business-to-business transactions.

## Future Extensions

* Support for non-payment transactions, such as confirmation to share sensitive medical data.

### Instrument selection

* See [issue 24](https://github.com/w3c/secure-payment-confirmation/issues/24) on instrument ID (when instrument selection is revisited).
* See [issue 21](https://github.com/w3c/secure-payment-confirmation/issues/21) regarding routing information if instrument ID is opaque.

## Editors

* Ian Jacobs
* Michel Weksler
