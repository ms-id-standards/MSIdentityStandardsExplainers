# Passkey Endpoints Well-Known URL

Author: [Tim Cappalli](https://github.com/timcappalli) &lt;tim.cappalli@microsoft.com&gt;

## Status of this Document

This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the most current standards venue and content location of future work and discussions.

- This document status: draft.02
- Current venue: tbd
- Current version: early draft

## Introduction

Passkeys are a new phishing-resistant credential for the web, leveraging FIDO2 and WebAuthn and designed for mass-scale adoption by users. All major operating systems and browsers now support passkeys.

As more sites start offering passkeys, [passkey providers](#terms) (traditionally known as password managers) and FIDO2 clients (apps, browsers, OS) want to help users upgrade their accounts from passwords to passkeys. To do this, sites (known as [relying parties](#terms)) need a way to advertise that they support passkeys and provide additional metadata to support a seamless interactive upgrade experience.

## Terms

- **passkey(s)**: the user-friendly name for FIDO2/WebAuthn discoverable credentials
- **passkey provider**: native platform authenticators or third party solutions that provide credential management (such as "password managers")
- **Relying Party (RP)**: see the [WebAuthn specification](https://w3c.github.io/webauthn/#relying-party)

## Goals

1. Enable [Relying Party (RP)](#terms) developers to advertise support for [passkeys](#terms).
2. Enable RP developers to advertise a direct passkey enrollment URL to [passkey providers](#terms) and/or FIDO2/WebAuthn clients to provide a more streamlined "upgrade" experience for end users.
3. Enable RP developers to advertise a direct passkey management URL to passkey providers to make it easier for users to manage the passkeys that are tied to their RP accounts.

## Non-Goals

- Require RPs to implement this feature
- Require passkey providers to implement this feature

## Use Cases

### "Upgrading" from passwords to passkeys

Most users currently use a password as their primary credential for accessing sites and services on the web today and password manager usage has been steadily rising as more and more options have become available to users.

Passkey providers and/or clients may wish to display an indication to the user that a given website/service (RP) has support for passkeys and provide a way for the user to access certain account management flows directly.

For example, a passkey provider may wish to provide a visual indication next to a password-only entry in their management UI, that when clicked, opens the user's default browser and automatically takes the user directly to the RP's passkey enrollment page.

Another example would be a system notification (sometimes referred to as a "toast") notifying the user that the app/service/site supports passkeys. Tapping the notification could open the user's default browser and automatically takes the user directly to the RP's passkey enrollment page.

These experiences can help remove some of the friction that comes with such a large scale change to the web.

### Managing passkeys

A passkey consists of both a public and private key. The public key is stored by the server (RP) and linked to the user's account, and the private key is stored and managed by the passkey provider. Deleting a passkey from the passkey provider does not remove the passkey from the user's account.

A passkey provider may want to offer the user a direct link to the RP's account settings page where credentials, specifically passkeys, can be managed. A direct link is a much better user experience than having the user search around their account settings.

This use case is very similar to the existing ["Change Password URL"](https://www.w3.org/TR/change-password-url/) model that some password managers and websites have implemented.

## Proposed Solution

We propose defining a new [well-known URL](https://www.rfc-editor.org/rfc/rfc5785.html), hosted by a WebAuthn [Relying Party](#terms) who supports passkeys.

A successful response would return:

- Status Code: `200 OK`
- Content-Type: `application/json`
- a JSON object containing one or both members defined in the table below

| Member   | JSON Type | Format | Required/Optional | Description                                                                         |
| :------- | :-------- | :----- | :---------------- | ------------------------------------------------------------------------------------|
| `enroll` | String    | URL    | Optional          | a URL that takes the user directly to the passkey creation page for their account   |
| `manage` | String    | URL    | Optional          | a URL that takes the user directly to the passkey management page for their account |

### Example

For the Relying Party `https://example.com`, the well-known URL would be `https://example.com/.well-known/passkey-endpoints` with a response of:

```http
HTTP/1.1 200 OK
Content-Type: application/json
 {
    "enroll": "https://example.com/account/manage/passkeys/create",
    "manage": "https://example.com/account/manage/passkeys"
 }
```

### File Location

The `passkey-endpoints` file must be found at `[RP ID]/.well-known/passkey-endpoints` (without an extension), as defined in [RFC 5785](https://www.rfc-editor.org/rfc/rfc5785.html).
