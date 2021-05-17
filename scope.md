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

## Unique Features of SPC

* **Streamlined UX**. FIDO authentication provides a very good user
experience. Our expectation is that the SPC "happy path" provides an
even more streamlined experience. One important goal is to remove the
need to redirect an enrolled user during a payment
transaction. Another important goal is to speed up authentication
compared to other approaches such as one-time passwords. A third goal
is to create a consistent and trusted authentication experience across
the Web.

* **Scalable and Ubiquitous**. SPC supports streamlined authentication
across merchant sites without additional enrollment. Note that with
FIDO, the Relying Party that creates FIDO credentials is the only
origin that can authenticate the user with those credentials. With
SPC, any origin can authenticate the user using another Relying
Party's credentials, provided that the authentication takes place
within a known payment context. At the current time, Payment Request
API establishes the payment context (but see [issue 56](https://github.com/w3c/secure-payment-confirmation/issues/56)). In addition, enabling payment
service providers or others to authenticate the user can reduce the
need to embed code provided by a Relying Party in a Web page, reducing
security risks.

* **Transaction Confirmation**. SPC generates cryptographic proof of
the user's confirmation of transaction details. This is intended to
help entities fulfill regulatory requirements (e.g., SCA for PSD2) or
support other customer authentication use cases.

* **Phishing-resistant**. Different entities may render the transaction
details that the user confirms. In many cases the browser will display
the transaction details in native UI, but SPC also supports display by
secure hardware. Because the browser or secure hardware controls the
display of transaction details, SPC is phishing-resistant.

* **Simpler Front-end Deployment**. The browser (or secure hardware)
manages the display of transaction confirmation, removing the need for
other parties (e.g., issuing banks or payment apps) to do so.

See also [more SPC benefits](https://github.com/w3c/webpayments/wiki/Secure-Payment-Confirmation#benefits).

## Definitions

**Instrument**
: A mechanism used to transfer value from a payer to a payee.

**SPC Credential**
: Data that represents the association between an instrument and an authentication credential. Note: Management of multiple relationships is an implementation detail (e.g., multiple authentications corresponding to a single instrument, or multiple instruments enrolled for a given authentication).

**SPC Credential Identifiers**
: Each SPC Credential Identifier refers to one SPC Credential. These identifiers are generated during enrollment and stored by the Relying Party in association with an instrument. An instrument may be addressable by more than one SPC Credential Identifier (e.g., when the user has authenticated through different devices for that instrument).

**SPC Request**
: Information provided as input to the API. It is likely to include
SPC Credential Identifiers, sources of randomness, and other data.

**SPC Assertion**
: The output of a successful SPC API authentication.

## Protocols and Systems Helping to Guide Requirements

This list is the result of people joining the SPC task force:

* EMV&reg; 3-D Secure (3DS)
* EMV&reg; Secure Remote Commerce (SRC)
* Open Banking APIs, with a focus on PISP use cases.
* Real time credit (see [issue 42](https://github.com/w3c/secure-payment-confirmation/issues/42)).

The following use cases may be of interest but are of distinctly lower
priority:

* Delegation to the merchant as Relying Party.
* QR-code triggered payment. User points phone at QR code which represents a Web Page. In vanilla mode, Web Page includes "Buy" button to trigger Payment Request. But in streamlined mode, SPC transaction dialog is immediately displayed to the user for a default instrument associated with the payment method.
* Tap-to-pay (NFC)

## User Stories

* Our initial use cases focus consumer-to-business transactions, including eCommerce and bill pay.
* SPC should be usable with a variety of payment methods, including card pull, card push, and account push.

### Enrollment outside a transaction

* While mobile banking, Alice enroll her authenticator to be used with all her credit cards issued by this bank.
* A few days later during checkout, Alice is prompted to confirm a transaction with one of those cards via her authenticator.

### In-transaction enrollment, authentication same merchant

* While checking out, Alice selects an instrument and is successfully authenticated via one-time password.
* She is then prompted with the opportunity to speed up future checkouts by
enrolling her authenticator with the bank in association with the same instrument.
* A few days later during checkout on the same merchant site, Alice is prompted (e.g., during an EMV&reg; 3-D Secure step up) to confirm a transaction with the same instrument by using the enrolled authenticator.

### Authentication different merchant

* Having enrolled an authenticator previously (either out-of-band or during a transaction on any merchant site), Alice is shopping on an unrelated merchant
site.
* During checkout, Alice selects the same instrument and is prompted (e.g., during an EMV&reg; 3-D Secure step up) to authenticate by using the enrolled authenticator.

### Authentication with out-of-band authenticator

* While mobile banking, Alice enrolls the authenticator in her phone to be used with all her credit cards issued by this bank.
* While shopping on a desktop browser, Alice initiates a checkout.
* She receives a push notification on her phone to authenticate with that device for the transaction.
* Upon successful authentication, the checkout experience on her desktop successfully completes.

3) Having enrolled an authenticator previously (either out-of-band or during a transaction on any merchant site), Alice is shopping on an unrelated merchant
site.
* During checkout, Alice selects the same instrument and is prompted to authenticate by using the enrolled authenticator.

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
* The browser prompts the user confirm the transaction details (e.g., by clicking the "Verify" button).
* Having confirmed the user's preference for express checkout on
this merchant, the browser does not require a user presence check. The authenticator is used to sign the transaction details, with additional information
that the transaction did not include a user presence check.

Notes:

* This is a "2 click" flow: buy button, then transaction confirmation dialog "verify".
* Web Authentication requires the user presence check, but CTAP does not.
* The Relying Party may make the decision for (or against) an express checkout flow based on a variety of information, including relationship to the
customer, transaction amount, regulatory requirements, etc.
* The Relying Party should be sure to provide enough information so
  that if the user does not accept express checkout, there is a path
  to complete authentication.

### Frictionless Checkout (no user presence check or transaction confirmation dialog)

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
this merchant, the authenticator is used to sign the transaction details, with additional information that the transaction did not include a transaction
confirmation dialog or user presence check.

Notes:

* This is a "1 click" flow: the "Buy" button.

### Authenticator unenrollment

* Alice drops her phone in the river.
* For housekeeping, she logs into her bank site and removes information about the authenticator. This causes the bank to remove any bindings between that authenticator and any instruments.
* Through her operating system or browser settings, Alice removes references to her authenticator. This causes the browser to remove any SPC Credentials that refer to that authenticator.

### Instrument detail update

* TBD

### Authentication by bank after redirect

* Alice has enrolled her authenticator with her bank.
* While Alice is shopping on a merchant site, the merchant redirects her to her bank site to authenticate.
* The bank invokes SPC using the previously enrolled authenticator.

## Mockups of some user stories

* [Stripe pilot mockups](https://docs.google.com/presentation/d/1kZEv2Cf9W5kqG1fCGtP2CNQc9sVeZNuSlbRJvx_irHo/edit#slide=id.gc60d028daa_0_19)
* [Visa SRC (Oct 2020)](http://www.w3.org/2020/Talks/visa-spc-20201020.pdf)
* [Entersekt example re: tracking](https://github.com/w3c/secure-payment-confirmation/issues/49#issuecomment-817552203)

## Assumptions and Requirements for User Journeys

There are two elements to the user authentication journey:

1) Enrollment
2) Authentication (at transaction time)

### Enrollment

Enrollment may happen outside of a transaction (see [issue 44](https://github.com/w3c/secure-payment-confirmation/issues/44)) or during a transaction.
Enrollment considerations include:

* It is desirable that previously enrolled authentication credentials (e.g., FIDO) can be "enhanced" to become SPC Credentials.

* In some cases the user may consent to a particular (future) authentication flow. For example, the user might agree to a lower friction flow for some types of transactions or some merchants.

### Authentication

* At transaction time, if the SPC Request does not match any SPC
Credentials stored in the browser, there is no user experience.  This
enables the merchant to provide a seamless fallback experience that
can leverage the modal browsing window provided by the browser.

* If the browser finds a match, the browser prompts the user to
confirm the transaction details including amount, merchant identification
and instrument information. The browser determines the user
experience, but the required user gestures may vary according to the
authentication method. For example, the user experience may involve a
multi-factor biometric authentication. Or it may involve less friction
if the user previously consented to such an experience.

Related issues:

 * [Issue 34](https://github.com/w3c/secure-payment-confirmation/issues/34) and [issue 29](https://github.com/w3c/secure-payment-confirmation/issues/29) on frictionless flows.
 * [Issue 30](https://github.com/w3c/secure-payment-confirmation/issues/30) on out-of-band authentication to a secondary device.
 * [Issue 28](https://github.com/w3c/secure-payment-confirmation/issues/28) on flows where the RP is not invoked before payment.
 * [Issue 48](https://github.com/w3c/secure-payment-confirmation/issues/48) on the merchant details displayed in the payment confirmation dialog

### Role of Payment Handlers

* If the user is responding to a Payment Request with a payment handler,
the payment handler may invoke SPC. In this case, the user journey is the
same, but the SPC Assertion is returned to the payment handler, not the
merchant.

## Browser Behaviors

### Enrollment

* The API will will support registration of a Payment Credential
  in the browser.
* Enrollment will be possible during or outside of a transaction (see [issue 44](https://github.com/w3c/secure-payment-confirmation/issues/44)).
* TBD whether we standardize Payment Credential enrollment UX.

### Transaction

* See [issue 8](https://github.com/w3c/secure-payment-confirmation/issues/8) on whether SPC can be invoked before Payment Request, or only during.
* Given an ordered list of SPC Credential Identifiers in the SPC
  Request, the browser picks the first SPC Credential that matches and
  uses that credential to authenticate the user. If the browser does
  not have any matching SPC Credentials, the API returns null without
  any user experience.
* The browser (or delegated secure hardware) displays transaction
  details to the user: amount, beneficiary, and instrument.
  The Payment Credential includes displayable information about
  the instrument.
* The browser prompts the user to confirm the transaction details.
  The extent of the user experience depends on the authentication
  method.
* After successful authentication, the API returns a standardized
  SPC Assertion. Unsuccessful authentication results in a different
  behavior (e.g., error).

### Role of Payment Handlers

* During Payment Request API a payment handler can invoke SPC
  authentication.
* After successful authentication, the API returns a standardized
  SPC Assertion to the payment handler.

### Lifecycle management

* Browsers need to enable users to manage (view, update, delete) SPC
  Credentials.
* See [issue 14](https://github.com/w3c/secure-payment-confirmation/issues/14) on handling stale display information.
* See [issue 63](https://github.com/w3c/secure-payment-confirmation/issues/63) about unenrollment.

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
