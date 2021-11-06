--- 
layout: "post"
title:  "How to Implement Multi-Factor Authentication Using YES IDENTITY"
date:   "2021-07-12 13:30:00 -0500"
categories: openid, oauth, ciba, identity, mfa, passwordless
---
This post describes how to implement Multi-factor authentication using [YES IDENTITY](https://yesidentity.com){:target="_blank"
}.

The [Demo application](https://demo.yesidentity.com){:target="_blank"} is a Java and Angular application with a
companion Android app. After logging in with username and password, users receive an authentication prompt when they
open their app, then they either approve (and authenticate using biometrics) or deny the request. It can be customized
so that only users with certain roles or who have enabled MFA receives the prompt.

<figure>
  <img src="{{site.url}}/img/demo.gif" alt="Demo App"/>
</figure>

In future posts, I'd describe how to implement passwordless authentication and transaction signing using the solution.

# Introduction

YES IDENTITY enables you to easily and quickly implement multi-factor, passwordless authentication and transaction signing in
your application using biometrics. Unlike similar solutions relying on closed protocols and implementations, YES IDENTITY is
based on the [OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749){:target="_blank"}
, [OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html){:target="_blank"}
and [CIBA](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html){:target="_blank"}
open standards. Unlike many forms of MFA such one-time codes (via SMS) or voice calls (which are transmitted in
cleartext and prone to attacks), it is based
on [Public key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography){:target="_blank"} and biometrics (
currently considered the most bulletproof form of MFA).

For those familiar with [BankID](https://www.bankid.com/){:target="_blank"}, YES IDENTITY is essentially your own private
authentication embedded in your mobile app for seamless user experience. The solution is API-based so you have complete
flexibility on how you design the authentication prompt in your app or the kind of requests you prompt users to
authorize. A major drawback with using BankID (as a form of MFA) is its reliance on Swedish social security numbers (
which could be seen as discriminatory) which means it cannot be deployed globally. Also, you need two apps instead of one!

During user enrollment, cryptographic keys (optionally locked by biometrics) are generated and used to authorize
requests. Private user data never leaves your servers and sensitive information, such as users' biometrics never leaves
their device enabling GDPR compliance.

The solution is low-cost, easy to deploy and requires no schema changes in your backend.

The following are examples of real world use cases:

* A customer service agent wants to **authenticate a customer on a call**. Current common practice is that the customer
  has to answer some security questions such as date of birth, account balance etc. which
  are [notoriously vulnerable to social engineering attacks](https://www.theguardian.com/money/2004/sep/16/yourfeedback1){:target="_blank"}
  . With this solution, the agent can initiate an `Authentication request` for the customer to approve on their
  smartphone.

* Users can **authorize a payment request** for a purchase initiated by a merchant or at point of sale terminal. The
  payment request information (amount, currency, message etc.) are retrieved by the user's device for authorization.
  Current common practice is that an OTP is sent (via SMS) to the user's phone number connected to their bank (also very
  susceptible to social engineering attacks).

* Implement PSD2 compliant **Open Banking** services by securely allowing customers to give consent for Third Party
  Providers (TTP) to access their financial data.

* Enable **two-factor authentication (2FA)** to confirm the user's identity using a combination of their
  username/password, and their device before they can gain access to your app.

* Enable **three-factor authentication (3FA)** to confirm the user's identity using a combination of username/password,
  their device and their biometrics (e.g., fingerprint) before they can gain access to your app.

* You can eliminate passwords altogether by implementing **Passwordless authentication** and there
  are [valid reasons](https://www.freecodecamp.org/news/360-million-reasons-to-destroy-all-passwords-9a100b2b5001/){:target="_blank"}
  to. The user simply scans a QR code with their device, then authorizes the `Login request` typically using biometric
  authentication.

* Secure your **Virtual Private Network (VPN)** with multi-factor authentication.

* You can implement **electronic signing** where the user needs to authorize the creation of signatures for one or more
  documents. The text to be signed or urls to the documents can be included in a `Signature request` which the user then
  authorize or deny.

and many more...

# Design

Pre mutli-factor authentication, a user would provide their credentials when authenticating and that would be matched
against some hashed value in the database. If they match, a bearer token is returned to the client which is passed in
the Authorization header of subsequent requests to protected resources. This authentication mechanism is called
stateless or sessionless (as it does not keep track of user sessions on the server) and this type of authentication is
called **single-factor authentication**. No matter what kind of authentication mechanisms you use, the idea is the same,
something (secret) the user knows is used to verify their identity before they can access protected resources.

With multi-factor authentication, additional factors are required to verify user's identity, i.e something the user
has (e.g smartphone, a key, bank card etc), something the user is (e.g fingerprint, voice etc.) and/or somewhere the
user is  (e.g GPS coordinates).

Below is a simplified diagram of how YES IDENTITY enables multi-factor Authentication.

![Multi-factor authentication demo](/img/mfa.png)

1. A user initiates a login using their credentials typically username and password. In your backend, you initiate an
   Authentication request to
   YES IDENTITY's [Authorize Endpoint](https://datatracker.ietf.org/doc/html/rfc6749#section-3.1){:target="_blank"} using a
   user identifier. YES IDENTITY's private endpoints are protected
   using [Oauth 2.0 Client Authentication methods](https://darutk.medium.com/oauth-2-0-client-authentication-4b5f929305d4){:target="_blank"}
   . A transaction ID (auth_req_id) with polling interval and expiration date is returned in the response to the client
   i.e your backend, then passed to the browser. YES IDENTITY also supports `PING` mode whereby your backend is notified via a callback when the user has
   acted on the request.

2. The client uses the transaction ID (auth_req_id) to check the status of the Authentication request. If approved, a
   bearer token is returned in the response body, and the browser can then use that to access protected resources.

3. The device/app is notified of an Authentication request. YES IDENTITY can notify the device on your behalf, or you can
   notify the device yourself.

4. The device retrieves the Authentication request.

5. The user approves/denies the Authentication request and signs it with their cryptographic key protected using
   biometrics.

# Implementation

Finally, let's implement Multi-factor authentication using YES IDENTITY :)

#### Step 1. Request Demo

Email [request@demo.yesidentity.com](mailto:request@demo.yesidentity.com) to request access to the dashboard and API
documentation.

#### Step 2. Create Client

Create a Client in the Dashboard to obtain credentials to authenticate with YES IDENTITY's API. You can
choose `client_secret_basic`, `client_secret_post` or `private_key_jwt`
authentication
methods. [Financial-grade API (FAPI)](https://darutk.medium.com/financial-grade-api-fapi-explained-by-an-implementer-d09fcf2ff932){:target="_blank"}
requires private_key_jwt as no secrets are sent on the wire, instead, a JWT signed with the client's private key is used
to authenticate the client.

#### Step 3. Register Device

When a user logs in to their **trusted** device for the first time, the device is registered with YES IDENTITY's backend using
the `Devices API`. The API expects a JWT token (in the Authorization header) signed using the client's private key
obtained from **step 1**.

An example using [Nimbus JOSE + JWT library](https://connect2id.com/products/nimbus-jose-jwt){:target="_blank"} to
create a signed JWT:

```java
public class JwtProvider {

    private final IdentityProperties identityProperties;

    public JwtProvider(IdentityProperties identityProperties) {
        this.identityProperties = identityProperties;
    }

    public String createdSignedJwt() {

        // header
        JWSHeader jwsHeader = new JWSHeader.Builder(JWSAlgorithm.RS256)
                .keyID(identityProperties.getClientId())
                .type(JOSEObjectType.JWT)
                .build();

        // the payload
        JWTClaimsSet jwtClaimsSet = new JWTClaimsSet.Builder()
                .jwtID(UUID.randomUUID().toString())
                .subject(identityProperties.getClientId())
                .issuer(identityProperties.getClientId())
                .audience(identityProperties.getIssuerUrl())
                .expirationTime(Date.from(Instant.now().plusSeconds(5)))
                .issueTime(Date.from(Instant.now()))
                .build();

        SignedJWT signedJWT = new SignedJWT(jwsHeader, jwtClaimsSet);
        try {
            // load the private key from path
            File file = new ClassPathResource(identityProperties.getKeyPairPath()).getFile();

            String jwkValue = new String(Files.readAllBytes(Paths.get(file.getPath())));

            JWK jwk = JWK.parse(jwkValue);

            // sign the jwt
            signedJWT.sign(new RSASSASigner(jwk.toRSAKey().toPrivateKey()));

            return signedJWT.serialize();
        } catch (JOSEException | ParseException | IOException e) {
            e.printStackTrace();
        }

        return null;
    }
}
```

The `Devices API` exposes endpoints to register, find, activate/deactivate devices etc. This can be used to build a
dashboard for users to manage their trusted devices.

In order to register a device, the `device name`, `public key` and the `user's identifier `is
required. [Android Keystore](https://developer.android.com/training/articles/keystore){:target="_blank"} provides API to
generate/store cryptographic keys.

A `KeyStoreUtil` class used in the Demo App:

```java
public class KeyStoreUtil {

    private KeyStore keyStore;

    private static final String KEY_PROVIDER = "AndroidKeyStore";
    private static final String KEY_ALIAS = "selfsigned";

    public KeyStoreUtil()
            throws KeyStoreException, CertificateException, NoSuchAlgorithmException, IOException {
        initKeyStore();
    }

    private void initKeyStore()
            throws KeyStoreException, CertificateException, NoSuchAlgorithmException, IOException {
        this.keyStore = KeyStore.getInstance(KEY_PROVIDER);
        this.keyStore.load(null);
    }

    public Signature initSignature() {
        try {
            Signature signature = Signature.getInstance("SHA256withRSA");
            signature.initSign(getPrivateKey());
            return signature;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public PrivateKey getPrivateKey() {
        try {
            if (keyStore.containsAlias(KEY_ALIAS)) {
                generateKeyPair();
            }
            return (PrivateKey) keyStore.getKey(KEY_ALIAS, null);

        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public PublicKey getPublicKey() {
        try {
            if (!keyStore.containsAlias(KEY_ALIAS)) {
                generateKeyPair();
            }
            return keyStore.getCertificate(KEY_ALIAS).getPublicKey();

        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    private void generateKeyPair() {
        try {
            if (!keyStore.containsAlias(KEY_ALIAS)) {

                GregorianCalendar startDate = new GregorianCalendar();
                GregorianCalendar endDate = new GregorianCalendar();
                endDate.add(Calendar.YEAR, 200);

                KeyPairGenerator kpg =
                        KeyPairGenerator.getInstance(KeyProperties.KEY_ALGORITHM_RSA, KEY_PROVIDER);

                kpg.initialize(
                        new KeyGenParameterSpec.Builder(
                                KEY_ALIAS, KeyProperties.PURPOSE_SIGN | KeyProperties.PURPOSE_VERIFY)
                                .setUserAuthenticationRequired(true)
                                .setUserAuthenticationValidityDurationSeconds(-1)
                                .setInvalidatedByBiometricEnrollment(true)
                                .setDigests(
                                        KeyProperties.DIGEST_NONE,
                                        KeyProperties.DIGEST_SHA256,
                                        KeyProperties.DIGEST_SHA512)
                                .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
                                .setCertificateSubject(new X500Principal(String.format("CN=%s", KEY_ALIAS)))
                                .setSignaturePaddings(KeyProperties.SIGNATURE_PADDING_RSA_PKCS1)
                                .setCertificateNotBefore(startDate.getTime())
                                .setCertificateNotAfter(endDate.getTime())
                                .build());

                kpg.generateKeyPair();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static String getKeyAlias() {
        return KEY_ALIAS;
    }

    public static String getKeyProvider() {
        return KEY_PROVIDER;
    }
}
```

`setUserAuthenticationRequired(true) `indicates that the key must be protected using the device's biometric
authentication.

For non-biometric authentication, this value must be set to false.

#### Step 4. Create Authentication Request

For the Demo App, I have implemented two endpoints.

The first endpoint verifies the user's credentials and returns a transaction ID (auth_req_id), interval for polling and
expiration date in the response body to the browser. In parallel, if configured, YES IDENTITY notifies your device/app of a
pending Authentication request.

```java

@RestController
@RequestMapping("/api")
public class UserJWTController {

    // .... omitted code

    @PostMapping("/mfa")
    public ResponseEntity<AuthenticationResponseDTO> mfa(@Valid @RequestBody LoginVM loginVM) {
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(
                loginVM.getUsername(),
                loginVM.getPassword()
        );

        authenticationManagerBuilder.getObject().authenticate(authenticationToken);

        AuthenticationResponseDTO authenticationResponseDTO = null;
        try {
            authenticationResponseDTO =
                    identityService.authorize(
                            identityProperties.getClientId(),
                            identityProperties.getClientSecret(),
                            CLIENT_ASSERTION_TYPE,
                            jwtProvider.createdSignedJwt(),
                            OPENID,
                            loginVM.getUsername(),
                            Base64.getEncoder().encodeToString(LOGIN_REQUEST.getBytes()),
                            UUID.randomUUID().toString()
                    );
        } catch (IOException e) {
            throw new RuntimeException(e.getMessage(), e);
        }
        return ResponseEntity.ok(
                new AuthenticationResponseDTO(
                        authenticationResponseDTO.getAuthReqId(),
                        authenticationResponseDTO.getInterval(),
                        authenticationResponseDTO.getExpiresIn()
                )
        );
    }
}
```

The browser then polls the `/authenticate` endpoint to obtain a JWT token that will be used to access protected
resources.

```java

@RestController
@RequestMapping("/api")
public class UserJWTController {

    // ... omitted code

    @PostMapping("/authenticate")
    public ResponseEntity<JWTToken> authenticate(@Valid @RequestBody UserJWTController.AuthenticationRequestVM authenticationRequestVM) {
        TokenResponseDTO tokenResponseDTO = null;
        try {
            tokenResponseDTO =
                    identityService.token(
                            identityProperties.getClientId(),
                            identityProperties.getClientSecret(),
                            CLIENT_ASSERTION_TYPE,
                            jwtProvider.createdSignedJwt(),
                            URN_OPENID_PARAMS_GRANT_TYPE_CIBA,
                            authenticationRequestVM.getAuthReqId()
                    );

            JWTClaimsSet jwtClaimsSet = tokenValidator.validateJwt(
                    tokenResponseDTO.getAccessToken(),
                    identityProperties.getJwksUrl(),
                    identityProperties.getIssuerUrl()
            );

            List<GrantedAuthority> grantedAuthorities = userRepository
                    .findOneWithAuthoritiesByEmailIgnoreCase(jwtClaimsSet.getSubject())
                    .orElseThrow()
                    .getAuthorities()
                    .stream()
                    .map(authority -> new SimpleGrantedAuthority(authority.getName()))
                    .collect(Collectors.toList());
            UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                    jwtClaimsSet.getSubject(),
                    null,
                    grantedAuthorities
            );
            SecurityContextHolder.getContext().setAuthentication(authentication);
            String jwt = tokenProvider.createToken(authentication, false);
            HttpHeaders httpHeaders = new HttpHeaders();
            httpHeaders.add(JWTFilter.AUTHORIZATION_HEADER, "Bearer " + jwt);
            return new ResponseEntity<>(new JWTToken(jwt), httpHeaders, HttpStatus.OK);
        } catch (IOException | BadJOSEException | ParseException | JOSEException e) {
            throw new RuntimeException(e.getMessage(), e);
        }
    }
}
```

While the request is pending (not yet approved by the user), YES IDENTITY would return a json body with
error_message `authorization_pending` and HTTP 400 Bad Request response status code. The browser should poll at the
specified interval until the error_message is no longer authorization_pending.

```typescript
@Injectable({providedIn: 'root'})
export class AuthServerProvider {

    // ... omitted code

    authenticate(param: { authReqId: any; interval: any; expiresIn: any }): Observable<void> {
        return this.http.post<JwtToken>(this.applicationConfigService.getEndpointFor('api/authenticate'), {authReqId: param.authReqId}).pipe(
            retryWhen(err =>
                err.pipe(
                    concatMap(error => {
                        if (error.error.detail.includes('authorization_pending')) {
                            return of(error);
                        }
                        return throwError(error);
                    }),
                    delay(1000 * param.interval)
                )
            ),
            map(response => this.authenticateSuccess(response, true))
        );
    }

}
```

#### Step 5. Approve Authentication Request

Once your device/app has received the notification, it should retrieve the Authentication request from your endpoint and
displays it to the user.
A [BottomSheetDialog](https://developer.android.com/reference/com/google/android/material/bottomsheet/BottomSheetDialog){:target="_blank"}
is used in the demo app. Your endpoint should proxy this request securely to YES IDENTITY's `Authentication Requests API`. Like
the `Devices API`, it is protected and expects a JWT token signed using the client's private key.

Depending on your security requirements, YES IDENTITY supports approving Authentication requests on the device using biometric
and non-biometric protected cryptographic keys. Either way you need
a [Signature](https://docs.oracle.com/javase/7/docs/api/java/security/Signature.html) object that is used to sign a JWT
that is sent in a `request` parameter to the `approve` endpoint.

Obtain a Signature from
a [CryptoObject](https://developer.android.com/reference/android/hardware/biometrics/BiometricPrompt.CryptoObject)
returned from a successful biometric authentication:

```java
public class ConsentDialog extends BottomSheetDialogFragment {

    // ... omitted code

    private void setUpBiometric() {
        biometricPrompt =
                new BiometricPrompt(
                        requireActivity(),
                        ContextCompat.getMainExecutor(requireContext()),
                        new BiometricPrompt.AuthenticationCallback() {
                            @Override
                            public void onAuthenticationError(
                                    int errorCode, @NonNull @NotNull CharSequence errString) {
                                super.onAuthenticationError(errorCode, errString);
                                onCancelled();
                            }

                            @Override
                            public void onAuthenticationSucceeded(
                                    @NonNull @NotNull BiometricPrompt.AuthenticationResult result) {
                                super.onAuthenticationSucceeded(result);
                                if (result.getCryptoObject() != null
                                        && result.getCryptoObject().getSignature() != null) {
                                    Signature signature = result.getCryptoObject().getSignature(); // Obtain Signature from CryptoObject
                                    try {
                                        String bearerToken = createBearerToken(signature);  // create signed jwt
                                        yesidentityApi
                                                .approveAuthRequest("Bearer " + bearerToken, authRequestDTO.getId())
                                                .enqueue(
                                                        new Callback<Void>() {
                                                            @Override
                                                            public void onResponse(Call<Void> call, Response<Void> response) {
                                                                if (response.isSuccessful()) {
                                                                    onApproved();
                                                                } else {
                                                                    onCancelled();
                                                                }
                                                            }

                                                            @Override
                                                            public void onFailure(Call<Void> call, Throwable t) {
                                                                onCancelled();
                                                            }
                                                        });

                                    } catch (SignatureException e) {
                                        onCancelled();
                                    }

                                } else {
                                    onCancelled();
                                }
                            }

                            @Override
                            public void onAuthenticationFailed() {
                                super.onAuthenticationFailed();
                                onCancelled();
                            }
                        });
    }
}

```

If they cryptographic keys are not locked by biometrics (i.e users are not prompted for biometric authentication when
they approve an Authentication request), you can initialize a Signature directly as shown in the `KeyStoreUtil` class.

That's it! Although this tutorial is quite high level, the actual implementation took about a day.
