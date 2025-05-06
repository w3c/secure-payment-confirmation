# Secure Payment Confirmation: Browser Based Key Requirements and Design Considerations

Status: This is a **draft** document without consensus.

Though this document we seek to build consensus around requirements and design considerations for adding a browser-based key pair (BBK) to the results of Secure Payment Confirmation to serve as a possession factor during authentication. For discussion, see [issue 271](https://github.com/w3c/secure-payment-confirmation/issues/271).

## Motivation

Secure Payment Confirmation provides a convenient "sign what you see" experience for a user to agree to the terms and conditions of a transaction, and where Web Authentication is used to generate cryptographic evidence of the user's agreement.

The payments industry has indicated that SPC would further benefit from a device binding capability. As WebAuthn passkeys can now be synced, it can be argued that they no longer meet strict 2FA requirements (being no longer a signal of device possession), and so SPC (like WebAuthn) is reduced to a single factor (biometric or knowledge, depending on the authentication method used).

The Web Authentication Working Group has considered but not adopted device binding proposals (cf. [SPK](https://github.com/w3c/webauthn/pull/1957) and [DPK](https://github.com/w3c/webauthn/issues/1658)).

Therefore, the Web Payments Working Group plans to add a "browser-based key (BBK)" to SPC. This document endeavors to capture requirements for the BBK functionality.

## Assumptions

* A relying party will perform some ID &amp; V process before trusting a (new) BBK. That ID &amp; V process might take place before a Web Authentication registration (and thus, if the BBK is returned as part of the Web Authentication registration, the RP would not likely step up the user a second time). In the case of a synched passkey, when the RP first sees a BBK on a new user device, in the absence of other trust signals, the RP would likely perform some ID &amp; V process in order to trust the new BBK, and we consider that an acceptable user experience on a new device.

## Requirements

***Notes:***

* These requirements are not prioritized.
* "MUST", "SHOULD", and "MAY" are used per [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).


### Association with passkeys

* A given passkey should have only one associated BBK at any given time for a given user agent user profile.

* A BBK must only ever be associated with one passkey. By implication, if a passkey is deleted by the user, any associated BBK should also be deleted.

* If a BBK is deleted, the user agent should generate a new BBK associated with the same passkey.

* At authentication time, the client data will be signed by the passkey and the associated BBK.

* To link the BBK to the passkey cryptographically, the BBK public key should be added in the client data.

### User agent user profiles

* A user agent may reuse the same BBK across user profiles of the same user agent instance.

### Device binding

* To meet anticipated security requirements, issuance of the BBK by the user agent should involve a device-binding process that ensures a unique connection between the user agent (user profile) and the device. This may be, for example, through hardware crypto-security (e.g., TPM), keys stored in the secure element, or registration of the web browser linking a browser to a device.

* Once a BBK has been bound to a device, it must not be usable outside that device.

* The user agent may return a BBK even in environments where a device-binding process is not readily available. Not every transaction requires the same level of security (e.g., low-value transactions), and so even a BBK that is not device-bound can be useful. 

* Each BBK should be associated with a signal indicating the nature of the device-binding process (e.g., corresponding to "secure element", "software", "no device binding").

### Attestation

_This section is in development._

### Security and privacy considerations

* In private browsing mode, if a user can access passkeys when using
  SPC, the user agent should also return the BBK.

* The BBK must only be available through the SPC API and otherwise isolated from the Web page environment.

* This proposal relies on underlying [FIDO security assumptions](https://fidoalliance.org/specs/common-specs/fido-security-ref-v2.1-ps-20220523.html#fido-security-assumptions) and related threat models.

## Questions

* If TPM is the focus of security, do we need any requirements related to BBK storage (e.g., must be encrypted storage)?

## Editor

* Ian Jacobs, borrowing in part from the original text [issue
  271](https://github.com/w3c/secure-payment-confirmation/issues/271)
  and also aggregating subsequent feedback.
