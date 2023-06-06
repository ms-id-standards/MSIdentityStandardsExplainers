# Passkey Endpoints Well-Known URL

Author: [Tim Cappalli](https://github.com/timcappalli) &lt;tim.cappalli@microsoft.com&gt;

## Status of this Document

This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the most current standards venue and content location of future work and discussions.

- This document status: **WIP DRAFT**
- Current venue: tbd
- Current version: early draft

## Introduction

> TO DO

## Terms

- **passkey(s)**:
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

| Member   | JSON Type | Required/Optional | Description                                                                                                            |
| :------- | :-------- | :---------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `enroll` | Object    | Optional          | an object defining URLs for web and apps which take the user directly to a passkey creation flow                       |
| `manage` | Object    | Optional          | an object defining URLs for web and apps which take the user directly to the passkey management page for their account |

`enroll` object:

| Member    | JSON Type | Format | Required/Optional | Description                                                                                   |
| :-------- | :-------- | :----- | :---------------- | --------------------------------------------------------------------------------------------- |
| `web`     | String    | URL    | Optional          | a URL that takes the user directly to the web-based passkey management page for their account |
| `android` | String    | URL    | Optional          | an Android specific URL that takes the user directly to a passkey creation flow in an app     |
| `ios`     | String    | URL    | Optional          | an iOS/iPadOS specific URL that takes the user directly to a passkey creation flow in an app  |
| `windows` | String    | URL    | Optional          | a Windows specific URL that takes the user directly to a passkey creation flow in an app      |
| `macos`   | String    | URL    | Optional          | a macOS specific URL that takes the user directly to a passkey creation flow in an app        |

`manage` object:

| Member    | JSON Type | Format | Required/Optional | Description                                                                                      |
| :-------- | :-------- | :----- | :---------------- | ------------------------------------------------------------------------------------------------ |
| `web`     | String    | URL    | Optional          | a URL that takes the user directly to the web-based passkey management page for their account    |
| `android` | String    | URL    | Optional          | an Android specific URL that takes the user directly to the passkey management page in an app    |
| `ios`     | String    | URL    | Optional          | an iOS/iPadOS specific URL that takes the user directly to the passkey management page in an app |
| `windows` | String    | URL    | Optional          | a Windows specific URL that takes the user directly to the passkey management page in an app     |
| `macos`   | String    | URL    | Optional          | a macOS specific URL that takes the user directly to the passkey management page in an app       |

### Example 1

For the Relying Party `https://example.com`, which has a website and native apps across all platforms, the well-known URL would be `https://example.com/.well-known/passkey-endpoints` with a response of:

```http
HTTP/1.1 200 OK
Content-Type: application/json
 {
    "enroll": {
        "web": "https://example.com/account/manage/passkeys/create",
        "android": "com.example.android.myapp://account/passkeys/create",
        "ios": "com.example.ios.myapp://account/passkeys/create",
        "windows": "myapp://account/passkeys/create",
        "macos": "myapp://account/passkeys/create"
    },
    "manage": {
        "web": "https://example.com/account/manage/passkeys",
        "android": "com.example.android.myapp://account/passkeys",
        "ios": "com.example.ios.myapp://account/passkeys",
        "windows": "myapp://account/passkeys",
        "macos": "myapp://account/passkeys"
    }
 }
```

### Example 2

For the replying party `https://example.org`, with only web and mobile apps, the well-known URL would be `https://example.org/.well-known/passkey-endpoints` with a response of:

```http
HTTP/1.1 200 OK
Content-Type: application/json
{
    "enroll": {
        "web": "https://example.org/myaccount/manage/passkeys/create",
        "android": "org.example.android.myapp://myaccount/passkeys/create",
        "ios": "org.example.ios.myapp://myaccount/passkeys/create"
    },
    "manage": {
        "web": "https://example.org/myaccount/manage/passkeys",
        "android": "org.example.android.myapp://myaccount/passkeys",
        "ios": "org.example.ios.myapp://myaccount/passkeys"
    }
}
```

### File Location

The `passkey-endpoints` file must be found at `[RP ID]/.well-known/passkey-endpoints` (without an extension), as defined in [RFC 5785](https://www.rfc-editor.org/rfc/rfc5785.html).

## Security Considerations

> To Do

## Privacy Considerations

> **To Do**
>
> - Require consent before background fetch

## Open Questions

- Should the FQDN defined in the metadata values be required to match the RP ID serving the metadata?
- Should we add an explicit delete passkey endpoint (vs just manage) that can actually perform a getAssertion to help the user more seamlessly delete the exact credential?
