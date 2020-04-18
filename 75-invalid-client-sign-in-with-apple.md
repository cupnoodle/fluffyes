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



"**iat**" means the time when this JWT was issued (created), this value should equal to the **UNIX [timestamp](https://www.unixtimestamp.com) in seconds** (not milliseconds) when your server generated the JWT. In Ruby, you can use **Time.now.to_i** to get this. If you are using Java, remember to convert this to seconds (instead of using milliseconds). This should be in **number value**, not enclosed in String, (ie: **exp: 1587204602**, instead of exp: "1587204602")



"**exp**" means the expiry time for this JWT, which the JWT will become invalid after this time. You should set a future time in **UNIX [timestamp](https://www.unixtimestamp.com) in seconds** (not milliseconds) for this field. The maximum acceptable value for this field is current time's timestamp + 15777000 seconds (6 months in the future) , usually I set it to 10 minutes from current time's timestamp ( eg: Time.now.to_i + 600 seconds ). This should be in **number value**, not enclosed in String, (ie: **exp: 1587204602**, instead of exp: "1587204602")



"**aud**" means the intended audience for this JWT, as we are sending this JWT to Apple's AppleID server, the value of this should always equal to "**https://appleid.apple.com**"



"**sub**" means the subject for this JWT. This should equal to the **client_id** value you used in the HTTP POST request. If you are using authorization code or access token gotten from iOS app, this field should equal to your iOS app bundle ID. If you are using authorization code or access token gotten from Apple's auth website redirect, this field should equal to your Services ID's identifier.





## Is your JWT signature correct?

Your JWT signature is the last part of the JWT string : 



![jwt signature](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/signature.png)



Here's a simplified diagram on how the signature is generated : 

![signature generation](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/signature_generation.png)



To check if the signature generated is correct, we can generate a public key from the .p8 private key, and decrypt the signature using the public key, and check if the decrypted SHA-256 hash matches the SHA-256 hash generated from hashing the header and payload. 



Fortunately https://jwt.io/#debugger has built in signature verification function, we can just paste the public key in and it will verify it for us, so we only need to generate the public key.



To generate a public key from the .p8 private key, open Terminal app, and navigate (cd) to the directory containing your .p8 private key.



And run this command : 

```bash
openssl ec -in AuthKey_123ABC456.p8 -pubout -out AuthKey_123ABC456_public.p8
```



Replace the "AuthKey_123ABC456.p8" with your private key file name, and replace "AuthKey_123ABC456_public" with the file name you want to use for the exported public key.



Running this command will generate a public key in the same folder. The public key file (.p8) file will look like this if you open it with text editor : 

```bash
-----BEGIN PUBLIC KEY-----
ABCDEFGHIJKLMNOP......
-----END PUBLIC KEY-----
```





Go to [JWT.io debugger](https://jwt.io/#debugger), select ‘**ES256**’  as the algorithm and paste your client secret JWT string, then at the bottom right section, paste the public key text (including the “—BEGIN PUBLIC KEY—-“ and “—END PUBLIC KEY—” lines) into the "verify signature" -> public key box.



![paste public key](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/paste_public_key.png)





You should see a  “Signature Verified” status at the bottom left if the JWT is signed correctly : 



![signature verified](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/signature_verified.png)



If you are seeing “Invalid Signature”, it means that you are  either using the wrong private key to sign it, or there’s something wrong with the signing step.



Make sure the JWT library you used supports ES256 elliptic curve encryption.



If your own code or the library you used is using **openssl\_sign** function to generate the signature for ES256, it will generate a signature which uses a DER-encoded ASN.1 structure (with size > 64).



The correct digital signature is the concatenation of two unsigned integers, denoted as R and S, which are the result of the Elliptic Curve (EC) algorithm. The length of R \|| S is 64.



The way to fix this is to convert the DER-encoded signature into a raw concatenation of the R and S values, as explained further in this [StackOverflow answer](https://stackoverflow.com/questions/59737488/apple-sign-in-invalid-client-sign-jwt-for-authentication-using-php-and-openss).



…..or you can use the libraries I listed below for generating / signing JWT, I have tested each of them myself and they produce the correct signature for ES256.

1. Python - [https://github.com/mpdavis/python-jose]
2. Node.js - [https://github.com/panva/jose]
3. PHP - [https://github.com/firebase/php-jwt]
4. Ruby - [https://github.com/jwt/ruby-jwt]





This article is an excerpt from the book [Practical Sign in with Apple](http://siwa.fluffy.es/?ref=invalidclient) , if you want a complete step by step guide on implementing Sign in with Apple (from iOS client side to backend communication with Apple to adding SIWA support on your website), with code samples which you can plug in directly (Ruby, PHP, Python and NodeJS), give the sample chapters a try!



CTA of download sample chapter