--- 
layout: "post"
title:  "How to Integrate Yes Identity with Your App"
date:   "2021-07-12 13:30:00 -0500"
categories: openid, oauth, ciba, identity, mfa, passwordless
---
This post describes how to implement Multi-factor authentication using [Yes Identity](https://yesidentity.com){:
target="_blank"}, a strong authentication solution that I have developed.

# Background

Yes Identity enables you to easily and quickly implement multi-factor, passwordless authentication and transaction
signing in your application using biometrics or security codes. Unlike most solutions developed in the wild, it is based
on [OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html){:target="_blank"}
, [OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749){:target="_blank"}
and [CIBA](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html){:target="_blank"}
open standards. Unlike many forms of MFA such one-time codes sent via SMS or voice calls (which are transmitted in
cleartext and prone to attacks), it is based
on [Public key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography){:target="_blank"} system.

For those familiar with [BankID](https://www.bankid.com/){:target="_blank"}, YI is essentially your own private
authentication embedded in your mobile app for seamless user experience.

Once a user has been authenticated, either using email/password or even BankID, cryptographic keys (protected by
biometric authentication) are generated and used to authorize requests. Private user data never leaves your servers and
sensitive information, such as users' biometrics never leaves their device enabling GDPR compliance.

The solution is low-cost, easy to deploy and requires no schema changes in your backend. It is also API-based and gives
you the flexibility to implement any kind of request, even for things like email or password change.

The following are examples of real world use cases:

* A **customer service agent** wants to authenticate a caller. Current common practice is that the user has to answer
  some security questions on a call. With this solution, the agent can initiate an `Authentication request` for the user
  to approve on their smartphone.

* Users can **authorize a payment request** for a purchase initiated by a merchant or at point of sale terminal. The
  payment request information are sent alongside the request to the user's device for authorization. Current common
  practice is that an OTP(via SMS) is sent to the user's phone number connected to their bank/card.

* Enable **two-factor authentication** to confirm the user's identity using a combination of their username/password,
  and their device before they can gain access to your app or digital resource.

* Enable **three-factor authentication** to confirm the user's identity using a combination of username/password, their
  device and their biometrics (e.g., fingerprint) before they can gain access to your app or digital resource.

* You can eliminate passwords altogether by implementing **Passwordless authentication** and there
  are [valid reasons](https://www.freecodecamp.org/news/360-million-reasons-to-destroy-all-passwords-9a100b2b5001/){:target="_blank"}
  to. The user simply scans a QR code with their device, then authorizes the `Login request` typically using biometric
  authentication.

* You can implement **electronic signing** where the user needs to authorize the creation of signatures for one or more
  documents. The text to be signed or urls to the documents can be included in a `Signature request` which the user then
  authorize or deny.

# Implementation

Let's implement Multi-factor authentication using YI :)

The demo application below is hosted at [https://demo.yesidentity.com/](https://demo.yesidentity.com/){:target="_blank"}
and is a Spring Boot and Angular application using Jwt Authentication.

![Multi-factor authentication demo](/img/demo.gif)

Pre mutli-factor authentication, you would typically provide your credentials when authenticating and that would be
matched against some hashed value in the database to check for validity. If the credentials are valid, a token is
returned to the client and are passed with subsequent request to access to protected resources. The kind of
authentication mechanism is called stateless and this tyoe if authentication is called single-fator authentication. No
matter what kind of authentication mechanisms you use, the idea is the same, something you know is used to verify your
identity before you can access protected resources.


