

BASIC UNDERSTANDING OF RHSSO 

Application

Relying Party
Identity
	Identity Server
	Identity Provider
	Identity Provider Federation 				-		a way to delegate authentication to another IDP

Realm											-		a namespace for a set of users, credentials, roles and groups.
	
User federation mapper 							-		a mapper that can map some ldap attribute to a ...... (User Model property/attribute ??)
	LDAP Mapper/group-ldap-mapper				-		take some CN's found in an LDAP DN and map them to local SSO groups 
	LDAP Mapper/user-attribute-ldap-mapper		-		take some attribute of an LDAP object (cn, givenName, mail, etc) and map it to a User Model property/attribute 
	LDAP Mapper/role-ldap-mapper				-		take some ldap attribute (think cn) of an ldap object (think CN) and map it to either a realm role or a client role 
	LDAP Mapper/certificate-ldap-mapper
	LDAP Mapper/full-name-ldap-mapper 
	LDAP Mapper/msad-user-account-control-mapper

Protocol Mapper 								-		for a given protocol, a mapper can map some user data (e.g. attribute) into a claim (OIDC protocol) or into an assertion (SAML protocol) 
														OR transform a request going between client and auth server
	Token Mapper/User Realm Role 
	Token Mapper/User Session Note 
	Token Mapper/User Address
	Token Mapper/Role Name Mapper 
	Token Mapper/User Client Role 
	Token Mapper/User Property					-		map some User Model property of a user to a token claim 
	Token Mapper/User Attribute 				-		map some User Model attribute of a user to a token claim 
	Token Mapper/User's Full Name 				-		map the User's first name and last name to OIDC token claim 'name'
	
UserStorageProvider

UserModel 
	property 									-		email, firstName, lastName 
	attribute 									-		profile (not to be confused with the client scope 'profile'), website 

UserSessionModel


User Federation 								-		a way of getting a list of users from some external source (as opposed to having a list in SSO)
	user federation mapper 


Role											-		a way of indicating permissions of a resource owner on a resource ( is this part of OAuth Spec?)
	Realm Role									-		on SSO, a role available to all clients in the realm
	Client Role 								-		on SSO, a role specific to a client
	Composite Role 								-		a role that can be associated with other roles (think how a group has users, a composite role has other roles in it)


User											-		a local SSO user could be created directly in SSO or come from LDAP 
	Role Mapping 								-		realm roles and client roles that have been ASSIGNED to a user, manual assignment (as opposed to token mapper assignment) can be done here
	
Group											-		a local SSO group could be created in SSO or come from LDAP 
	Default Group 								-		a newly created user is automatically added to a default group 
	Role Mapping								-		realm roles and client roles that have been ASSIGNED to a group, manual assignment (as opposed to token mapper assignment) can be done here


Client Scope									-		a way to limit the roles that get declared inside an access token based on the client invoking the request
														so a client cannot get all the roles of the user, only those that fall within the scope of what the client is allowed to have 
														this also helps to have re-usable settings between clients 
														the scope parameter can be specified in the request URL
	protocol mapper 
	scope (scope mappings) 						-		the list of realm roles and client roles available in the Client Scope, manual assignment can be done here 
	
	
Client
	protocol mapper 
	client scope 
	client role 
	
	
Claim on a token 
  from a realm role assigned to the user  

  from a protocol mapper on a client scope that is available to the client 
  from a client role assigned to the user  	
  from a protocol mapper on a client			
  
  EITHER from a user federation mapper for a realm role 
  OR from a user federation mapper for a client role	


Flow 											-		what does a client want to do 
  Login (Get a Token) 
    Browser
	Direct Grant 
	Docker Auth 
	HTTP Challenge 
	<Custom Flow> 
  Something Else 
    Registraition 
	Reset Credentials 
  Need to sort these 
	Clients 
	First Broker Login 
	
OAuth and OIDC Compliance 
	Standard Flow 								-		a way of fulfilling OAuth2 'Authorization Code Flow' specification 
	Implicit Flow 								-		a way of fulfilling OAuth2 'Implicit Flow' specification 
	Direct Access grant							-		a way of fulfilling the OAuth2 'Resource Owner Password Credentials Grant' (Password Grant) specification  
														clients use a REST invocation for direct grants
	Service Account								-		a way of fulfilling OAuth2 'Client Credentials Grant' specification 
	OAuth 2.0 Device Authorization Grant		-		a way of fulfilling OAuth2 'Device Authorization Grant' 
	OIDC CIBA Grant Enabled


==================	
	
MENU OF RHSSO

<realm>
  Configure
	Realm Settings
		General
		Login
		Keys 
		Email
		Themes
		Localization
		Cache
		Tokens
		Client Registration 
		Client Policies
		Security Defenses
	Clients 
		<client>
			Access Type									-	confidential 	- 	client requires a secret to initiate login protocol 
															public			-	client does not require a secret
															bearer only		-	clients are web services that never initiate a login
			Settings
				Authentication Flow Overrides for			
					Browser Flow
					Direct Grant Flow 
			
				Client Protocol
				Service Account Enabled					-	if yes, the client is able to obtain tokens on its own behalf																	{Client Credentials Grant}
															a user in the form service-account-<client> will be created
			Credentials									-	as a Client ID and Secret OR as a X509 certificate OR as a Signed JWT
			
			Keys
			
			Roles										-	these are Client roles (limited to the client, one client cannot see the role of another client) 
			Client Scopes
			Mappers										-	Token mapper (User client role,  User session note, 
			Scope
			Revocation
			Sessions
			Offline Access
			Clustering
			Installation
			Service Accounts Roles 						-	a Realm role or Client role can be applied to a client's service account 
															so when a client retrieves a token on its own behalf, the token will contain roles listed in 'Service Account Roles'
	
	Client Scopes 										-	a common set of protocol 'mappers and role' which are shared between clients
		<client_scope>
			settings
			mappers
				<mapper> 								-	maps some property from the ID Provider to a claim in an 'ID Token' or 'access token' or 'user info'
														-	could be a "User Attribute" mapper 
															e.g. 'profile' mappper maps the User Attribute "profile" from ldap to a token claim called "profile"
			scope 										-	scope mappings allow you to restrict which user role (realm and client) are included within the access token requested by the client 
	
	Roles												
		Realm roles										-	these are Realm roles, these can be used across all clients 
		Default roles 									-	realm roles that will be assigned to all users 
	
	Identity Providers 
	
	User Federation
		<ldap_provider>
			Settings 
			Mappers 									-	can be user-attribute-ldap-mapper, group-ldap-mapper, etc.
															e.g. User Federation/LDAP/LDAP Mappers/dsm-groups => ou=dsm,ou=_groups,dc=transpower,dc=co,dc=nz
		<kerberos_provider>
	
	Authentication
		can_have 
			Flows
				can_be
					Browser
					Direct Grant
					Registration
					Reset Credentials
					Clients
					First Broker Login
					Docker Auth 
					HTTP Challenge
				can_have
					Auth Type 
					Requirement
				
				
	
  Manage	
	Users
	  <user>
		Details
		Attributes
		Credentials
		Role Mappings 
			Realm Roles 								-	what realm roles are mapped to a token claim for this user 
			Client Roles 
				<client>								-	what client roles are mapped to a token claim for this user 
		Groups
		Consents
		Sessions 
	Groups
	  <group>
		Settings 
		Attributes
		Role Mappings
			Realm Roles 
			Client Roles 
				<client>
		Members 

=======================================

INTERNAL APPLICATIONS AND CLIENTS OF RHSSO

Applications
	Internal
		Security Admin Console 					-	this application is used to manage the master realm 
		Account Console 						-	per realm (including master), to help a user with their personal info, signing in, device activity, etc.
		
Client											-	each application uses a client to talk to SSO, internal applications have their own client as well
	Internal
		admin-cli 
		account 
		account-console 
		security-admin-console
		<realm>-realm							-	used by a user to manage a realm (via the master realm url https://sso.apps.sdcses.ocp.transpower.co.nz/auth/admin/master/console/#/realms/<realm>
		

========================================

ACCESS TOKEN 

Example 1 - from ndcprd-4 prod realm 
----------

read -s password			# this is my domain password 

curl -s \
-H "Content-Type: application/x-www-form-urlencoded" \
-d 'client_id=monitoring' -d 'client_secret=43f2fba6-bd62-4902-9ff1-23d0ebd603a3' -d 'username=banatg' -d "password=${password}" -d 'grant_type=password' \
'https://sso.apps.ndcprd.ocp.transpower.co.nz/auth/realms/prod/protocol/openid-connect/token'  | jq .

Response:
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICIxNDZaeGF3ZkFoTTBmLWJVc010MTlmV2ZvdURiOXdpSXdOTTk1OHRReTRjIn0.eyJleHAiOjE2ODI2NzEyODgsImlhdCI6MTY4MjY3MDY4OCwianRpIjoiOTMyMGRiMDktY2I0Ni00NTMzLTgyOTEtYzc3MDA5ZDhmYjA1IiwiaXNzIjoiaHR0cHM6Ly9zc28uYXBwcy5zZGNwcmQub2NwLnRyYW5zcG93ZXIuY28ubnovYXV0aC9yZWFsbXMvcHJvZCIsImF1ZCI6ImFjY291bnQiLCJzdWIiOiJiNzBhMTQwZC04OTMzLTRhMDAtYTM3Mi0yNjQ0MTJkNmJjODIiLCJ0eXAiOiJCZWFyZXIiLCJhenAiOiJtb25pdG9yaW5nIiwic2Vzc2lvbl9zdGF0ZSI6ImFmMTdmNGYxLTU0ZDgtNDZjNi1iNzZiLTJjNGQzNTJjYWViYiIsImFjciI6IjEiLCJhbGxvd2VkLW9yaWdpbnMiOlsiKiJdLCJyZWFsbV9hY2Nlc3MiOnsicm9sZXMiOlsib2ZmbGluZV9hY2Nlc3MiLCJ1bWFfYXV0aG9yaXphdGlvbiIsIm1rdC1hZG1pbiJdfSwicmVzb3VyY2VfYWNjZXNzIjp7Im1vbml0b3JpbmciOnsicm9sZXMiOlsibmFnaW9zX2FjY2VzcyIsImVsa19hY2Nlc3MiXX0sImFjY291bnQiOnsicm9sZXMiOlsibWFuYWdlLWFjY291bnQiLCJtYW5hZ2UtYWNjb3VudC1saW5rcyIsInZpZXctcHJvZmlsZSJdfX0sInNjb3BlIjoiZW1haWwgcHJvZmlsZSIsInNpZCI6ImFmMTdmNGYxLTU0ZDgtNDZjNi1iNzZiLTJjNGQzNTJjYWViYiIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6IkdheiBCYW5hdHdhbGEiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJiYW5hdGciLCJnaXZlbl9uYW1lIjoiR2F6IiwiZmFtaWx5X25hbWUiOiJCYW5hdHdhbGEiLCJlbWFpbCI6ImdoYXphbmZhci5iYW5hdHdhbGFAdHJhbnNwb3dlci5jby5ueiJ9.AdQwHVJ3yPTczsb8p7iJUyXUt2lfsNZXs0sOe2ObZqJaKcY4KArXlqbiFnoavE1nm4gFEDLuGRYFsI63bohD0BSQJWCATHDGhq0CH4adltUwMDutl8gfa180xLuDxWqdSTJPwMc6s4m39OaxuCh298BoG4wtK-7QiLvZmx-mHOsYZ-HsoU9XlkEbOwCTRi96lWngOd6GyFuyvqZi6KfALAmUbrOhB4g8oyPtob19SVMMc_fRLEd2eayPDE9oMlCX50TW_WnXVkk_s4ATOlmzTeLUGq9_8uBOInlCfWUkZqXN6MWo8yQjWS4fL30xoZ-uhmhE01LXuqSD3cforxyqHg",
  "expires_in": 600,
  "refresh_expires_in": 10800,
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI0OTY4OTMwNi02ZTgxLTQwOTYtYjVkYS04M2NkNDcxNTlmZDIifQ.eyJleHAiOjE2ODI2ODE0ODgsImlhdCI6MTY4MjY3MDY4OCwianRpIjoiYmM5NDhkOWEtYjM4Ni00MjYwLWFiZjctMDUzNmY0NDU0NzQ2IiwiaXNzIjoiaHR0cHM6Ly9zc28uYXBwcy5zZGNwcmQub2NwLnRyYW5zcG93ZXIuY28ubnovYXV0aC9yZWFsbXMvcHJvZCIsImF1ZCI6Imh0dHBzOi8vc3NvLmFwcHMuc2RjcHJkLm9jcC50cmFuc3Bvd2VyLmNvLm56L2F1dGgvcmVhbG1zL3Byb2QiLCJzdWIiOiJiNzBhMTQwZC04OTMzLTRhMDAtYTM3Mi0yNjQ0MTJkNmJjODIiLCJ0eXAiOiJSZWZyZXNoIiwiYXpwIjoibW9uaXRvcmluZyIsInNlc3Npb25fc3RhdGUiOiJhZjE3ZjRmMS01NGQ4LTQ2YzYtYjc2Yi0yYzRkMzUyY2FlYmIiLCJzY29wZSI6ImVtYWlsIHByb2ZpbGUiLCJzaWQiOiJhZjE3ZjRmMS01NGQ4LTQ2YzYtYjc2Yi0yYzRkMzUyY2FlYmIifQ.KIi2DVpJmr5vLAqQLc0uEAw5jBedWjln24llW5nAHcU",
  "token_type": "Bearer",
  "not-before-policy": 1676424634,
  "session_state": "af17f4f1-54d8-46c6-b76b-2c4d352caebb",
  "scope": "email profile"
}


access_token:
Header
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "146ZxawfAhM0f-bUsMt19fWfouDb9wiIwNM958tQy4c"
}
Payload
{
  "iat": 1683097865,
  "jti": "fe5d7e6b-ff2e-45ea-bd3f-d51a02072006",
  "iss": "https://sso.apps.ndcprd.ocp.transpower.co.nz/auth/realms/prod",
  "aud": "account",											 					// audience for the token, the application that is meant to be the final recipient of the token.
  "sub": "27f91469-c9d9-4986-a2cb-7593fe5b1a07",								// this looks like the id of the user in SSO
  "typ": "Bearer",																// the type of token - in this case i think this means that the token can be used in HTTP Authorization header
  "azp": "monitoring",															// client-id of the client requesting the token (authorizing party, i.e. the party doing the authorization)
  "session_state": "90687cb8-6112-45ff-97d2-7a0beb5ff0f3",
  "acr": "1",
  "sid": "90687cb8-6112-45ff-97d2-7a0beb5ff0f3",
  "allowed-origins": [
    "*"
  ],
  "scope": "profile email",							// Scopes applicable for the combination of this user+client (WARNING - other scopes may be on this token but not listed in scope key)
  
  "name": "Ghazanfar Banatwala",					// Client Scopes/profile-->Mapper/full name ('User's full name' mapper)
  "preferred_username": "banatg",					// Client Scopes/profile->Mapper/username (User Property mapper) :: maps UserModel Property/username to Token Claim/preferred_username
  "given_name": "Ghazanfar",						// Client Scopes/profile->Mapper/given name (User Property mapper) :: maps UserModel Property/firstName to Token Claim/given_name
  "family_name": "Banatwala",						// Client Scopes/profile->Mapper/family name (User Property mapper) :: maps UserModel Property/lastName to Token Claim/family_name 
  
  "email": "ghazanfar.banatwala@transpower.co.nz"	// Client Scopes/email->Mapper/email (User Property mapper) :: maps Property/email to Token Claim Name/email
  "email_verified": false,							// Client Scopes/email->Mapper/email verified (User property mapper):: maps Property/emailVerified to Token Claim Name/email_verified	boolean


  "realm_access": {									# CHECK WITH roles coming from the 'Prod' realm for user banatg (Prod-->Users-->banatg-->Role Mappings-->Realm Roles)	 
    "roles": [										// realm_access.roles claim - Client Scopes/role->Mappers/realm roles																
      null,
	  "mkt-admin",
      "default-roles-prod",
      "uma_authorization",
      "offline_access",
      "fn_crp_sift_access_istMiscFileUpload",
      "fn_crp_sift_role_adminView",
      "fn_crp_sift_access_moiMiscFileUpload",
      "fn_crp_sift_role_standardUser"
    ]
  },
  
  "resource_access": {								 
      "monitoring": {								# CHECK WITH roles coming from the client 'account' for user banatg (Prod-->Users-->banatg-->Role Mappings-->Client Roles-->account)
        "roles": [									// resource_access.${client_id}.roles claim - Client Scopes/roles->Mappers/client roles (${client_id} is monitoring)
          "nagios_access",							// Clients/monitoring->Roles/nagios_access
          "elk_access"								// Clients/monitoring->Roles/elk_access
        ]
      },
      "account": {									# CHECK WITH roles coming from the client 'account' for user banatg (Prod-->Users-->banatg-->Role Mappings-->Client Roles-->account)
        "roles": [									// resource_access.${client_id}.roles claim - Client Scopes/roles->Mappers/client roles (${client_id} is account)
          "manage-account",							// Clients/account->Roles/manage-account
          "manage-account-links",					// Clients/account->Roles/manage-account-links 
          "view-profile"							// Clients/account->Roles/view-profile 
        ]
      }
  },
  
  "access_roles": [									// access_roles claim - Clients->monitoring->Mappers->monitoring-role-mapper-->Mapper Type=User Client Role (maps User Client Role to Token Claim Name "access_roles")
    "nagios_access",														
    "elk_access"															
  ],
}
Signature
RSASHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  {
  "e": "AQAB",
  "kty": "RSA",
  "n": "mb_b3ljWekfzc5nLv9JgtNhoC1UQPDnyOJtXOqK9hUE7eO7mtvwJCLmMUSwlC9fJdswXaFPSKkIPEcJgq_y0ulrWum1oSvReyAE0GdfJH2z3I8rNoCPVAGiIcMkbNUR9ND3rVAV7LewbhY7QwSvXZ29BLUJE0j6yxFyIO6-cUwB4WXZYvhOfbaYPcOjADykvwENSoW9vOgETDygSth2Cs66xHE2jK9N8k9c14XkAkjqVR1ACF5-CAIf4EG6r85lfSbq785M0DKpOEGmpBV__Fcuik5TaluurzT4nBtChPPaS_RO1x4VxH__vX78bUZ9AZh-pFf0H919LfFcLIvXu5Q"
  }
)



Example 2 - from ndcses-4 dev realm 
-----------------------------------

read -s password 

curl -s \
-H "Content-Type: application/x-www-form-urlencoded" \
-d 'client_id=dcm-demo' -d 'username=banatg' -d "password=${password}" -d 'grant_type=password' \
'https://sso.apps.ndcses.ocp.transpower.co.nz/auth/realms/dev/protocol/openid-connect/token' | jq .


Response: 
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJwRVh4MklKYnZMY1J4eHFxTUZUSUc4YU5XX0I4NGwzNTBuQ2hDbmtBTUJzIn0.eyJleHAiOjE3MTQxODU0MDUsImlhdCI6MTcxNDE4NDgwNSwianRpIjoiNWJmNTUyMDctNWQyMC00NGIyLThiMTgtYTU3ZWZmZDY0ZWJjIiwiaXNzIjoiaHR0cHM6Ly9zc28uYXBwcy5uZGNzZXMub2NwLnRyYW5zcG93ZXIuY28ubnovYXV0aC9yZWFsbXMvZGV2IiwiYXVkIjpbIm1vbml0b3JpbmciLCJhY2NvdW50Il0sInN1YiI6ImIzZTNmMDliLWM2NzUtNDNiNy04Zjk2LWVkNDcyNmU3Mzc1MCIsInR5cCI6IkJlYXJlciIsImF6cCI6ImRjbS1kZW1vIiwic2Vzc2lvbl9zdGF0ZSI6ImQ1ZjY5NjEyLTYwZjQtNDlhYi04NWQ3LWU1MWQ5N2E3OTg5YiIsImFjciI6IjEiLCJyZWFsbV9hY2Nlc3MiOnsicm9sZXMiOlsibWt0LWFkbWluIl19LCJyZXNvdXJjZV9hY2Nlc3MiOnsiZGNtLWRlbW8iOnsicm9sZXMiOlsiZGNtX3N1cHBvcnQiXX0sIm1vbml0b3JpbmciOnsicm9sZXMiOlsibmFnaW9zX2FjY2VzcyIsImVsa19hY2Nlc3MiXX0sImFjY291bnQiOnsicm9sZXMiOlsibWFuYWdlLWFjY291bnQiLCJtYW5hZ2UtYWNjb3VudC1saW5rcyIsInZpZXctcHJvZmlsZSJdfX0sInNjb3BlIjoiZW1haWwgcHJvZmlsZSIsInNpZCI6ImQ1ZjY5NjEyLTYwZjQtNDlhYi04NWQ3LWU1MWQ5N2E3OTg5YiIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6IkdoYXphbmZhciBCYW5hdHdhbGEiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJiYW5hdGciLCJnaXZlbl9uYW1lIjoiR2hhemFuZmFyIiwiZmFtaWx5X25hbWUiOiJCYW5hdHdhbGEiLCJlbWFpbCI6ImdoYXphbmZhci5iYW5hdHdhbGFAdHJhbnNwb3dlci5jby5ueiJ9.Bc0SQA9jmUBMBJiVJlZYf4mjYAdrrZquvYdBIcq3Hby2CYhQpPIZD5tK_E5DBQePCAmH3uknMenTsDToXBxodcw0ayIniwT1KR3Rti8v3jDFdB8HyVO7r_Z5j0v3TspySbPqCWitEuUuJBSYkDU_XJdShypH32q1vPzUcoBdWrOoljGdSxm8dbigvKheIEEl8w4gT9_lGowJJa567FAWpcJh2w3KdDcG7EmyD_XHsUm3Qs3f5zxlQOLs7QQPW3unOEYFkGSJ6dwf833siWEDKFWkFSE8oQ7uLtYaHEhzNy9IflW9c89Db2bT-EdqpFkxvACGJnPR7hkuFlfhkBziLw",
  "expires_in": 600,
  "refresh_expires_in": 432000,
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICIwNzczYjk2NS1kMmE1LTRjZTMtODIzMi04YWEwNjRjNWY2N2MifQ.eyJleHAiOjE3MTQ2MTY4MDUsImlhdCI6MTcxNDE4NDgwNSwianRpIjoiMjAzNWNiYzEtOGNlYi00OWMyLWE5NTEtZWViMjY3Njg4ZWI3IiwiaXNzIjoiaHR0cHM6Ly9zc28uYXBwcy5uZGNzZXMub2NwLnRyYW5zcG93ZXIuY28ubnovYXV0aC9yZWFsbXMvZGV2IiwiYXVkIjoiaHR0cHM6Ly9zc28uYXBwcy5uZGNzZXMub2NwLnRyYW5zcG93ZXIuY28ubnovYXV0aC9yZWFsbXMvZGV2Iiwic3ViIjoiYjNlM2YwOWItYzY3NS00M2I3LThmOTYtZWQ0NzI2ZTczNzUwIiwidHlwIjoiUmVmcmVzaCIsImF6cCI6ImRjbS1kZW1vIiwic2Vzc2lvbl9zdGF0ZSI6ImQ1ZjY5NjEyLTYwZjQtNDlhYi04NWQ3LWU1MWQ5N2E3OTg5YiIsInNjb3BlIjoiZW1haWwgcHJvZmlsZSIsInNpZCI6ImQ1ZjY5NjEyLTYwZjQtNDlhYi04NWQ3LWU1MWQ5N2E3OTg5YiJ9.6_hX3uBf6L1GzW1q1s4omaceoGn_CKGkbmwXHsPPGd0",
  "token_type": "Bearer",
  "not-before-policy": 1684797754,
  "session_state": "d5f69612-60f4-49ab-85d7-e51d97a7989b",
  "scope": "email profile"
}

access_token: 
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "pEXx2IJbvLcRxxqqMFTIG8aNW_B84l350nChCnkAMBs"
}

{
  "exp": 1714185405,
  "iat": 1714184805,
  "jti": "5bf55207-5d20-44b2-8b18-a57effd64ebc",
  "iss": "https://sso.apps.ndcses.ocp.transpower.co.nz/auth/realms/dev",
  "sub": "b3e3f09b-c675-43b7-8f96-ed4726e73750",
  "typ": "Bearer",
  "azp": "dcm-demo",
  "session_state": "d5f69612-60f4-49ab-85d7-e51d97a7989b",
  "acr": "1",
  "sid": "d5f69612-60f4-49ab-85d7-e51d97a7989b",
  "aud": [
    "monitoring",
    "account"
  ],
    
  "scope": "email profile",  
  "email_verified": false,
  "name": "Ghazanfar Banatwala",
  "preferred_username": "banatg",
  "given_name": "Ghazanfar",
  "family_name": "Banatwala",
  "email": "ghazanfar.banatwala@transpower.co.nz"
  
  "realm_access": {
    "roles": [
      "mkt-admin"
    ]
  },
  
  "resource_access": {
    "dcm-demo": {
      "roles": [
        "dcm_support"
      ]
    },
    "monitoring": {
      "roles": [
        "nagios_access",
        "elk_access"
      ]
    },
    "account": {
      "roles": [
        "manage-account",
        "manage-account-links",
        "view-profile"
      ]
    }
  },
}

RSASHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  {
  "e": "AQAB",
  "kty": "RSA",
  "n": "oszLQ-cGHCrXT-MwXIdE4J54bT3IiHuGIlvKqs2ljeH27YIsA-3zVwX3GmAcAofWNJR2ybEHoe-Gp5kvLsG2Q60bQVZ6vZ1Hxunf7DCl99ycJO0G-qwwsb4yY1KISlJ76Hmg6U1V83qCbungGdtaViO2qyvT2kii-1kiWMEi6ek8lsR_3TpBX87Bw9eT9Q4Zq8033j13YNwNm5HosTj88IY2qMJ2SneF-XAXFtXZ5dbhyEwqdHImX3npbzMnTmmGTiQ_I8h5ZrX2evfkunQ0HX0PBaqFwr4FZy2Ss4YWLHHbymJwXGnkoUNXdFiXIedWi2X63aIV1IzbErQce18NPw"
} 
)


Example 3 - from Red Hat SSO 
----------

export username=gaz.banat
read -s RHCP_PASSWORD	# this is my Red Hat Customer Portal password 
 
curl -sL -u "${username}:${RHCP_PASSWORD}" "https://sso.redhat.com/auth/realms/rhcc/protocol/redhat-docker-v2/auth?service=docker-registry&client_id=curl&scope=repository:rhel:pull" | jq .

Response:
{
  "token": "eyJhbGciOiJSUzI1NiIsImtpZCIgOiAiUUdybGszdTkzM3Jja0pSd3piRTdra1I0RjRTU0VHY0dkeHZDVjBFaHI2OCJ9.eyJleHAiOjE3MTQxOTIyMDksIm5iZiI6MTcxNDE5MTkwOSwiaWF0IjoxNzE0MTkxOTA5LCJqdGkiOiI1NDczMDA3OC0xYmEwLTRiYjUtYTE4Ni05ZjgwZmUwYTc0MTIiLCJpc3MiOiJodHRwczovL3Nzby5yZWRoYXQuY29tL2F1dGgvcmVhbG1zL3JoY2MiLCJhdWQiOiJkb2NrZXItcmVnaXN0cnkiLCJzdWIiOiJnYXouYmFuYXQiLCJ0eXAiOiJCZWFyZXIiLCJhenAiOiJkb2NrZXItcmVnaXN0cnkiLCJhY2Nlc3MiOltdfQ.eFhtj9rNmia4PkMhXfCeNleRAtPGtSB4zeYBwpI_HdZBxtl0B-93SN6lgW8qjUKI6eXJOfLdykCw2g72WTM2SwMnD160I9w_JxSkDhVhvpFZSLsxT5kgFos809LRKL_cwkE2CQP_tRL33gBH9Nz-SUElN6DcGkr4UlueAvvtX_alQomdX7ivpZp5LUn5mxqxhAdMdPQ-rc1PKUlowrrtJQb663P2LNXNAPewFN_u5w_f5ttMEQl2h45q7L5K1w3bfAwqq9DFm_31IUabsX0_rji51YpZa77XwG0-NjbLODod4ZFZX-0LTjiCldDEISDBBl_HLzmVd84vQKeeX5wiXwdYgqN0EaiL3zZ8sqvIhllxN1GI09Sz6hJGNUiviUz-HFkdin8b77MO57uHfFQWd-C6gj4ud0WaGQZ_kCJF6WCgkUb38n7f982adhsC7NzUYDVTZfPR5Z6Jqr5dzMTGIqNbOgeXJmOTaJrgKOFGYdXN2UhdOuhSal1l_CJdHhnem_A_2S4GZr41u2incScp6htJi8oeU03NkfPr2crd4Sazw5RfBPUhU1kPz6o4SPRRcm8fCaO1F4EHJzVSPeD2PHc54m7eEBK4DbMcpDgKC3DMdY0ZeVk3JiOgbUQ1BZXqSmsazQeQNvsES0R82pQTKbM-W50rSWajls1wv9jyH9k",
  "access_token": "eyJhbGciOiJSUzI1NiIsImtpZCIgOiAiUUdybGszdTkzM3Jja0pSd3piRTdra1I0RjRTU0VHY0dkeHZDVjBFaHI2OCJ9.eyJleHAiOjE3MTQxOTIyMDksIm5iZiI6MTcxNDE5MTkwOSwiaWF0IjoxNzE0MTkxOTA5LCJqdGkiOiI1NDczMDA3OC0xYmEwLTRiYjUtYTE4Ni05ZjgwZmUwYTc0MTIiLCJpc3MiOiJodHRwczovL3Nzby5yZWRoYXQuY29tL2F1dGgvcmVhbG1zL3JoY2MiLCJhdWQiOiJkb2NrZXItcmVnaXN0cnkiLCJzdWIiOiJnYXouYmFuYXQiLCJ0eXAiOiJCZWFyZXIiLCJhenAiOiJkb2NrZXItcmVnaXN0cnkiLCJhY2Nlc3MiOltdfQ.eFhtj9rNmia4PkMhXfCeNleRAtPGtSB4zeYBwpI_HdZBxtl0B-93SN6lgW8qjUKI6eXJOfLdykCw2g72WTM2SwMnD160I9w_JxSkDhVhvpFZSLsxT5kgFos809LRKL_cwkE2CQP_tRL33gBH9Nz-SUElN6DcGkr4UlueAvvtX_alQomdX7ivpZp5LUn5mxqxhAdMdPQ-rc1PKUlowrrtJQb663P2LNXNAPewFN_u5w_f5ttMEQl2h45q7L5K1w3bfAwqq9DFm_31IUabsX0_rji51YpZa77XwG0-NjbLODod4ZFZX-0LTjiCldDEISDBBl_HLzmVd84vQKeeX5wiXwdYgqN0EaiL3zZ8sqvIhllxN1GI09Sz6hJGNUiviUz-HFkdin8b77MO57uHfFQWd-C6gj4ud0WaGQZ_kCJF6WCgkUb38n7f982adhsC7NzUYDVTZfPR5Z6Jqr5dzMTGIqNbOgeXJmOTaJrgKOFGYdXN2UhdOuhSal1l_CJdHhnem_A_2S4GZr41u2incScp6htJi8oeU03NkfPr2crd4Sazw5RfBPUhU1kPz6o4SPRRcm8fCaO1F4EHJzVSPeD2PHc54m7eEBK4DbMcpDgKC3DMdY0ZeVk3JiOgbUQ1BZXqSmsazQeQNvsES0R82pQTKbM-W50rSWajls1wv9jyH9k",
  "expires_in": 300,
  "issued_at": "2024-04-27T04:25:09Z"
}


access_token:
{
  "alg": "RS256",
  "kid": "QGrlk3u933rckJRwzbE7kkR4F4SSEGcGdxvCV0Ehr68"
}
{
  "exp": 1713998411,
  "nbf": 1713998111,
  "iat": 1713998111,
  "jti": "5f99989d-ac40-4cdc-9a62-6a92d0fdc635",
  "iss": https://sso.redhat.com/auth/realms/rhcc,
  "aud": "docker-registry",
  "sub": "gaz.banat",
  "typ": "Bearer",
  "azp": "docker-registry",
  "access": []
}
RSASHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
	{
	"e": "AQAB",
	"kty": "RSA",
	"n": "x-LArGk9uBkSAsM040uUkiJEXAfVGcWxFkI8AULV3J46Sj3OPq1azY_mmXt7pMdfmzCVnD4owd4pb4HN7a-d75SJAdqHO-nTSuc8qItXBriUbMC4qWpHITwBBIAuu3Bg0sMyp-31qthlRMYbp0A4dP3npKZFzI7-E4XdzpAlTgHkZvZOLja0-G4cf0YRRtw8DQ7TUkpREFAROi6EcqmDa6MfSUHPG0Gqav83aK8NvA3D3O6nD2otbypnN6d-gpbZMqVpOm_Mz4cPZfVqZnWWozE0oy-P3w1RzT5Ger9yHtJ373N7ORQuyPnLM-hUJ1kkbVPsk7uwqvJIUmJlfjP0Y1cuXvnhV0cRWU9iFJye6iyOuGXW2Id_dlBSfxqa1-pM7Mj2Y-1M09FKo0pZPYlSZyUwnwOynVTylb2g3vkgu6MnXXR2na1wq7SUxs017eECW04hrV7HwGKRTOM4q1T8HVEmiac1oczHgKI8xracaLnndE8yL8rAFAdcqA739x780q2w4L3qVk3Ajxxsw8xTCm0m_YKj9hRdPQc72f3NFxfCvsiZF6gGRAPPhX61egX2rxKq1_W1UMa1YzC3ec_iXNO1rlGGZj1OBJAvhPcrXNDAMxaMmGiqxFIHsekoQ99NK1Dl2QC5ZAoy-pxVoCS75a-iKsfgacjlXobubDh6UZ8"
	}
)


======================================

ID TOKEN 



USERINFO 







												
							