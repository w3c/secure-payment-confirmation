# Secure Payment Confirmation: Browser Bound Key Requirements and Design Considerations

Status: This document has been discussed and revised by the Web Payments Working Group and thus informally represents Working Group consensus (even if incomplete).

Though this document we seek to build consensus around requirements and design considerations for adding a browser-bound key pair (BBK) to the results of Secure Payment Confirmation to serve as a possession factor during authentication. For discussion, see [issue 271](https://github.com/w3c/secure-payment-confirmation/issues/271).

## Motivation

Secure Payment Confirmation provides a convenient "sign what you see" experience for a user to agree to the terms and conditions of a transaction, and where Web Authentication is used to generate cryptographic evidence of the user's agreement.

The payments industry has indicated that SPC would further benefit from a device binding capability. As WebAuthn passkeys can now be synced, it can be argued that they no longer meet strict 2FA requirements (being no longer a signal of device possession), and so SPC (like WebAuthn) is reduced to a single factor (biometric or knowledge, depending on the authentication method used).

The Web Authentication Working Group has considered but not adopted device binding proposals (cf. [SPK](https://github.com/w3c/webauthn/pull/1957) and [DPK](https://github.com/w3c/webauthn/issues/1658)).

Therefore, the Web Payments Working Group plans to add a "browser-bound key (BBK)" to SPC. This document endeavors to capture requirements for the BBK functionality.

## Assumptions

* Device binding is the fundamental property of a BBK. Once a BBK has been associated with a device, it cannot be reused outside that device.
* Although in theory a BBK could be shared among multiple [installed browser programs](#installed-browser-program) on the same device, we assume here that a BBK will also be bound to an [installed browser program](#installed-browser-program) (as the name "browser bound key" indicates).
* The consensus of the Web Payments Working Group is that, initially, each BBK will be paired with a passkey. We assume here that a BBK will be bound to a single passkey. In the future the Web Payments Working Group may explore the use of BBKs without passkeys.
* A given passkey corresponds to an online identity. To distinguish online identities, the user will use different passkeys. 
* A relying party will perform some ID &amp; V process before trusting a (new) BBK. That ID &amp; V process might take place before a Web Authentication registration (and thus, if the BBK is returned as part of the Web Authentication registration, the RP would not likely step up the user a second time). In the case of a synced passkey, when the RP first sees a BBK on a new user device, in the absence of other trust signals, the RP would likely perform some ID &amp; V process in order to trust the new BBK, and we consider that an acceptable user experience on a new device.

## Definitions

* <dfn id="bbk-binding">BBK binding</dfn>: a unique triplet consisting of a specific passkey, a specific [installed browser program](#installed-browser-program), and a specific device.
* <dfn id="installed-browser-program">Installed browser program</dfn>: a specific installation of a web browser on a device. Note that a browser embedded in another application (e.g., WebView)  is considered a part of the embedding application and thus a distinct installed browser program.
## Requirements

***Notes:***

* These requirements are not prioritized.
* "MUST", "SHOULD", and "MAY" are used per [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

### Device binding

* A BBK must only ever be associated with one [BBK binding](#bbk-binding). Thus, a BBK will not be usable with any other device, [installed browser program](#installed-browser-program), or passkey. The device binding process may be, for example, through hardware crypto-security (e.g., TPM), keys stored in the secure element, or registration of the web browser linking a browser to a device.

* A [BBK binding](#bbk-binding) may have more than one BBK associated with it (e.g., if the user has multiple profiles using the same passkey on the same [installed browser program](#installed-browser-program), each profile may have its own associated BBK).

* A user agent may choose not to create a BBK if it deems there is no suitable mechanism available for establishing the device binding.

* Each BBK should be associated with a signal indicating the nature of the device-binding process (e.g., corresponding to "secure element", "software", "no device binding"). <b>Status Note:</b> As of May 2025, this feature is not yet defined in the SPC specification, nor has it been implemented in Chrome.

### Association with passkeys

* At passkey registration, if the passkey has the "payment" extension and there is no BBK associated with a [BBK binding](#bbk-binding), the user agent should create one and associate it with that [BBK binding](#bbk-binding).

* At SPC authentication time, whether or not the passkey has the "payment" extension, if there is no BBK associated with a [BBK binding](#bbk-binding), the user agent should create one and associate it with that [BBK binding](#bbk-binding).

* Once the user agent has associated a BBK with a [BBK binding](#bbk-binding), the user agent should use that BBK whenever the relevant passkey is used with SPC authentication on this device. 

* At both passkey registration and SPC authentication time, the client data will be signed by the passkey and the associated BBK.

* To link the BBK to the passkey cryptographically, when the user agent does provide a BBK signature, the BBK public key must be added in the client data.

### Deletion

* The user agent may delete BBKs for a variety of reasons (e.g., when the user uninstalls the browser, deletes passkeys, or chooses to clear all stored data).

* If a passkey is deleted, any associated BBKs should be deleted.

* The user agent is not required to provide a dedicated user experience for deleting individual BBKs.

### Attestation

_This section is in development._

### Security and privacy considerations

* In private browsing mode, if a user can access passkeys when using SPC authentication, the user agent should also return the BBK.

* In private browsing mode, the user agent should not generate new BBKs.

* The BBK must only be available through the SPC API and otherwise isolated from the Web page environment.

* This proposal relies on underlying [FIDO security assumptions](https://fidoalliance.org/specs/common-specs/fido-security-ref-v2.1-ps-20220523.html#fido-security-assumptions) and related threat models.

## FAQ
* Can a BBK be reused across different accounts on the same installed browser program?
  * This is an implementation choice left to the installed browser program.
* Can a BBK be reused across an installed browser program and a web view version of that browser?
  * No; each webview is considered a distinct installed browser program bounded by the embedding application.
* Can a BBK be reused across an installed browser program and other installed browser programs using the same underlying engine (e.g., Chromium)?
  * No; it cannot be shared among distinct installed browser programs (even if they share the same engine).
* Can a BBK be reused across different browsers on the same device if they use the same underlying passkey manager?
  * No; it cannot be shared among distinct installed browser programs.
* If different apps use the same installed browser program to compose content (e.g., via Android Custom Tabs or Trusted Web Activity), can the same BBK be reused across those apps?
  * Yes. In this case, the installed browser program is not embedded but invoked independently.

## Questions

* If TPM is the focus of security, do we need any requirements related to BBK storage (e.g., must be encrypted storage)?

## Editor

* Ian Jacobs, borrowing in part from the original text [issue
  271](https://github.com/w3c/secure-payment-confirmation/issues/271)
  and also aggregating subsequent feedback.
