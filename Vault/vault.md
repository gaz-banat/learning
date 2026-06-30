vault list -format json auth/token/accessors | \
jq -r .[] | \
xargs -I '{}' vault token lookup -format json -accessor '{}' | \
jq -r 'select(.data.policies | any(. == "root"))

# GENERAL COMMANDS 

```shell
# First time initialization of new vault - generates unseal keys and root token
vault operator init

# Unseal a sealed vault
vault operatore unseal <unseal key>
```


# A NOTE on vault 'read' and 'write'

if reading/writing to an auth method the path will start with - auth/
if reading/writing for system functions then path will start with - sys/
if reading/writing to a secrets engine the path will start with where the engine is mounted - e.g. identity/



# UNDERSTANDING PATHS

## Root protected paths

auth/token/create-orphan			# create an orphan token
pki/root/sign-self-issued			# sign a self-issued certificate
sys/rotate							      # rotate the encryption key
sys/seal							        # manually seal Vault
sys/step-down						      # force the leader to give up active status

NOTE: a capability of 'sudo' in a policy will allow access to root protected paths



## Path Patterns

* 				    -	 	glob, matches one or more characters after the given path, e.g. secrets/application1/* matches anything AFTER secrets/application1
+				      -		matches a single directory, e.g. secrets/application1/+/db, secrets/console/+/+/keycloak
{{<var>}}		  -		templating/variable interpolation, e.g. secrets/application1/{{identity.entity.id}}/*
Note: https://developer.hashicorp.com/vault/tutorials/policies/policy-templating 



# TOKENS

## Types of tokens
hvs.XXXXXXXXXXX		        -	service token		persisted to storage, renew/revoke, create child token, 
hvb.XXXXXXXXXXX		        -	batch token			not persisted to storage, no renew/revoke, are blobs (binary large objects)
                                            ideal for high volume ops like encryption, 
											                      replicated to other clusters in a replica set
hvr.XXXXXXXXXXX		        -	recovery token


## Service Tokens
service token - periodic		  -		has a TTL but no max TTL, can be renewed indefinitely, 
service token - use limit
service token - orphan			  -		not impacted by the lifecycle of its parent, can expire after is parent expires
service token - cidr-bound	 	-		bound to a specific network/s



## Paths for tokens
auth/token/create


## Manage Tokens
```shell
vault token lookup

# without any arguments AND invoked as root, you will get back a root token
vault token create							

# create a token with <policy> attached and get back the token
vault token create -policy=<policy> 

# create a token with <policy> attached and get back the token with a ttl of 60m
vault token create -policy=<policy> -ttl=60m 

	# create a periodic token, it has ttl but no max ttl
vault token create -policy=<policy> -period=<N>h			
```

## Token Accessors
it is a reference to a token - can be use to lookup,renew,revoke,capabilities of a token
cannot be used to authenticate with vault

```shell
vault token lookup -accessor <accessor>
```

## Generate a root token
```shell
# Get an OTP for decoding
vault operator generate-root -init
# run this command 3 times, each time with a different unseal key
vault operator generate-root
# decode the the encoded string from the 3rd run from above using the otp
vault operator generate-root -decode=<encoded_string> -otp=<otp>
```





# AUDIT DEVICES

## Audit Devices
```shell
vault audit list
```



# POLICIES

NOTE: There are 2 default policies:
- 'root' policy is the superuser policy and tied to root token
- 'default' policy is the default policy and provides common permissions


## Capabilities (think permissions)
create			-		allows creating data at a path							        -	POST, PUT
read			  -		allows reading data stored at a path					      -	GET
update			-		allows changing data stored at a path					      -	POST, PUT
delete			-		allows deleting data stored at a path					      -	DELETE
patch			  -		allows partial updates to data stored at a path			-	PATCH
list			  -		allows listing subpaths under a path					      -	This one is a bit tricky, list will not allow listing data stored at a path, only listing subpaths at a path
sudo 			  -		allows access to root protected paths					      -	This capability is used in addition to other capabilities needed for a specific path
deny			  -		disallows access


## Understanding Paths in Policies

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


## A standard policy
```shell
## Allow read, create, update and patch beneath secret/k8s/opsk8sdev/auscert-parser
path "secret/data/k8s/opsk8sdev/auscert-parser/*" {
  capabilities = ["read", "create", "update", "patch"]
}

# allow listing of metadata at a path
path "secret/metadata/k8s/opsk8sdev/auscert-parser" {
  capabilities = ["list"]
}

# allow listing of metadata at subpaths
path "secret/metadata/k8s/opsk8sdev/auscert-parser/*" {
  capabilities = ["list"]
}
```

## Commands
```shell				
vault policy list									# list policies
vault list sys/policy								# list policies
vault policy write <policy> <policy_file>			# a policy file contents are formatted exactly as you might see from a read
vault policy read <policy>							# read the contents of <policy>
```


# WORKING WITH ENTITY AND ENTITY-ALIAS

## Entity (in the identity secrets engine)
```shell
# creates an entity and attaches a policy to it
vault write identity/entity name="<entity_name>" policies=<policy>
# list entities
vault list identity/entity/id
# read entity
vault read identity/entity/id/<id>
```

## Entity-Alias (in the identity secrets engine)
```shell
# show all the entity-aliases
vault list identity/entity-alias/id			
# read the configuration of an entity-alias
vault read identity/entity-alias/id/<id>		


# Add an alias to an entity (e.g. with userpass auth method)
vault list auth/userpass/users		# select the username
vault auth list 					        # get the accessor for userpass method
vault list identity/entity/id		  # select the id of the entity

## now write the alias for the entity
vault write identity/entity-alias name=<username> canonical_id=<entity_id> mount_accessor=<accessor>
```





# WORKING WITH AUTH METHODS

## Type of Auth Methods					

userpass
ldap
token
approle
oidc
jwt
tls certs
kubernetes
okta
radius
various cloud methods (e.g. aws)

```shell
vault auth list
```

## userpass auth method - example with auth method userpass mounted at path userpass/

```shell
# enable userpass auth method at path userpass/
vault auth enable [-description=<description>] userpass

# tokens from userpass/ will expire after 1 day
vault auth tune -default-lease-ttl=24h userpass/

# add a user to userpass auth method with password and policy
vault write auth/userpass/users/<user> password=<password> policies=<policies>		

# list the local users in the auth method userpass
vault list auth/userpass/users

# read configuration for a user
vault read auth/userpass/users/<user>

# login via userpass auth method
vault login -method=userspass username=<user> password=<password>					
```


## approle auth method - examples with auth method approle mounted at path approle/

```shell
# Enable the auth method
vault auth enable approle												

# Add an approle and attach a policy to it, think of a role as having some configuration settings specific to certain app/s
vault write auth/approle/role/<role> \
policies=<policy> [token_ttl=<N>s|m|h] [token_max_ttl=<N>s|m|h] 		

# List the roles in approle
vault list auth/approle/role											

# read the configuration of a role
vault read auth/approle/role/<role>									

# Get the role-id for a role in approle
vault read auth/approle/role/<role>/role-id								

# this will generate the secret-id and the secret accessor id, because -f was used
# think of the secret-id as the password, you can't get it back, so if you create with -f make sure to record it	 
vault write -field secret_id -f auth/approle/role/<role>/secret-id		

# Get the secret accessor id for a role
vault list auth/approle/role/<role>/secret-id							

# Login from cli using approle, Note: not the secret-id accessor
vault write auth/approle/login role_id=<role-id> secret_id=<secret-id>	
```


## jwt auth method - examples with auth method jwt mounted at path jwt-opsk8s-lab

```shell
# enable an auth method
vault auth enable -path=jwt-opsk8s-lab jwt

# write the jwt auth method config, the public key is in the file public_key.pem
vault write auth/jwt-opsk8s-lab/config jwt_supported_algs=RS256 jwt_validation_pubkeys=@public_key.pem						

# read the config for the auth method
vault read auth/jwt-opsk8s-lab/config

# create a role for the login method, think of a role as carrying some of the config info, e.g. bound_audience, bound_subject, etc.
# at login time the role will be specified
vault write auth/jwt-opsk8s-lab/role/eso \
bound_audiences=https://talos.k8s.aarnet.net.au:6443 \
bound_subject=system:serviceaccount:external-secrets:external-secrets \
policies=opsk8s/lab/eso \
user_claim=sub \										
role_type=jwt 											

# list available roles on the auth method
vault list auth/jwt-opsk8s-lab/role						

# read the configuration for a role
vault read auth/jwt-opsk8s-lab/role/eso				

# login to vault using jwt authentication method, a vault token with opsk8s/lab/eso policy attached should be returned
vault write auth/jwt-opsk8s-lab/login role=eso jwt=<jwt>			
OR
vault write -field token auth/jwt-opsk8s-lab/login role=eso jwt=<jwt>	# still need to understand what -field is doing


# delete the role on an auth method
vault delete auth/jwt-opsk8s-lab/role/eso

# disable an auth method
vault auth disable auth/jwt-opsk8s-lab
```

NOTE: think about the jwt auth config as something that applies to every jwt token and 
the role on the jwt auth config as something that applies to config that can change between tokens






# WORKING WITH SECRET ENGINES

## Types of Secret Engines  				

kv
pki
cubbyhole
transit
aws
identity
generic
ssh
database
system	

```shell
# you should see the secret engines and the path at which they are available
vault secrets list			
```

## Secret Engine of type kv - examples with a secret engine called 'secret'

```shell
# Enable a kv secret engine
vault secrets enable -path=secret -description="<description>" kv
vault secrets enable -path=secret -description="<description>" kv-v2

# upgrades the version for kv engine from 1 to 2, cannot downgrade after this
vault kv enable-versioning secret

# list the paths under the secret/ mount
vault kv list secret							# here the secret engine is called 'secret' 		
vault kv list -mount=secret

# list the paths under secret/lxadm/ 
vault kv list secret/lxadm						# only the items that are paths will be listed, if not path present in lxadm the command will fail with no value found 
vault kv list -mount=secret lxadm/

vault kv list secret/lxadm/applications			# so you can keep listing paths
vault kv list -mount=secret lxadm/applications/ 

# for a kv store use 'get' instead of list
vault kv get secret/cloudstor/grub				# this will show the key and value pairs at the cloudstor/grub
vault kv get -mount=secret cloudstor/grub		# is the same as above, but uses -mount flag 
vault kv get [-version=x] secret/cloudstor/grub	# optionally specify version if using kv-v2

# to write a key value pair
vault kv put secret/test-creds/store1 username=gaz password=happy123
vault kv put secret/test-creds/store1 @passwords.json

vault write secret/credentials1 username=gaz password=happy123	# This is an older way, suggest not using it since it does not seem to work with kv-v2

# to softdelete a kv store
vault kv delete secret/cloudstor/grub			# for kv-v2 this is a soft delete and can be recovered using undelete, for version1 it is gone burgers

# recover a soft deleted kv store
vault kv undelete -versions=<versions> 

# hard delete a kv store
vault kv get secret/path/to/kv_store                                  # note the versions you want to delete
vault kv destroy -versions=1,2,3 secret/path/to/kv_store 							# -versions flag is compulsary
vault kv metadata delete secret/path/to/kv_store					            # in kv-v2 this command removes metadata of the kv store as well
                                                                      # now it cant be listed after destroy

# patch a kv store
vault kv patch												# this is 'patching' the existing kv store

# rollback to a previous version of a kv store
vault kv rollback -version=<version_number>	secret/path/to/kv_store

# work with metadata for a kv store 
vault kv metadata get secret/path/to/kv_store						# gets the metadata for a kv store
vault kv metadata put -custom-metadata=<key>=<value>


# NOTE: for kv-v2 use <mount>/data/ and <mount>/metadata when working with API or writing policies, not required for vault cli
```


## Secret Engine of type pki

```shell
# Enable a pki secret engine at the default path of pki_int/
vault secrets enable -path=pki_int -description="intermediate ca" pki		

# enable certificate life for max 10 years
vault secrets tune -max-lease-ttl=87600h pki_int							

## First the root CA

# Now add the root ca - generate a self signed root ca
vault write -field=certificate pki_int/root/generate/internal common_name="rootca.gaz.net.nz" ttl=87600h > rootca.gaz.net.nz.crt			

# Configure the urls for where certificates will be issued and crl information
vault write pki_int/config/urls issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" \
crl_distribution_points="http://127.0.0.1:8200/v1/pki/crl"					


## Then intermediate ca

# generate a csr for the intermediate ca 
vault write -format=json pki_int/intermediate/generate/internal common_name="intermediateca.gaz.net.nz" \
| jq -r '.data.csr' > intermediateca.gaz.net.nz.csr							

# ask root to sign intermediate
vault write -format=json pki_int/root/sign-intermediate csr=@intermediate.gaz.net.nz.csr format=pem_bundle ttl=43800h \
| jq -r '.data.certificate' > intermediateca.gaz.net.nz.crt				


# set the signed certificate for the intermediate ca
vault write pki_int/intermediate/set-signed certificate=@intermediateca.gaz.net.nz.crt									

## Now add a role for a domain for the pki secret engine

# create a role to allow signing certs for gaz.net.nz domain
vault write pki_int/roles/gaznetnz \
allowed_domains="gaz.net.nz" \
allow_subdomains=true \						
max_ttl="26280h" \
ou="IT Dept" \																
organization="GAZNET"														


# OR

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

# NOTE: a role carries some of the configuration, e.g. you might configure a role for a specific domain   
# NOTE: if reconfiguring role make sure to include all the parameters not just one	

# show the available roles on the intermediate ca
vault list pki_int/roles													

# read the configuration of a role
vault read pki_int/roles/gaznetnz											

## and now if you want a certificate issued

# issue a cert from the intermediate ca using the role gaznetnz
vault write pki_int/issue/gaznetnz common_name="web.gaz.net.nz" ttl="26000h"									

# OR

# you have a CSR and you just want the issuer to sign the csr
vault write pki_int/sign/gaznetnz common_name="web.gaz.net.nz" csr=@web.gaz.net.nz.csr
```

### General commands
```shell
# list the certificates issued by pki_int/
vault list pki_int/certs

# read a cert					
vault read pki_int/cert/<serial-num>										
vault read [-field=<field>] pki_int/config/issuers

# read the default issuer for pki secret engine at pki_int/
vault read pki_int/issuer/default											

# list intermediates of root ca at mount path pki/
vault pki list-intermediates pki/issuer/default								

# get back the health status and paths for the pki_int secret engine
vault pki health-check pki_int									
# Get the CA Chain			
curl -k https://vault.aarnet.edu.au/v1/pki_int/ca_chain                     
```

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

## Secret Engine of type cubbyhole

a kv store on a per token basis for arbitrary secrets
lifetime of item in cubbyhole is linked to the token that wrote it

```shell
vault write cubbyhole/<path> <key>=<value>
vault read cuubbyhole/<path>
```

wrapping token - single use token with ttl

```shell
# returns a wrapping token of single use with 5m ttl which will get the key value pairs from beta into its own cubbyhole
vault kv get -wrap-ttl=5m secret/alpha/beta

# get back the key value pairs in beta using the wrapping token
vault unwrap <wrapping-token>					
# OR
export VAULT_TOKEN=<wrapping-token> vault unwrap
# OR
vault login <wrapping-token>; vault unwrap
```



## Secret Engine of type transit

receives data from app (data must be base64 encoded), encrypts and returns ciphertext, enc key is only in vault
encrypted data is NOT stored in vault
apps need to have permissions to use key for enc/dec ops, permissions are via policy attached to the app token

create,rotate,delete,export of an encryption key is possible

keys can be rotated to newer versions
keyring - contains all versions of an encryption key
rewrap ciphertext -	send ciphertext encrypted with an older version of a key and get back ciphertext encrypted with a newer version of the key


covergent encryption mode -	get back the same ciphertext when you encrypt the same data (so you can have searchable ciphertext)

```shell
# enabled at default path of transit/
vault secrets enable transit							
```

### with key type aes256-gcm96
```shell
# force the creation of a new enc key at transit/keys/ called keyone, type is aes256-gcm96
vault write -f transit/keys/keyone			
# list the keys in transit/ secret engine			
vault list transit/keys						
# read the configuration of keyone, note to write config you will need /config at end of path			
vault read transit/keys/keyone							

# encrypt at transit/encrypt
vault write transit/encrypt/keyone plaintext=$(base64 <<< "This is the secret")			

# decrypt at transit/decrypt
vault write transit/decrypt/keyone ciphertext=<ciphertext> | jq -r .data.plaintext base64 -d												

# rotate to a new version
vault write -f transit/keys/keyone/rotate

# set minimum key version to decrypt with
vault write transit/keys/keyone/config min_decryption_version=<N>								

# rewrap ciphertext to latest version
vault write transit/rewrap/keyone ciphertext=<ciphertext>									

# Allow key to be deleted
vault write transit/keys/keyone/config deletion_allowed=true

# the config property {"deletion_allowed": true} should be set or it wont work
vault delete transit/keys/keyone						
```

### with key type rsa
```shell
# this time with type of key specified, since rsa is asymmetric you will get back public key
vault write -f transit/keys/keytwo type="rsa-4096"		
```









## Secret Engine of type aws

```shell
# Enable the aws secret engine
vault secret enable -path=aws -description="meaningful description" aws

# Configure the engine 
vault write aws/config/root access_key=<access_key> secret_key=<secret_key> region=<region>
```


# WORKING WITH SYSTEM FUNCTIONS

## Paths
sys/license			- 		configure license
sys/init			-		initialize vault
sys/config/ui		-		configure UI in vault
sys/rekey/*			-		allows rekey of unseal keys for vault
sys/rotate			-		allows rotation of master key
sys/seal			-		allows sealing of vault
