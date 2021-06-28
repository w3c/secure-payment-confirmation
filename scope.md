# Secure Payment Confirmation: Scope

Status: This is a draft document without consensus.

Secure Payment Confirmation (SPC) is a Web API to support streamlined
authentication during a payment transaction. It is designed to scale
authentication across merchants, to be used within a wide range of
authentication protocols, and to produce cryptographic evidence that
the user has confirmed transaction details.

Through this document we seek to build consensus around the scope of SPC.

See also: [SPC Requirements and Design Considerations](requirements.md) for discussion about concrete requirements.

We have been discussing three steps in a payment flow where browser
capabilities could help streamline the user experience:

* Identify the User (e.g., cookie, session, installed payment app). See also [issue 62](https://github.com/w3c/secure-payment-confirmation/issues/62).
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
across multiple merchant sites following a single enrollment.

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
across merchant sites and relying parties.

* **Cryptographic evidence**. Payment confirmation generates
cryptographic evidence of the user's confirmation of payment
details.

* **Cross-origin authentication**. With FIDO, the Relying Party that
creates FIDO credentials is the only origin that can generate an assertion 
with those credentials to authenticate the
user. With SPC, any origin can generate an assertion during a transaction 
even by leveraging another Relying Party's credentials.

## Protocols and Systems Helping to Guide Requirements

This list is the result of people joining the SPC task force:

* EMV&reg; 3-D Secure (3DS)
* EMV&reg; Secure Remote Commerce (SRC)
* Open Banking APIs, with a focus on PISP use cases.
* Real time credit (see [issue 42](https://github.com/w3c/secure-payment-confirmation/issues/42)).

## User Stories

* Our initial use cases focus on consumer-to-business transactions, including eCommerce and bill pay.
* SPC should be usable with a variety of payment methods, including card pull, card push, and account push.

### Priority

#### Authentication different merchant

* Having enrolled an authenticator previously with her bank (e.g., while mobile banking or during a transaction on any merchant site), Alice is shopping on an unrelated merchant
site.
* During checkout, Alice selects the same instrument and is prompted (e.g., during an EMV&reg; 3-D Secure step up) to authenticate by using the enrolled authenticator.

#### In-transaction enrollment, later authentication same merchant

* While checking out, Alice selects an instrument and is successfully authenticated by her bank via one-time password, Web Authentication, or other means.
* She is then prompted with the opportunity to speed up future checkouts by
enrolling her authenticator with her bank and associating it with the same instrument.
* A few days later during checkout on the same merchant site, Alice is prompted (e.g., during an EMV&reg; 3-D Secure step up) to confirm a payment with the same instrument by using the enrolled authenticator.

Note: This use case intends to capture the "in-transaction enrollment" use case. We also expect to support enrollment independent of a transaction.

#### Enrollment of multiple instruments with one authentication

* While mobile banking, Alice enrolls her authenticator to be used with all her credit cards issued by this bank.
* A few days later during checkout, Alice is prompted to confirm a transaction with one of those cards via her authenticator.

#### Association of new instrument with existing authentication credential

* While visiting her bank site, Alice enrolls her authenticator in association with two instruments.
* The following week she associates a third instrument with the same authentication credential.

#### EMV&reg; Secure Remote Commerce (SRC) System as Relying Party

* Alice checkouts on a merchant web site with SRC, which triggers the SRC Digital Card Facilitator (DCF) to be displayed. The SRC DCF asks whether she wants to use biometric authentication to streamline payment. She agrees and SRC DCF redirects her to her bank where she goes through an ID&V process with her bank for the credit card she wishes to use.
* As an alternative, Alice visits her bank, authenticates to her bank, enrolls into biometric authentication, and selects card(s) that she wants to make available to SRC. The bank (the Relying Party) shares the authentication credential with the SRC System.
* The following week Alice checkouts with a merchant enabled with SRC. The SRCi/DCF prompts Alice to do biometric authentication. The SRC System reviews the authentication results, and the bank authorizes the transaction.

Note: We anticipate that flows of "SPC with 3DS" and "SPC with SRC" will be similar, but with different entities as Relying Party.

### Lower Priority

#### Enrollment for both payment authentication and account login

* During a guest checkout experience, Alice selects an instrument. As part of authenticating, she enrolls her authenticator with her bank in association with the instrument.
* In addition, Alice is prompted to set up a user account with this particular merchant, leveraging the same authentication credentials.
* The goal is thus to enable Alice to use the same authentication credential to (1) make payments on multiple sites, and (2) log into this site.

Notes:

* It has been pointed out that "sharing authentication credentials" may be closer to a FIDO topic than an SPC topic, but we include this use case to support that discussion.

#### Express Checkout (no user presence check)

This use cases is like the previous authentication use cases (same merchant or different merchant) but removes the user presence check. Because doing so removes a security feature, we suggest native browser UX as a way to help ensure that the user has not been tricked into agreeing to less friction.

* Alice is prompted to enroll her authenticator during a transaction on a merchant site.
* Through a browser-native UX, Alice selects the "express checkout" option for this merchant. The browser does not share this user preference with any party.
* During a future checkout on the same merchant site, the Alice clicks the "Buy" button.
* The merchant communicates a preference for express checkout for this transaction. The Payment Service Provider conveys this preference to the Relying Party
when seeking SPC Payment Identifiers.
* The Relying Party makes a decision whether to accept express checkout
authentication for this transaction and communicates that along
with any SPC Payment Identifiers. ***Note*** The SPC Payment Identifiers might themselves be the mechanism for communicating the preference for express checkout authentication.
* The Payment Service Provider triggers SPC with this information.
* Having confirmed the user's preference for express checkout on
this merchant, the browser does not require a user presence check.
* The browser prompts the user confirm the payment details (e.g., by clicking the "Verify" button).
* The authenticator is used to sign the payment details, with additional information
that the confirmation did not include a user presence check.

Notes:

* This is a "2 click" flow: buy button, then payment confirmation dialog "verify".
* Web Authentication requires the user presence check, but CTAP does not.
* The Relying Party may make the decision for (or against) an express checkout flow based on a variety of information, including relationship to the
customer, payment amount, regulatory requirements, etc.
* The Relying Party should be sure to provide enough information so
  that if the user does not accept express checkout, there is a path
  to complete authentication.

#### Frictionless Checkout (no user presence check or payment confirmation dialog)

This use cases is like the Express Checkout use case except that, in
addition, there is no browser-supplied confirmation dialog.

* Alice is prompted to enroll her authenticator during a transaction on a merchant site.
* Through a browser-native UX, Alice selects the "frictionless checkout" option for this merchant. The browser does not share this user preference with any party.
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

### Additional Considerations

These use cases represent additional considerations, some of which (e.g., unenrollment) may be orthogonal to the primary flows above. 

#### Merchant as Relying Party

* Alice logs into her favorite merchant using a merchant proprietary mechanism or using biometric authentication.   
* The merchant asks Alice if she wants to use biometric authentication to streamline payment. She agrees and goes through an ID&V process with her bank for the credit card she wishes to use. (The merchant may decide to perform IDamp;&V during the checkout or outside of the checkout.)
* The merchant is the relying party for this authentication credential, and shares authentication data with Alice’s bank and/or payment network to allow for partial or full validation of authentication results in subsequent checkouts.
* The following week Alice checks out on the merchant site and is prompted by the merchant to do biometric authentication. The merchant uses SPC then shares authentication results with Alice’s bank and/or payment network, which reviews the data. The bank authorizes the transaction.

#### Authentication by Relying Party after redirect

* Alice has enrolled her authenticator with a Relying Party (e.g. her bank)
* While Alice is shopping on a merchant site, the merchant redirects her to the Relying Party site to authenticate.
* The Relying Party invokes SPC using the previously enrolled authenticator.

#### Authenticator unenrollment

* Alice drops her phone in the river.
* For housekeeping, she logs into her bank site and removes information about the authenticator. This causes the bank to remove any bindings between that authenticator and any instruments.
* Through her operating system or browser settings, Alice removes references to her authenticator. This causes the browser to remove any SPC-related information related to that authenticator.

### Low Priority

The following use cases may be of interest but are of distinctly lower
priority:

* Authentication with out-of-band authenticator. See [issue 30](https://github.com/w3c/secure-payment-confirmation/issues/30).
* QR-code triggered payment. User points phone at QR code which represents a Web Page. In vanilla mode, Web Page includes "Buy" button to trigger Payment Request. But in streamlined mode, SPC payment confirmation dialog is immediately displayed to the user for a default instrument associated with the payment method.
* Tap-to-pay (NFC)

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
