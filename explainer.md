# Digital Credentials API

Government recognized digital credentials (e.g. driver’s licenses) are being increasingly utilized on the web. We propose a deliberate and flexible API to enable browsers to continuously improve the balance of benefits and risks for users and the entire online community.

# Why?

Government-recognized documents play a big and constructive role in society (e.g., drivers licenses, passports, etc.). Increasingly, with the movement of government and financial services online, and regulation (e.g. [eIDAS](https://en.wikipedia.org/wiki/EIDAS) and various [age verification regulations](https://en.wikipedia.org/wiki/Age_verification_system)), these paper-based documents are gaining digital counterparts

Along with all of their potential from the physical world, the presentation of government recognized  digital credentials also brings their associated risks of abuse, such as the potential for an increase in [surveillance](https://github.com/w3cping/credential-considerations/blob/main/credentials-considerations.md#no-phoning-home), [censorship](https://github.com/w3cping/credential-considerations/blob/main/risks.md#censorship-and-reduction-in-access-to-free-information), [discrimination](https://github.com/w3cping/credential-considerations/blob/main/credentials-considerations.md#free-expression), and intrusion to the online world.

The most recent online presentation protocols (e.g. [OpenID4VP](https://openid.github.io/OpenID4VP/openid-4-verifiable-presentations-wg-draft.html)) and regulations (e.g. eIDAS) were designed around a Web that lacked the intentional support for such a critical task; depending instead on general purpose primitives such as custom schemes. Unfortunately, the use of [custom schemes](https://github.com/WICG/digital-identities/blob/main/custom-schemes.md#concerns-with-custom-schemes-for-identity-presentment) left us with two problems:

Because custom schemes can be largely opaque to user agents, they substantially limit the user agent’s ability to exercise its agency in reducing the risk of abuse. Users rely on their browser to help provide transparency and control over the use of their data. From the subtle, like providing [UI](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/security/url_display_guidelines/url_display_guidelines.md) [cues](https://blog.chromium.org/2023/05/an-update-on-lock-icon.html) as to the privacy and security risks of various operations. To the more severe, like [warning](https://blog.google/products/chrome/google-chrome-safe-browsing-real-time/) about interacting with known phishing sites.

Even in highly-regulated and non-abusive use cases (e.g. eIDAS), custom schemes have  [security](https://github.com/WICG/digital-identities/blob/main/custom-schemes.md#can-wallets-limit-requests-to-secure-contexts) and [privacy](https://github.com/WICG/digital-identities/blob/main/custom-schemes.md#what-are-the-privacy-implications-of-a-wallet-accepting-custom-schemes) risks, as well as a series of suboptimal [user experiences. We posit that these limitations](https://github.com/WICG/digital-identities/blob/main/custom-schemes.md#user-experience-concerns) can be addressed by a purpose-built browser API. While some have reasonably argued that these limitations will slow-down and limit the deployment (and so, the abuse) of such technology, it seems more advantageous  to rely on intentional design choices to manage this tradeoff than such accidents. 

In the pursuit of establishing a standard for web-based identity sharing, it is paramount that we not only strive for ease of use but also exercise utmost caution to uphold high standards of security and privacy. This is essential to safeguard users from potential identity theft and ensure they are fully informed about the implications of online identity sharing before providing their consent. We firmly believe that relying on a browser API and the mobile platform is the most reliable approach for identity digital credentials to be shared securely online.


# What?

First we must acknowledge, architecturally, that this is a rapidly evolving space with a large number of moving parts at various different levels of maturity. Users, regulations (e.g. eIDAS), wallets, verifiers, issuers, formats (e.g. [ISO mDocs](https://www.iso.org/obp/ui/en/#iso:std:iso-iec:18013:-5:ed-1:v1:en) and [W3C VCs](https://verifiablecredentials.dev/), just to name a few) and protocols (e.g. [OpenID4VP](https://openid.github.io/OpenID4VP/openid-4-verifiable-presentations-wg-draft.html), ISO REST’s API, [VC API](https://github.com/w3c-ccg/vc-api)), operating systems and browsers (e.g. evolving mitigation strategies, for [example](https://docs.google.com/document/d/1L68tmNXCQXucsCV8eS8CBd_F9FZ6TNwKNOaFkA8RfwI/edit#heading=h.8gq5f7p3it8q)) are all still in various stages of formation and need the space and autonomy to evolve efficiently.

So, to the extent that we can, it is important to leave levels of indirections that would allow us to extend the API without having to redesign it. Sometimes, extensibility, autonomy (eg. of different standards bodies) and the ability/incentives of ecosystem adoption are in tension with other goals, such as the user’s privacy and security, so we often have to find a good and principled balance (e.g. more often than not, [users first, developers second, and browser engines third](https://www.w3.org/TR/design-principles/#priority-of-constituencies), guides our choices at the W3C). 

To balance this tension we propose an API with the following key properties:

- By separating the act of requesting from the specific protocol, we can enable flexibility and adaptability in both the protocol and credential formats. This way, the pace of changes in browsers won't hinder progress or block new developments.
- Require request transparency, enabling user-agent inspection for risk analysis
- Assume response opacity (encrypted responses), enabling verifiers and holders to control where potentially sensitive PII is exposed
- Prevent website from silently querying for the availability of digital credentials and communicating with wallet providers without explicit user consent 

At its core, the API is designed for a website ("verifier") to [transparently](https://github.com/w3cping/credential-considerations/blob/main/credentials-considerations.md#in-context-explanations) request the [selective disclosure](https://github.com/w3cping/credential-considerations/blob/main/credentials-considerations.md#selective-disclosure) of attributes from (issued) digital credentials that were provisioned - ahead of time - to wallets ("holders"), in a manner that is seamlessly compatible with existing architectural choices (such as [OpenID4VP integration](https://github.com/openid/OpenID4VP/issues/125)).

Here is an example of how the  the API might be used in practice:

```javascript
    const digitalCredential = await navigator.identity.get({
      digital: {
        providers: [{
          // Protocol extensibility:
          protocol: "oid4vp",
          // An example of an OpenID4VP request to wallets.
          // Based on https://github.com/openid/OpenID4VP/issues/125
          request: {
            nonce: "n-0S6_WzA2Mj",
            presentation_definition: {
              // Presentation Exchange request, omitted for brevity
            }
          }
        }],
      },
    });
    // To be decrypted on the server...
    const encryptedData = digitalCredential.data;
```
You can read a more detailed and technical description of the API in the [specification draft](https://wicg.github.io/digital-identities/).


# Alternatives Considered

There are many alternatives that were considered, most notably:

- [Do nothing](https://github.com/w3cping/credential-considerations/blob/main/risks.md#the-consequences-of-a-failure-to-act-are-as-valid-as-those-of-acting) (the intentional status quo in browsers for the past several years)
- Various different [API proposals](https://github.com/WICG/digital-identities/tree/main/proposals): [an mDoc-specific API](https://github.com/WICG/digital-identities/blob/main/proposals/mobile-document-request-api-proposal.md), [extending the Credential Management API](https://github.com/WICG/digital-identities/blob/main/proposals/digital-credential-proposal.md), [extending the FedCM API](https://github.com/WICG/digital-identities/blob/main/proposals/identity-credential-proposal.md) and [introducing navigator.identity](https://github.com/WICG/digital-identities/blob/main/proposals/navigator-identity-proposal.md)

# Open Questions

There are still many open questions, but a few big ones:

- To what extent does the browser introspect the request to wallets (for privacy and security reasons)? How much of that needs to interoperate between browsers, vs. be browser-specific points of differentiation in offering privacy features to users?
- Will existing protocols ([example](https://github.com/openid/OpenID4VP/issues/125)) adopt this API?
- Will regulation ([example](https://digital-strategy.ec.europa.eu/en/library/european-digital-identity-architecture-and-reference-framework-outline)) adopt this API?

# Out of Scope

The following topics are currently out of scope for the API:

- A website (issuer) requesting the issuance of a digital credential to a digital wallet
- A website (verifier) explicitly requesting multiple digital credentials from multiple wallets in the same request

# [Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)

> 01.  What information does this feature expose,
>      and for what purposes?

The Digital Credential API exposes a one-time user-mediated end-to-end encrypted communication channel from websites, to digital wallet applications, and back to websites. It is designed to be a better option than established lower-level communication channels like custom schemes, QR codes, and server-to-server network communication. How exactly this channel is used is up to the wallet applications and host operating system, but we are designing it to be suitable for conveying presentations of digital credentials such as claims in mobile driver’s licenses.

> 02.  Do features in your specification expose the minimum amount of information
>      necessary to implement the intended functionality?

A primary use case of the digital credentials API is for selective disclosure of identity properties such as a cryptographic attestation that the user holds a California driver’s license of an adult. The use of selective disclosure, however, is a decision for the verifier website, wallet and credential issuer based on the use case. There are legitimate scenarios, such as creating or recovering an account on a government website, where uniquely identifiable and potentially non-resettable PII is potentially exposed.

The API is designed to expect the use of response encryption so that this PII is exposed only to the requesting server and not any code running in the web page or browser. It is an [open question](https://github.com/WICG/digital-identities/issues/109) whether this response encryption is something we can reasonably enforce at this layer or not.

The API is also designed to require request transparency to enable user agents and operating systems to appropriately inform users about the level of privacy risk involved in the request. A major area of outstanding work is to outline in the specification our privacy and security guidance to implementers, but we already know that the [Chromium implementation](https://docs.google.com/document/d/1L68tmNXCQXucsCV8eS8CBd_F9FZ6TNwKNOaFkA8RfwI/edit) intends to show users stronger warnings in riskier scenarios such as those that lack the use of selective disclosure.

> 03.  Do the features in your specification expose personal information,
>      personally-identifiable information (PII), or information derived from
>      either?

Yes, the API is designed to help facilitate communication of PII from secure wallet applications to verifier web servers. 

> 04.  How do the features in your specification deal with sensitive information?

Today online identity verification (eg. for KYC) is usually done by submitting photos of government identity documents (for example, uploading photos containing the PII via the web’s `<input type=file>` mechanism. Due to the privacy and security limits of this approach, credential issuers and verifiers are working to move to the use of wallet applications which can do selective disclosure of cryptographically attested properties. Today those approaches rely on generic web->app communication paths like custom schemes and server-to-server communication. 

The use of wallets aims to improve on the status quo of uploading images in a number of ways:
 * Enable the use of selective disclosure (eg. for more privacy-friendly age 18+ verification)
 * Enable the PII to be end-to-end encrypted between the wallet application and the relevant verification server
 * Enable the use of request authentication where a wallet is required to cryptographically verify that the requester has permission to access the credential

Further, this specification aims to improve over the existing communication channels used by websites to talk to wallet apps by:
 * Enabling browsers and operating systems to provide meaningful credential selection and permission screens to users prior to wallet applications becoming aware of the presentation request
 * Securely communicate the origin of the requesting site to the wallet application so that it can implement it’s own MITM protection with support of WebPKI
 * Enabling browsers to inspect requests and provide additional UI affordances to users

> 05.  Do the features in your specification introduce state
>      that persists across browsing sessions?

No. In particular, the current scope is just about credential presentation (read-only). We have had requests to expand scope to consider credential issuance APIs but no work has begun on that.

> 06.  Do the features in your specification expose information about the
>      underlying platform to origins?

Not directly. Wallets may (intentionally or inadvertently) expose some information through this API (such as some indication of which wallet app the user is using). But the primary motivation for these wallets over traditional federated authentication systems is that the wallet can act in the user’s interest to protect their privacy even from the credential issuer. Issuers generally choose which wallets to support their credentials in and can impose privacy requirements on those wallets. Users will often have their choice of multiple wallets and are expected to use reputation for privacy as part of their decision making.

> 07.  Does this specification allow an origin to send data to the underlying
>      platform?

Yes. The credential presentation request is sent to the underlying platform to mediate.

> 08.  Do features in this specification enable access to device sensors?

No

> 09.  Do features in this specification enable new script execution/loading
>      mechanisms?

No

> 10.  Do features in this specification allow an origin to access other devices?

Not yet, but we expect to expand the API to enable cross-device presentation flows using the same mechanism used by passkeys ([FIDO CTAP](https://fidoalliance.org/specs/fido-v2.2-rd-20230321/fido-client-to-authenticator-protocol-v2.2-rd-20230321.html).

> 11.  Do features in this specification allow an origin some measure of control over
>      a user agent's native UI?

Not control, no. User agents may add additional affordances for user transparency and control. 

> 12.  What temporary identifiers do the features in this specification create or
>      expose to the web?

None

> 13.  How does this specification distinguish between behavior in first-party and
>      third-party contexts?

The API is currently available only in first-party contexts, but there is an [open issue](https://github.com/WICG/digital-identities/issues/78) to explore adding a permission policy to enable requests in third-party contexts.

> 14.  How do the features in this specification work in the context of a browser’s
>      Private Browsing or Incognito mode?

Like other browser authentication (eg. WebAuthn) and identification (eg. autofill) features, the feature is available to users to use in private browsing. Regardless of whether private browsing is in use, the feature is designed to not communicate information without the user’s explicit permission each and every time it's invoked. There are legitimate use cases for this API in private browsing such as privacy-preserving age verification.

> 15.  Does this specification have both "Security Considerations" and "Privacy
>      Considerations" sections?

Not yet, but it will. The specification is still being written as we incubate the API and gain experience with real-world experiments. 

> 16.  Do features in your specification enable origins to downgrade default
>      security protections?

No

> 17.  What happens when a document that uses your feature is kept alive in BFCache
>      (instead of getting destroyed) after navigation, and potentially gets reused
>      on future navigations back to the document?

Implementations should fail or postpone any requests which occur while the page is not visible to the user. The spec may introduce a [requirement for user activation](https://github.com/WICG/digital-identities/issues/91) which could help with this.

> 18.  What happens when a document that uses your feature gets disconnected?

The API is not supported outside of a top-level browsing context. If we change that, then I imagine it would be reasonable for the API to fail when invoked from a disconnected document.

> 19.  What should this questionnaire have asked?

What are the security and privacy implications of not shipping this feature? How does this feature fit into the larger privacy risk landscape. We believe the feature will lead to a reduced risk relative to the status quo, but this is very subjective and hard to demonstrate difinitively. 

# Related Work

- [Real-world identity on the web - risks and mitigations](https://github.com/w3cping/credential-considerations/blob/main/risks.md#the-consequences-of-a-failure-to-act-are-as-valid-as-those-of-acting), Rick Byers
- [Concerns with custom schemes for identity presentment](https://github.com/WICG/digital-identities/blob/main/custom-schemes.md), Rick Byers
- [Digital Credentials API Standards Position](https://github.com/WebKit/standards-positions/issues/332#issuecomment-2019400609), Marcos Caceres
- [User considerations for credential presentation on the Web](https://github.com/w3cping/credential-considerations/blob/main/credentials-considerations.md), Nick Doty
- [Privacy Principles](https://w3ctag.github.io/privacy-principles/#identity), Jeffrey Yasskin and Robin Berjon
- [Intent to deprecate forwarding of mdoc-scheme URLs as Android Intents](https://groups.google.com/a/chromium.org/g/blink-dev/c/wcCrcMTELS0/m/ZSPxAT0LAgAJ), Adam Langley
- [Wallets on the web - PING @TPAC](https://docs.google.com/presentation/d/1Z7blMTME1tAQAdO-Wr42oVNN3CRIbklASjbJdB1JYOc/edit?resourcekey=0-ockU2NbemVbLEeF94-peNA#slide=id.p), Rick Byers
- [Chromium: RWI privacy risk mitigation design](https://docs.google.com/document/d/1L68tmNXCQXucsCV8eS8CBd_F9FZ6TNwKNOaFkA8RfwI/edit#heading=h.8gq5f7p3it8q), Rick Byers
