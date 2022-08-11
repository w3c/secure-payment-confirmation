# FAST Checklist for Secure Payment Confirmation

The following evaluation is based on the [FAST checklist](https://w3c.github.io/apa/fast/checklist.html).

## If technology allows visual rendering of content

SPC does not include any features that result in rendering content in a page.

When Secure Payment Confirmation is called at authentication time, the browser displays the following information in a transaction dialog owned by the browser:

* payeeName and/or payeeOrigin (both strings)
* instrument displayName and icon (an image or link to an image). The specification discusses how to use the displayName as alt text for the icon.
* Transaction total and currency (both strings).

## If technology provides author control over color

N/A

## If technology provides features to accept user input

N/A

## If technology provides user interaction features

* The transaction dialog includes browser-owned buttons to cancel or proceed with Web Authentication.
* [Accessibility considerations for WebAuthn](https://www.w3.org/TR/webauthn/#sctn-accessiblility-considerations) are documented in that specification.

## If technology defines document semantics

N/A

## If technology provides time-based visual media

N/A

## If technology provides audio

N/A

## If technology allows time limits

* SPC relies on the timeout parameter of Web Authentication.
* Relevant [accessibility considerations for WebAuthn](https://www.w3.org/TR/webauthn/#sctn-accessiblility-considerations) are documented in that specification.

## If technology allows text content

* The SPC transaction dialog includes both an icon and string to help the user identify the relevant payment instrument. 
## If technology creates objects that don't have an inherent text representation

N/A

## If technology provides content fallback mechanisms, whether text or other formats

N/A

## If technology provides visual graphics

N/A

## If technology provides internationalization support

N/A

## If technology defines accessible alternative features

N/A

## If technology provides content directly for end-users

N/A

## If technology defines an API

* SPC relies on the useragent to generate a user interface (the transaction dialog). Previous review of Secure Payment Confirmation concluded there was no need to review the specification ([issue 14](https://github.com/w3c/a11y-request/issues/14#issuecomment-915393934)) and the specification is largely the same since that review.


## If technology defines a transmission protocol

N/A