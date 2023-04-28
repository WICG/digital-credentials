
# Identity Credential

## Authors

- Sam Goto
- Adam Langley

## Introduction

This document contains a specification of `IdentityCredential`—a credential type in the [Credential Management](https://www.w3.org/TR/credential-management-1/) framework for all types of identity verification. Types of identity credentials would extend `IdentityCredential` in the same way that different authentication methods extend [Credential Management](https://www.w3.org/TR/credential-management-1/) today.

<img src="./structure.svg" alt="Diagram of API structure" width="300px">

In this proposal, we lift `IdentityCredential` out of [Federated Credential Management](https://fedidcg.github.io/FedCM/) (FedCM). FedCM would be one extension of `IdentityCredential` but user agents would not need to implement FedCM in order to support other types of identity credentials.

This proposal is informed by a few design points:

1. identity credentials are going to come in multiple formats, determined in part by legislation and non-web standards
1. some browsers may choose to be largely agnostic to the credential format, acting mostly as a conduit for the underlying platform or wallet applications
1. identity verification is not authentication

First, `IdentityCredential` supports multiple identity verification schemes because we envision that sites may want to support multiple identity types in a single identity credential picker while retaining some influence over the order in which they are presented to the user.

For example, student university affiliations are often asserted via [SAML](https://www.oasis-open.org/committees/download.php/56776/sstc-saml-core-errata-2.0-wd-07.pdf) today, but student IDs are going to be increasingly made available in wallet apps ([example](https://www.purdue.edu/newsroom/releases/2023/Q2/purdue-launches-purdue-mobile-id-for-students-allowing-them-to-get-around-campus-with-just-a-simple-tap-of-their-mobile-device.html)) and so could be verifiable credentials (e.g. [mdocs](https://www.ul.com/resources/new-isoiec-standard-electronic-credentials) or [W3C VCs](https://www.w3.org/TR/vc-data-model/)) in the future. An online library (e.g. nature.com) that needs users to prove their student affiliations may need to support both mdocs and federation in order to support students from multiple institutions.

Second, we think it is key to make requests for identity verification different from requests for authentication. The former is (potentially) a much more significant operation because it may disclose an irrevocable, cross-site identity. Because of this we considered building a `navigator.identity` namespace but ultimately decided to use [Credential Management](https://www.w3.org/TR/credential-management-1/), and thus `navigator.credentials`. This was because FedCM already uses `navigator.credentials.get({identity: …})` and thus the term `identity` would appear twice in the Web Platform if `navigator.identity` was added. But the use of `identity` for `IdentityCredential` was not a mistake: federation, and thus FedCM, provides identity verification and over time we would like authentication to be handled by [`PublicKeyCredential`](https://www.w3.org/TR/webauthn-2/#iface-pkcredential) (e.g. &ldquo;passkeys&rdquo;). However, today, FedCM is used for authentication too and so it lives in `navigator.credentials`. 

Thus we want to support identity requests that can be satisfied by either mdocs or federation, and authentication requests that can be satisfied by either passwords, passkeys, or federation. This implies that all those mechanisms end up in `navigator.credentials` and that has informed this design.

All identity requests are grouped together, rather than made top-level types in Credential Management, to communicate that they are separate from the authentication mechanisms. In general, user agents that receive a request that combines authentication and identity verification, e.g. a request for either an mdoc or a passkey, SHOULD immediately reject such requests with a `NotSupportedError` exception.

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

`IdentityCredentialRequestOptions` mirrors the pattern of Credential Management and is extended with a list of identity verification providers:

```webidl
dictionary IdentityCredentialRequestOptions {
  // Common request fields, e.g. the context API.

  sequence<IdentityProviderConfig> providers;
}
```

`IdentityProviderConfig` represents a scheme-specific source of identity credentials, and it expected to be extended by other specifications (e.g. the federated or mdocs scheme).

```
dictionary IdentityProviderConfig {
  // More common fields across all identity provider types:
  // ...
};
```

As an illustrative example, the definition of an identity scheme would add members to that dictionary:

```webidl
partial dictionary IdentityProviderConfig {
  MdocIdentityProviderConfig mdoc;
}
```

When [DiscoverFromExternalSource](https://w3c.github.io/webappsec-credential-management/#dom-credential-discoverfromexternalsource-slot) is invoked, the `DiscoverFromExternalSource` function of each of the different identity providers  specified in the `IdentityCredentialRequestOptions.providers` is run in parallel. If any throws an error then that error is the result of `IdentityCredential`'s DiscoverFromExternalSource. If one or more return an `IdentityCredential` then the user agent picks one of the values as the result, as its discretion. Otherwise the result is `null`.

## Examples

These are examples of extensions to `IdentityCredential` that we would expect, and how / why they'd fit together.

- [mdocs](#mdocs)
- [FedCM](#fedcm)
- [W3C Verifiable Credentials](#w3c-verifiable-credentials)

### MDocs

The [MDocs API](https://github.com/agl/mobile-document-request-api/tree/identityapi#examples) extends the `IdentityCredential` API to allow mdocs to be requested:

```js
// Gets a CBOR with specific fields out of mobile driver's license as an mdoc
const {response} = await navigator.credentials.get({
  identity: {
    providers: [{
      mdoc: {
        nonce: "gf69kepV+m5tGxUIsFtLi6pwg=",
        readerPublicKey: "ftl+VEHPB17r2 ... Nioc9QZ7X/6w...",
        retentionDays: 90,
        documentType: "org.iso.18013.5.1.mDL",
        requestedElements: [
          { namespace: "org.iso.18013.5.1", name: "document_number" },
          { namespace: "org.iso.18013.5.1", name: "portrait" },
          { namespace: "org.iso.18013.5.1", name: "driving_privileges" },
          { namespace: "org.iso.18013.5.1.aamva", name: "organ_donor" },
        ],
      }
    }],
  }
});
```

### FedCM

The [FedCM](https://fedidcg.github.io/FedCM/) also extends the `IdentityCredential` API to provide a binding to [OpenID](https://openid.net/specs/openid-connect-core-1_0.html) and [SAML](https://seamlessaccess.org/posts/2023-02-20-fedcm/): 

```js
// Gets a JWT from a OIDC provider. 
const {response} = await navigator.credentials.get({
  identity: {
    providers: [{
      federated: {
        configURL: "https://university.edu/students",
        clientId: "123",
        nonce: "m5tGxUIsFtLi6pwg"
      }
    }]
  }
}
```

### W3C Verifiable Credentials

We also expect that we'd at some point extend `IdentityCredential` to provide a binding to [CHAPI](https://github.com/fedidcg/FedCM/issues/374) and [OIDC4VP](https://openid.net/specs/openid-connect-4-verifiable-presentations-1_0-07.html).

While this is still something that we are actively exploring with that community ([example](https://w3c-ccg.github.io/meetings/2023-01-24/) and early [proposal](https://github.com/fedidcg/FedCM/issues/374)), here is a possible example of what that could look like:

> It is worth noting how closely related this looks to the [mdocs](#mdocs) extension.

```js
// Gets a SD-JWT from a W3C VC holder.
const {response} = await navigator.credentials.get({
  identity: {
    providers: [{
      w3cvc: {
        format: {"vc+sd-jwt": { alg: ["EdDSA", "ES256"]}},
        inputDescriptors: [{
          constraints: {
            fields: [{
              path: [                  
                "$.credentialSubject.dateOfBirth",
                "$.credentialSubject.dob",
                "$.vc.credentialSubject.dateOfBirth",
                "$.vc.credentialSubject.dob",                     
              ]
            }]
          }
        }]
      },
    }]
  }
}
```

### Reconcilation

User agents can support multiple identity schemes, and they can be requested at the same time:

```js
// Requests a verifiable affilitation to a university, accepting mdocs,
// W3C VCs and SAML assertions.
const credential = await navigator.credentials.get({
  identity: {
    // Requests the user's university affiliation from either 
    // a W3C VC, an MDoc or a JWT.
    providers: [{
      // The university may have a device-bound certificate ...
      mdoc: {
        nonce: "1234",
        retentionDays: 90,
        documentType: "org.iso.18013.5.1.UniversityDegree",
        readerPublicKey: "...ftl+VEHpdNioc9QZ7X/6w...",
        requestedElements: [
          { namespace: "org.iso.18013.5.1", name: "affiliation" },
        ],
      }
    }, {
      // ... or a SAML assertion ...
      federated: {
        clientId: "myapp",
        nonce: "1234",
        configURL: "https://idp.university.edu",
      }
    }, {
      // ... or a Verifiable Credential ...
      w3cvc: {
        inputDescriptors: [{
          constraints: {
            type: "UniversityDegreeCredential",
            fields: [{
              path: [                  
                "$.credentialSubject.alumniOf",                 
              ]
            }]
          }
        }]
      },
    }]
  }
});

```


# Alternatives Considered

There are many alternatives that we are actively exploring and considering trade-offs. Here are a few:

- Keep the presentment of MDocs/VCs independent from the presentment of OpenID/SAML assertions ([example](https://github.com/WICG/mobile-document-request-api#examples)). This is a plausible alternative, but the early intuition is that there are more use cases where these coincide than not, and it will be useful for the verifier-holder-issuer schemes to piggy back on each other (e.g. requesting either VCs or MDocs at the same time, increasing the chances that one will be available) and/or be able to fallback on the deployment of the rp-idp schemes (e.g. federated flows).
- Move identity verification into its own namespace `navigator.identity.get()` as opposed to `navigator.credentials.get({identity: ...})`. That's also a plausible alternative, but (a) seemed duplicative of the `identity` namespace and (b) FedCM would have to co-exist in both places.
- Allow the specialization of identity verification schemes at the `IdentityCredentialRequestOptions` rather than the `IdentityCredentialRequestConfig` level. That's also a plausible alternative, but the intuition is that it would be useful to have a level that would allow us to have common things between identity verification schemes (for example, the [RP context API](https://github.com/fedidcg/FedCM/issues/456)). 
- Unify the query languages, have all of the identity verification schemes use the same query language (for example, [Presentation Exchange](https://identity.foundation/presentation-exchange/) or a variation of) in the `IdentityCredentialRequestConfig`. That's also plausible and desirable, but it seemed like a good idea to give each identity scheme the autonomy to explore their own design space and to unify once we find more convergence between them.

These are all valid options that we'd love feedback from the community as to what trade-offs to take.
