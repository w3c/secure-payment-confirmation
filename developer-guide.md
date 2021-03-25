# Secure Payment Confirmation Developer Guide

The experimental Secure Payment Confirmation is available for developer testing
in Chrome Canary, and will soon be available in stable channels in M91 as an
[Origin Trial](https://github.com/GoogleChrome/OriginTrials). This document
describes how to start using this API on your website.

## Step 1: Get a compatible computer

Currently, the Secure Payment Confirmation trial is available on:
* MacOS with
[secure enclave](https://support.apple.com/en-ca/guide/security/sec59b0b31ff/web)
(e.g., 2016 MacBookPro or later).
* Windows 10 with Windows Hello (e.g., version 1607 or later).

Note that the feature is designed for use with biometric authenticators such as
Touch ID on Mac, but more generally it is compatible with
[user-verifying platform authenticators](https://www.w3.org/TR/webauthn/#user-verifying-platform-authenticator)
which may use a device PIN, for example.

## Step 2: Get a compatible version of Chrome

* Canary channel: 91.0.4459.0 or later

Until the origin trial is available in M91, you can test out the feature by
enabling the flag chrome://flags#enable-experimental-web-platform-features.
You can verify that you have the right version of Chrome by trying out this
[test page](https://rsolomakhin.github.io/pr/spc/).

## Step 3: Enable the Origin Trial on your website

Stay tuned at paymentrequest@chromium.org for updates on origin trial
availability coming in M91. Once it's available, you can follow the
[Origin Trial Developer Guide](https://github.com/GoogleChrome/OriginTrials/blob/gh-pages/developer-guide.md)
to obtain a trial token and use it on your website.

## Frequently Asked Questions

### Q: What are the maximum dimensions of the payment instrument icon when registering a PaymentCredential?
A: Chrome's current implementation expects an icon that is 32px wide by 20px high.

## Questions & Feedback

Please contact payments-dev@chromium.org.
