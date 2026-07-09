
HASHING

Changing a message into a string so that the message can be verified



Algorithms

md5

sha1

sha256


Use case
- software download


ENCRYPTION

SYMMETRIC

Only a single key is used


Symmetric Algorithms

DES - 56 bit key

3DES -

AES - 128/192/256 bit key sizes,

BLOWFISH -

TWOFISH -


Use case
- encrypting drives



ASYMMETRIC

2 Keys are used, a public key and a private key.
Server side keys - Public key will encrypt the message and the private
key will decrypt the message
Client side keys - Public key will be used for verifying and private key
for signing (aka authentication)

Asymmetric Algorithms

DSA -

RSA - 1024/2048/3072/7680/15360 bit key sizes
DER encoding - this is a binary encoding
PEM encoding - this is a base64 encoding of binary data


ECDSA - 160-233/224-255/256-383/384-511/512+ bit key sizes




Key Formats

PKCS#8 - key type is included in the key data
base64 encoded data

PKCS#1 - just an RSA key (only the key object from PKCS#8)
base64 encoded data

OPENSSH - for use with OpenSSH
private key
base64 encoded data but there is no DER data, rather it is a proprietary
OpenSSH format
public key
ssh-rsa abcd\...\...\...\...\...\...\...\.....xyz \[user@computer\]

PPK - for use with PuTTY



File Formats

PEM - header, base64 encoded DER data, footer


Use case
- Communication over the wire {SSL/TLS (protocol could be http, smtp,
ftp, etc.), PGP, SSH}
- Digital signing of files (DSA is better suited for this)




USE CASES FOR KEYS


encryption

decryption

signing This key usage allows a key to be used to create digital
signatures that verify the identity of the signer.

signature verification

non-repudiation Ensures that a signer cannot deny the authenticity of
their signature.

key encipherment Used for encrypting keys that can then be used for
decrypting data.

data decipherment Directly used for encrypting data to maintain
confidentiality





THE CASE FOR PGP

PGP uses a combination of symmetric key encryption and public key
encryption

\[gaz:\~\] \$ gpg \--help
gpg (GnuPG) 2.4.5
libgcrypt 1.10.3
Copyright (C) 2024 g10 Code GmbH
\...

Home: /Users/gaz/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
CAMELLIA128, CAMELLIA192, CAMELLIA256
AEAD: EAX, OCB
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224





KEY EXCHANGE

ECDHE_RSA

Diffie-Hellman



CIPHER

AES_256_GCM





CIPHER SUITE

A cipher suite is a grouping of configuration choices

compression

key exchange algorithm

bulk cipher


Some known ciphers

[[TLS_PSK_WITH_AES_128_CBC_SHA](https://ciphersuite.info/cs/TLS_PSK_WITH_AES_128_CBC_SHA/)]{.underline} 





CERTIFICATES


CERTIFICATE COMPONENTS

Subject Name
Issuer Name
Public Key Info
Extensions
Fingerprint
Extensions
subjectAltName
basicConstraints
keyUsage



CERTIFICATE FORMATS


DER - binary encoding

PEM - base64 ASCII encoding of DER binary encoding with header and
footer lines
- header and meta data information


PKCS7

PKCS10


PKCS12 - microsoft enhanced standard
- password container format
- fully encrypted
- for both public and private certificate pairs

P7b - format used by windows for certificate interchange




CERTIFICATE EXTENSIONS

.crt, .cer - certificate
.key
.csr




COMPONENTS OF A CSR

Version

Subject

Public Key Info

Attributes

Requested Extensions

Signature Algorithm




CERTIFICATE PROCESS BETWEEN CLIENT AND SERVER

1.  Client receives the public key of the server (public key is included
    in the certificate)

2.  Client generates a symmetric key

3.  Client encrypts the symmetric key with the public key of the server

4.  Client sends the encrypted symmetric key to the server

5.  Server decrypts the encrypted symmetric key (at this stage, client
    and server have the same key, so key exchange is done)

6.  Client and server use the symmetric key to encrypt their
    communication




OPENSSL

OPENSSL COMMANDS


1. Create a CA certificate
First create a key - openssl genrsa \[-aes256\] -out MY-CA.key 2048
Create certificate from the key - openssl req -x509 -new -nodes -key
MY-CA.key -sha256 -days 1826 -out MY-CA.crt -subj
'/C=NZ/L=WELLINGTON/O=Transpower/CN=TEMP CA
DEVIDE03/emailAddress=gaz.banat@transpower.co.nz'

Create a config file for the ca - \~/gazca/GAZ_CA.conf
Create directories accordingly - mkdir ca; cd ca; mkdir ca.db.certs;
touch ca.db.index; echo \"1234\" \> ca.db.serial


2. Create a cert to be signed by the CA
Create a key - openssl genrsa \[-aes256\] -out SERVER.key 2048
Create a csr - openssl req -new -key SERVER.key -out SERVER.csr -subj
"/C=NZ/ST=WELLINGTON/L=WELLINGTON/O=DR SCIENCE/OU=IT/CN=\<domain\>"
OR
with one command - openssl req -new -newkey rsa:2048 -nodes -keyout
yourdomain.key -out yourdomain.csr -subj \"/C=US/ST=Utah/L=Lehi/O=Your
Company, Inc./OU=IT/CN=yourdomain.com\"


3\. Sign the csr and return a crt

openssl ca -config GAZ_CA.conf -in my_request_1/server.csr -out
my_request_1/server.crt \[-days 1862 -keyfile GAZ_CA.key -keyform PEM
-cert GAZ_CA.crt\]



4. Create a self signed certificate
With a new key

openssl req -new -newkey rsa:2048 -nodes -keyout server.key -x509 -days
365 -out server.crt -subj \"/CN=www.gaz.net/emailAddress=me@gaz.net\"
With an existing key
openssl req -new server.key -x509 -days 365 -out server.crt -subj
\"/CN=www.gaz.net/emailAddress=me@gaz.net\"

