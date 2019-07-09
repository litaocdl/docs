This documents describe various kinds of authentication and authorization 

## Server Authentication vs Client Authentication

`Certification identify someone or something, and authentication confirm the identity.` from redhat doc. 

When talking about server authentication and client authentiction, we are talking about x509 certification based authentication. In general, server authentication means server side provide the something to let the client side verify, and client authentication means client side provide something to let the server side to verify. 

`Server certificate authentication`  Server send its certification to client side, and client side leverage the trust CA to verify the signature of the server certification to decide trust or not trust. 



Those are all server authentication:
1. web server enabled the https, and client side verify the server certification. 

2. client use jdbc to connect to database which enabled the ssl, client need to provide the truststore. the truststore is used to store the trust certifications which provided by the server side (self-signed), or use public CA if server is configured CA based certification. 

`Client certificate authentication` Enabled in server side, happens after the client verify the server certification. client send its certification to server side, and the server verfiy the certification. this is an optional steps defined in following diagram

if we are not only talking about the certification authentication, all the password based authentication can belongs to client authentication, as username and password are provided by the client and server do the verification.

![](https://access.redhat.com/documentation/en-US/Red_Hat_Certificate_System/8.0/html/Deployment_Guide/images/02cert.png)

[Reference: Certification and Authentication](https://access.redhat.com/documentation/en-US/Red_Hat_Certificate_System/8.0/html/Deployment_Guide/Introduction_to_Public_Key_Cryptography-Certificates_and_Authentication.html)
[Reference: Client Authentication](https://docs.oracle.com/cd/E19424-01/820-4811/aakhe/index.html)

![](https://blog.cloudflare.com/content/images/2017/05/illustration-tls-ssl-client-auth.png)

## Basic Authentication 

Http basic authentication is server requests a username and password from the client and server verify the username and password pairs. basic authentication is not a secure authentication mechanism, basic authentication sends the user names and passwords over the internet as base64 encoded text and the target server is not authenticated. however, if the secure http channel is used like SSL or security at the network level such as IPSEC or VPN, some of those concern could be alleviated. 

Basic authentication in http header 

use `-u` or `--user` to specific the user. 
```
curl -u "username:password" http://example.com
curl --user "username:password" http://example.com
```

[Reference: Basic authentication](https://docs.oracle.com/cd/E19226-01/820-7627/bncbo/index.html)

## Digest Authentication 

Like HTTP Basic Authentication, Http Digest Authentication authenticates against username and password, unlike basic authentication, digest authentication send the password hash to the server side. 

```
curl --digest --user daniel:secret http://example.com/
```

## Form based authentication 

Form based authentication is putting the username and password into http body not in http headers, it is not secure unless ssl is used. 

```
curl -X POST http://example.com -d "{"username":<>,"password":<>}"
```

## OAUTH2 Authentication 


## IAM 

## OPEN ID

## SAML


## Reference
