# Digital Identities

- [Digital identities explainer](explainer.md)
- [Digital identities draft spec](https://wicg.github.io/digital-identities/)

This incubation is attempting to specify an API for user agents that would mediate access to, and representation of, verifiably-issued digital identities.
These identities can range from government-issued documents, such as driver's licenses and passports, to start with, to other credential types, potentially in the future.

The [Digital Identities API](https://wicg.github.io/digital-identities/) builds upon the [Credential Management standard](https://www.w3.org/TR/credential-management-1/) to enable the secure and private exchange of digital identity information. It facilitates authenticated interactions by representing digital identities through the `DigitalIdentity` interface, which embodies verifiable claims about an individual's identity.

## Scope

This project covers the requesting mechanisms for digital identities, including secure presentation aspects. It does not yet encompass the issuance process for establishing a digital identity nor UI/UX considerations beyond privacy aspects related to data protection during the request process.

## Trying out the API

**🚧 Note: The API is still extremely unstable and undergoing a lot of changes (almost daily!). 🚧**

However, if you want to try it out:

- [Try out the API in Chrome](https://github.com/WICG/digital-identities/wiki/HOWTO%3A-Try-the-Prototype-API-in-Chrome-Android)

## Contributing

This is an unofficial proposal under development. Contributions, feedback, and discussions are highly encouraged to refine and enhance the API specification.
[Join the WICG](https://www.w3.org/community/wicg/) to help us in shape the future of digital identity management on the web.

### Weekly Meetings

* Slack ["wicg-identity-cred"](https://w3ccommunity.slack.com/archives/C05UG0EJUDB) in W3C Community instance.
* [Meeting notes](https://github.com/WICG/identity-credential/wiki/Meeting-Notes)
* 📆 [ICS file (calendar items)](https://drive.google.com/file/d/15MOQmqSA8PHIv7mT86Md37XOuaCH7GUM/view?usp=sharing) (24DST)
* [Agenda doc](https://docs.google.com/document/d/1Sq9tjh4Hv887Mzjoor-ZauXJ1glq6MCdjTsyUYNHjWA/)

## Initial proposals

The initial proposals have been moved to the proposals folder. They are still available as historical references.
