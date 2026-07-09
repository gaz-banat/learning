

TERRAFORM EDITIONS

https://developer.hashicorp.com/terraform/intro/terraform-editions

Terraform Community Edition	-	the downloadable tool that works on the cli
Terraform Cloud			-	is now HCP Terraform
HCP Terraform 				-	is an application that helps teams use Terraform together, available at https://app.terraform.io
							In this case terraform runs in a stable remote environment and securely stores state and secrets.
	free edition
	plus edition
Terraform Enterprise			-	self-hosted distribution of HCP Terraform.


ENVIRONMENT VARIABLES FOR TERRAFORM PROCESS

TF_LOG=[TRACE | DEBUG | INFO | WARN | ERROR]
TF_LOG_PATH=/path/to/logfile.txt




WORKING DIR

main files and directories
	.terraform/								-	do not commit to version control
		providers/
		modules/

	.terraform.lock.hcl						-	has provider dependency selections, always commit to version control

	terraform.tfstate							-	state file, do not commit to version control instead put in gitignore, 
	terraform.tfstate.*						-	backup state files, do not commit to version control, put in gitignore,

	.terraform.tfstate.lock.info					-	created and deleted automatically when you run terraform apply

	*.tfvars								-	if it contains sensitive information do not commit to git repo

	*.tftest.hcl								-	test files
	*.tftest.json

various opinonated .tf files
	main.tf								-	data, resource and module blocks
	terraform.tf							-	a single terraform block with required_version and required_providers
	variables.tf							-	variables in alphabetical order
	backend.tf								-	the backend configuration
	outputs.tf								-	outputs in alphabetical order
	providers.tf							-	provider blocks
	locals.tf								-	local variables and values
	override.tf								-	override definitions for your configuraiton (Terraform loads this and all files ending with _override.tf last)



Example .gitignore file for terraform repo - https://github.com/github/gitignore/blob/main/Terraform.gitignore



HashiCorp Configuration Language (HCL)


Language Elements:

1. Blocks and Arguments
<BLOCK TYPE> "<BLOCK LABEL>" “[<BLOCK LABEL>]” {		# BLOCK HEADER
  # Block body
  	NAME = <EXPRESSION>					# ARGUMENT (aka ATTRIBUTE)
}

NOTE: Argument names, block type names, and the names of most Terraform-specific constructs like resources, input variables, etc. are all identifiers.

2. meta-arguments
  depends_on
  count				-	the number of resources to create if the resources are identical
  for_each				-	the number of resources to create when the resources are not identical, for_each accepts a set or map value
  provider				-	used for resources
  providers				-	used for modules
  lifecycle
    create_before_destroy	-	if the resource needs to be replaced then first create before destroying (the default behaviour is to destroy then create)
	ignore_changes
	prevent_destroy


3. parameter ?
  sensitive 


4. Objects

It seems that objects are things enclosed by { } but not all the time. Maps are also enclosed by { } but are not objects.




Configuration Blocks:

Terraform Block			terraform { }
Provider Block			provider “<provider_name>” { }
Resource Block			resource “<resource_type>” “<resource_name>” { }
Data Block			data “<data_type>” “<data_name>”				calling:  data.<type>.<name>.<attribute>
Input Variables Block		variable “<variable_name>” { }					calling:  var.<variable_name>, “${var.<variable_name>}”
Local Variables Block	locals { <variable_name> = <value> }				calling:  ${local.<variable_name>}
Output Values Block		output “<output_name>”
Modules Block			module “<module_name>”						a module is a reusable container of resources that are frequently used
Dynamic Block	


Variables:

1. Variable Precedence from high to low

Command Line: -var and -var-file
*.auto.tfvars OR *.auto.tfvars.json
terraform.tfvars.json
terraform.tfvars
Environment Variables					-	export TF_VAR_<variable_name> (e.g. variable “vpc_name” in configuration would become TF_VAR_vpc_name)
Variable defaults


2. Special Variables

terraform.workspace
local_file


3. Variable types
primitive
  number
  string
  bool
complex/collection
  list		-	a sequence of similar types
  tuple	-	a sequence of types that need not be similar
  set
  map	-	map(string), 			- the value of each key is a string, the key name is up to you
			map(number), 			- the value of each key is a number, the key name is up to you
			map(bool),			- the value of each key is a bool, the key name is up to you
			map(list(string)),		- the value of each key is a list of strings, the key name is up to you
			map(set), 				- the value of each key is a set, the key name is up to you
			map(object( { ... } ))		- the value of each key is an object, the key name is up to you			
			calling: ${var.<map_variable_name>[“<key>”]}


4. Variable Validation

validation { }



Example variable block

variable “some_var” {
	type = 
	description =
	default = 
	sensitive =
	validation { }
}



Functions:

1. length()
var “aList” { type = list(string) }
resource “type” “name” {
	count = length(var.aList)				# count is a meta-argument
	<identifier> = var.aList[count.index]
}

2. tolist()
3. toset()
4. index()
5. can()
6. cidrsubnet()
7. slice()
8. lower()
9. regex()
10. jsonencode()


Loops:

1. for loop

{ for k, v in aws_instance.web : k => v.private_ip }

{ for key, value in var.<map_variable> : name => “${key}-bucket” }


Conditions:

1. Terenary condition
var.<variable_name> ? <action_if_true> : <action_if_false>
e.g. count = var.enable_metrics ? 1 : 0



Language Constructs:

in 		-	keyword
=>		-	used to separate a key from a value in an expression when using a for loop on a map
:




Linter:

TFLint





PROVIDERS

https://registry.terraform.io

aws provider				-	registry.terraform.io/hashicorp/aws




PROVISIONERS

local-exec
remote-exec



MODULES

What:
a way to package resources into configuration files


Where can I source a module from:
Local Path			-	./modules/<module>
Terraform Registry		-	https://registry.terraform.io/, also look at https://developer.hashicorp.com/terraform/cloud-docs/registry
						module repository must be in the form “terraform-<PROVIDER>-<NAME>”
GitHub
Bitbucket
Mercurial
HTTP URL
S3 Bucket
GCS Bucket


MODULE REPOSITORY and REGISTRY



STATE

Lock file for state:
.terraform.tfstate.lock.info				-	a json file that describes who is in the process of modifying the state file.

Backends for state:
  Standard Backends (state is stored remote but terraform exectuion is local)

  	s3	 										-	supports locking if dynamodb is used  [ think about versioing, encryption and locking ]
	http
  	gcs											-	supports locking
  	azurerm	 									-	supports locking


  Enhanced Backend (state is stored remote and terraform execution is remote OR state is store local and terraform execution is local)
	local											-	supports locking  [ this is the default backend when no other is specified ]
	remote										-	supports locking							


State sharing:

tfe_outputs data source				-	when you are using HCP Terraform or Terraform Enterprise and need to reference resources across workspaces




CREDENTIALS FILE

$HOME/.terraform.d/credentials.tfrc.json



SECRETS MANAGEMENT

basically the state file contains sensitive data

Terraform Community	-	
HCP Terraform			-	hashicorp vault for state encryption, use of dynamic provider credentials is recommended
Terraform Enterprise		-	hashicorp vault for state encryption, use of dynamic provider credentials is recommended


provider credentials
dynamic provider credentials
Terraform vault provider



TESTS



POLICIES



WORKSPACES




COMMANDS

terraform version					-	show version of terraform and providers
terraform providers					-	show required providers and modules

terraform init						-	initialize working directory, gets providers and modules, create or update the .terraform.lock.hcl dependency file, setup the backend for storing the state file
									(this command does not touch state or configuration files)
terraform init -upgrade				-	when you add a new provider or upgrade an existing provider to an existing TF project
terraform init -migrate-state			-	migrate the state file to a new location specified in the terraform { } block
terraform init -reconfigure				-	something changed in the terraform configuration and the state needs to be reconfigured, e.g. you changed the backend where terraform state should be kept

terraform fmt [-recursive]

terraform validate					-	validate the configuration, checks if the code is syntactically valid and internally consistent
terraform validate -json

terraform plan						-	create execution plan, compare configuration file to terraform state
terraform plan -refresh-only			-	a way of finding changes to terraform managed resources outside of terraform, so a way of detecting drift.
terrafrom plan -destroy				-	create an execution plan for destroying

terraform apply	[ <plan_file> ]			-	will apply the plan, optionally you can provide a plan file
terraform apply --replace=<resource>
terraform apply -refresh-only			-	update one or more resources in the state with configuration drift (config changes that were made outside of terraform)
terraform apply --destroy
terraform apply --lock-timeout=<n>s

terraform show						-	show the terraform state in an easy to read format
terraform state list					-	list the resources in the current state
terraform state show <resource>		-	show a resource, good way of getting all resource arguments that make up the resource
terraform refresh					-	first it compares state with what is actually deployed
									helps to update terraform state with resource configuration drift that may have happened outside terraform
									THIS COMMAND HAS BEEN DEPRECATED, use terraform plan -refresh-only
									this command automatically updates state


terraform output [-json] [<output_name>]	-	list the outputs in the current state (remember that sensitive objects that did not display at cli after tf apply due to being sensitive can still be retrieved from the state file)

terraform destroy

terrraform taint

terraform workspace

terraform console

terraform login

terraform test




SOLUTIONS

A resource configuration changed outside terraform and i want to bring the configuraiton into the state and configuration file
terraform plan -refresh-only
terraform apply -refresh-only
then update the configuration in the proper configuration file



I want to see the documentation of a resource of a provider
https://registry.terraform.io/providers/<provider_org>/<provider_name>/latest/docs
e.g. https://registry.terraform.io/providers/hashicorp/aws/latest/docs



I want to see the documentation of a module
https://registry.terraform.io/modules/<module_path>/latest
e.g. https://registry.terraform.io/modules/terraform-aws-modules/iam/aws/latest

