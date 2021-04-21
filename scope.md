# Secure Payment Confirmation: Scope, Requirements, and Priorities

Status: This is a draft document without consensus.

Secure Payment Confirmation is a Web API to support streamlined
authentication during a payment transaction. It is designed for use
within a wide range of authentication protocols and to produce
cryptographic evidence that the user has confirmed transaction
details.

Though this document we seek to build consensus around the scope,
requirements, and priorities of Secure Payment Confirmation (SPC).

We have been discussing three steps in a payment flow where browser
capabilities could help streamline the user experience:

* Establish user identity (e.g., cookie, session, payment app)
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

* **Transaction Confirmation**. SPC generates cryptographic proof of the
user's confirmation of transaction details. This is intended to help
entities fulfill regulatory requirements (e.g., when used with 3DS2
for PSD2).

* **Phishing-resistant**. Different entities may render the transaction
details that the user confirms. In many cases the browser will display
the transaction details in native UI, but SPC also supports display by
secure hardware. Because the browser or secure hardware controls the
display of transaction details, SPC is phishing-resistant.

* **Simpler Front-end Deployment**. Because the browser or secure hardware
controls the display, the merchant site (or payment service provider)
does not need to provide user experience for authentication.

* **Strong Security Boundary**. With FIDO, the Relying Party that creates
FIDO credentials is the only origin that can authenticate the user
with those credentials. With SPC, any origin can authenticate the user
using another Relying Party's credentials, provided that the
authentication takes place within a known payment context. At the
current time, Payment Request API establishes the payment context. In
practice, SPC can reduce the need to embed code provided by a Relying
Party in a Web page, reducing security risks.

See also [more SPC benefits](https://github.com/w3c/webpayments/wiki/Secure-Payment-Confirmation#benefits).

## Definitions

**SPC Credential**
: The data stored in the browser that represents the association between a payment instrument and some authentication credential(s). An SPC Credential includes one or more SPC Credential Identifiers.

**SPC Credential Identifiers**
: These identifiers are generated during enrollment and stored by the Relying Party in association with a payment instrument. 

**SPC Request**
: Information provided as input to the API. It is likely to include
SPC Credential Identifiers, sources of randomness, and other data.

**SPC Assertion**
: The output of a successful SPC API authentication.

## Use Cases Helping to Guide Requirements

This list is the result of people joining the SPC task force:

* EMV&reg; 3-D Secure (3DS)
* EMV&reg; Secure Remote Commerce (SRC)
* Open Banking APIs

The following use cases may be of interest but are of distinctly lower
priority:

* Delegation to the merchant as Relying Party.
* QR-code triggered payment. User points phone at QR code which represents a Web Page. In vanilla mode, Web Page includes "Buy" button to trigger Payment Request. But in streamlined mode, SPC transaction dialog is immediately displayed to the user for a default instrument associated with the payment method.
* Tap-to-pay (NFC)

## User Journeys

There are two elements to the user journey:

1) Enrollment
2) Authentication (at transaction time)

### Enrollment

Enrollment may happen outside of a transaction or during a transaction.
Enrollment considerations include:

* It is desirable that previously enrolled authentication credentials (e.g., FIDO) can be "enhanced" to become SPC Credentials.

* In some cases the user may consent to a particular (future) authentication flow. For example, the user might agree to a lower friction flow for some types of transactions or some merchants.

### Authentication

* At transaction time, if the SPC Request does not match any SPC
Credentials stored in the browser, there is no user experience.  This
enables the merchant to provide a seamless fallback experience.

* If the browser finds a match, the browser prompts the user to
confirm the transaction details. The browser determines the user
experience, but the required user gestures may vary according to the
authentication method. For example, the user experience may involve a
multi-factor biometric authentication. Or it may involve less friction
if the user previously consented to such an experience.

### Role of Payment Handlers

* If the user is responding to a Payment Request with a payment handler,
the payment handler may invoke SPC. In this case, the user journey is the
same, but the SPC Assertion is returned to the payment handler, not the
merchant.

## Browser Behaviors

### Enrollment

* The API will will support registration of a Payment Credential
  in the browser.
* TBD whether we standardize Payment Credential enrollment UX.

### Transaction

* Given an ordered list of SPC Credential Identifiers in the SPC
  Request, the browser picks the first SPC Credential that matches and
  uses that credential to authenticate the user. If the browser does
  not have any matching SPC Credentials, the API returns null without
  any user experience.
* The browser (or delegated secure hardware) displays transaction
  details to the user: amount, beneficiary, and payment instrument.
  The Payment Credential includes displayable information about
  the payment instrument.
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

## Requirements and Design Considerations

Note: As with all W3C technologies, we anticipate strong
considerations of security, privacy, accessibility, and
internationalization. In this summary document, we do not focus on
those requirements, but we anticipate they will be part of discussions
and the SPC specification.


### General

* It must be possible to determine programmatically whether a browser supports SPC.
* Many authentication protocols include a source of randomness to ensure freshness. How does SPC support that (e.g., as input data?)? It may not always come from the RP.
* It should be possible to trigger SPC from an iframe.
* It should be possible to trigger SPC from a payment app (PH API).
* The API should support different sources of preferences:
   * Merchant
   * Relying Party
   * User (e.g., "I always want full-friction experiences.")
* Should there be a standardized SPC enrollment user experience?
* When enrollment happens as part of a transaction, if the transaction can complete independently of enrollment, that should be supported.

### SPC Credential

* SPC Credential Identifiers are opaque and origin bound to improve
  privacy. The browser maps SPC Credential Identifiers to stored SPC Credentials. See [issue 49](https://github.com/w3c/secure-payment-confirmation/issues/49#issuecomment-817552203)
* An SPC Credential is likely to include the following kind of data:
   * Displayable information about the instrument, provided by the Relying Party.
   * Authentication method-specific (private) data. For example, for FIDO, this structure might include a credential id and an rpid.
* SPC Credentials need to be distinguishable from other types of credentials (e.g., SPC Credential recognized as different from a FIDO Credential).

### SPC Assertion

* The SPC Assertion includes a version of the transaction details cryptographically signed after user confirmation. This data may be validated by the Relying Party or any other party with the appropriate key.
* Transaction details include merchant origin/identifier, amount and currency, transaction id. 
* Information about the user's journey (e.g., with or without user presence check. (Should this data be standardized or private?)

### FIDO-Specific Considerations

* FIDO credentials should be "enhanceable" to SPC Credentials.
* SPC should support both local and roaming authenticators.
* Some authenticators may be capable of rendering the transaction confirmation dialog; the API should take this into account.
* [Large Blob](https://www.w3.org/TR/webauthn-2/#sctn-large-blob-extension) (WebAuthn Level 2) may be used to create portable stored data to reduce enrollment costs. Use case: I enroll my authenticator via one browser, but stored data can be used in another browser.

## Out of Scope

* ID & V to establish real world identity during enrollment.

## Future Extensions

* Support for non-payment transactions, such as confirmation to share sensitive medical data.

## Editors

* Ian Jacobs
* Michel Weksler
