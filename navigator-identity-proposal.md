This is an alternative design that optimizes for decoupling identity verification and authentication further (so that they have more autonomy to 
evolve independently) while still accomplishing the goal of being able to request them at the same time in the same dialog.

In this proposal, we create a new namespace for identity verification, say `navigator.identity`, as opposed to integrate with the existing 
authentication-oriented [Credential Manager](https://www.w3.org/TR/credential-management-1/) being used by WebAuthn, WebOTP and FedCM.

```javascript
const result = await navigator.identity.get({
  selector: {
    format: ["mdoc"],
    doctype: "org.iso.18013.5.mDL",
    fields: ["org.iso.18013.5.1.family_name"]
  },
  params: {
    nonce: "1234",
    readerPublicKey: "567"
  },
});
``` 

>  NOTE: should / how would we factor in here federated identities?

In this variation, there is no API affordance for a developer to make the mistake of requesting identity verification and authentication mechanisms 
at the same time.
