# Configure an app or service with Microsoft Graph

finish

https://docs.microsoft.com/en-us/graph/auth-v2-user

https://docs.microsoft.com/en-us/graph/auth-v2-service

https://docs.microsoft.com/en-us/graph/use-the-api

https://docs.microsoft.com/en-us/graph/permissions-reference

## Reading material:
https://docs.microsoft.com/en-us/graph/overview

https://docs.microsoft.com/en-us/graph/users-you-can-reach

https://docs.microsoft.com/en-us/graph/overview-major-services

https://docs.microsoft.com/en-us/graph/azuread-users-concept-overview

https://docs.microsoft.com/en-us/graph/auth-overview

https://docs.microsoft.com/en-us/graph/api/resources/users?view=graph-rest-1.0

https://docs.microsoft.com/en-us/graph/auth-v2-user

https://jwt.io/introduction/

https://oauth.net/articles/authentication/

https://auth0.com/docs/protocols/oidc

https://www.hcidata.info/base64.htm

## Videos

## What is Microsoft Graph?
Microsoft Graph is the gateway to data and intelligence in Microsoft 365. Microsoft Graph provides a unified programmability model that you can use to take advantage of the tremendous amount of data in Office 365, Enterprise Mobility + Security, and Windows 10. Use Microsoft Graph to reach users with Microsoft personal accounts, such as @outlook.com, @hotmail.com, or @live.com accounts. With their consent, you can use Microsoft Graph to access users' profiles, their Office services like OneDrive and Outlook mail, calendar, and contacts, and their Windows 10 devices and activities.

![Microsoft Graph](https://github.com/microsoftgraph/python-sample-auth/raw/master/static/images/authworkflow.png)

## What is Microsoft Graph Connect?
Microsoft Graph Data Connect enables bulk - rather than the traditional transactional - access to Office 365 data.

## What is Oath?
OAuth 2.0 is an authorization framework, not an authentication protocol. You can think of this framework as a common denominator for authorization. Oauth2 was left generic so that it could be applied to many authorization requirements, like API access management or posting on someone’s wall.

## What is OpenID Connect?
OpenID Connect is a profile of OAuth 2.0 that defines a workflow for authentication. The big difference between OpenID Connect and OAuth2 is the id_token. There is no id_token defined in OAuth2 because the id_token is specific to federated authentication.

## Definitions
1. Base64: Encoded data is a string of characters that contains only a-z, A-Z, 0-9, + and / characters and is often used in situations when sending non-text information via a text only transmission protocol. Base 64 Encoding takes a stream of characters and converts them to characters that belong to the universal ASCII character set. Once a stream of characters has been converted to characters that belong to the universal ASCII character set (Base 64 encoded) they can be transported with ease over the Internet.

2. Azure AD Access Token: Access tokens issued by Azure AD are base 64 encoded JSON Web Tokens (JWT). They contain information (claims) that web APIs secured by Azure AD, like Microsoft Graph, use to validate the caller and to ensure that the caller has the proper permissions to perform the operation they're requesting.

3. JSON Web Tokens: JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA or ECDSA.

4. Directory permissions: Directory permissions are highly privileged permissions and always require administrator consent.

## Misc notes

### Azure AD

* Azure AD exposes two sets of endpoints, Azure AD and Azure AD v2.0. The main difference between them is that Azure AD endpoint supports only work or school accounts (that is, accounts that are associated with an Azure AD tenant), while Azure AD v2.0 also supports Microsoft accounts like Live.com or outlook.com accounts. This means that if you use the Azure AD endpoint, your app can target only organizations, but with Azure AD v2.0 it can target both consumers and organizations

* There are some additional differences with Azure AD v2.0:

  * Your app can use a single Application ID for multiple platforms. This simplifies app management for both developers and administrators.

  * Support for dynamic and incremental consent. With this feature your app can request additional permissions during runtime, pairing the request for the user's consent with the functionality that requires it. This provides a much more comfortable experience for users than having to consent to a long list of permissions when they sign-in for the first time.

  * Windows integrated authentication for federated tenants is not supported. This means that users of federated Azure AD tenants cannot silently authenticate with their on-premises Active Directory instance. They will have to re-enter their credentials.

* Tokens from the Azure AD endpoint are not interchangeable with those from the Azure AD v2.0 endpoint, so before you begin work on an app for production, you must choose between the endpoints.

* OAuth 2.0 is an authorization protocol. It defines how your app can get access tokens by authenticating directly with Azure AD or by redirecting a user to authenticate with Azure AD and consent to the permissions your app requests. In the first case, your app gets an access token that it can use to call Microsoft Graph as itself. In second case, your app gets an access token that it can use to call Microsoft Graph on behalf of a user. 

* With OAuth 2.0, your app does not receive any information about the user or how they were authenticated by Azure AD. OAuth 2.0 flows are most often used by mobile or native apps, which already know the identity of the user; or by apps like background services or daemons, which call Microsoft Graph under their own identity and not on behalf of a user.

* OpenID Connect extends OAuth 2.0 to provide an identity layer. With OpenID Connect, in addition to an access token, your app can also get an id token from Azure AD. OpenID Connect id tokens contain claims about the user's identity and details about how and where they were authenticated. OpenID Connect flows are typically used by web apps, including single page apps (SPAs). These apps can use the id token to customize their behavior for the user they've requested an access token for, and, in many cases, will outsource sign-in of their users to Azure AD and enable experiences like Single Sign-on (SSO).

### Microsoft Graph

* Before your app can get a token from Azure AD, your app must be registered. For the Azure AD v2.0 endpoint, you use the Microsoft App Registration Portal to register your app. For the Azure AD endpoint, you use the Azure portal.

* To call Microsoft Graph, your app must acquire an access token from Azure Active Directory (Azure AD), Microsoft's cloud identity service. The access token contains information (or claims) about your app and the permissions it has for the resources and APIs available through Microsoft Graph. To get an access token, your app must be able to authenticate with Azure AD and be authorized by either a user or an administrator for access to the Microsoft Graph resources it needs.

* The basic steps required to use the OAuth 2.0 authorization code grant flow to get an access token from the Azure AD v2.0 endpoint are:

  * Register your app with Azure AD.
  * Get authorization.
  * Call Microsoft Graph with the access token.
  * Use a refresh token to get a new access token.

* You attach the access token as a Bearer token to the Authorization header in an HTTP request. 

  ```
  HTTP/1.1
  Authorization: Bearer EwAoA8l6BAAU ... 7PqHGsykYj7A0XqHCjbKKgWSkcAg==
  Host: graph.microsoft.com`
  GET https://graph.microsoft.com/v1.0/me/
  ```

* There are two types of permissions:

  * Delegated permissions are used by apps that run with a user present. The user's privileges are delegated to the app which makes calls on behalf of the user to Microsoft Graph. Many of these permissions can be consented to by a user, but others require administrator consent.

  * Application permissions are used by apps that run without a user. These often grant an app broad privileges within an organization and always require the consent of an administrator.

* Authenticating your application with admin consent enables you to work with and update a wider range of entities associated with a user.

* Microsoft Graph permission names follow a simple pattern: **resource.operation.constraint**. For example, User.Read grants permission to read the profile of the signed-in user, User.ReadWrite grants permission to read and modify the profile of the signed-in user, and Mail.Send grants permission to send mail on behalf of the signed-in user.

* You can access users through Microsoft Graph in two ways:

  * By their ID, /users/{id | userPrincipalName}

  * By using the /me alias for the signed-in user, which is the same as /users/{signed-in user's id}

### OAuth 2

* Authentication in the context of a user accessing an application tells an application who the current user is and whether or not they're present. OAuth tells the application none of that. OAuth says absolutely nothing about the user, nor does it say how the user proved their presence or even if they're still there. As far as an OAuth client is concerned, it asked for a token, got a token, and eventually used that token to access some API. It doesn't know anything about who authorized the application or if there was even a user there at all. In fact, much of the point of OAuth is about giving this delegated access for use in situations where the user is not present on the connection between the client and the resource being accessed.

### JSON Web Tokens

* In its compact form, JSON Web Tokens consist of three parts separated by dots (.), which are:

  * Header
  * Payload
  * Signature

* **A JWT typically looks like this xxxxx.yyyyy.zzzzz**. Do note that for signed tokens this information, though protected against tampering, is readable by anyone. Do not put secret information in the payload or header elements of a JWT unless it is encrypted.

  * The first part, or header, typically consists of two parts: the type of the token, which is JWT, and the signing algorithm being used, such as HMAC SHA256 or RSA. this JSON is Base64Url encoded to form the first part of the JWT.

  ``` 
      {
        "alg": "HS256",
        "typ": "JWT"
      } 
  ```
  * The second part of the token is the payload, which contains the claims. Claims are statements about an entity (typically, the user) and additional data. There are three types of claims: registered, public, and private claims. The payload is then Base64Url encoded to form the second part of the JSON Web 

  ```
    {
      "sub": "1234567890",
      "name": "John Doe",
      "admin": true
    }
  ```

    * Registered claims: These are a set of predefined claims which are not mandatory but recommended, to provide a set of useful, interoperable claims. Some of them are: iss (issuer), exp (expiration time), sub (subject), aud (audience), and others. Notice that the claim names are only three characters long as JWT is meant to be compact.

    * Public claims: These can be defined at will by those using JWTs. But to avoid collisions they should be defined in the IANA JSON Web Token Registry or be defined as a URI that contains a collision resistant namespace.

    * Private claims: These are the custom claims created to share information between parties that agree on using them and are neither registered or public claims.

* The third part is the Signature. To create the signature part you have to take the encoded header, the encoded payload, a secret, the algorithm specified in the header, and sign that. The signature is used to verify the message wasn't changed along the way, and, in the case of tokens signed with a private key, it can also verify that the sender of the JWT is who it says it is.

### Base64

* Regarding the equal characters often seen in Base64 strings:
  * The 65th character ("=" sign) is used only as a complement in the final process of encoding a message.

    You will not have a '=' sign if your string has a multiple of 3 characters number, because Base64 encoding takes each three bytes (8bits) and represents them as four printable characters in the ASCII standard.

    (a) If you want to encode ABCDEFG <=> [ABC] [DEF] [G

    Base64 will deal(producing 4 characters) with the first block and the second (as they are complete) but for the third it will add a double == in the output in order to complete the 4 needed characters.Thus, the result will be QUJD REVG Rw== (without space)

    (b) If you want to encode ABCDEFGH <=> [ABC] [DEF] [GH

    Similarly, it will add just a single = in the end of the output to get 4 characters the result will be QUJD REVG R0g= (without space)