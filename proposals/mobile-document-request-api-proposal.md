# ⚠️ This proposal has been superseded ⚠️

- Please refer to the [Digital identities spec](https://wicg.github.io/digital-identities/) instead. It represents the community's consensus.

## Mobile Document Request API

## Authors

- [@dcrousso](https://github.com/dcrousso)
- [@hlozi](https://github.com/hlozi)
- [@martijnharing](https://github.com/martijnharing)


## Participate

- [Issue tracker](https://github.com/WICG/mobile-document-request-api/issues)


## Introduction

This is a proposal for a new web API that allows websites to request a mobile document as defined in ISO/IEC 18013-5 using structures defined in the same ISO in order to achieve an online presentment. The API would protect the mobile document credential data, supports selective disclosure, and allow the user to control the release of credential data.

The ISO/IEC 18013-5:2021 standard specifies how a driver license (mDL) is represented on a mobile device and how it is presented to a physical reader.  Users who chose to have their mDL on device can present it to compatible readers instead of the physical driver license.  The ISO standard defines the protocols and the exchanges between the mDL and the mDL reader for different physical uses case (e.g. TSA, Hotel, store,... ).  The ISO standard provides building blocks to allow remote presentment of the the mobile driver’s license over the Web such as data structures and operational phases.  The ISO standard ISO/IEC 18013-5:2021 for mobile driver’s license does not define a web API to allow presentement via user agent.

The mechanisms defined by ISO/IEC 18013-5 are designed to support any type of mobile document, called mdoc. Besides the mDL, this API can therefore be used to request and transmit any mobile document that implements the ISO mdoc specification.



## Goals

- Allow websites to request the presentment of a mobile document in mdoc format such as driver’s license. 
- Enable the user to have a consent based control of what data is released to the website. 
- Protect the released data using encryption to ensure only the website the data is intended for can access the data.


## Non-goals

- Define issuance and provisioning of an mdoc.
- Define the communication between the User Agent and the website server is out of scope. 
- Define the native communication between the User Agent and the application holding the mdoc.
- Define how the response to the request is used by the server is out of scope. 
- Requesting multiple mdocs in one API call.


## Proposed Solution

The API request should contain all the information needed to build up and encrypt the response.  The response data from the mdoc should be a CBOR-encoded blob encrypted end-to-end for the intended relying party using Hybrid Public Key Encryption (HPKE) defined in RFC 9180.  Only the targeted relying party, holder of the private part corresponding to the public key used for the response encryption, should be able to decrypt the response (with the assumption that the server always holds and protects the  private key).  The API should allow the server to pass as a parameter a requester identity object that allows the mdoc device to extract the public key material needed for the encryption.  The passed object can be anything that allows the mdoc to determine which public key to encrypt the response to, like a certificate, a key or a key identifier.


### Request

The website would need to provide a document type and desired data elements, as well as a requester identity object that allows the device to extract the encryption public key (and the mdoc to optionally verify the eligibility of the website to request the mdoc presentment).

Once mdoc application verifies the server identity object to make sure it is a legitimate request (if needed), it would build up the response according to ISO/IEC 18013-5, including the credential data itself, the authentication by the issuer, and the authentication by the device, encrypting everything with the key derived from the requester identity object using HKPE so that only the server can decrypt the binary blob (assuming the server keeps and protects the private key).

After sending the encrypted data to the server, it would be decrypted to verify the presented mdoc issuer signature and device signature according to ISO/IEC 18013-5.


### Response

The response from the server would be the `DeviceResponse` object as defined in ISO/IEC 18013-5 clause 8.3.2.1.2.2 encoded with CBOR according to ISO/IEC 18013-5, encrypted using HPKE with the following settings:
```
mode: mode_base
KEM: DHKEM(P-256, HKDF-SHA256)
KDF: HKDF-SHA256
AEAD: AES-128-GCM
```

Part of the signature generated for `DeviceAuth` is done using the `SessionTranscript` structure as defined in ISO/IEC 18103-5 section 9.1.5.1.  The `DeviceEngagementBytes` and the `EReaderKeyBytes` are both not present, and therefore have a value of `null`.  The `Handover` element shall be as defined below:
```
BrowserHandover = [
    "BrowserHandoverv1",
    Nonce,
    OriginInfoBytes,
    RequesterIdentity,
    pkEm
]
```
where `Nonce` is the nonce provided by the request, `OriginInfoBytes` contains the origin of the request and is defined in ISO/IEC 18013-7, and `RequesterIdentity` is the requester identity object from the request.

This is all included inside a CBOR encoded `CredentialDocument` with the following structure:
```
CredentialDocument = {
    "version": tstr,
    "encryptionParamaters": EncryptionParamaters,
    "data": bstr
}

EncryptionParamaters = {
    "version": tstr,
    "EDeviceKey": bstr,
    "originInfoBytes": OriginInfoBytes
}
```


### API

One idea a new `CredentialRequest` (et al)
```
dictionary CredentialElement {
    required DOMString namespace;  // As defined in ISO 18013-5 clause 8.
    required DOMString name;
};

dictionary CredentialStorageDuration {
    // At least one of these is required.

    boolean forever;  // Cannot be used with any other properties.

    long days;  // Cannot (currently) be used with any other properties.
};

dictionary CredentialDocumentDescriptor {
    required DOMString documentType;  // As defined in ISO 18013-5 clause 8.

    required sequence<CredentialElement> requestedElements;

    CredentialStorageDuration desiredStorageDuration;  // Not providing this is equivalent to not asking to store.
};

dictionary CredentialDocument {
    object data;  // The CBOR encoded `CredentialDocument` defined above.
};

dictionary RequestConfiguration {
    required DOMString nonce;
};

[
    SecureContext,
    Exposed=Window,
] interface CredentialRequest {
    constructor(DOMString requesterIdentity, CredentialDocumentDescriptor documentDescriptor);  // This throws if anything in the `documentDescriptor` is not recognized (e.g. an invalid `documentType`).

    Promise<CredentialDocument> requestDocument(RequestConfiguration configuration);

    Promise<undefined> abort();
};
```


## Examples

```js
// Driver's License
let mDLCredentialRequest = new CredentialRequest(certificate, {
    documentType: "org.iso.18013.5.1.mDL",
    requestedElements: [
        { namespace: "org.iso.18013.5.1", name: "document_number" },
        { namespace: "org.iso.18013.5.1", name: "portrait" },
        { namespace: "org.iso.18013.5.1", name: "driving_privileges" },
        { namespace: "org.iso.18013.5.1.aamva", name: "organ_donor" },
        { namespace: "org.iso.18013.5.1.aamva", name: "hazmat_endorsement_expiration_date" },
    ],
    desiredStorageDuration: {
        days: 7,
    },
});
mDLCredentialRequest.request({ nonce }).then((credentialDocument) => { ... });
```

```js
// Vaccination Card
let micovCredentialRequest = new CredentialRequest(certificate, {
    documentType: "org.micov.1",
    requestedElements: [
        { namespace: "org.micov.attestation.1", name: "PersonId_dl" },
        { namespace: "org.micov.attestation.1", name: "portrait" },
    ],
});
micovCredentialRequest.request({ nonce }).then((credentialDocument) => { ... });
```


## Privacy & Security Considerations

This API provides the ability for websites to retrieve an ISO 18013-5 mDoc already on the device.  An important part of the security and privacy mechanisms are provided by ISO 18013-5:
- Protection against forgery with passive data authentication defined in ISO 18013-5 based on Mobile Security Object (MSO).
- Protection against cloning using active authentication leveraging a signature over session data.  The signature is produced by the device using the device private key as defined in ISO 18013-5.  The Device Public key is part of the MSO signed by the issuer.
- The ability for the user to see the requested data before disclosure, in addition to being able to choose what to disclose.
- The ability for the website to request a subset of data, rather than any sort of all-or-nothing.
- Protection against eavesdropping during transactions by using HPKE to encrypt the response containing the mDoc to the requesting server.
- Man in the middle attacks are detectable by the requesting server via validation of the device signature over session data.


## References & acknowledgements

Many thanks for valuable feedback and advice from:

- [@15characterlimi](https://github.com/15characterlimi)
- [@agl](https://github.com/agl)
- [@balfanz](https://github.com/balfanz)
- [@bslassey](https://github.com/bslassey)
- [@cwilso](https://github.com/cwilso)
- [@davidz25](https://github.com/davidz25)
- [@divegeek](https://github.com/divegeek)
- [@hober](https://github.com/hober)
- [@jyasskin](https://github.com/jyasskin)
- [@samuelgoto](https://github.com/samuelgoto)
- [@sethmoo](https://github.com/sethmoo)
