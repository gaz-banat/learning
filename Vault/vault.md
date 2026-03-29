

# GENERAL COMMANDS 

First time initialization of new vault. generates unseal keys and root token
vault operator init

Unseal a sealed vault
vault operatore unseal <unseal key>



# A NOTE on vault 'read' and 'write'

if reading/writing to an auth method the path will start with - auth/
if reading/writing for system functions then path will start with - sys/
if reading/writing to a secrets engine the path will start with where the engine is mounted - e.g. identity/



# UNDERSTANDING PATHS

## Root protected paths

auth/token/create-orphan			# create an orphan token
pki/root/sign-self-issued			# sign a self-issued certificate
sys/rotate							# rotate the encryption key
sys/seal							# manually seal Vault
sys/step-down						# force the leader to give up active status

NOTE: a capability of 'sudo' in a policy will allow access to root protected paths



## Path Patterns

* 				-	 	glob, matches one or more characters after the given path, e.g. secrets/application1/* matches anything AFTER secrets/application1
+				-		matches a single directory, e.g. secrets/application1/+/db, secrets/console/+/+/keycloak
{{<var>}}		-		templating/variable interpolation, e.g. secrets/application1/{{identity.entity.id}}/*
Note: https://developer.hashicorp.com/vault/tutorials/policies/policy-templating 



# TOKENS

## Types of tokens
hvs.XXXXXXXXXXX		-	service token		persisted to storage, renew/revoke, create child token, 
hvb.XXXXXXXXXXX		-	batch token			not persisted to storage, no renew/revoke, are blobs (binary large objects), ideal for high volume ops like encryption, 
											replicated to other clusters in a replica set
hvr.XXXXXXXXXXX		-	recovery token


## Service Tokens
service token - periodic		-		has a TTL but no max TTL, can be renewed indefinitely, 
service token - use limit
service token - orphan			-		not impacted by the lifecycle of its parent, can expire after is parent expires
service token - cidr-bound	 	-		bound to a specific network/s



## Paths for tokens
auth/token/create


## Manage Tokens
vault token lookup
vault token create												# without any arguments AND invoked as root, you will get back a root token
vault token create -policy=<policy> 							# create a token with <policy> attached and get back the token
vault token create -policy=<policy> -ttl=60m 					# create a token with <policy> attached and get back the token with a ttl of 60m
vault token create -policy=<policy> -period=<N>h				# create a periodic token, it has ttl but no max ttl


## Token Accessors
it is a reference to a token - can be use to lookup/renew/revoke/capabilities of a token
cannot be used to authenticate with vault

vault token lookup -accessor <accessor>


## Generate a root token
vault operator generate-root -init
vault operator generate-root
vault operator generate-root -decode=x -otp=y






# AUDIT DEVICES

## Audit Devices
vault audit list






# POLICIES

NOTE: There are 2 default policies:
- 'root' policy is the superuser policy and tied to root token
- 'default' policy is the default policy and provides common permissions


## Policies						
vault policy list									# list policies
vault list sys/policy								# list policies
vault policy write <policy> <policy_file>			# a policy file contents are formatted exactly as you might see from a read
vault policy read <policy>							# read the contents of <policy>


## Capabilities (think permissions)
create			-		allows creating data at a path							-	POST, PUT
read			-		allows reading data stored at a path					-	GET
update			-		allows changing data stored at a path					-	POST, PUT
delete			-		allows deleting data stored at a path					-	DELETE
patch			-		allows partial updates to data stored at a path			-	PATCH
list			-		allows listing subpaths under a path					-	This one is a bit tricky, list will not allow listing data stored at a path, only listing subpaths at a path
sudo 			-		allows access to root protected paths					-	This capability is used in addition to other capabilities needed for a specific path
deny			-		disallows access


## Path

### Policies are based on API paths
For a secret engine called "secret"
secret/data/...
secret/metadata/...
secret/delete/...
secret/undelete/...
secret/destroy/...

### using glob character "*" can only be used at end of path
secret/data/k8s		    -		only secret/k8s
secret/data/k8s-*	    -		secret/k8s-one, secret/k8s-two, secret/k8s-three and so on (but not secret/k8s)
secret/data/k8s/*	    -		secret/k8s/one, secret/k8s/two/three, secret/k8s/four/five/six and son on (but not secret/k8s)

### using wildcard character +, matches any path segement
secret/k8s/+/api	-	secret/k8s/dev/api, secret/k8s/test/api, etc.




## The root policy
NOTE: this is the equivalent of a root policy, "sudo" is for root protected paths, "deny" capability has obviously been left out
path "*" {
  capabilities = ["create", "read", "update", "delete", "patch", "list", "sudo"]
}




# WORKING WITH ENTITY AND ENTITY-ALIAS

## Entity (in the identity secrets engine)
vault list identity/entity/id
vault read identity/entity/id/<id>
vault write identity/entity name="<entity_name>" policies=<policy>		# creates an entity and attaches a policy to it


## Entity-Alias (in the identity secrets engine)
vault list identity/entity-alias/id				# show all the entity-aliases
vault read identity/entity-alias/id/<id>		# read the configuration of an entity-alias


Add an alias to an entity (e.g. with userpass auth method)
vault list auth/userpass/users		# select the username
vault auth list 					# get the accessor for userpass method
vault list identity/entity/id		# select the id of the entity

## now write the alias for the entity
vault write identity/entity-alias name=<username> canonical_id=<entity_id> mount_accessor=<accessor>






# WORKING WITH AUTH METHODS

## Auth Methods					# types of auth methods - userpass, ldap, okta, approle, oidc, jwt, tls certs, kubernetes, okta, radius, various cloud methods
vault auth list


## userpass auth method - example with auth method userpass mounted at path userpass/

vault auth enable [-description=<description>] userpass								# enable userpass at userpass/
vault auth tune -default-lease-ttl=24h userpass/									# tokens from userpass/ will expire after 1 day
vault write auth/userpass/users/<user> password=<password> policies=<policies>		# add a user to userpass auth method with password and policy
vault list auth/userpass/users														# list the local users in the auth method userpass
vault read auth/userpass/users/<user>												# read configuration for a user
vault login -method=userspass username=<user> password=<password>					# login via userpass auth method



## approle auth method - examples with auth method approle mounted at path approle/

vault auth enable approle												# Enable the auth method

vault write auth/approle/role/<role> \
policies=<policy> [token_ttl=<N>s|m|h] [token_max_ttl=<N>s|m|h] 		# Add an approle and attach a policy to it, think of a role as having some configuration settings specific to certain app/s

vault list auth/approle/role											# List the roles in approle

vault read auth/approle/role/<role>										# read the configuration of a role

vault read auth/approle/role/<role>/role-id								# Get the role-id for a role in approle

																		# think of the secret-id as the password, you can't get it back, so if you create with -f make sure to record it	 
vault write -field secret_id -f auth/approle/role/<role>/secret-id		# this will generate the secret-id and the secret accessor id, because -f was used

vault list auth/approle/role/<role>/secret-id							# Get the secret accessor id for a role

vault write auth/approle/login role_id=<role-id> secret_id=<secret-id>	# Login from cli using approle, Note: not the secret-id accessor



## jwt auth method - examples with auth method jwt mounted at path jwt-opsk8s-lab

```shell

# enable an auth method
vault auth enable -path=jwt-opsk8s-lab jwt

vault write auth/jwt-opsk8s-lab/config \
jwt_supported_algs=RS256 \
jwt_validation_pubkeys=@public_key.pem					# write the jwt auth method config, the public key is in the file public_key.pem	

vault read auth/jwt-opsk8s-lab/config					# read the config for the auth method

vault write auth/jwt-opsk8s-lab/role/eso \
bound_audiences=https://talos.k8s.aarnet.net.au:6443 \
bound_subject=system:serviceaccount:external-secrets:external-secrets \
policies=opsk8s/lab/eso \
user_claim=sub \										# create a role for the login method, think of a role as carrying some of the config info, e.g. bound_audience, bound_subject, etc.
role_type=jwt 											# at login time the role will be specified

# the eso role should be listed
vault list auth/jwt-opsk8s-lab/role						

vault read auth/jwt-opsk8s-lab/role/eso					# read the configuration for a role

vault write auth/jwt/login role=eso jwt=<jwt>			# login to vault using jwt authentication method, a vault token with opsk8s/lab/eso policy attached should be returned
OR
vault write -field token auth/jwt/login role=eso jwt=<jwt>	# still need to understand what -field is doing
```

NOTE: think about the jwt auth config as something that applies to every jwt token and 
the role on the jwt auth config as something that applies to config that can change between tokens






# WORKING WITH SECRET ENGINES

## Types of Secret Engines  				

kv
pki
cubbyhole
identity
generic
ssh
database
transit
system	


vault secrets list				# you should see the secret engines and the path at which they are available
  

## Secret Engine of type kv - examples with a secret engine called 'secret'

Enable a kv secret engine
vault secrets enable -path=secret -description="<description>" kv
vault secrets enable -path=secret -description="<description>" kv-v2

vault kv enable-versioning secret 				# upgrades the version for kv engine from 1 to 2, cannot downgrade after this


list the paths under the secret/ mount
vault kv list secret							# here the secret engine is called 'secret' 		
vault kv list -mount=secret

list the paths under secret/lxadm/ 
vault kv list secret/lxadm						# only the keys that are paths will be listed, if not path present in lxadm the command will fail with no value found 
vault kv list -mount=secret lxadm/

vault kv list secret/lxadm/applications			# so you can keep listing paths
vault kv list -mount=secret lxadm/applications/ 

for a kv store use 'get' instead of list
vault kv get secret/cloudstor/grub				# this will show the key and value pairs at the cloudstor/grub
vault kv get -mount=secret cloudstor/grub		# is the same as above, but uses -mount flag 
vault kv get [-version=x] secret/cloudstor/grub	# optionally specify version if using kv-v2

to write a key value pair
vault kv put secret/test-creds/store1 username=gaz password=happy123
vault kv put secret/test-creds/store1 @passwords.json

vault write secret/credentials1 username=gaz password=happy123	# This is an older way, suggest not using it since it does not seem to work with kv-v2

to delete a kv store
vault kv delete secret/cloudstor/grub			# for kv-v2 this is a soft delete and can be recovered using undelete, for version1 it is gone burgers


Specific to kv-v2
vault kv undelete -versions=<versions> 
vault kv destroy -versions=1,2,3							# -versions flag is compulsary
vault kv patch												# this is 'patching' the existing kv store

vault kv rollback -version=<version_number>	

vault kv metadata get /path/to/kv_store						# gets the metadata for a kv store
vault kv metadata put -custom-metadata=<key>=<value>
vault kv metadata delete /path/to/kv_store					# in kv-v2 this command removes metadata of the kv store as well, now it cant be listed after destroy

NOTE: for kv-v2 use <mount>/data/ and <mount>/metadata when working with API or writing policies, not required for vault cli




## Secret Engine of type cubbyhole

a kv store on a per token basis for arbitrary secrets
lifetime of item in cubbyhole is linked to the token that wrote it

vault write cubbyhole/<path> <key>=<value>
vault read cuubbyhole/<path>

wrapping token - single use token with ttl

vault kv get -wrap-ttl=5m secret/alpha/beta		# returns a wrapping token of single use with 5m ttl which will get the key value pairs from beta into its own cubbyhole
vault unwrap <wrapping-token>					# get back the key value pairs in beta using the wrapping token
OR
export VAULT_TOKEN=<wrapping-token> vault unwrap
OR
vault login <wrapping-token>; vault unwrap




## Secret Engine of type transit

receives data from app (data must be base64 encoded) , encrypts and returns ciphertext, enc key is only in vault
encrypted data is NOT stored in vault
apps need to have permissions to use key for enc/dec ops, permissions are via policy attached to the app token

create/rotate/delete/export of an encryption key is possible

keys can be rotated to newer versions
keyring - contains all versions of an encryption key
rewrap ciphertext -	send ciphertext encrypted with an older version of a key and get back ciphertext encrypted with a newer version of the key


covergent encryption mode -	get back the same ciphertext when you encrypt the same data (so you can have searchable ciphertext)

vault secrets enable transit							# enabled at default path of transit/

### with key type aes256-gcm96
vault write -f transit/keys/keyone						# force the creation of a new enc key at transit/keys/ called keyone, type is aes256-gcm96
vault list transit/keys									# list the keys in transit/ secret engine
vault read transit/keys/keyone							# read the configuration of keyone, note to write config you will need /config at end of path

vault write transit/encrypt/keyone \					
plaintext=$(base64 <<< "This is the secret")			# encrypt at transit/encrypt

vault write transit/decrypt/keyone \
ciphertext=<ciphertext> | jq -r .data.plaintext \
base64 -d												# decrypt at transit/decrypt

vault write -f transit/keys/keyone/rotate				# rotate to a new version
vault write transit/keys/keyone/config \
min_decryption_version=<N>								# set minimum key version to decrypt with

vault write transit/rewrap/keyone \
ciphertext=<ciphertext>									# rewrap ciphertext to latest version

vault write transit/keys/keyone/config \
deletion_allowed=true									# Allow key to be deleted
vault delete transit/keys/keyone						# the config property {"deletion_allowed": true} should be set or it wont work


### with key type rsa
vault write -f transit/keys/keytwo type="rsa-4096"		# this time with type of key specified, since rsa is asymmetric you will get back public key






## Secret Engine of type pki

### Enable a secret engine of type pki and tune it
vault secrets enable -path=pki_int -description="intermediate ca" pki		# enable a pki secret engine at the default path of pki_int/

vault secrets tune -max-lease-ttl=87600h pki_int							# enable certificate life for max 10 years

### Now add the root ca
vault write -field=certificate \
pki_int/root/generate/internal \
common_name="rootca.gaz.net.nz" ttl=87600h > rootca.gaz.net.nz.crt			# generate a self signed root ca

vault write pki_int/config/urls \
issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" \
crl_distribution_points="http://127.0.0.1:8200/v1/pki/crl"					# configure the urls for where certificates will be issued and crl information


### Now add the intermediate ca

vault write -format=json \
pki_int/intermediate/generate/internal \
common_name="intermediateca.gaz.net.nz" \
| jq -r '.data.csr' > intermediateca.gaz.net.nz.csr							# generate a csr for the intermediate ca 


vault write -format=json \
pki_int/root/sign-intermediate \
csr=@intermediate.gaz.net.nz.csr format=pem_bundle ttl=43800h \
| jq -r '.data.certificate' > intermediateca.gaz.net.nz.crt					# ask root to sign intermediate


vault write pki_int/intermediate/set-signed \
certificate=@intermediateca.gaz.net.nz.crt									# set the signed certificate for the intermediate ca

### Now add a role for the pki secret engine                                
vault write pki_int/roles/gaznetnz \                                        # a role carries some of the configuration, e.g. you might configure a role for a specific domain
allowed_domains="gaz.net.nz" \
allow_subdomains=true \						
max_ttl="26280h" \
ou="IT Dept" \																# create a role to allow signing certs for gaz.net.nz domain
organization="GAZNET"														# if reconfiguring role make sure to include all the parameters not just one	

OR

vault write pki_int/roles/gaznetnz \
issuer_ref="$(vault read -field=default pki_int/config/issuers)" \
allowed_domains="gaz.net.nz" \
allow_subdomains=true \
allow_ip_sans=false \
allow_localhost=false \
generate_lease=false \
client_flag=true \
server_flag=true \
key_bits=4096 \
max_ttl=8784h \
use_csr_sans=true \
no_store=false \
organization="GAZNET"


	
vault list pki_int/roles													# show the available roles on the intermediate ca

vault read pki_int/roles/gaznetnz											# read the configuration of a role

### and now if you want a certificate issued 
vault write pki_int/issue/gaznetnz \
common_name="web.gaz.net.nz" ttl="26000h"									# issue a cert from the intermediate ca using the role gaznetnz

### OR

### you have a CSR and you just want the issuer to sign the csr
vault write pki_int/sign/gaznetnz \
common_name="web.gaz.net.nz" \
csr=@web.gaz.net.nz.csr


### General commands
vault list pki_int/certs													# list the certificates issued by pki_int/
vault read pki_int/cert/<serial-num>										# read a cert
vault read [-field=<field>] pki_int/config/issuers
vault read pki_int/issuer/default											# read the default issuer for pki secret engine at pki/
vault read pki_int/issuer/default											# read the default issuer for pki secret engine at pki_int/
vault pki list-intermediates pki/issuer/default								# list intermediates of root ca at mount path pki/
vault pki health-check pki_int												# get back the health status and paths for the pki_int secret engine
curl -k https://vault.aarnet.edu.au/v1/pki_int/ca_chain                     # Get the CA Chain


### Create vault policy that allows writing to a pki secret engine
```shell
## cluster specific policy
vault policy write pki_k8s_opsk8slab -<<___EOT
path "pki_k8s/*" {
    capabilities = ["read", "list"]
}

path "pki_k8s/issue/opsk8slab" {
    capabilities = ["create"]
}

path "pki_k8s/sign/opsk8slab" {
    capabilities = ["create", "update"]
}
___EOT
```





## Secret Engine of type aws

1. Enable the aws secret engine
vault secret enable -path=aws -description="meaningful description" aws

2. Configure the engine 
vault write aws/config/root access_key=<access_key> secret_key=<secret_key> region=<region>



# WORKING WITH SYSTEM FUNCTIONS

## Paths
sys/license			- 		configure license
sys/init			-		initialize vault
sys/config/ui		-		configure UI in vault
sys/rekey/*			-		allows rekey of unseal keys for vault
sys/rotate			-		allows rotation of master key
sys/seal			-		allows sealing of vault



