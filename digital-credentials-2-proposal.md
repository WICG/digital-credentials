# Towards a preliminary API to retrieve Digital Credentials

Digital identities encompass a range of use cases, from age verification to sharing verifiable identity proofs, along with numerous other credential types. Managing and retrieving these identities is complicated by the varied formats in which they exist. A standardized approach, adaptable to diverse identity formats is essential for the Web, as it's expected that this API will be used in an extensive range of jurisdictions (which in turn may use different credential representation and retrieval standards). Moreover, such an approach must prioritize user privacy and guarantee robust security while allowing sites to verify the credentials they request and receive while keeping users in control over what, when, and with who, is share.

This document presents an API solution designed to deal with the varying digital identity formats. The API builds on the existing W3C Credential Management API and focuses on upholding user privacy and ensuring comprehensive data protection, streamlining the digital identity experience for all users.

## Why leverage Credential Management API

The Credential Management API is well positioned as the foundation for our proposed solution. True to its name, this API was designed with the concept of "credentials" at its core, specifically how these credentials are managed.

Of even greater significance, the API seamlessly integrates with the web's security model. It's designed to delegate the user experience to the browser, such as presenting digital wallet(s) from which users can select a requested credential. The specification is designed to be extensible, with comprehensive integration points. Each of these points has customizable security properties, like requiring a user gesture or being callable only from a top-level browsing context, to name a few.

However, given its initial design orientation towards password-style credentials, we believe some modifications to the Credential Management API are in order to optimize it for handling identity credentials.

## Extensions to Credential Management API: `requestIdentity()` method

We propose extending the `CredentialsContainer` interface with a new method: `requestIdentity()`. Unlike the existing .get() method, this new method is crafted to improve the retrieval and management of digital identities.

The rationale for introducing a new method, instead of using the existing .get(), is as follows:

- **Structured Requests**: The proposed `requestIdentity()` method is designed to accept a sequence of `IdentityRequestOptions` (discussed in more details below). This dictionary type is purposefully conceived to meet the specialized requirements of requesting identity credentials.
- **Clear Differentiation**: With `requestIdentity()`, we aim to prevent potential mix-ups between different request types, offering a clear distinction between tasks such as password retrieval and age verification (which could happen with using just the `.get()`method).
- **Developer Intuition**: Through the naming of `requestIdentity()`, we anticipate offering developers immediate clarity that an identity is being solicited.
- **Enhanced Flexibility**: `requestIdentity()`, as proposed, is designed with adaptability in mind, allowing for potential expansions and adjustments in line with future requirements.
- **Security Integrity**: As a suggested extension to the `CredentialsContainer`, the `requestIdentity()` method would inherit the robust security features of the Credential Management infrastructure.
- **Efficient Underpinnings**: At its core, `requestIdentity()` is intended to serve as a specialized wrapper around the existing `.get()` method, addressing its inherent limitations, particularly its challenges with type distinctions.
- **Storage Limitations**: An essential aspect of our proposal is to underline that `requestIdentity()` would focus exclusively on retrieval and would not encompass mechanisms for credentials storage. This may change in the future, but right now we only deal with identity credential retrieval.

The API extension would be:

```WebIDL
partial interface CredentialsContainer {
 Promise<DigitalCredential> requestIdentity(IdentityRequestOptions options);
};
```

## The `IdentityRequestOptions` dictionary

The `IdentityRequestOptions` dictionary would be specifically designed to handle requests in a "protocol" agnostic manner, while also handling cryptographic requirements. We envision a "protocol" member serving to identify the standard being used to make the request (e.g.,"OpenID4VP", "mDoc", etc). The request itself would likely just be a string, and the dictionary would need some means to pass along a public key.

The above expresses as WebIDL would be:

```WebIDL
dictionary IdentityRequestOptions {
 AbortSignal signal; // tears down the UI, if needed.
 required sequence<IdentityRequestProvider> providers;
};

dictionary IdentityRequestProvider {
 required DOMString protocol; // See "Protocol Registry" below
 required DOMString request;
 required DOMString publicKey;
 // Other members
};
```

What other members would be included in both dictionaries is up for discussion (e.g., nonce, reason for request, etc.).

## The `DigitalCredential` interface

An "digital credential" is the result from the promise resolving as a result of calling the .requestIdentity() method. We envision this interface exposing a read-only `.response` attribute, which would contain the encrypted response for the corresponding request.

Our expectation is that all responses are encrypted by default.

```WebIDL
[Exposed=Window, SecureContext]
interface DigitalCredential {
  [SameObject] readonly attribute ArrayBuffer response;
};
```

## Registry of credential identity protocols

This would be a simple registry to help identify which "protocol" is being used to make a request (i.e. a string identifying the standard, and its request format, which the browser or OS can then use to locate the requested identity credential).

Following in the tradition of the Web, we hope to make this registry open and developer friendly. For instance, "OpenID for Verifiable Presentations" might be identified as "`OpenID4VP`" and perhaps "ISO/IEC 18013-5:2021" could simply be identified as "mDL" (mobile drivers license), and so on.

The registry would likely be a W3C “[_Registry Track_](https://www.w3.org/2023/Process-20230612/#registries)" document so the Web community can make sure that, in order to be included, a standard meets privacy, security, and usability expectations.
