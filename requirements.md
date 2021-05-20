# Secure Payment Confirmation: Requirements and Design Considerations

Status: This is a draft document without consensus.

Though this document we seek to build consensus around requirements
and design considerations for SPC.

***Notes:***

* These requirements are not prioritized.
* "MUST", "SHOULD", and "MAY" are used per [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

See also: [SPC Scope](scope.md) for use cases.

## Definitions

**Instrument** <a name="dfn-instrument"></a>
: A mechanism used to transfer value from a payer to a payee.

**SPC Credential** <a name="dfn-spc-credential"></a>
: Data that represents the association between an instrument and an authentication credential. Note: Management of multiple relationships is an implementation detail (e.g., multiple authentications corresponding to a single instrument, or multiple instruments enrolled for a given authentication).

**SPC Credential Identifiers** <a name="dfn-credential-id"></a>
: Each SPC Credential Identifier refers to one SPC Credential. These identifiers are generated during enrollment and stored by the Relying Party in association with an instrument. An instrument may be addressable by more than one SPC Credential Identifier (e.g., when the user has authenticated through different devices for that instrument).

**SPC Request** <a name="dfn-spc-request"></a>
: Information provided as input to the API. It is likely to include
[SPC Credential Identifiers](#dfn-credential-id), sources of randomness, and other data.

**SPC Assertion** <a name="dfn-spc-assertion"></a>
: The output of a successful SPC API authentication.

## API Independence
* We endeavor to describe requirements independently of specific APIs, in particular Web Authentication and Payment Request. See [issue 65](https://github.com/w3c/secure-payment-confirmation/issues/65)
* Where requirements have been suggested that are specific to an API, we list those in self-contained sections.

## Feature Detection

* It must be possible to determine programmatically whether a browser supports SPC.

## Web Context Support

* It must be possible to call SPC from a Web site or a payment handler. Note: This implies changes to the Payment Handler API are likely.
* Because many checkout experiences are offered through iframes, it must be possible to call SPC from an iframe. See [issue 68](https://github.com/w3c/secure-payment-confirmation/issues/68).

## Enrollment

* It must be possible to enroll an [SPC Credential](#dfn-spc-credential) outside of a transaction.
* It must be possible to enroll an [SPC Credential](#dfn-spc-credential) following a prior (non-SPC) authentication during a transaction. This enrollment should not prevent timely completion of the transaction.
* It must be possible to enroll multiple instruments for a single authentication. Each resulting [SPC Credential](#dfn-spc-credential) (that is: each instrument/authentication credential binding) must be independently addressable.
* It is not a requirement that the relying party be able to enroll an [SPC Credential](#dfn-spc-credential) in a third-party context.
* It must be possible to enroll an [SPC Credential](#dfn-spc-credential) from a payment handler.
* Each browser should natively support an SPC credential enrollment user experience.
* One of the authentication methods for enrolling an [SPC Credential](#dfn-spc-credential) should be Web Authentication.

### Instrument Information

* Enrollment of an instrument must include display information for it.
* Because instrument display information is available to the relying party (e.g., provided by the relying party itself, the merchant, or some other party), it is not a requirement that this information be stored in the browser as part of the [SPC Credential](#dfn-spc-credential).
* The relying party should be able to update the instrument information of an enrolled [SPC Credential](#dfn-spc-credential) (e.g., for new card art).

## Transaction Confirmation User Experience

* To add: user activation gesture requirement.
* Each browser must natively support a transaction confirmation user experience.
* Although we anticipate that in most cases the browser will render the transaction confirmation user experience, the protocol must support rendering by other entities (e.g., the operating system or authenticator).
* The transaction confirmation user experience must display instrument information such as label and icon.
* The transaction confirmation user experience must display amount and currency of the payment.
* For regulatory reasons, the party that invokes SPC must be able to specify a timeout for the user experience. See [issue 67](https://github.com/w3c/secure-payment-confirmation/issues/67).
* The transaction confirmation user experience should include the beneficiary name, and optionally the title and favicon of the page where it was called. See [issue 48](https://github.com/w3c/secure-payment-confirmation/issues/48) on merchant information display.

### Sources of Instrument Information

* The protocol should support multiple ways of accessing the instrument display information, including browser storage, authenticator storage, and merchant-provided data. Note that merchant data may not be trustworthy, but during subsequent
authorization it can be validated against relying-party stored instrument display data.

### Cross-Browser Support

* If the user enrolled an [SPC Credential](#dfn-spc-credential) when using one instance of a browser, it should be possible to leverage that authentication from a different instance of the same browser (e.g., both browsers are Firefox)
* If the user enrolled an [SPC Credential](#dfn-spc-credential) when using one instance of a browser, it should be possible to leverage that authentication from any browser (e.g., one browser is Firefox and the other is Chrome).

### Low Friction Flows

* The browser should support transaction confirmation without hardware
  authentication (e.g., no FIDO user presence check) when requested by
  the relying party.

* For each transaction, a merchant should be able to express to the relying party a preference for a low-friction flow (or not to use a low-friction flow).

* If the browser supports a low-friction flow option, the browser must
  support a user preference to override that option and maintain the
  full hardware-supported (e.g., biometric) flow.

### Zero Friction Flows

* TBD

## Unenrollment

* The ecosystem should enable the user to communicate to the relying party to forget [SPC Credential Identifiers](#dfn-credential-id). This might happen in a variety of ways (e.g., forget this authenticator and all associated instruments; forget any authenticators associated with this instrument, etc.). See [issue 63](https://github.com/w3c/secure-payment-confirmation/issues/63).

## SPC Credentials

* See [issue 69](https://github.com/w3c/secure-payment-confirmation/issues/69) for discussion of requirements when more than one [SPC Credential](#dfn-spc-credential) matches input data.
* When no [SPC Credential](#dfn-spc-credential) matches input data, the protocol should terminate without any user experience to allow for seamless fallback behaviors.

### Lifecycle Management

* The user must be able to remove individual [SPC Credentials](#dfn-spc-credential) from a browser instance.

## SPC Assertions

* The [SPC Assertion](#dfn-spc-assertion) must include at least: a merchant origin/identifier, amount and currency, transaction id.
The [SPC Assertion](#dfn-spc-assertion) must include the cryptographic nonce / challange if provided within the SPC request.
* The [SPC Assertion](#dfn-spc-assertion) must also include information about the user's journey. This information may be part of the authenticator's own assertion. For example, the assertion must indicate whether the user completed the transaction without a user presence check.
* The [SPC Assertion](#dfn-spc-assertion) must also include information about which entity displayed the transaction confirmation user experience (browser, OS, or authenticator).
* The [SPC Assertion](#dfn-spc-assertion) must include a signature over that data (based on the associated authenticator). This signature may be validated by the Relying Party or any other party with the appropriate key.

## Security and Privacy Considerations

* [SPC Credential Identifiers](#dfn-credential-id) must be origin-bound to reduce the risk
  of cross-site tracking through the protocol. This implies that the
  RP generates new [SPC Credential Identifiers](#dfn-credential-id) across merchants. The
  browser maps [SPC Credential Identifiers](#dfn-credential-id) to stored [SPC Credentials](#dfn-spc-credential).

* It is not a requirement to obfuscate [SPC Credential Identifiers](#dfn-credential-id) used
  as input to SPC.

## FIDO Considerations

* FIDO credentials should be "enhanceable" to [SPC Credentials](#dfn-spc-credential).
* SPC credentials should also usable as ordinary FIDO credentials. See [issue 39](https://github.com/w3c/secure-payment-confirmation/issues/39).
* SPC credentials must be programmatically distinguishable from FIDO credentials.
* SPC should support both local and roaming authenticators. See [issue 31](https://github.com/w3c/secure-payment-confirmation/issues/31) on discoverable credentials and [issue 12](https://github.com/w3c/secure-payment-confirmation/issues/12) on roaming authenticator behaviors.
* [Large Blob](https://www.w3.org/TR/webauthn-2/#sctn-large-blob-extension) (WebAuthn Level 2) may be used to create portable stored data to reduce enrollment costs. Use case: I enroll my authenticator via one browser, but stored data can be used in another browser.

## Detailed Considerations

Note: Ideally this level of detail would not be part of the scope
document. These musings are likely to migrate to a specification
once that becomes available.

### SPC Credential

* An [SPC Credential](#dfn-spc-credential) is likely to include the following kind of data:
* See [issue 13](https://github.com/w3c/secure-payment-confirmation/issues/13) on cardinality between [SPC Credential](#dfn-spc-credential) and instruments). 
* Authentication-method specific data (e.g., rpid).
* See [issue 62](https://github.com/w3c/secure-payment-confirmation/issues/62) about associating a credential with a user profile. This issue discusses the idea of making that profile information available prior to instrument selection, which could support additional selection use cases.

### SPC Credential Identifiers
* See [issue 49](https://github.com/w3c/secure-payment-confirmation/issues/49) and [issue 10](https://github.com/w3c/secure-payment-confirmation/issues/10) on the nature of the SPC Credential Identifier.

### Sources of Randomness

Many authentication protocols include a source of randomness to ensure freshness. 

* See [issue 28](https://github.com/w3c/secure-payment-confirmation/issues/28) on any requirements for the nonce (e.g., random? secret?).
* See [issue 26](https://github.com/w3c/secure-payment-confirmation/issues/26) on not constraining the source of randomness

### SPC Assertion

* What is the nature of the signature? See [issue 40](https://github.com/w3c/secure-payment-confirmation/issues/40) and [issue 28](https://github.com/w3c/secure-payment-confirmation/issues/28)

### Security and Privacy Considerations

* See [issue 28](https://github.com/w3c/secure-payment-confirmation/issues/28) on security properties.
* See [issue 13](https://github.com/w3c/secure-payment-confirmation/issues/13) on cardinality between [SPC Credential](#dfn-spc-credential) and instruments). 

## Editors

* Ian Jacobs

