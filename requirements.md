# Secure Payment Confirmation: Requirements and Design Considerations

Status: This is a draft document without consensus.

Though this document we seek to build consensus around requirements
and design considerations for SPC.

***Notes:***

* These requirements are not prioritized.
* "MUST", "SHOULD", and "MAY" are used per [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

See also: [SPC Scope](scope.md) for use cases as well as the [issues list](https://github.com/w3c/secure-payment-confirmation/issues).

## Definitions

**Instrument** <a name="dfn-instrument"></a>
: A mechanism used to transfer value from a payer to a payee.

**SPC Credential** <a name="dfn-spc-credential"></a>
: Data that represents the association between an instrument and an authentication credential. Note: Representation of multiple relationships is an implementation detail (e.g., multiple authentications corresponding to a single instrument, or multiple instruments enrolled for a given authentication).

**SPC Credential Identifiers** <a name="dfn-credential-id"></a>
: Each SPC Credential Identifier refers to one SPC Credential. These identifiers are generated during enrollment and returned by the API to the Relying Party.

**SPC Request** <a name="dfn-spc-request"></a>
: Information provided as input to the API. It is likely to include
[SPC Credential Identifiers](#dfn-credential-id), sources of randomness, and other data.

**SPC Assertion** <a name="dfn-spc-assertion"></a>
: The output of a successful SPC API authentication.

## Design Goals
* We endeavor to describe requirements independently of specific APIs, in particular Web Authentication and Payment Request. See [issue 65](https://github.com/w3c/secure-payment-confirmation/issues/65)
* Where requirements have been suggested that are specific to an API, we list those in self-contained sections.

## Requirements

### Feature Detection

* It must be possible to determine through JavaScript feature detection whether a user agent supports SPC.

### Web Context Support

* It must be possible to call SPC from a Web site or a payment handler. Note: This implies changes to the Payment Handler API are likely.
* Because many checkout experiences are offered through iframes, it must be possible to call SPC from an iframe. SPC usage within an iframe must be enabled through a permission policy. See [issue 68](https://github.com/w3c/secure-payment-confirmation/issues/68).

### Enrollment

* It must be possible to enroll an [SPC Credential](#dfn-spc-credential) outside of a transaction.
* It must be possible to enroll an [SPC Credential](#dfn-spc-credential) following a prior (non-SPC) authentication during a transaction. This enrollment should not prevent timely completion of the transaction.
* It is not a requirement that the relying party be able to enroll an [SPC Credential](#dfn-spc-credential) in a third-party context. However, it could improve common payment flows if the relying party could enroll an SPC credential from a cross-origin iframe.
* It must be possible to enroll an [SPC Credential](#dfn-spc-credential) from a payment handler.

#### Instrument Information at Enrollment

* The Relying Party must not be required to provide definitive
  information about a specific instrument at enrollment time. In other
  words, the API supports dynamic binding to a specific instrument at
  authentication time. This feature renders unnecessary additional
  functionality to update stored instrument information.
* It is not a requirement that instrument information be stored in the
  browser as part of the [SPC Credential](#dfn-spc-credential).
* The protocol should support multiple ways of accessing the instrument display information, including browser storage and authenticator storage.

#### Enrollment User Experience

* Each browser should natively support an [SPC Credential](#dfn-spc-credential) enrollment user experience.
* The user agent should [require and consume at least one transient user activation](https://html.spec.whatwg.org/multipage/interaction.html#activation-consuming-api) in order to display an SPC enrollment user experience.

### Payment Confirmation

#### Instrument Information at Payment Confirmation

* For each [SPC Credential](#dfn-spc-credential), instrument
  information (e.g., a display string and icon) must be available for
  use within the payment confirmation user experience.
* The party that calls the API must be able to provide instrument
  information for each input SPC Credential Identifier. Note: The
  Relying Party is the authoritative source of instrument information.
  If another party provides conflicting instrument information as
  input to the API, the Relying Party can detect this during
  subsequent validation.

#### Payment Confirmation User Experience

* Each browser must natively support a payment confirmation user experience.
* The user agent must [require and consume at least one transient user activation](https://html.spec.whatwg.org/multipage/interaction.html#activation-consuming-api) in order to display an SPC user experience.
* Although we anticipate that in most cases the browser will render the payment confirmation user experience, the protocol must support rendering by other entities (e.g., the operating system or authenticator).
* The payment confirmation user experience must display instrument information such as label and icon.
* The payment confirmation user experience must display amount and currency of the payment.
* For regulatory reasons, the party that invokes SPC must be able to specify a timeout for the payment confirmation user experience. See [issue 67](https://github.com/w3c/secure-payment-confirmation/issues/67).
* The payment confirmation user experience should include the beneficiary name, and optionally the title and favicon of the page where it was called. See [issue 48](https://github.com/w3c/secure-payment-confirmation/issues/48) on merchant information display.

#### Levels of User Interaction during Payment Confirmation

* The API must support payment confirmation with two factor authentication (e.g., with FIDO user verification check).
* The API must support payment confirmation with one factor authentication with user presence check.
* The API should support payment confirmation with one factor authentication (possession) without user presence check.
* TBD: The API should support payment confirmation with one factor authentication (possession) but without a visible dialog and without a user presence check.
* The API must allow the relying party to express a preference for any of the supported levels of user interaction.
* For each transaction, a merchant should be able to express to the relying
party a preference to use (or not use) any of the supported levels of user interaction.
* If the browser supports a low-friction flow option, the browser must support a user preference to override that option and maintain authentication with more user interaction.

#### Cross-Browser Support

* If the user enrolled an [SPC Credential](#dfn-spc-credential) when using one instance of a browser, it should be possible to leverage that authentication from a different instance of the same browser (e.g., both browsers are Firefox)
* If the user enrolled an [SPC Credential](#dfn-spc-credential) when using one instance of a browser, it should be possible to leverage that authentication from any browser (e.g., one browser is Firefox and the other is Chrome).

### SPC Credentials

* See [issue 69](https://github.com/w3c/secure-payment-confirmation/issues/69) for discussion of requirements when more than one [SPC Credential](#dfn-spc-credential) matches input data.
* When no [SPC Credential](#dfn-spc-credential) matches input data, the protocol should terminate without any user experience to allow for seamless fallback behaviors.
* The user agent should support a user experience of leveraging multiple instruments for a single authentication. Each SPC Credential must be independently addressable. Note: This binding might take place at either enrollment or authentication time.


### SPC Assertions

* The [SPC Assertion](#dfn-spc-assertion) must include at least: a merchant origin/identifier, amount and currency, transaction id.
The [SPC Assertion](#dfn-spc-assertion) must include the cryptographic nonce / challange if provided within the SPC request.
* The [SPC Assertion](#dfn-spc-assertion) must also include information about the user's journey. This information may be part of the authenticator's own assertion. For example, the assertion must indicate whether the user completed the transaction without a user presence check.
* The [SPC Assertion](#dfn-spc-assertion) must also include information about which entity displayed the payment confirmation user experience (browser, OS, or authenticator).
* The [SPC Assertion](#dfn-spc-assertion) must include a signature over that data (based on the associated authenticator). This signature may be validated by the Relying Party or any other party with the appropriate key.

### Lifecycle Management

* The user must be able to remove individual [SPC Credentials](#dfn-spc-credential) from a browser instance.
* The ecosystem should enable the user to communicate to the relying party to forget [SPC Credential Identifiers](#dfn-credential-id). This might happen in a variety of ways (e.g., forget this authenticator and all associated instruments; forget any authenticators associated with this instrument, etc.). See [issue 63](https://github.com/w3c/secure-payment-confirmation/issues/63).

### Security and Privacy Considerations

* [SPC Credentials](#dfn-credential) must be origin-bound. All [SPC Credential Identifiers](#dfn-credential-id) on the same origin are expected be distinct.
* The API should allow relying parties to reduce the risk of cross-site tracking that might arise through the reuse of [SPC Credential Identifiers](#dfn-credential-id). See [issue 77](https://github.com/w3c/secure-payment-confirmation/issues/77).

### FIDO Considerations

* FIDO credentials should be "enhanceable" to [SPC Credentials](#dfn-spc-credential). See [issue 79](https://github.com/w3c/secure-payment-confirmation/issues/79).
* SPC credentials should also usable as ordinary FIDO credentials. See [issue 39](https://github.com/w3c/secure-payment-confirmation/issues/39).
* SPC credentials must be programmatically distinguishable from FIDO credentials.
* SPC should support both platform and roaming authenticators. See [issue 31](https://github.com/w3c/secure-payment-confirmation/issues/31) on discoverable credentials and [issue 12](https://github.com/w3c/secure-payment-confirmation/issues/12) on roaming authenticator behaviors.
* [Large Blob](https://www.w3.org/TR/webauthn-2/#sctn-large-blob-extension) (WebAuthn Level 2) may be used to create portable stored data to reduce enrollment costs. Use case: I enroll my authenticator via one browser, but stored data can be used in another browser.

## Editor

* Ian Jacobs

