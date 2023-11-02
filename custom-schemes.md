# Concerns with custom schemes for identity presentment

Existing specifications like [OpenID4VP](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html#name-wallet-invocation) and [ISO 18013-7](https://www.iso.org/standard/82772.html) currently define the use of custom URL schemes (eg. `openid4vp:` and `mdoc:`) to enable verifier websites to invoke holder (wallet) applications. This approach works in some scenarios, but has a number of problems we aim to solve by designing a browser API which can replace the use of custom URL schemes in these specifications. This DRAFT document aims to outline the questions and concerns with the use of custom URL schemes for identity presentment in order to provide more detail on potential benefits of a dedicated browser API.

## Can wallets reliably determine their invoker?

   - Android and iOS offer no facility to determine the web origin which triggered a navigation to a custom URL scheme.
   - This seems to increase the opportunity for attacks such as replay attacks, man-in-the-middle attacks and phishing. It also seems to limit the opportunity for abuse detection and prevention.
   - A browser API can securely pass the requesting origin on to the wallet so that the wallet can know with confidence what origin the user was on when the request was initiated.
   
## How are wallet apps supposed to pass info back to the invoking page?

  - Does the page give the wallet a URL to navigate back to as part of the custom scheme URL it uses to launch the wallet?

If so, that's a problem because it won't go back to the same tab. Apps don't have the ability to return information to the same page that invoked them after a custom scheme invocation; they can just open a new page in a new tab.

## Can wallets limit requests to secure contexts?

   - If a user uses a website via http (not https) and it invokes a custom scheme, it's possible that an attacker with access to the network could eavesdrop on or even modify the request (such as by replacing the verifier's public key with their own).
   - A browser API could (and almost certainly should) take the position that it will function only within the context of a secure context (https).

## User experience concerns

### What is the user experience when multiple wallet apps are installed?

   - When multiple Android applications register for the same custom URL scheme, the operating system presents the user with a picker every time that scheme is invoked. Even if the verifier provides the user with appropriate context, a user may have trouble remembering which application has which credentials and so be more likely to cancel or fail the request.
   - On iOS, when multiple applications are installed that register for the same custom URL scheme, the behavior is [undefined](https://stackoverflow.com/questions/13130442/multiple-apps-with-the-same-url-scheme-ios). At present, one indeterminate wallet will be invoked without any indication that other wallets were available.

### What is the user experience when no wallet app is installed or when the user cancels wallet invocation?

   - Browsers generally silently ignore navigation to URL schemes with no handler installed. This likely means a website cannot distinguish between nothing happening while waiting for a wallet to process a request and nothing happening because no wallet app is installed.
   - Additionally browsers on Android and iOS often offer the user the choice of whether or not to invoke an application for a custom URL scheme navigation (even if there is only one such app). The user canceling at this stage is also indistinguishable from no valid application being installed.
   - A browser-based API could show the user some meaningful error message when no relevant wallets are installed.

### What is the user experience on a desktop operating system?

   - Most wallet applications exist only as mobile applications, and users are generally reluctant if not unable (due to enterprise security settings, for instance) to install native applications on desktop operating systems. What fallback will flows relying on custom URL schemes provide for users of desktop operating systems?
   - A browser-based API can potentially provide low-friction integration between a desktop browser and a wallet application running on a mobile phone or a website that provides an in-browser wallet experience on the same device.

### Friction and app switching

Opening a custom URL scheme will switch apps, which is a high friction user flow. A browser API could present relevant UI (even from third party wallet apps) in a sheet over the webpage, which is a less disruptive flow.

## What are the privacy implications of a wallet accepting custom schemes?

   - [Android aims](https://developer.android.com/training/package-visibility) to keep the list of installed applications private from other applications, but this capability is reduced for applications which handle arbitrary intents like custom URL schemes. By registering to support custom URL schemes, a wallet application may effectively be leaking PII (such as the user's likely citizenship status or state of residency) to other applications on the device.
   - A purpose-built OS API for wallet invocation (on top of which a browser API would be built) can avoid any such information leakage.

## How can users be assured that they have control over where their personal information is shared?

   - A buggy or poorly designed wallet application may store or transmit sensitive information about the user (such as which website(s) the user was visiting and when) without necessarily ensuring the user has provided informed consent. Malicious websites may exploit bugs in wallets to cause them to leak information without requiring any consent from the user. Wallets and credentials may come from any source, but users will expect their operating systems and browsers to protect their privacy from buggy or malicious wallet applications.

<hr>

Due to these concerns, Google Chrome has [proposed](https://groups.google.com/a/chromium.org/g/blink-dev/c/wcCrcMTELS0/m/ZSPxAT0LAgAJ) blocking website access to specific high-risk custom URL schemes, and instead invoking wallets via a browser-mediated API. Anyone interested in contributing to the design of such an API is encouraged to join the W3C's [Web Incubator Community Group](https://www.w3.org/community/wicg/) and contribute to the [Identity Credential proposal](https://github.com/WICG/identity-credential/). While websites will likely always be able to communicate directly with wallet applications in some fashion, it is the group's hope that a browser-based API can be designed which is a better option for wallets and verifiers, and which also provides users with confidence in having their sensitive identity information potentially exposed to websites they visit.
