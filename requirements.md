# Secure Payment Confirmation: Requirements and Design Considerations

Status: This is a draft document without consensus.

Though this document we seek to build consensus around requirements
and design considerations for SPC.

See also: [SPC Scope](scope.md) for definitions and more information.

## Feature Detection

* It must be possible to determine programmatically whether a browser supports SPC.

## Web Context Support

* It must possible to invoke SPC from an iframe. See [issue 68](https://github.com/w3c/secure-payment-confirmation/issues/68).
* It must possible to invoke SPC from within Payment Request API.
* It must possible to invoke SPC from within a Payment Handler.

## Enrollment

* It must be possible to enroll and SPC Credential outside of a transaction.
* It must be possible to enroll and SPC Credential during a transaction. This enrollment should not prevent timely completion of the transaction.
* It must be possible to enroll and SPC Credential from code on a merchant site.
* It must be possible to enroll and SPC Credential from a payment handler.
* If the user enrolled an SPC Credential when using one instance of a browser, it should be possible to leverage that authentication from a different instance of the same browser (e.g., both browsers are Firefox)
* If the user enrolled an SPC Credential when using one instance of a browser, it should be possible to leverage that authentication from any browser (e.g., one browser is Firefox and the other is Chrome).
* Each browser should natively support an SPC credential enrollment user experience.

## Transaction Confirmation User Experience

* Each browser must natively support a transaction confirmation user experience.
* Although we anticipate that in most cases the browser will render the transaction confirmation user experience, the protocol must support rendering by other entities (e.g., the operating system or authenticator).
* See [issue 48](https://github.com/w3c/secure-payment-confirmation/issues/48) on merchant information display.
* For regulatory reasons, the party that invokes SPC must be able to specify a timeout for the user experience. See [issue 67](https://github.com/w3c/secure-payment-confirmation/issues/67).

### Low Friction Flows

* The browser should support FIDO authentication without a user presence check when requested by the relying party.

* For each transactoin, a merchant should be able to express to the relying party a preference for a low-friction flow (or not to use a low-friction flow).

* If the browser supports FIDO authentication without a user presence
  check (requested by the RP), the browser must support a user
  preference to override the RP preference and to maintain the user
  presence check.

### Zero Friction Flows

* TBD

## Unenrollment

* The ecocystem should enable the user to communicate to the relying party to forget SPC Credential Identifiers. This might happen in a variety of ways (e.g., forget this authenticator and all associated instruments; forget any authenticators associated with this instrument, etc.). See [issue 63](https://github.com/w3c/secure-payment-confirmation/issues/63).

## SPC Credentials

* When more than one SPC Credential matches input data, the browser must choose the authenticator in the order of the input SPC Credential Identifiers.
* When no SPC Credential matches input data, the protocol should terminate without any user experience to allow for seamless fallback behaviors.
* If the protocol supports more than one instrument per authenticator (e.g., within the same SPC Credential), then each instrument must be uniquely addressable and have unique display information.

### Lifecycle Management

* The user must be able to remove individual SPC Credentials from a browser instance.

## SPC Assertions

* The SPC Assertion must include at least: a merchant origin/identifier, amount and currency, transaction id.
* The SPC Assertion must also include information about the user's journey. This information may be part of the authenticator's own assertion. For example, the assertion must indicate whether the user completed the transaction without a user presence check.
* The SPC Assertion must include a signature over that data (based on the associated authenticator). This signature may be validated by the Relying Party or any other party with the appropriate key.

## Security and Privacy Considerations

* For privacy, the protocol must enable the user to require that two payment instruments NOT be associated with the same authenticator.

* SPC Credential Identifiers must be origin-bound to reduce the risk
  of cross-site tracking through the protocol. This implies that the
  RP generates new SPC Credential Identifiers across merchants. The
  browser maps SPC Credential Identifiers to stored SPC Credentials.

* It is not a requirement to obfuscate SPC Credential Identifiers used
  as input to SPC.

## FIDO Considerations

* FIDO credentials should be "enhanceable" to SPC Credentials.
* SPC credentials should also usable as ordinary FIDO credentials. See [issue 39](https://github.com/w3c/secure-payment-confirmation/issues/39).
* SPC credentials must be programmatically distinguishable from FIDO credentials.
* SPC should support both local and roaming authenticators. See [issue 31](https://github.com/w3c/secure-payment-confirmation/issues/31) on discoverable credentials and [issue 12](https://github.com/w3c/secure-payment-confirmation/issues/12) on roaming authenticator behaviors.
* [Large Blob](https://www.w3.org/TR/webauthn-2/#sctn-large-blob-extension) (WebAuthn Level 2) may be used to create portable stored data to reduce enrollment costs. Use case: I enroll my authenticator via one browser, but stored data can be used in another browser.

## Detailed Considerations

Note: Ideally this level of detail would not be part of the scope
document. These musings are likely to migrate to a specification
once that becomes available.

### SPC Credential

* An SPC Credential is likely to include the following kind of data:
* One or more Payment Credential Identifiers. See [issue 13](https://github.com/w3c/secure-payment-confirmation/issues/13) on cardinality between SPC Credential and instruments). Also see [issue 27](https://github.com/w3c/secure-payment-confirmation/issues/27) about allowing user to request that each instrument have its own credential.
* Authentication-method specific data (e.g., rpid).
* See [issue 62](https://github.com/w3c/secure-payment-confirmation/issues/62) about associating a credential with a user profile. This issue discusses the idea of making that profile information available prior to instrument selection, which could support additional selection use cases.

### SPC Credential Identifiers
* See [issue 49](https://github.com/w3c/secure-payment-confirmation/issues/49) and [issue 10](https://github.com/w3c/secure-payment-confirmation/issues/10) on the nature of the Payment Credential Identifier.

### Sources of Randomness

Many authentication protocols include a source of randomness to ensure freshness. 

* See [issue 28](https://github.com/w3c/secure-payment-confirmation/issues/28) on any requirements for the nonce (e.g., random? secret?).
* See [issue 26](https://github.com/w3c/secure-payment-confirmation/issues/26) on not constraining the source of randomness

### SPC Assertion

* What is the nature of the signature? See [issue 40](https://github.com/w3c/secure-payment-confirmation/issues/40) and [issue 28](https://github.com/w3c/secure-payment-confirmation/issues/28)

### Security and Privacy Considerations

* See [issue 28](https://github.com/w3c/secure-payment-confirmation/issues/28) on security properties.
* See [issue 13](https://github.com/w3c/secure-payment-confirmation/issues/13) on cardinality between SPC Credential and instruments). Also see [issue 27](https://github.com/w3c/secure-payment-confirmation/issues/27) about allowing user to request that each instrument have its own credential.

## Editors

* Ian Jacobs

