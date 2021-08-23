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

**SPC Credential Identifiers** <a name="dfn-credential-id"></a>
: These identifiers are generated during registration (formerly "enrollment") and returned by the API to the Relying Party. The Relying Party is the authoritative source for the meaning of each identifier (e.g., whether it refers to a user account, a specific instrument, or something else). It is the responsibility of the Relying Party to
communicate clearly to the user what the user is consenting to during
an [SPC Registration](#dfn-spc-registration). The Relying Party determines whether it is satisfied with a given [SPC Assertion](#dfn-spc-assertion) for a given use case.

**SPC Registration** <a name="dfn-spc-registration"></a>
: The process of generating an SPC Credential Identifier for a Relying party. Registration may optionally involve other activities (e.g., browsers may choose to store other information in conjunction with the SPC Credential Identifier).

**SPC Request** <a name="dfn-spc-request"></a>
: Information provided as input to the API to generate an SPC Assertion. It is likely to include [SPC Credential Identifiers](#dfn-credential-id), instrument information such as a string and icon, sources of randomness, and other data.

**SPC Assertion** <a name="dfn-spc-assertion"></a>
: The output of a successful SPC API authentication. The SPC Assertion includes a signature over the data presented to the user during the [Payment Confirmation User Experience](#payment-confirmation-user-experience).

## Design Goals
* We endeavor to describe requirements independently of specific APIs, in particular Web Authentication and Payment Request. See [issue 65](https://github.com/w3c/secure-payment-confirmation/issues/65)
* Where requirements have been suggested that are specific to an API, we list those in self-contained sections.

## Requirements

### Feature Detection

* It must be possible to determine through JavaScript feature detection whether a user agent supports SPC.

### Web Context Support

* It must be possible to call SPC from a Web site or a payment handler. Note: This implies changes to the Payment Handler API are likely.
* Because many checkout experiences are offered through iframes, it must be possible to call SPC from an iframe. SPC usage within an iframe must be enabled through a permission policy. See [issue 68](https://github.com/w3c/secure-payment-confirmation/issues/68).

### Registration

* It must be possible to do an [SPC Registration](#dfn-spc-registration) outside of a transaction.
* It must be possible to do an [SPC Registration](#dfn-spc-registration) following a prior (non-SPC) authentication during a transaction. This registration should not prevent timely completion of the transaction.
* It is not a requirement that the relying party be able to do an [SPC Registration](#dfn-spc-registration) in a third-party context. However, it could improve common payment flows if the relying party could register from a cross-origin iframe.
* It must be possible to do an [SPC Registration](#dfn-spc-registration) from a payment handler.

#### Instrument Information at Registration

* The Relying Party must not be required to provide definitive
  information about a specific instrument at registration time. In other
  words, the API supports dynamic binding to a specific instrument at
  authentication time. This feature renders unnecessary additional
  functionality to update browser-stored instrument information.
* It is not a requirement that instrument information be stored in the
  browser as part of an [SPC Registration](#dfn-spc-registration).
* The protocol should support multiple ways of accessing the instrument display information, including browser storage and authenticator storage.

#### Registration User Experience

* Each browser should natively support an [SPC Registration](#dfn-spc-registration)  user experience.
* The user agent should [require and consume at least one transient user activation](https://html.spec.whatwg.org/multipage/interaction.html#activation-consuming-api) in order to display an SPC registration user experience.
* The party that invokes SPC registration must be able to specify a timeout for the registration user experience.


### Payment Confirmation

#### Origin Policies

* Any origin (including the Relying Party) must be able to invoke SPC payment confirmation with the credential id(s) of the Relying Party. 

#### Instrument Information at Payment Confirmation

* Instrument information (e.g., a display string and icon) must be available for
  use within the payment confirmation user experience.
* The party that calls the API must be able to provide instrument
  information as input. Note: The
  Relying Party is the authoritative source of instrument information.
  If another party provides conflicting instrument information as
  input to the API, the Relying Party can detect this during
  subsequent validation.

#### Payment Confirmation User Experience

* Each browser must natively support a payment confirmation user experience.
* The user agent must [require and consume at least one transient user activation](https://html.spec.whatwg.org/multipage/interaction.html#activation-consuming-api) in order to display an SPC user experience.
* Although we anticipate that in most cases the browser will render the payment confirmation user experience, the protocol should support rendering by other entities (e.g., the operating system or authenticator or for out-of-band authentication). Note: This feature would address use cases that require particularly secure display of information.
* The payment confirmation user experience must display instrument information such as label and icon.
* The payment confirmation user experience must display amount and currency of the payment.
* The party that invokes SPC must be able to specify a timeout for the payment confirmation user experience. See [issue 67](https://github.com/w3c/secure-payment-confirmation/issues/67).
* The payment confirmation user experience must include the origin of the top level where the API is called.
* The payment confirmation user experience should include the title and favicon of the page where it was called. See [issue 48](https://github.com/w3c/secure-payment-confirmation/issues/48) on merchant information display.

#### Levels of User Interaction during Payment Confirmation

* The API must support payment confirmation with two factor authentication (e.g., with FIDO user verification check).
* The API should support payment confirmation with one factor authentication with user presence check.
* The API should support payment confirmation with one factor authentication (possession) without user presence check.
* The API might at some point support payment confirmation with one factor authentication (possession) but without a visible dialog and without a user presence check.
* The API should allow the relying party to express a preference for any of the supported levels of user interaction.
* For each transaction, a merchant should be able to express to the relying
party a preference to use (or not use) any of the supported levels of user interaction.
* If the browser supports any relying party option less secure than two-factor, the browser must support a user preference to override that option and maintain two-factor authentication.

#### Cross-Browser Support

* If the user did an [SPC Registration](#dfn-spc-registration) when using one instance of a browser, it should be possible to leverage that authentication from a different instance of the same browser (e.g., both browsers are Firefox)
*If the user did an [SPC Registration](#dfn-spc-registration) when using one instance of a browser, it should be possible to leverage that authentication from any browser (e.g., one browser is Firefox and the other is Chrome). Note:  [Large Blob](https://www.w3.org/TR/webauthn-2/#sctn-large-blob-extension) (WebAuthn Level 2) may be used to create portable stored data to reduce the total number of registrations.

### SPC Credential Identifiers and Matching

* It it left to the user agent how the user selects an authentication path when more than one matches the [SPC Request](#dfn-spc-request). See [issue 69](https://github.com/w3c/secure-payment-confirmation/issues/69) for discussion.
* When the user agent has no information matching the [SPC Request](#dfn-spc-request), for privacy reasons the browser may inform the user that the API has been invoked but failed. Note: Whoever might call the API can determine in advance that the user has never done an [SPC Registration](#dfn-spc-registration), and choose not to call the API.

### SPC Assertions

* The [SPC Assertion](#dfn-spc-assertion) must include at least: top level origin, caller origin, relying party origin (rpid), one-time challenge, instrument information, transaction amount and currency.
* If different user journeys are possible for a given authenticator (e.g., a low friction option), the [SPC Assertion](#dfn-spc-assertion) must also include information about the user's journey. This information may be part of the authenticator's own assertion. For example, the assertion must indicate whether the user completed the transaction without a user presence check.
* The [SPC Assertion](#dfn-spc-assertion) must include a signature over that data (based on the associated authenticator). This signature may be validated by the Relying Party or any other party with the appropriate key.

### Lifecycle Management

* If the user agent stores [SPC Credential Identifiers](#dfn-credential-id) (and any associated information), the user must be able to remove individual [SPC Credential Identifiers](#dfn-credential-id) from the user agent.
* The API should not make it impossible for the user to communicate to the relying party to forget [SPC Credential Identifiers](#dfn-credential-id). This might happen in a variety of ways (e.g., forget this authenticator and all associated instruments; forget any authenticators associated with this instrument, etc.). See [issue 63](https://github.com/w3c/secure-payment-confirmation/issues/63).

### Security and Privacy Considerations

* [SPC Credential Identifiers](#dfn-credential-id) must be origin-bound. All [SPC Credential Identifiers](#dfn-credential-id) on the same origin are expected be distinct.
* The API should allow relying parties to reduce the risk of cross-site tracking that might arise through the reuse of [SPC Credential Identifiers](#dfn-credential-id). See [issue 77](https://github.com/w3c/secure-payment-confirmation/issues/77).

### FIDO Considerations

* FIDO credentials should be usable for SPC activities and vice-versa, but the user agent must clearly communicate to the user how credentials are being used. See [issue 79](https://github.com/w3c/secure-payment-confirmation/issues/79) and  [issue 39](https://github.com/w3c/secure-payment-confirmation/issues/39).
* SPC should support both platform and roaming authenticators. See [issue 31](https://github.com/w3c/secure-payment-confirmation/issues/31) on discoverable credentials and [issue 12](https://github.com/w3c/secure-payment-confirmation/issues/12) on roaming authenticator behaviors.

## Editor

* Ian Jacobs

