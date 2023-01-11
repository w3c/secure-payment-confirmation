# Self-Review Questionnaire: Security and Privacy

## 1.  What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary? 

This feature allows a web site that is not the original Relying Party of a
`PublicKeyCredential` to exercise the credential in order to sign over
transaction data. This is to allow a merchant or a Payment Service Provider
(PSP) that represents the merchant to request strong authentication of the
payment instrument used in the transaction without redirecting to the issuing
bank of the payment instrument. Minimizing redirection or any form of friction
is critical to avoid cart abandonment during a checkout process. In many
payment systems, the Account Provider wishes (or is required by regulation) to
authenticate the user for fraud protection; SPC allows them to fulfill the
requirement to validate the authentication while enabling the merchant to
manage the user experience.

Thus, following a browser prompt and user consent, the information
that is exposed to the Web site is the Web Authentication assertion
that includes signed transaction data.

## 2. Do features in your specification expose the minimum amount of information necessary to enable their intended uses? 

Yes.

## 3. How do the features in your specification deal with personal information, personally-identifiable information (PII), or information derived from them?

Any information displayed through this feature is provided by the
caller (e.g., the merchant). The Web Authentication credentialID in
the assertion that is generated after a browser prompt and user
consent is considered public information (not PII).

## 4. How does this specification deal with sensitive information?

Although the credentialID that results from Web Authentication is not considered
PII, it may be considered sensitive. credentialIDs are provided as input
to the API. The API reveals that the user has an associated authenticator only
after a browser prompt and user consent.

## 5. Do the features in your specification introduce new state for an origin that persists across browsing sessions? 

Secure Payment Confirmation relies on Web Authentication, which introduces state. Secure Payment Confirmation does not introduce additional state.

Secure Payment Confirmation includes an "opt-out" feature: an
(optional) input parameter that instructs the user agent to provide
the user with an opportunity to request that the relying party forget
previously stored authentication credentials.

## 6. Do the features in your specification expose information about the underlying platform to origins?

Secure Payment Confirmation relies on Web Authentication, which
exposes some underlying information.  Secure Payment Confirmation,
like Web Authentication, takes care not to expose e.g. credential
existence (or lack thereof) to any caller.

## 7. Does this specification allow an origin to send data to the underlying platform? 

Secure Payment Confirmation relies on Web Authentication, which supports interactions with the
underlying platform (that is, authenticators). Secure Payment Confirmation sends transaction
data to authenticators to be included in the resulting assertion using a standard Web
Authentication mechanism (clientData).

## 8. Do features in this specification enable access to device sensors? 

Secure Payment Confirmation relies on Web Authentication, which supports interactions with the
underlying platform (that is, authenticators).

## 9. Do features in this specification enable new script execution/loading mechanisms? 

No.

## 10. Do features in this specification allow an origin to access other devices? 

No. Secure Payment Confirmation relies on Web Authentication; discussions about passkeys in Web Authentication are ongoing.

## 11. Do features in this specification allow an origin some measure of control over a user agent’s native UI? 

Yes. This feature allows an origin (usually a bank) to provide a set of
information that will be shown to the user in a native UI:

* Payment instrument icon and string
* Payment amount and currency
* Intended payee

This information is displayed to the user in the user agent's native UI when
this credential is exercised. If the merchant provides incorrect or fraudulent
data to the API for confirmation by the user, the Relying Party can detect this
when validating the assertion and reject the transaction.

## 12. What temporary identifiers do the features in this specification create or expose to the web? 

None.

## 13. How does this specification distinguish between behavior in first-party and third-party contexts? 

This feature mostly inherits the behavior from [Web Authentication], with the
following exceptions:

- Web Authentication Level 2 does not allow credential creation in a cross-origin iframe; Secure Payment Confirmation does (with delegated permission and user activation). Discussions about whether Web Authentication Level 3 should allow credential creation in a cross-origin iframe are ongoing.
- A SPC-enabled `PublicKeyCredential` can be exercised in any context where
  Payment Request API is allowed, i.e. a first-party secure context, or a
  third-party secure context with [delegated permission] from the top-level
  context. Note: As of January 2023 the specifiation includes a requirement that the user agent consumer a user activation at authentication time, but the Working Group plans to remove that requirement; see [issue 216](https://github.com/w3c/secure-payment-confirmation/issues/216) including the Chrome Team's [security and privacy consideration notes](https://github.com/w3c/secure-payment-confirmation/issues/216#issue-1455821580).

[delegated permission]: https://w3c.github.io/payment-request/#permissions-policy

## 14. How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?

Secure Payment Confirmation relies on [Web Authentication], which
behaves identically regardless of Private Browsing or "incognito"
mode.

## 15. Does this specification have both "Security Considerations" and "Privacy Considerations" sections? 

Yes.

## 16. Do features in your specification enable origins to downgrade default security protections?

Secure Payment Confirmation differs (currently) from Web Authentication in two ways related to default security protections:

* Web Authentication Level 2 does not allow credential creation in a cross-origin iframe; Secure Payment Confirmation does (with delegated permission and user activation).
* Web Authentication Level 2 credentials may only be used at authentication time by the Relying Party that created them. If a Relying Party approves (at credential creation time), Secure Payment Confirmation allows other parties to use credentials, with an associated user agent prompt and user consent.

## 17. How does your feature handle non-"fully active" documents?

Secure Payment Confirmation inherits behavior from Payment Request API related
to non-"fully active" documents (see sections on show(), retry(), and complete()).

## 18. What should this questionnaire have asked?

None.

[Payment Request API]: https://w3c.github.io/payment-request
[PublicKeyCredential]: https://www.w3.org/TR/webauthn/#iface-pkcredential
[AuthenticatorAssertionResponse]: https://www.w3.org/TR/webauthn/#authenticatorassertionresponse
[Web Authentication]: https://www.w3.org/TR/webauthn
