## How to solve invalid_client error in Sign in with Apple



One of the major roadblock on implementing Sign in with Apple is generating the **client_secret** parameter, which is required when sending a HTTP POST request to Apple's token validation endpoint, which exchange authorization code for an access token.



![invalid client error](https://iosimage.s3.amazonaws.com/2020/75-invalid-client-sign-in-with-apple/error_client.png)



**client_secret** is a JWT (JSON Web Token) string you generate to prove that the HTTP request indeed comes from you (or your code), not originated from possible attacker.



Invalid client error usually happens when one of these occur : 

1. Using non matching app bundle identifier in the HTTP request
2. JWT header doesn't contain all required parameters
3. JWT payload doesn't contain all required parameters
4. JWT signature is incorrect



We will walk through each of these below. If you are confident that your JWT payloads and HTTP request are correct, you can jump to section 4 directly.



## Using non matching app bundle identifier in the HTTP request





