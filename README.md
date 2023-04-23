# Identity Credential

## Authors

- Sam Goto
- Adam Langley


## Introduction

This document contains a specification of `IdentityCredential`—a credential type in the [Credential Management](https://www.w3.org/TR/credential-management-1/) framework for all types of identity information. Types of identity credentials would extend `IdentityCredential` in the same way that different methods extend [Credential Management](https://www.w3.org/TR/credential-management-1/) today.

<img src="./structure.svg" alt="Diagram of API structure" width="300px">

`IdentityCredential` is being [lifted out of](https://www.w3.org/2023/04/21-webauthn-minutes.html) Federated Credential Management (FedCM). FedCM would be one user of `IdentityCredential` but user agents would not need to implement FedCM in order to support other types of identity credentials. 

`IdentityCredential` supports multiple identity types because we envision that sites may want to support multiple identity types in a single request. For example, student affiliations are often asserted via [SAML](https://www.oasis-open.org/committees/download.php/56776/sstc-saml-core-errata-2.0-wd-07.pdf) today, but student IDs are often kept in wallet apps and so could be mdocs in the future. A library that needs users to prove their student status may need to support both mdocs and federation in order to support students from multiple institutions. 

Requests for identity are different from requests for authentication. The former is (potentially) a much more significant operation because it may disclose an irrevocable, cross-site identity. Because of this we considered building a `navigator.identity` namespace but ultimately decided to use [Credential Management](https://www.w3.org/TR/credential-management-1/), and thus `navigator.credentials`. This was because FedCM already uses <kbd>navigator.credentials.get({_identity_: …})</kbd> and thus the term `identity` would appear twice in the Web Platform if `navigator.identity` was added. But the use of `identity` for `IdentityCredential` was not a mistake: federation, and thus FedCM, provides identity information and over time we would like authentication to be handled by [`PublicKeyCredential`](https://www.w3.org/TR/webauthn-2/#iface-pkcredential) (e.g. &ldquo;passkeys&rdquo;). However, today, FedCM is used for authentication too and so it lives in `navigator.credentials`.

Thus we want to support identity requests that can be satisfied by either mdocs or federation, and authentication requests that can be satisfied by either passwords, passkeys, or federation. This implies that all those mechanisms end up in `navigator.credentials` and that has informed this design.

All identity requests are grouped together, rather than made top-level types in Credential Management, to communicate that they are separate from the authentication mechanisms. User agents that implement FedCM may allow federation requests to be combined with other authentication types due to the current reality that federation is often used for sign-in. But that's an exception and, in general, user agents that receive a request that combines authentication and identity mechanisms, e.g. a request for either an mdoc or a passkey, SHOULD immediately reject such requests with a `NotSupportedError` exception.

## The IdentityCredential Interface

`IdentityCredential` is a new type of [Credential](https://w3c.github.io/webappsec-credential-management/#credential):

```webidl
[Exposed=Window, SecureContext]
interface IdentityCredential : Credential {
  readonly attribute USVString scheme;
  readonly attribute USVString? response;
};
```

The `scheme` specifies the type of identity credential returned and schemes will be defined in other, scheme-specific documents. The `response` is a value with semantics defined by that scheme. If this structure is insufficient, schemes may subclass `IdentityCredential` to add more members.

(This differs from what's specified in FedCM. The intent is that FedCM will adjust to conform.)

[CredentialRequestOptions](https://w3c.github.io/webappsec-credential-management/#dictdef-credentialrequestoptions) is extended with a new member for identity requests:

```webidl
partial dictionary CredentialRequestOptions {
  IdentityCredentialRequestOptions identity;
};
```

`IdentityCredentialRequestOptions` mirrors the pattern of Credential Management and is extended by scheme-specific documents:

```webidl
dictionary IdentityCredentialRequestOptions {
}
```

As an illustrative example, the defintion of an identity scheme would add members to that dictionary:

```webidl
partial dictionary IdentityCredentialRequestOptions {
  sequence<IllustrativeIdentitySchemeRequestOptions> illustrativeSchemeProviders;
}
```

The member need not be a sequence depending on the structure of the identity scheme in question.

When [DiscoverFromExternalSource](https://w3c.github.io/webappsec-credential-management/#dom-credential-discoverfromexternalsource-slot) is invoked, the `DiscoverFromExternalSource` function of each of the different identity schemes specified in the `IdentityCredentialRequestOptions` is run in parallel. If any throws an error that that error is the result of `IdentityCredential`'s DiscoverFromExternalSource. If one or more return an `IdentityCredential` then the user agents picks one of the values to the result, as its discretion. Otherwise the result is `null`.
