# Secure Payment Confirmation Capabilities explained

## tl;dr

The **Secure Payment Confirmation (SPC) Capabilities API** is a proposed extension to [Secure Payment Confirmation](https://w3c.github.io/secure-payment-confirmation/) that allows detecting support for specific features before invoking a "secure-payment-confirmation" payment request. Specifically, this proposal introduces feature detection for Browser Bound Keys (BBK).

It provides an API surface modeled after WebAuthn's `getClientCapabilities()`, allowing Relying Parties to determine whether they can proceed with an SPC flow without forcing the user through unnecessary authentication dialogs when required features are unavailable.

## Motivation

The existing SPC [Browser Bound Keys](https://github.com/w3c/secure-payment-confirmation/issues/271) (BBKs) feature provides strong security guarantees by signing the SPC payment response with a device-bound key-pair, returning both the signed data and the public key. This ensures that any subsequent responses signed with that exact public key originate from the exact same device. While BBKs are utilized for "secure-payment-confirmation" payment requests whenever possible, they are not universally available (e.g., if a device lacks a hardware security chip).

When a transaction requires BBK support but the user's device doesn't support it, the Relying Party discovers this only *after* invoking SPC and examining the response. The user is forced to navigate the SPC dialog and an authentication interaction that is ultimately discarded because the Relying Party will reject an SPC response that lacks a BBK signature. This creates a wasteful user experience, particularly if the user has multiple devices with a mix of BBK availabilities. See [issue #315](https://github.com/w3c/secure-payment-confirmation/issues/315) for additional background.

### Goals

* Provide a mechanism for API callers to detect if the BBK feature is available before making the "secure-payment-confirmation" payment request.
* Save users from wasted authentication dialog interactions when required hardware capabilities are absent.
* Provide an extensible API design to easily incorporate future SPC capabilities without significant specification churn.
* Minimize any privacy impact relating to being able to fingerprint user devices.

## Proposed Solution

The proposed solution adds a static API to enable callers to detect whether specific capabilities, such as the BBK hardware feature, are available before initiating a "secure-payment-confirmation" payment request.

### Proposed APIs

The caller queries a new static `getSecurePaymentConfirmationCapabilities()` method on the `PaymentRequest` object to receive a record that maps capability strings to boolean availability values.

```javascript
partial interface PaymentRequest {
  static Promise<SecurePaymentConfirmationCapabilities> getSecurePaymentConfirmationCapabilities();
};

typedef record<DOMString, boolean> SecurePaymentConfirmationCapabilities;
```

The boolean value conveys whether the SPC capability is available. A capability key might be missing from the map entirely, in which case the API user should interpret the availability of that specific capability as "unknown".

The primary capability introduced is `browserBoundKeyHardware`. A value of `true` means that hardware necessary for storing non-extractable keys is available.

```javascript
// Check if getSecurePaymentConfirmationCapabilities function is defined.
if (PaymentRequest.getSecurePaymentConfirmationCapabilities) {
  const capabilities = await PaymentRequest.getSecurePaymentConfirmationCapabilities();
  
  // Checking for a specific capability (will be falsy if the key doesn't exist).
  if (capabilities.browserBoundKeyHardware) {
    // Proceed with SPC request requiring BBK
  }

  // Iterating over all available capabilities.
  for (const [key, value] of Object.entries(capabilities)) {
    // Handle capabilities
  }
}
```

This design is modeled after the [WebAuthn `getClientCapabilities()` method](https://www.w3.org/TR/webauthn-3/#sctn-getClientCapabilities), but the scope is kept within SPC to neatly support future SPC-specific features without adding bloat to a generalized `PaymentRequest` capability enum. 

A `DOMString` is utilized as the key rather than a formal enum to allow string values outside the standard definitions to be tested and prototyped without needing immediate specification updates. A boolean is used over an enum value for simplicity; if more granular responses are required in the future, distinct capability strings can simply be added.

## Alternatives Considered

### Dedicated `browserBoundKeyAvailability()` method

An alternative approach is to create a highly specific `browserBoundKeyAvailability()` method that returns an enum with detailed state information about BBK availability.

```javascript
partial interface PaymentRequest {
  static Promise<BrowserBoundKeyAvailability> browserBoundKeyAvailability();
};

enum BrowserBoundKeyAvailability {
  "available",
  "unavailable-unknown-reason",
  "unavailable-no-hardware-security-chip"
};
```

This alternative returns an enum value instead of a boolean, allowing API users to receive detailed context, while allowing the response to be nuanced in the future (e.g., dynamically returning statuses for software-backed BBKs). While easier for developers to immediately understand, the major drawback is extensibility. Wanting to add brand new features to SPC in the future would compel the creation of new bespoke availability methods for every feature.

### Extending `securePaymentConfirmationAvailability()`

Another alternative is to inject a new enum value into the existing `securePaymentConfirmationAvailability()` method to signify that SPC is available with BBKs:

```javascript
enum SecurePaymentConfirmationAvailability {
  "available-with-bbk".  // New enum for BBK availability.
  "available",
  "unavailable-unknown-reason",
  "unavailable-feature-not-enabled",
  "unavailable-no-permission-policy",
  "unavailable-no-user-verifying-platform-authenticator",
};
```

Although this represents the simplest change to surface the BBK feature, it is broadly undesirable. It is unsustainable if we add multiple, independent capabilities to SPC in the future, as all combinatoric permutations of those capabilities would need defining. Additionally, it creates a backwards compatibility hazard, as API users would be forced to modify their code to accept both `available-with-bbk` and `available`, even if they do not care about BBK support.

## Privacy Considerations

The primary concern with this proposal is that it presents a potential fingerprinting risk, revealing whether a given device is equipped with a hardware security chip. 

However, similar low-level feature detections are already readily available. In discussions regarding DBSC (Device Bound Session Credentials), TPM detection is not generally considered an unmanageable fingerprinting vector given that timing attacks (slow TPM vs. fast TPM vs. no TPM) already expose information about hardware.

To mitigate this risk, clients can elect to limit the capabilities returned to the API user. Specifically, clients should prefer to omit capabilities entirely rather than definitively returning a boolean `false`. Omitting the capability value altogether still provides a negative signal to an API user relying on the capability, but limits explicit confirmation about what hardware is definitively absent, making robust fingerprinting harder to achieve.
