
# Browser bound key (BBK) for Secure Payment Confirmation (SPC): Security and Privacy Questionnaire

This questionnaire covers the BBK addition to SPC. See also the [Secure Payment
Confirmation Questionnaire](security-privacy-questionnaire.md).

2.1. What information does this feature expose, and for what purposes?

* BBK public key: The public key of the BBK key pair that is randomly generated
  and bound to a specific passkey, a specific browser instance and a specific device.
* BBK signature: The signature using the BBK private key (not exposed).

When a user agent has no suitable method for generating and storing the private
key, no BBK is exposed.

Browser bound keys enhance SPC for regulatory environments that require a
possession factor as part of multifactor authentication. The consensus of the
payments ecosystem is that synched passkeys do not meet requirements for
possession factors, and so after discussion between the Web Authentication
Working Group and the Web Payments Working Group, we are adding this feature to
SPC instead of adding a general-purpose possession factor (via extension) to Web
Authentication.

2.2. Do features in your specification expose the minimum amount of information necessary to implement the intended functionality?

Yes. Only the public key and signature are exposed.

2.3. Do the features in your specification expose personal information, personally-identifiable information (PII), or information derived from either?

No. The BBK public-private key pair is generated randomly.

2.4. How do the features in your specification deal with sensitive information?

N/A

2.5. Does data exposed by your specification carry related but distinct information that may not be obvious to users?

N/A

2.6. Do the features in your specification introduce state that persists across browsing sessions?

Yes. The BBK public-private key pair persists across sessions.

2.7. Do the features in your specification expose information about the underlying platform to origins?

Because BBKs may be implemented in a variety of ways, in general it may be
difficult to deduce information about the underlying platform from the presence
of a BBK public key and signature. However, in practice, if user agents only
implement BBKs via secure elements, the availability of a secure element may be
inferred by the presence of the BBK public key and signature.

2.8. Does this specification allow an origin to send data to the underlying platform?

No.

2.9. Do features in this specification enable access to device sensors?

No.

2.10. Do features in this specification enable new script execution/loading mechanisms?

No.

2.11. Do features in this specification allow an origin to access other devices?

No.

2.12. Do features in this specification allow an origin some measure of control over a user agent’s native UI?

No.

2.13. What temporary identifiers do the features in this specification create or expose to the web?

No temporary identifiers.

However, the BBK public key is returned across successful Secure Payment
Confirmation invocations (and therefore across sessions) that return the passkey
credential ID that was passed in. In other words, the passkey credential ID must
provided and user must accept the transaction before the BBK public key is
exposed.

2.14. How does this specification distinguish between behavior in first-party and third-party contexts?

There is no difference. BBKs are returned in first-party and third-party Secure
Payment Confirmation invocations. However see the same question in the [Secure
Payment Confirmation Questionnaire](security-privacy-questionnaire.md).

2.15. How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?

In private browsing mode:
* If the user can access a passkey and there is an associated BBK, that BBK
  provides its public key and signature.
* When a BBK for a given passkey does not exist, no BBK is generated and
  persisted to avoid leaving a record on disk. And, no public key or signature
  are provided.

2.16. Does this specification have both "Security Considerations" and "Privacy Considerations" sections?

Yes.

2.17. Do features in your specification enable origins to downgrade default security protections?

No.

2.18. What happens when a document that uses your feature is kept alive in BFCache (instead of getting destroyed) after navigation, and potentially gets reused on future navigations back to the document?

Nothing is cached. If BBKs are created they are persisted rather than cached.

2.19. What happens when a document that uses your feature gets disconnected?

N/A

2.20. Does your spec define when and how new kinds of errors should be raised?

No

2.21. Does your feature allow sites to learn about the user’s use of assistive technology?

No

2.22. What should this questionnaire have asked?

None
