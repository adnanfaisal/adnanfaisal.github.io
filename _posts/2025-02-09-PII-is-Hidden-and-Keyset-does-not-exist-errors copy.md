## SSL Certificate issues on Sitecore Id Server - PII is hidden and Keyset does not exist

Few days ago I faced two different issues related to SSL certificate expiration on the Sitecore Identity Server. In this article, I will tell about the symptoms and How I resolved them. 

## PII Is Hidden Error
I noticed that my Sitecore CM Server is bypassing the Identity Server. You can tell by just by looking at your browser url that for authentication, you are not forwarded to the Id server. I know that Identity Server can be disabled by disabling the `Sitecore.Owin.Authentication.IdentityServer.config` file on the CM server. But, this was not the case. I wanted my Id server to work. 

I tried to check if my Id server is running. From IIS, I found that the web server was running. Then on my browser I tried to browse `https://mysc10.idserver/.well-known/openid-configuration` and from the web browser address bar, I found that my browser flagging the site as **Not secure**, meaning the SSL certificate has expired. 

In addition, I also found the following error on my CM server log. 

```
ERROR Unable to reach an external identity provider.
Exception: System.InvalidOperationException
Message: IDX20803: Unable to obtain configuration from: '[PII is hidden]'.
Source: Microsoft.IdentityModel.Protocols
   at Microsoft.IdentityModel.Protocols.ConfigurationManager`1.<GetConfigurationAsync>d__24.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.Owin.Security.OpenIdConnect.OpenIdConnectAuthenticationHandler.<ApplyResponseChallengeAsync>d__10.MoveNext() in /_/src/Microsoft.Owin.Security.OpenIdConnect/OpenidConnectAuthenticationHandler.cs:line 148
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.Owin.Security.Infrastructure.AuthenticationHandler.<ApplyResponseCoreAsync>d__40.MoveNext() in /_/src/Microsoft.Owin.Security/Infrastructure/AuthenticationHandler.cs:line 179
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.Owin.Security.Infrastructure.AuthenticationHandler.<ApplyResponseAsync>d__39.MoveNext() in /_/src/Microsoft.Owin.Security/Infrastructure/AuthenticationHandler.cs:line 167
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.Owin.Security.Infrastructure.AuthenticationHandler.<TeardownAsync>d__34.MoveNext() in /_/src/Microsoft.Owin.Security/Infrastructure/AuthenticationHandler.cs:line 96
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.Owin.Security.Infrastructure.AuthenticationMiddleware`1.<Invoke>d__5.MoveNext() in /_/src/Microsoft.Owin.Security/Infrastructure/AuthenticationMiddleware.cs:line 32
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.Owin.Security.Infrastructure.AuthenticationMiddleware`1.<Invoke>d__5.MoveNext() in /_/src/Microsoft.Owin.Security/Infrastructure/AuthenticationMiddleware.cs:line 30
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.Owin.Security.Infrastructure.AuthenticationMiddleware`1.<Invoke>d__5.MoveNext() in /_/src/Microsoft.Owin.Security/Infrastructure/AuthenticationMiddleware.cs:line 30
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.Owin.Security.Infrastructure.AuthenticationMiddleware`1.<Invoke>d__5.MoveNext() in /_/src/Microsoft.Owin.Security/Infrastructure/AuthenticationMiddleware.cs:line 30
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.AspNet.Identity.Owin.IdentityFactoryMiddleware`2.<Invoke>d__0.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.AspNet.Identity.Owin.IdentityFactoryMiddleware`2.<Invoke>d__0.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.Owin.Mapping.MapMiddleware.<Invoke>d__3.MoveNext() in /_/src/Microsoft.Owin/Mapping/MapMiddleware.cs:line 72
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Sitecore.Owin.Middlewares.GlobalExceptionHandlerMiddleware.<Invoke>d__4.MoveNext()
   ```
The solution was to update the SSL certificate on the Id server. You can follow [this document by Vinicius](https://viniciusdeschamps.com.br/sitecore-using-ssl-part-1/) to know how to install self-signed SSL certificate on your dev machine. 

### Same or Different certificate for each server?
It's your choice. You can either use the same certificate or different certificates for the different server. It does not make any difference. 

On the Id server, you'll also have to update the `CertificateThumbprint` value (found from the freshly installed SSL certificate) in the `Sitecore.IdentityServer.Host.xml` file with the thumbprint of your Id server's SSL certificate. You don't change any other values on this file, no change of `Client secrets`. 

### Client Secret vs SSL Certificate
**ClientSecret** is the shared secret used by IdentityServer and the client application (like Sitecore, xConnect) to authenticate requests securely. The ClientSecrets in Sitecore.IdentityServer.Host.xml should match the secret configured for the client (such as Sitecore, xConnect, or Identity Server) when registering the client in IdentityServer. ClientSecrets in the context of Sitecore IdentityServer are not directly related to SSL certificates. ClientSecrets are used for authenticating the client, not securing the communication itself.

On the other side, **SSL Certificates** are used for encrypting communication between the client and server and ensuring that no one can eavesdrop on the data being exchanged. An SSL certificate (often in .pfx or .crt format) is used for securing communications between the client (e.g., Sitecore CM) and IdentityServer using HTTPS (SSL/TLS encryption). The SSL certificate ensures that all data exchanged over the network is encrypted, and it establishes trust between the server and the client. SSL certificates are necessary for securing the communication channel (like HTTPS) and preventing man-in-the-middle (MITM) attacks. This is especially important when transmitting sensitive data like authentication tokens or user credentials.

Although they serve different purposes, SSL certificates and ClientSecrets can work together in a secure authentication flow.
SSL ensures that the communication is encrypted and secure.
ClientSecrets ensures that the client is authenticated by IdentityServer.


So, in short, while both are related to security, ClientSecrets authenticate the client, and SSL certificates secure the communication channel. They are not the same thing.

## Keyset Does Not Exist error

After installing new SSL certificate for the Sitecore Identity Server, things were still not perfect. Now the Id server is running, I can log in, but nothing shows up after that, I am stuck on the login page / identity server. 

I checked the logs of the CM server but could not find anything. Then I checked the logs of the Id server and found the following error.  

```
[FTL] (Sitecore Identity/WIN-MachineName) Unhandled exception: "Keyset does not exist"
Internal.Cryptography.CryptoThrowHelper+WindowsCryptographicException: Keyset does not exist
   at System.Security.Cryptography.CngKey.Open(String keyName, CngProvider provider, CngKeyOpenOptions openOptions)
   at System.Security.Cryptography.CngKey.Open(String keyName, CngProvider provider)
   at Internal.Cryptography.Pal.CertificatePal.GetPrivateKey[T](Func`2 createCsp, Func`2 createCng)
   at Internal.Cryptography.Pal.CertificatePal.GetRSAPrivateKey()
   at Internal.Cryptography.Pal.CertificateExtensionsCommon.GetPrivateKey[T](X509Certificate2 certificate, Predicate`1 matchesConstraints)
   at System.Security.Cryptography.X509Certificates.RSACertificateExtensions.GetRSAPrivateKey(X509Certificate2 certificate)
   at Microsoft.IdentityModel.Tokens.X509SecurityKey.get_PrivateKey()
   at Microsoft.IdentityModel.Tokens.X509SecurityKey.get_PrivateKeyStatus()
   at Microsoft.IdentityModel.Tokens.AsymmetricSignatureProvider.FoundPrivateKey(SecurityKey key)
   at Microsoft.IdentityModel.Tokens.AsymmetricSignatureProvider..ctor(SecurityKey key, String algorithm, Boolean willCreateSignatures)
   at Microsoft.IdentityModel.Tokens.AsymmetricSignatureProvider..ctor(SecurityKey key, String algorithm, Boolean willCreateSignatures, CryptoProviderFactory cryptoProviderFactory)
   at Microsoft.IdentityModel.Tokens.CryptoProviderFactory.CreateSignatureProvider(SecurityKey key, String algorithm, Boolean willCreateSignatures, Boolean cacheProvider)
   at Microsoft.IdentityModel.Tokens.CryptoProviderFactory.CreateForSigning(SecurityKey key, String algorithm, Boolean cacheProvider)
   at Microsoft.IdentityModel.Tokens.CryptoProviderFactory.CreateForSigning(SecurityKey key, String algorithm)
   at Microsoft.IdentityModel.JsonWebTokens.JwtTokenUtilities.CreateEncodedSignature(String input, SigningCredentials signingCredentials)
   at Microsoft.IdentityModel.JsonWebTokens.JsonWebTokenHandler.CreateTokenPrivate(JObject payload, SigningCredentials signingCredentials, EncryptingCredentials encryptingCredentials, String compressionAlgorithm, IDictionary`2 additionalHeaderClaims, String tokenType)
   at Microsoft.IdentityModel.JsonWebTokens.JsonWebTokenHandler.CreateToken(String payload, SigningCredentials signingCredentials, IDictionary`2 additionalHeaderClaims)
   at Duende.IdentityServer.Services.DefaultTokenCreationService.CreateJwtAsync(Token token, String payload, Dictionary`2 headerElements) in /_/src/IdentityServer/Services/Default/DefaultTokenCreationService.cs:line 128
   at Duende.IdentityServer.Services.DefaultTokenCreationService.CreateTokenAsync(Token token) in /_/src/IdentityServer/Services/Default/DefaultTokenCreationService.cs:line 75
   at Duende.IdentityServer.Services.DefaultTokenService.CreateSecurityTokenAsync(Token token) in /_/src/IdentityServer/Services/Default/DefaultTokenService.cs:line 269
   at Duende.IdentityServer.ResponseHandling.AuthorizeResponseGenerator.CreateImplicitFlowResponseAsync(ValidatedAuthorizeRequest request, String authorizationCode) in /_/src/IdentityServer/ResponseHandling/Default/AuthorizeResponseGenerator.cs:line 184
   at Duende.IdentityServer.ResponseHandling.AuthorizeResponseGenerator.CreateHybridFlowResponseAsync(ValidatedAuthorizeRequest request) in /_/src/IdentityServer/ResponseHandling/Default/AuthorizeResponseGenerator.cs:line 126
   at Duende.IdentityServer.ResponseHandling.AuthorizeResponseGenerator.CreateResponseAsync(ValidatedAuthorizeRequest request) in /_/src/IdentityServer/ResponseHandling/Default/AuthorizeResponseGenerator.cs:line 107
   at Duende.IdentityServer.Endpoints.AuthorizeEndpointBase.ProcessAuthorizeRequestAsync(NameValueCollection parameters, ClaimsPrincipal user, Boolean checkConsentResponse) in /_/src/IdentityServer/Endpoints/AuthorizeEndpointBase.cs:line 138
   at Duende.IdentityServer.Endpoints.AuthorizeEndpointBase.ProcessAuthorizeRequestAsync(NameValueCollection parameters, ClaimsPrincipal user, Boolean checkConsentResponse) in /_/src/IdentityServer/Endpoints/AuthorizeEndpointBase.cs:line 150
   at Duende.IdentityServer.Endpoints.AuthorizeCallbackEndpoint.ProcessAsync(HttpContext context) in /_/src/IdentityServer/Endpoints/AuthorizeCallbackEndpoint.cs:line 48
   at Duende.IdentityServer.Hosting.IdentityServerMiddleware.Invoke(HttpContext context, IEndpointRouter router, IUserSession session, IEventService events, IIssuerNameService issuerNameService, IBackChannelLogoutService backChannelLogoutService) in /_/src/IdentityServer/Hosting/IdentityServerMiddleware.cs:line 84
   ```


This error happens when IIS app user account does not have permission on this certificate. Adding the permission should resolve the issue. I followed the instructions of [this article](https://sitecorefootsteps.blogspot.com/2019/09/sitecore-identity-server-error-with-ca.html) to resolve the issue with two excpetions.

1. The user to give permission for me was not IIS_IUSRS. From the IIS App Pool, I found that my Id Server's Identity column showed `ApplicationPoolIdentity`, meaning whatever value I have for the App Pool Name (e.g., mysc10.idserver) is the user to whom I have to give permission. This user name can be also confirmed from the *Task Manager -> Details* and then *User name* column for the corresponding *w3wp* process. Since `ApplicationPoolIdentity` was the value in the `Identity` column, I had to give permission to the user `IIS AppPool\mysc10.idserver`.

2. In the above article, it is mentioned that the certificate was moved back to *Trusted Root Certification Authorities*. In my case moving did not resolve the issue. I had to copy it. In other words, my same certificate with added user permission exists in both *Trusted Root Certification Authorities* and *Personal* stores. 

Finally, I was able to resolve SSL certificate issues on my Id Server. Hooray!