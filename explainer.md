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
- Websites cannot access any information without user interaction. This API ensures that sites cannot silently query for digital credentials or communicate with wallet providers without the user's active participation and confirmation of each action.

At its core, the API is designed for a website ("verifier") to [transparently](https://github.com/w3cping/credential-considerations/blob/main/credentials-considerations.md#in-context-explanations) request the [selective disclosure](https://github.com/w3cping/credential-considerations/blob/main/credentials-considerations.md#selective-disclosure) of attributes from (issued) digital credentials that were provisioned - ahead of time - to wallets ("holders"), in a manner that is seamlessly compatible with existing architectural choices (such as [OpenID4VP integration](https://github.com/openid/OpenID4VP/issues/125)).

Here is an example of how the  the API might be used in practice:

The API needs to be initiated through a user gesture, such as a button click:

```html
<button onclick="requestLicense()">Request Driver's license<button>
```


```javascript
async function requestLicense() {
  const oid4pv = {
    // Protocol extensibility:
    protocol: "oid4vp", // An example of an OpenID4VP request to wallets. // Based on https://github.com/openid/OpenID4VP/issues/125
    request: {
      nonce: "n-0S6_WzA2Mj",
      presentation_definition: {
        // Presentation Exchange request, omitted for brevity
      },
    },
  };
  const digitalCredential = await navigator.identity.get({
    digital: {
      requests: [oid4pv],
    },
  });
  // To be decrypted on the server...
  const encryptedData = digitalCredential.data;
}
```

You can read a more detailed and technical description of the API in the [specification draft](https://wicg.github.io/digital-identities/).

### Using the API from another origin

The specification allows usage of the API from a remote/third-party origin via the "digital-credentials-get" Permissions Policy. This is useful for scenarios where a website wants to request digital credentials from a wallet provider that is hosted on a different origin. The Permissions Policy can be set on an iframe that embeds the website that wants to use the API. Here is an example of how the Permissions Policy can be set on an iframe:

```HTML
<iframe allow="digital-credentials-get"></iframe>
```

## Horizontal reviews

* [Security and privacy TAG Questionnaire](horizontal-reviews/security-privacy.md)

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

# Related Work

- [Real-world identity on the web - risks and mitigations](https://github.com/w3cping/credential-considerations/blob/main/risks.md#the-consequences-of-a-failure-to-act-are-as-valid-as-those-of-acting), Rick Byers
- [Concerns with custom schemes for identity presentment](https://github.com/WICG/digital-identities/blob/main/custom-schemes.md), Rick Byers
- [Digital Credentials API Standards Position](https://github.com/WebKit/standards-positions/issues/332#issuecomment-2019400609), Marcos Caceres
- [User considerations for credential presentation on the Web](https://github.com/w3cping/credential-considerations/blob/main/credentials-considerations.md), Nick Doty
- [Privacy Principles](https://w3ctag.github.io/privacy-principles/#identity), Jeffrey Yasskin and Robin Berjon
- [Intent to deprecate forwarding of mdoc-scheme URLs as Android Intents](https://groups.google.com/a/chromium.org/g/blink-dev/c/wcCrcMTELS0/m/ZSPxAT0LAgAJ), Adam Langley
- [Wallets on the web - PING @TPAC](https://docs.google.com/presentation/d/1Z7blMTME1tAQAdO-Wr42oVNN3CRIbklASjbJdB1JYOc/edit?resourcekey=0-ockU2NbemVbLEeF94-peNA#slide=id.p), Rick Byers
- [Chromium: RWI privacy risk mitigation design](https://docs.google.com/document/d/1L68tmNXCQXucsCV8eS8CBd_F9FZ6TNwKNOaFkA8RfwI/edit#heading=h.8gq5f7p3it8q), Rick Byers
