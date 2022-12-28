---
title: "Overview of JWTs"
description: "This article gives an overview of a JWT, and goes over the structure of a JWT"
date: "2022-12-26T14:38:00.987Z"
categories: ["Tech"]
tags: ["Web Security"]
published: true
---

This article gives an overview of a JWT, and goes over the structure of a JWT.

## Background

Traditionally, when a user logs into an application, the server generates a cookie and sends it back to the client. This cookie will contain information about the user, so that the server will "remember" the user when he/she logs in next time. The client/browser will store this cookie locally and send this cookie back to the server to continue the session/tell the server who the user is. This is a stateful way of storing a user's information, as the cookie is stored both on the client as well as the server, to remember the user's session.
> Cookies also provide additional features, like scoping to a particular domain (In the case of Google, `google.com` or `maps.google.com`). More information about these features can be found [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies).

With the advocation of APIs nowadays, there are caveats to this approach. APIs are used for broader interation between different services/applications. For example, your account on `amazon.in` might use an API endpoint provided and maintained by Google to retrieve credentials, to authenticate to Amazon using your Google account. This is just a simple example, but APIs only require one to provide authentication information to get a response back (Like your username and password from Google in this case), and they inherently don't store any information, unlike how cookies work.

> APIs are a broad topic that is beyond the scope of this article. I will cover this subject in a later article.

## What are JWTs?

JWT stands for JSON Web Tokens, and were implemented to make it easier for multiple systems to interact with each other. A JWT is a self-contained string that gives information about the user (called claims) that can be used to authenticate a user or request.

### Structure of a JWT

A typical JWT looks like this(taken from [jwt.io](https://jwt.io)):

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

A JWT uses the `$header.$payload.$signature` format, where:

1. The `$header` is a base64url encoded string, that gives information about the algorithm used to sign the `$header` and `$payload`.
2. The `$payload` is a base64url encoded string, that gives information about the user, and other data used for authentication. These are called `claims`.
3. The `$signature` is a base64url encoded string, that gives the signature data. This signature is used to verify whether:
    1. The header+payload was modified during transit. If these values were modified, we will have a totally different hash that results from this new data.

        > Let's look at an example of this. On [jwt.io](jwt.io), try changing the value of the payload. The default payload that is shown is seen below:
        > ```json
        >    {
        >        "sub": "1234567890",
        >        "name": "John Doe",
        >        "iat": 1516239022
        >    }
        > ```
        >
        > The JWT looks like this: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`
        >
        > Let's change this payload to a malicious one.
        > ```json
        >    {
        >        "sub": "1234567890",
        >        "name": "Administrator",
        >        "iat": 1516239022
        >    }
        > ```
        >
        > The JWT becomes `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkFkbWluaXN0cmF0b3IiLCJpYXQiOjE1MTYyMzkwMjJ9.tSvs4a5wYtDaz4Coyj-Qg1tAC7XzTDu2a24d8uyegxc`.
        >
        > As you can see, the signature section(`tSvs4a5wYtDaz4Coyj-Qg1tAC7XzTDu2a24d8uyegxc`) is completely different from before(`SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`).
    2. In the case of asymmetric algorithms that use a private key, we can also verify that the data was signed by the actual owner, since the private key stays hidden!

The client sends this long string in the HTTP request as part of the `Authorization: ` header. An example:

```
GET /my-account HTTP/1.1
Host: domain.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```
The server will receive this request containing the token, and use the secret key to verify the signature in the request. If the verification fails, the request must be denied.

### Secret key

The secret key is a symmetric key that is used for both signing and verification, however, there are asymmetric algorithms that make use of 2 keys for this signing process like RS256. For example: if the algorithm specified was HS256(HMAC SHA-256), the signature is generated using this formula:

```
HS256(
    base64UrlEncode($header),
    base64UrlEncode($payload),
    secret
)
```
Likewise, for RSA:
```
RS256(
    base64UrlEncode($header),
    base64UrlEncode($payload),
    secret
)
```
> Both symmetric and asymmetric algorithms are used for different use-cases. Symmetric algorithms are good for performance reasons, but asymmetric algorithms give you better security (provided all configurations are in tact) since they use a public/private key pair. You must look at your application's use-case before deciding on the algorithm to use.

**A word of caution**: Make sure this secret key is stored securely, like a password. If this key is compromised, an attacker can sign a malicious payload using this key to create a valid token that will bypass existing security controls on the server. For example, an attacker can create a new token such as this one:
>
```json
{
    "sub": "1234567890",
    "name": "Administrator",
    "iat": 1516239022
}
```

## Inspecting a JWT

1. You can use [jwt.io](https://jwt.io) to debug a token (easiest way)
2. You can also decode this token locally using the `basenc` package in *nix systems. Although, you might get errors using this method that  requires additional tweaking to work according to our needs:

    ```sh
    echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ" | basenc --base64url -d
    ```

In a separate article, I will go over possible attack techniques that can be leveraged against misconfigured JWTs.

## References

1. [PortSwigger Web Academy](https://portswigger.net/web-security/jwt/)