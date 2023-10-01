This is an alternative design that optimizes for decoupling federation and wallets further (so that they have more autonomy to evolve independently) 
while still accomplishing the goal of being able to request them at the same time in the same dialog.

In this proposal, we create a new Credential Manager Credential type, say a WalletCredential, as opposed to integrate with the existing 
[IdentityCredential](https://fedidcg.github.io/FedCM/#browser-api-identity-credential-interface) type being used by FedCM.

> One of the unfortunate parts of this proposal is that FedCM is already built on top of IdentityCredential and the `identity` 
> namespace in the Credential Manager API. 
> I couldn't come up with a better name other than WalletCredential (HolderCredential, VerifiableCredential, DecentralizedCredential,
> DigitalIDCredential and RealWorldCredential occurred to me but none of them seem quite right to me), so if anyone has a name they 
> like best, feel free to make a suggestion. So, aside from the name of the credential, the point of this design alternative remains.

```javascript
const result = await navigator.credentials.get({
  wallet: {
    selector: {
      format: ["mdoc"],
      doctype: "org.iso.18013.5.mDL",
      fields: ["org.iso.18013.5.1.family_name"]
    },
    params: {
      nonce: "1234",
      readerPublicKey: "567"
    },
  }
});
```

Because the API is format agnostic, you'd be able to query VCs equally well:

```javascript
const result = await navigator.credentials.get({
  wallet: {
    format: ["vc+sd-jwt"],
    selector: [{
      doctype: "UniversityDegreeCredential",
      fields: [
        "credentialSubject.dateOfBirth",
        "credentialSubject.dob",
        "vc.credentialSubject.dateOfBirth",
        "vc.credentialSubject.dob",
      ]
    }],
    params: {
      nonce: "m5tGxUIsFtLi6pwg",
    }
  }
});
``` 

The user agent can choose to fail this request if more than "wallet" credentials are requested from the credential manager, as it normally would for 
other credential types (e.g. you can't request a `publicKey` credential along with an `identity` or `otp` or a `password` credential at the moment).

If, and ever, we wanted to support querying wallets and federated identity providers at the same time, this is what it could look like:

```javascript
const result = await navigator.credentials.get({
  wallet: {
    selector: {
      format: ["mdoc"],
      doctype: "org.iso.18013.5.mDL",
      fields: ["org.iso.18013.5.1.family_name"]
    },
    params: {
      nonce: "1234",
      readerPublicKey: "567"
    },
  },
  identity: {
    configURL: "https://accounts.google.com/gsi/fedcm.json",
    clientId: "212342",
    nonce: "234231",
  }
});
``` 

