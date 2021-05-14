Q: Meta-question; are the uses of 'must', 'should' etc here to be interpreted per [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) ?

Q: Meta-question; these requirements encompass a fairly large scope. I believe that we can spec and ship SPC in 'chunks' towards this goal, where each stage might add a few more capabilities meeting these requirements, but not have every single requirement in v1. Do others agree?

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
: Data that represents the association between an instrument and an authentication credential. Note: Management of multiple relationships is an implementation detail (e.g., multiple authentications corresponding to a single instrument, or multiple instruments enrolled for a given authentication).

**SPC Credential Identifiers** <a name="dfn-credential-id"></a>
: Each SPC Credential Identifier refers to one SPC Credential. These identifiers are generated during enrollment and stored by the Relying Party in association with an instrument. An instrument may be addressable by more than one SPC Credential Identifier (e.g., when the user has authenticated through different devices for that instrument).

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

* It must be possible to determine programmatically whether a browser supports SPC.

### Web Context Support

* It must be possible to call SPC from a Web site or a payment handler. Note: This implies changes to the Payment Handler API are likely.
* Because many checkout experiences are offered through iframes, it must be possible to call SPC from an iframe. See [issue 68](https://github.com/w3c/secure-payment-confirmation/issues/68).

### Enrollment

* It must be possible to enroll an [SPC Credential](#dfn-spc-credential) outside of a transaction.
* It must be possible to enroll an [SPC Credential](#dfn-spc-credential) following a prior (non-SPC) authentication during a transaction. This enrollment should not prevent timely completion of the transaction.
* It must be possible to enroll multiple instruments for a single authentication. Each resulting [SPC Credential](#dfn-spc-credential) (that is: each instrument/authentication credential binding) must be independently addressable.
* It is not a requirement that the relying party be able to enroll an [SPC Credential](#dfn-spc-credential) in a third-party context.
* It must be possible to enroll an [SPC Credential](#dfn-spc-credential) from a payment handler.
* Each browser should natively support an SPC credential enrollment user experience.

#### Instrument Information

* Enrollment of an instrument must include display information for it.

Q: Should this be a 'must' if we're allowing for per-RP credentials (i.e. as opposed to requiring per-payment-instrument)? I could see it as an optional include in that case.

* Because instrument display information is available to the relying party (e.g., provided by the relying party itself, the merchant, or some other party), it is not a requirement that this information be stored in the browser as part of the [SPC Credential](#dfn-spc-credential).
* The relying party should be able to update the instrument information of an enrolled [SPC Credential](#dfn-spc-credential) (e.g., for new card art).

### Transaction Confirmation User Experience

* Each browser must natively support a transaction confirmation user experience.
* Although we anticipate that in most cases the browser will render the transaction confirmation user experience, the protocol must support rendering by other entities (e.g., the operating system or authenticator).
* The transaction confirmation user experience must display instrument information such as label and icon.
* The transaction confirmation user experience must display amount and currency of the payment.
* For regulatory reasons, the party that invokes SPC must be able to specify a timeout for the user experience. See [issue 67](https://github.com/w3c/secure-payment-confirmation/issues/67).
* The transaction confirmation user experience should include the beneficiary name, and optionally the title and favicon of the page where it was called. See [issue 48](https://github.com/w3c/secure-payment-confirmation/issues/48) on merchant information display.

#### Sources of Instrument Information

* The protocol should support multiple ways of accessing the instrument display information, including browser storage, authenticator storage, and merchant-provided data. Note that merchant data may not be trustworthy, but during subsequent
authorization it can be validated against relying-party stored instrument display data.

#### Cross-Browser Support

* If the user enrolled an [SPC Credential](#dfn-spc-credential) when using one instance of a browser, it should be possible to leverage that authentication from a different instance of the same browser (e.g., both browsers are Firefox)
* If the user enrolled an [SPC Credential](#dfn-spc-credential) when using one instance of a browser, it should be possible to leverage that authentication from any browser (e.g., one browser is Firefox and the other is Chrome).

Note: These two will conflict fairly directly with 'Sources of Instrument Information' above - if information is stored in the browser it will be by default not cross-browser (not even cross-instance of browser possibly). That might be fine, we might just say "ok you need to do merchant-provided data then".

#### Levels of User Interaction during Transaction Confirmation

* The API must support transaction confirmation with two factor authentication (e.g., with FIDO user verification check).
* The API must support transaction confirmation with one factor authentication with user presence check.

Q: Is the goal here to still produce a cryptogram from the authenticator? Or is it just being able to say 'well, I called the SPC API and the user clicked yes', with no actual 'proof' of that for anyone?

Overall, I'm not sure how we'll view this from the browser side, mostly due to lack of knowledge/experience on my part. I can only go from the fact that WebAuthn doesn't support user-not-present I believe, and there is assumedly a reason they did that (albeit that may change in the future!).

* The API should support transaction confirmation with one factor authentication (possession) without user presence check.
* TBD: The API should support transaction confirmation with one factor authentication (possession) but without a visible dialog and without a user presence check.
* The API must allow the relying party to express a preference for any of the supported levels of user interaction.
* For each transaction, a merchant should be able to express to the relying
party a preference to use (or not use) any of the supported levels of user interaction.
* If the browser supports a low-friction flow option, the browser must support a user preference to override that option and maintain authentication with more user interaction.

### SPC Credentials

* See [issue 69](https://github.com/w3c/secure-payment-confirmation/issues/69) for discussion of requirements when more than one [SPC Credential](#dfn-spc-credential) matches input data.
* When no [SPC Credential](#dfn-spc-credential) matches input data, the protocol should terminate without any user experience to allow for seamless fallback behaviors.

Note: Whilst I'm not saying this will or won't be a challenge for SPC, please note this does violate WebAuthn's [Authentication Ceremony Privacy](https://www.w3.org/TR/webauthn-2/#sctn-assertion-privacy). The explainer does [recognize this today](https://github.com/w3c/secure-payment-confirmation/blob/gh-pages/README.md#probing), but I'm still exploring the ramifications on our side.

### SPC Assertions

* The [SPC Assertion](#dfn-spc-assertion) must include at least: a merchant origin/identifier, amount and currency, transaction id.
* The [SPC Assertion](#dfn-spc-assertion) must include the cryptographic nonce / challange if provided within the SPC request.
* The [SPC Assertion](#dfn-spc-assertion) must also include information about the user's journey. This information may be part of the authenticator's own assertion. For example, the assertion must indicate whether the user completed the transaction without a user presence check.
* The [SPC Assertion](#dfn-spc-assertion) must also include information about which entity displayed the transaction confirmation user experience (browser, OS, or authenticator).
* The [SPC Assertion](#dfn-spc-assertion) must include a signature over that data (based on the associated authenticator). This signature may be validated by the Relying Party or any other party with the appropriate key.

### Lifecycle Management

* The user must be able to remove individual [SPC Credentials](#dfn-spc-credential) from a browser instance.
* The ecosystem should enable the user to communicate to the relying party to forget [SPC Credential Identifiers](#dfn-credential-id). This might happen in a variety of ways (e.g., forget this authenticator and all associated instruments; forget any authenticators associated with this instrument, etc.). See [issue 63](https://github.com/w3c/secure-payment-confirmation/issues/63).

### Security and Privacy Considerations

* [SPC Credential Identifiers](#dfn-credential-id) must be origin-bound to reduce the risk
  of cross-site tracking through the protocol. This implies that the
  RP generates new SPC Credential Identifiers across merchants. The
  browser maps SPC Credential Identifiers to stored SPC Credentials.

Q: I'm very unclear on this requirement. It seems very far from everything we've discussed on SPC so far, and is introducing a redirection layer?

* It is not a requirement to obfuscate SPC Credential Identifiers used
  as input to SPC.

### FIDO Considerations

* FIDO credentials should be "enhanceable" to [SPC Credentials](#dfn-spc-credential).
* SPC credentials should also usable as ordinary FIDO credentials. See [issue 39](https://github.com/w3c/secure-payment-confirmation/issues/39).
* SPC credentials must be programmatically distinguishable from FIDO credentials.

Q: I'm not clear why this is necessary? (Not saying we shouldn't do it :D)

* SPC should support both local and roaming authenticators. See [issue 31](https://github.com/w3c/secure-payment-confirmation/issues/31) on discoverable credentials and [issue 12](https://github.com/w3c/secure-payment-confirmation/issues/12) on roaming authenticator behaviors.
* [Large Blob](https://www.w3.org/TR/webauthn-2/#sctn-large-blob-extension) (WebAuthn Level 2) may be used to create portable stored data to reduce enrollment costs. Use case: I enroll my authenticator via one browser, but stored data can be used in another browser.

## Editor

* Ian Jacobs

