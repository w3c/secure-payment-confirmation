# Explainer: Connected roaming authenticators

## Author:

Stephen McGruer \<smcgruer@chromium.org\>

(With thanks to Nina Satragno)

## Status:

_Document state: **Proposal**_

_Last updated: 28-Jul-2026_

## Summary

This explainer proposes loosening the restrictions on SPC-related credentials
to allow non-platform authenticators with non-internal transports. Doing so
would allow user agents to support connected roaming authenticators (e.g., a
plugged-in security key) with SPC's current flow. It does not address the
problem of supporting non-connected roaming authenticators.

This proposal should have no impact on integrators of SPC who do not want to
allow roaming authenticators, other than its reliance on [issue
336](https://github.com/w3c/secure-payment-confirmation/issues/336) which is a
breaking change. Relying Parties who do not want to allow roaming
authenticators simply continue to specify "platform" attachment at credential
creation time and to pass only "internal" transports at authentication time.

## Background

The SPC specification does not currently allow use of [roaming
authenticators](https://w3c.github.io/webauthn/#roaming-authenticators) (such
as USB security keys or a separate authenticator device accessed via NFC,
bluetooth, etc):

- When creating a new credential, the `payment` extension [specifies
  that](https://w3c.github.io/secure-payment-confirmation/#client-extension-processing-registration)
  the `navigator.credentials.create` call should be rejected if the
  [authenticatorAttachment](https://w3c.github.io/webauthn/#dom-authenticatorselectioncriteria-authenticatorattachment)
  is not ["platform"](https://w3c.github.io/webauthn/#dom-authenticatorattachment-platform).

- When responding to a payment request, the "secure-payment-confirmation"
  payment method hard-codes (in step 6 of [this
  algorithm](https://w3c.github.io/secure-payment-confirmation/#sctn-steps-to-respond-to-a-payment-request))
  that the [transports](https://w3c.github.io/webauthn/#dom-publickeycredentialdescriptor-transports)
  for the credentials is a list with a single item whose value is
  ["internal"](https://w3c.github.io/webauthn/#dom-authenticatortransport-internal).

These spec steps enforce that (1) credentials cannot be created on a roaming
authenticator device when the `payment` extension is set, and (2) even if such
a credential existed, they cannot be used for SPC authentication as the
WebAuthn layer would only look for "internal" transports (i.e. those offered by
platform authenticators).

The restriction to only allow platform authenticators was originally put in
place for two reasons that I am aware of:

1. Initial Relying Parties for SPC were only interested in platform
   authenticators, and not supporting roaming authenticators simplified the
   implementation.

2. SPC relies on the ability to [silently determine if a credential is
   available for the current
   device](https://w3c.github.io/secure-payment-confirmation/#steps-to-silently-determine-if-a-credential-is-available-for-the-current-device),
   and we didn't think that was possible for roaming authenticators.

   - In particular, we were worried about a security key not being plugged in
     at the time of use, and thus not being queryable.

As of July 2026, we have seen increased interest in supporting roaming
authenticators.  Furthermore, the assumption that we could not silently query a
**connected** roaming authenticator was incorrect. This explainer proposes a
path forward to support **connected** roaming authenticators for SPC.

## Proposal

Enable a Relying Party to (1) create an SPC-compatible passkey stored on a
roaming authenticator, and (2) use passkeys stored on roaming authenticators
(both with and without the `payment` bit set) for SPC authentication.

For FIDO-compatible keys, the browser is able to **silently** query a connected
roaming authenticator (e.g., a plugged-in USB security key) for credential
existence and third-party payment support, as long as:

- The browser provides an explicit list of credential IDs, and
- The keys are created with a
  [credProtect](https://fidoalliance.org/specs/fido-v2.3-ps-20260226/fido-client-to-authenticator-protocol-v2.3-ps-20260226.html#sctn-credProtect-extension)
  value that is **not** `userVerificationRequired`.

Since SPC requires an input list of credential IDs, the browser can use those
and make a User Presence disabled (UP=0) request to any connected roaming
authenticator to determine if (1) a given credential ID exists, and (2) if that
credential ID has the third-party payment bit set on it. Those two bits of
information map exactly to the information needed by SPC to filter the input
list of credential IDs.

**Note**: This proposal does NOT allow the use of a roaming authenticator that
is not currently plugged in for SPC authentication. This is discussed further below.

### Impact on existing SPC integrators

This proposal should have no immediate impact on existing integrators of SPC.
While it does depend on [issue
336](https://github.com/w3c/secure-payment-confirmation/issues/336), which is a
backwards incompatible change, it will be up to user agents to handle
compatibility for that (e.g., by continuing to also support `credentialIds` for
a deprecation period). Aside from that issue, everything here is opt-in for a
Relying Party. In order to have a roaming authenticator be used for SPC, the
Relying Party must specify a non-platform authenticator attachment when
creating the credential, and must choose a non-internal transport when
triggering authentication.

### Necessary spec changes

#### `payment` client extension processing steps

Under
[registration](https://w3c.github.io/secure-payment-confirmation/#client-extension-processing-registration),
remove the requirement for "authenticatorAttachment" to be "platform".

#### Specifying the transport

In order to allow roaming authenticators, we need to allow non-"internal"
transport modes. Unfortunately SPC's API was designed with the assumption that
its credential IDs would always be "internal", and so currently takes in a
`sequence<BufferSource>` of the credentialIDs, rather than the correct
`sequence<PublicKeyCredentialDescriptor>`, which would then allow for
specifying the transport.

Fixing this is tracked in [issue
#336](https://github.com/w3c/secure-payment-confirmation/issues/336).

#### Allowing non-internal transports

Along with the above change to allow specifying the transport, we need to
remove the restriction that the transport must be "internal". Instead, any
transport should be allowed, and it is up to the user agent as to whether or
not it is able to successfully silently query a credential for the given
transport and connected authenticators.

#### Silently querying roaming authenticators

Currently the SPC steps to silently query for [credential
existence](https://w3c.github.io/secure-payment-confirmation/#steps-to-silently-determine-if-a-credential-is-available-for-the-current-device)
and for [third-party enabled
state](https://w3c.github.io/secure-payment-confirmation/#steps-to-silently-determine-if-an-spc-credential-is-third-party-enabled)
are actually undefined. As such, we do not technically need to change this to
allow roaming authenticators, and instead the requirement is just on the user
agent to implement support.

That said, we could do better for roaming authenticators. It should be possible
to specify explicit steps to silently query for credentials and for the
`thirdPartyPayment` bit on those credentials in the case of FIDO compliant
authenticators. Doing so would be a beneficial step for the SPC specification,
although we would still have to leave the undefined option as well for the user
agent to use when it talks to platform authenticators (whose APIs are
proprietary).

## Supporting non-connected roaming authenticators

As noted above, this proposal does not enable support for non-connected roaming
authenticators. This may cause it to be of limited use, as there is no way
(outside of website UX) for a user to know that they have to plug in their
roaming authenticator *before* an SPC call is triggered - and indeed some
roaming authenticators cannot be "plugged in" at all.

I am also working on an explainer for supporting non-connected roaming
authenticators in SPC, which will be linked from here once available.
