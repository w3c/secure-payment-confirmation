# Authenticators and SPC

In this document we explain the landscape (and complexities) of FIDO authenticators and how they can be used with Secure Payment Confirmation (SPC). One goal of this document is to facilitate conversations within the Web Payments Working Group about priorities for increasing support for authenticators that work with SPC. Questions? Please contact Stephen McGruer and Ian Jacobs on the Web Payments Working Group's mailing list [public-payments-wg@w3.org](https://lists.w3.org/Archives/Public/public-payments-wg/).

## Background

Because SPC adds functionality on top of FIDO authentication, to understand the relationship between SPC and supported authenticators, it is useful to consider two categories of capability and how they interact:

* FIDO ecosystem capabilities for scale, interoperability, and usability
* Capabilities specific to the SPC experience

### Capabilities of the FIDO ecosystem for scale, interoperability, and usability

Primarily driven by login use cases (rather than payments), the FIDO ecosystem seeks to increase usability by enabling credentials to be shared:

* Across browsers on the same device
* Across devices
* Across operating systems
* Across authenticator solution providers (including platform providers as well as third-party password/passkey providers)

### Capabilities specific to the SPC experience

When credentials are available for use with SPC, the browser presents a transaction dialog for authentication. When credentials are not available for use with SPC, the browser presents a “fallback” dialog box (and does so for consistency with the privacy model of Web Authentication). In order to choose which experience to provide, the browser needs to silently determine whether any of the credentials passed as input to the API are **immediately available** (either on the current device or via hybrid, for example). 

Ordinary Web Authentication credentials can only be used by the relying party (RP) that created them. SPC is designed to support use cases where the authentication “ceremony” can be conducted by one party (e.g., a payment service provider operating within a merchant’s page) and the authentication results can be validated by another (e.g., a bank). Credentials need to be **usable with SPC** in all these scenarios:

* An RP using its own credential, either in a 1p context or 3p context (iframe)
* Another party using that credential (with RP permission) in 1p or 3p context

The two main capabilities that support the above requirements are thus:

1. Identifying a credential as being usable with SPC (“an SPC credential”), either by the RP or another party (e.g., a PSP).
2. Understanding from available authenticators whether one or more input credentials are immediately available for usage with SPC.

### Challenges arising from the interplay of these capabilities

To the extent that SPC functionality remains close to FIDO functionality, SPC should generally benefit from capabilities that increase interoperability and usability. However, some challenges arise from SPC’s dependency on the full FIDO ecosystem and the capabilities that support scale, interoperability, and usability:

* How does the browser silently determine if a roaming authenticator has an available credential if the authenticator is not plugged in?
* How does the browser silently determine via hybrid if a nearby mobile phone has a relevant credential?
* The FIDO ecosystem involves platform-specific APIs as well as CTAP, and is rapidly evolving, which complicates SPC implementations.

### Challenges related to whether a credential is usable with SPC

For credentials to be used only by a relying party in a 1p or 3p context, no special information is required as part of the credential for it to be usable with SPC.

However, for credentials to be used by a party other than the relying party in a 1p or 3p context, the credential must be marked as such at credential creation time. In order to be able to use that credential across browsers or devices, any information that characterizes it as usable with SPC by another origin needs to be stored with the credential rather than in the browser. 

Today there are two parts to the proposed SPC solution: 

1. The first is the ‘payment’ extension at the Web Authentication level. 
2. When that extension is used with an authenticator that supports the latest version of CTAP, it should cause the authenticator to create the CTAP cross-origin payment bit ("thirdPartyPayment") for that credential.

However, because platform authenticators typically don’t implement CTAP at all, they would not by default implement the CTAP cross-origin bit. The Web Payments Working Group has lobbied platform authenticator providers to support this feature.

As a result of current lack of interoperable support for the CTAP cross-origin bit, on most platforms, for the time being, the browser records information about cross-origin usage in the user profile. This means that this information does not scale to other browser, devices, etc.


### Challenges related to whether a credential is immediately available

Different approaches may be taken to determine whether a credential is immediately available.

**Note**: This capability is also often required to implement the WebAuthn 'autofill-like' conditional mediation experience, as well as the new [immediate mediation proposal](https://github.com/w3c/webauthn/wiki/Explainer:-WebAuthn-immediate-mediation).

#### Simple query

One way to accomplish this is with a **simple query**:

1. The SPC caller provides a list of an RP’s credentials that could be used to authenticate the user for the transaction.
2. The browser silently queries all reachable authenticators for all credentials they have for that RP. The browser then compares the returned list of all credentials against the input list from the caller.
3. If there is a match (an overlap), the browser initiates the authentication experience, otherwise the fallback experience.

**Note**: This is how Chrome implements this capability on Android.

#### Simple response

Another way to accomplish this is with a **simple response**:

1. The SPC caller provides a list of an RP’s credentials that could be used to authenticate the user for the transaction.
2. For each of those credentials, the browser silently queries all reachable authenticators with the query “Do you have this credential?”. The authenticator returns either true or false.
3. If there is a match, the browser initiates the authentication experience, otherwise the fallback experience.

**Note**: This approach is more suitable if the only way that the authenticator will respond silently to a query about credential availability is to provide a “yes” or “no” response. In such a case, the query will need to be more complex than simply providing a credential id and asking “do you have this one?” The query will also need to include information about whether the credential supports cross-origin use. 

#### Cache information at creation time

Another approach has been used as well:

1. During credential creation, the browser caches the fact that a passkey has been created.
2. Later at transaction time, the SPC caller provides a list of an RP’s credentials that could be used to authenticate the user for the transaction.
3 .The browser looks at its own cache of credentials, and compares them against the input list from the caller.
4. If there is a match (an overlap), the browser initiates the authentication experience, otherwise the fallback experience.

**Note**: This is how Chrome implements this capability only for payment-enabled passkeys on MacOS and Windows. This approach does not scale; the browser only recognizes as “immediately available” those credentials created  through that specific browser (and perhaps browser+user profile, etc). 

## Implementation (July 2025)

As mentioned above, ideally SPC will benefit from advances in the FIDO ecosystem, and SPC-specific requirements will inform API development in that ecosystem. In the short term, the Chrome implementation of SPC:

* Leverages some but not all APIs that are being developed for the FIDO ecosystem.
* Relies in some cases on browser-specific cached information.

In the following sections we look at the state of SPC implementations on different platforms, as of July 2025.

### Platform Authenticators

First we describe the landscape of platform authenticators in the FIDO ecosystem today and the means for the browser to interact with them. This is the set of platform authenticators that **could potentially be used with SPC**. The table below organizes authenticators and ways to interact with them by operating system. 
In the table:

* **GPM** refers to “Google Password Manager (for Chrome only)”
* **Chrome internal** refers to “Chrome internal profile credential store (for Chrome only)”

<table>
  <tr>
    <th></th>
    <th></th>
    <th></th>
    <th></th>
    <th></th>
	<th colspan="3">For 3p passkey providers</th>
  </tr>
  <tr>
    <th></th>
    <th>Windows Hello</th>
    <th>GPM</th>
    <th>iCloud Keychain</th>
    <th>Chrome internal</th>
	<th>Credential Manager API</th>
	<th>Plugin passkey manager</th>
	<th>ASPasskey Credential Request</th>	
  </tr>
  <tr>
	<th>Windows</th>
	<td>&#x2713;</th>
	<td>&#x2713;</th>
	<td></td>
	<td></td>
    <td></td>
	<td>&#x2713;</th>		
    <td></td> 
  </tr>
  <tr>
	<th>MacOS</th>
	<td></td>	
	<td>&#x2713;</th>
	<td>&#x2713;</th>
	<td>&#x2713;</th>	
	<td></td>
	<td></td>
	<td>&#x2713;</th>		
  </tr>
  <tr>
	<th>Android</th>
	<td></td>	
	<td>&#x2713;</th>
	<td></td>	
	<td></td>	
    <td>&#x2713;</th>	
	<td></td>
	<td></td>
  </tr>
  <tr>
	<th>iOS</th>
	<td></td>	
	<td></td>	
	<td>&#x2713;</th>
	<td></td>	
	<td></td>	
	<td></td>
	<td>&#x2713;</th>
  </tr>
  <tr>
	<th>ChromeOS</th>
	<td></td>	
	<td>&#x2713;</th>
	<td></td>	
	<td></td>	
	<td></td>	
	<td></td>
	<td></td>
  </tr>
  <tr>
	<th>Linux</th>
	<td></td>	
	<td>&#x2713;</th>
	<td></td>	
	<td></td>	
	<td></td>	
	<td></td>
	<td></td>
  </tr>
</table>

### Platform Authenticators used with SPC

In order for SPC to work with the above platform authenticators, the capabilities described earlier must be supported in those authenticators, or the browser must provide a (temporary) alternative solution.

The following table describes Chrome’s implementation of SPC as of July 2025, by operating system. In the table:

* **GPM** refers to “Google Password Manager (for Chrome only)”. Note that Chrome currently calls GPM directly rather than through the Credential Manager API on Android
* **Chrome internal** refers to “Chrome internal profile credential store (for Chrome only)”

<table>
  <tr>
    <th></th>
    <th>Authenticator type supported</th>
    <th>Authenticator supports cross-origin CTAP bit? (If yes, used by SPC implementation?)</th>
	<th>Authenticator supports silent listing of immediately available credentials (If yes, used by SPC implementation?)</th>	
  </tr>
  <tr>
	<th>Windows</th>
	<td>Windows Hello</td>
	<td>Yes<sup>*</sup> (No<sup>&#x2020;</sup>)</td>
	<td>Yes<sup>*</sup> (No)</td>
</tr>
  <tr>
	<th>MacOS</th>
	<td>Chrome internal</td>
	<td>No</td>
	<td>Yes (No)</td>
  </tr>
  <tr>
	<th>Android</th>
	<td>GPM</td>
	<td>Yes (Yes)</td>
	<td>Yes, via direct usage of GPM (Yes)</td>
  </tr>
  <tr>
	<th>iOS</th>
	<td>N/A</td>
	<td>N/A</td>
	<td>N/A</td>
  </tr>
  <tr>
	<th>ChromeOS</th>
	<td>N/A</td>
	<td>N/A</td>
	<td>N/A</td>
  </tr>
  <tr>
	<th>Linux</th>
	<td>N/A</td>
	<td>N/A</td>
	<td>N/A</td>
  </tr>
</table>

<sup>*</sup> Supported in Windows 11 only<br/>
<sup>&#x2020;</sup> Chrome stores the fact that the credential can be used cross-origin in the user profile database.

## Avenues to support more authenticators

As the sections above should make clear, there are many avenues for increasing the number and range of supported authenticators by:

* Increasing the number of platform authenticators supported by a given implementation of SPC.
* Leveraging platform APIs instead of using alternative approaches (e.g., caching information in the user profile).
* Adding initial support for roaming authenticators (a long-standing goal of the Web Payments Working Group).

Below we detail some specific opportunities identified by the Chrome Team. The items listed here are purely speculative and simply meant to educate as well as gather potential interest from readers.

### Increased support on Windows

Windows Hello supports a list-credentials API (Windows 11 only), similar to what Chrome uses for Android with Google Password Manager. Adopting this would allow the use of 'normal' WebAuthn passkeys for SPC in the first-party case (rp.com using an rp.com passkey).

Windows Hello supports the third-party payment bit (Windows 11 only), so in theory Chrome on Windows could move to storing the bit in the credential rather than in the user profile database. In combination with using the list-credentials API (previous paragraph), this would mean that different Chromium-based browsers (e.g., Chrome Stable, Chrome Beta, Edge) that ship SPC on the same device would all be able to see the third-party bit for a passkey.

Chrome for Windows also supports Google Password Manager as a passkey provider today. In theory, SPC on Chrome for Windows could also allow Google Password Manager. A technical analysis still needs to be carried out to confirm if GPM on Chrome for Windows supports the list-credentials and third-party payment bit API, although technically right now we do not use either of those on Windows (see above).

### Increased support on MacOS

The Chrome internal profile credential store supports a list-credentials API, similar to what Chrome uses for Android with Google Password Manager. Adopting this would allow the use of 'normal' WebAuthn passkeys for SPC in the first-party case (rp.com using an rp.com passkey). However, many 'normal' WebAuthn passkeys are no longer created in this credential store on MacOS - users prefer GPM or iCloud Keychain. As such, it might be worth waiting until Chrome supports GPM and/or iCloud Keychain instead (see below).

Chrome for MacOS also supports Google Password Manager as a passkey provider today. In theory, SPC on Chrome for MacOS could also allow Google Password Manager, or even switch entirely from the internal Chrome profile store to GPM. A technical analysis still needs to be carried out to confirm if GPM on Chrome for MacOS supports the list-credentials and third-party payment bit API, although technically right now Chrome does not use either of those on MacOS (see above).

Chrome for MacOS also supports iCloud Keychain as a passkey provider today. In theory Chrome could enable this as a passkey provider for SPC, however iCloud Keychain does not support either listing credentials or the third-party payment bit, so if Chrome started using either of those APIs then this would become a regression.

### Support for third-party passkey providers

All major platforms either already support or are adding support for third-party passkey providers (i.e., provided neither by the browser itself nor directly by the OS vendor).

#### Windows

This might happen to Chrome silently depending on what Windows does; it is unclear if third-party providers will be a separate API or just accessed via the same APIs Chrome uses for Windows Hello today.

Questions and implications:

* TBD - are third-party providers required/able to support a list-credentials API?
* It would be up to individual providers as to whether or not they support the third-party payment bit. For those that don't, Chrome would have to continue to store the bit locally and it would not work across browsers on the same device (e.g., Chrome and Edge on the same Windows device).

#### MacOS

SPC on Chrome would have to explicitly allow third-party providers (TBD if this is synonymous with allowing iCloud Keychain).

Questions and implications:

* Would third-party providers (via whatever MacOS APIs) would be able to support a list-credentials API? (Current assumption is “no.”)
* It would be up to individual providers as to whether or not they support the third-party payment bit. For those that don't, Chrome would have to continue to store the bit locally and it would not work across browsers on the same device (e.g., Chrome Stable and Chrome Beta on the same Macbook).

#### Android

SPC on Chrome would have to explicitly allow third-party providers, by switching to  / supporting the Credential Manager API.

Questions and implications:

* Would lose ability to silently list credentials (for locked passkey providers)
* It would be up to individual providers as to whether or not they support the third-party payment bit. For those that don't, Chrome would have to start storing the bit locally on Android and it would not work across browsers on the same device (e.g., Chrome and Edge on the same Android phone).

