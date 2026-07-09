

WHY GPG
It is good for encryption and decryption - public key will encrypt and
secret key will decrypt
signing and verifying messages, content, etc. - secret key will sign and
public key will verify


It is asymmetric - there is a secret key (think private) and a public
key



ENVIRONMENT VARIABLES

1. GNUPGHOME - where is the .gnupg directory for GPG

export GNUPGHOME=./.gnupg



COMPONENTS

key - has encryption type, size, expiration date, passphrase,
keychain
configuration file - \~/.gnupg/gpg.conf
keybox - \~/.gnupg/pubring.kbx
trustdb - \~/.gnupg/trustdb.gpg
cert revocation dir - \~/.gnupg/openpgp-revocs.d


WHAT IS A SUBKEY?



COMMON TASKS

generate a keypair (secret and public) for yourself

gpg \--quick-generate-key \--batch \--passphrase "\[passphrase for
key\]" \<email\>
or
gpg \--full-generate-key


list public keys in my keychain
gpg \--list-keys \--fingerprint

list secret keys in my keychain
gpg \--list-secret-keys \--fingerprint


export a key (this gives the public part of a key)
gpg \--export \--armor \<email\> \> file.asc


import someones public key
gpg \--import file.asc



edit a key
gpg \--edit-keys \<email\>



VALIDATE A VENDOR

Get the vendor's public key
curl get the key

import the key into your GPG Keychain
gpg \--import \<file\>

sign the imported key
gpg \--sign-key \[ID of imported key\]

verify the public key ID and fingerprint with gpg

gpg \--fingerprint \--list-signatures \"HashiCorp Security\"

Check the output against the vendors page

Download the vendors product

Download the checksum for the product

Download the checksum signature file (this is a .sig file)

Verify the signature file and the checksums file
gpg \--verify checksum_file.sig checksum_file

verify the sha256 checksum of the vendor archive
shasum \--algorithm 256 \--check checksum_file
