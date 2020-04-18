## How to solve invalid_client error in Sign in with Apple



One of the major roadblock on implementing Sign in with Apple is generating the **client_secret** parameter, which is required when sending a **HTTP POST** request to Apple's token validation endpoint (https://appleid.apple.com/auth/token), which exchange authorization code for an access token.



![invalid client error](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/error_client.png)



**client_secret** is a JWT (JSON Web Token) string you generate to prove that the HTTP request indeed comes from you (or your code), not originated from possible attacker.



Here's the usual suspect when invalid client error happens : 

1. Are you using the correct client_id in your HTTP request?
2. Does your JWT header contains all the required parameters?
3. Does your JWT payload contains all the required parameters, correctly?
4. Is your JWT signature correct?



We will walk through each of these below and how to fix them. If you are confident that your JWT payloads and HTTP request are correct, you can jump to section 4 directly. We will be using this online JWT debugger (https://jwt.io/#debugger) to debug and verify JWT.



## Are you using the correct client_id in your HTTP request?

If the authorization code comes from your **iOS app**, the **client_id** should be your **iOS app bundle identifier**.

![app client id](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/app_client_id.png)





If the authorization codes comes from your **website / Android app** (Apple redirect URI), the **client_id** should be your **Services ID identifier**.

![web client id](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/web_client_id.png)



## Does your JWT header contains all the required parameters?

Paste in your JWT string into the "encoded" section of this JWT debugger (https://jwt.io/#debugger) 

![header](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/header.png)





Your JWT header should only contain "**kid**" and "**alg**" field. 



The "**kid**" value should equal to your **key ID**, which is the .p8 key file generated in the Apple developer portal, with Sign in with Apple capability. If you don't have access to Apple developer portal, your .p8 key file should have the filename like "AuthKey_**ABCDEF**.p8",  the **ABCDEF** part is your Key ID.

![key id](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/key_id.png)



The "**alg**" value should equal to "**ES256**", as Apple's server expect your JWT to be signed using [Elliptive Curve Digital Signature Algorithm](https://ldapwiki.com/wiki/ES256) using P-256 and SHA-256.



## Does your JWT payload contains all the required parameters, correctly?

Paste in your JWT string into the "encoded" section of this JWT debugger (https://jwt.io/#debugger) 



![payload](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/payload.png)



Your JWT payload should only contain "**iss**", "**iat**", "**exp**", "**aud**" and "**sub**" field.



"**iss**" means issuer of this JWT, which is you or your company, this value should equal to your **Team ID** as shown in the [Apple developer portal membership section](https://developer.apple.com/account/#/membership/) : 



![team ID](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/issuer.png)



"**iat**" means the time when this JWT was issued (created), this value should equal to the **UNIX [timestamp](https://www.unixtimestamp.com) in seconds** (not milliseconds) when your server generated the JWT. In Ruby, you can use **Time.now.to_i** to get this. If you are using Java, remember to convert this to seconds (instead of using milliseconds).



"**exp**" means the expiry time for this JWT, which the JWT will become invalid after this time. You should set a future time in **UNIX [timestamp](https://www.unixtimestamp.com) in seconds** (not milliseconds) for this field. The maximum acceptable value for this field is current time's timestamp + 15777000 seconds (6 months in the future) , usually I set it to 10 minutes from current time's timestamp ( eg: Time.now.to_i + 600 seconds ).



"**aud**" means the intended audience for this JWT, as we are sending this JWT to Apple's AppleID server, the value of this should always equal to "**https://appleid.apple.com**"



"**sub**" means the subject for this JWT. This should equal to the **client_id** value you used in the HTTP POST request. If you are using authorization code or access token gotten from iOS app, this field should equal to your iOS app bundle ID. If you are using authorization code or access token gotten from Apple's auth website redirect, this field should equal to your Services ID's identifier.





## Is your JWT signature correct?







