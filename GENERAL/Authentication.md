
client Server with OAuth SSO OpenID Provider


IDENTITY PROVIDERS
===================

Directory Server

Active Directory

Local/Files

OpenID Provider
identity token

SAML Identity Provider




AUTHENTICATION FRAMEWORKS
===========================

SASL (Simple Authentication and Security Layer)


GSSAPI (Generic Security Services Application Programming Interface)
gssapi-keyex
gssapi-with-mic




AUTHORIZATION
===============

OAuth 2.0

3 main phases
request an authorizaiton grant
exchange the authorization grant for an access token access the resource
with an access token

Flows/Grants
Commonly Used
Authorization Code - the idea is to exchange an authorization code for
an access token

clients that use this flow could be public clients (do not provide a
secret to the authorizer) or confidential clients (provide a secret to
the authorizer)

(browser based protocol, used to authenticate and authorize browser
based applications??)
Proof Key for Code Exchange (PKCE) is an extension to prevent CSRF and
authorization code injection attacks

Client Credentials - used by clients to obtain an access token outside
of the context of a user
typical use case is clients accessing resources about themselves rather
than to access a user's resources

Device Code - the idea is to exchange a previously obtained device code
for an access token
aka Device Authorization Grant
typical use case is for browserless or iinput-constrained devices

Refresh Token - the idea is to exchange a refresh token for an access
token
the use cas is for clients to continue to have a valid access token
without further interaction with the user
Legacy
Implicit Flow - a simple flow where an app (client) could get an access
token without an extra authorization code exchange step (no refresh
token is involved)
this is now discouraged in favour of authorization code grant/flow

Password Grant - a way to exchange a user's credentials
(username/password) for an access token
aka Resource Owner Password Credentials Grant
not recommended (but still used in places) because the client gets
access to the user's credentials

token types
access token in JWT Format
refresh token in JWT Format



AUTHENTICATION PROTOCOLS (aka AUTHENTICATION MECHANISMS)
============================================================


Kerberos


Negotiate (SPNEGO) - Simple and Protected GSSAPI Negotiation Mechanism
used to authenticate transparently through the web browser after the
user has been authenticated with log in


OpenID Connect (OIDC) - generally works on top of the OAuth Framework
uses an Identity token

makes heavy use of JWT set of standards - i.e an identity token in JSON
format with ways to digitally sign and encrypt that data in a compact
and web friendly way



SAML - Normally between Service Providers and Identity Providers


LDAP


RADIUS


SASL Plain


SASL Digest


SASL SCRAM - Salted Challenge Response Authentication Mechanism


SASL EXTERNAL - uses client certificates


SASL OAUTHBEARER



SASL GS2-KRB5 - GS2 is a protocol bridge between GSS-API and SASL. It
allows every GSS-API mechanism that supports mutual auth and channel
bindings to be used as a SASL mechanism
GS2-KRB5 is a way of using the SASL framework with Kerberos V5
authentication

EAP


PAP


CHAP


TACACS+


NTLM



FIT THIS SOMEWHERE - ways of doing authentication

Basic (aka Username/Password)

Digest

Public Key

Cookie








TOKEN TYPES
=======================

FIRST UNDERSTAND THE JWT FORMAT:

JSON Web Token (JWT)

Parts of JWT
header - contains type of token (bearer or mac) and the encryption
algorithm it uses
payload - provides authentication credentials and "claims" the server
will use to verify the user's identity
signature - contains a cryptographic key that can be used to validate
the authenticity of the information in the payload



THEN UNDERSTAND THE TOKEN TYPES


Bearer Token

MAC Token

API Key

Open ID Connect (OIDC)

====================


Flows in OIDC

Flows in OAuth
CIBA (Client Initiated Backchannel Authentication)
Hybrid - OAuth implicit + authorization flows

Tokens in OIDC

ID Token - An ID token is an artifact that proves that the user has been
authenticated
ID tokens are meant to be read by the OAuth client
ID tokens should not be sent to the resource/api
JWT format

Access Token - Access Token is an artifact that grants access when
presented to a resource server
Access tokens are meant to be read by the resource/api
Access tokens are not meant to be read by the OAuth client (e.g. a
browser, curl, etc.)
can be JWT format OR a random string

Refresh Token - Refresh Token are typically longer lived than Access
Tokens and used to request a new Access Token without forcing user
authentication






KERBEROS AUTHENTICATION
=========================

\*\*Terms

Principal - kerberos authenticates a principal
which is either:
user - primary is username , e.g. gazb
hosts - primary is word 'host' e.g. HOST
service - primary is servicename e.g. nfs
and
an optional identifier - usually specifies the host name of the system
the primary is associated with

Service Principal - an identity assigned to an application service that
is accessed through Kerberos
Service Principal Name - SPN is a unique identifier of a service
instance. SPN is used by Kerberos to associate a service instance with a
service logon account, e.g. nfs/server1.example.com@TRANSPOWER.CO.NZ or
HTTP/server2.example.com@AARNET.EDU.AU


Realm - identified by DNS named domains, a namespace for authentication,
upper case by convention

Kerberos application server - any system providing access to resources
that need client auth through kerberos, e.g. file server, print server,
sshd server, custom application, etc.



\*\*Components

Client (kinit??)


Kerberos Application Server


KDC
Runs as a single process and provides 2 services
Authentication Server - provides the TGT
Ticket Granting Server - application server that issues service tickets
and a database
Kerberos database (aka Admin Server) - the database has the record of
each principal. It is a centralized repository of kerberos and contains
the identification of clients and their access




Kerberos Ticket
Think of this as a certificate issued by an authentication server,
encrypted using the server key
issues to a client
2 types of tickets
TGT
Service Ticket


Ticket Granting Ticket (authentication ticket)
the following are encrypted by the TGS secret key
client ID
client network address
timestamp
lifetime


Service Ticket
Principal - could be a user, host or service
session key - encrypted by the KDC server key, used for authentication
of the principal to the verifier
timestamp -



\*\*Secret Keys

Client/User - hash derived from the user's password
TGS secret key - hash of the password employed in determining the TGS
Server secret key - hash of the password used to determine the server
providing the service



\*\*Session Keys

session key 1
session key 2 - a service session key



\*\*Summary of flow

client \-\-- Authentication Server Request \-\--\> KDC/Authentication
Server
client \<\-\-- Authentication Server Response (TGT and Session Key)
\-\-- KDC/Authentication Server
client \-\-- Service Ticket Request \-\--\> KDC/TGS

client \<\-\-- Service Ticket Response (Service Ticket and credentials)
\-\-- KDC/TGS (encrypted by session key)

client \-\-- Application Server Request (Service Ticket) \-\--\>
Application Server

client \<\-\-- Application Server Response (kerberos auth of app server)
\-\-- Applicaiton Server optional, only if requested by the client




\*\*Keytab

A keytab is a file containing pairs of Kerberos principals and encrypted
keys that are derived from the kerberos password.
The key is derived from a password using a certain type of encryption,
e.g. aes256-cts-hmac-sha1-96
This keytab file can be used to log on to kerberos without being
prompted for a password
The most common use of keytab files is to allow scripts to authenticate
to Kerberos without human interaction or without storing the password in
a plain text file.




Service Principal Name
- a unique identifier of a service instance offerred by a particular
host within an authentication domain
- SPN is used by Kerberos authentication to associate a service instance
with a service logon account
- this allows a client application to request that the service
authenticate an account even if the client does not have the account
name
\<service class\>/\<fdqn\>@REALM
IMAP/mail.example.com@EXAMPLE.COM


User Principal Name
- an entity performing client requests to some service.
- entity may be human, service account, machine
\<name\>@\<REALM\>
Ghazanfar.Banatwala@transpower.co.nz
OR
\<name\>/\<privilege\>@REALM
gaz.banat/admin@AARNET.EDU.AU

NOTE: a UPN retrieves a service ticket for an SPN to use that actual
service



Windows User:
CN
displayName
distinguishedName
givenName
name
sAMAccountName
servicePrincipalName
userPrincipalName




AUTHENTICATION - HTTP
======================

FIRST UNDERSTAND THIS

Headers
Server:
WWW-Authenticate: lists all the authentication methods that a server
supports
Negotiate
Bearer
Basic
Basic realm=\<realm\>

Client:
Authorization: the authorization mechanism the client is using
Negotiate \<spnego-token\>
Bearer \<token\>



NOW THE METHODS (aka Authentication Schemes)


Basic - curl -u \<username\> -p \<password\>


Bearer - curl -H 'Authorization: Bearer \<token\>' (API Keys are kinds
of tokens)


Digest


Negotiate (SPNEGO) - curl -H "Authorization: Negotiate
\<spnego-token\>'"


HTTP Challenge



HOBA


Mutual


AWS4-HMAC-SHA256


Client Cert


NTLM


Cookie






AUTHENTICATION - SSH
====================


username/password

private/public key

gssapi-keyex

gssapi-with-mic











WIRELESS AUTHENTICATION METHODS
=================================


Open Authentication



WPA2-PSK (Pre Shared Key)












