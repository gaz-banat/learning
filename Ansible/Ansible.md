
ARCHITECTURE


PluginLoader


Executor engine	--	Action Plugin	--	Module


Controller





ANSIBLE VERSIONS

ansible-core			-	2.10 (ansible-base) 2.11, 2.12, 2.13, ..... and so on
						no semver
						includes language, runtime and builtin plugins
						developed and maintained in ansible/ansible repository
						primarily for developers and users who want to install only the collections they need


ansible community		-	2.9, 2.10, 3.0.0, ....... and so on
						uses semver
						includes language, runtime and selected Collections
						developed and maintained in Collection repositories
						more than 85 collections containing thousands of modules and plugins



INSTALLING ANSIBLE

1. With a package manager like yum or brew
2. Install with pip
3. Install with pipx
4. Create a python venv and install ansible into that
5. Install ansible to a container
3. Install from source (this will give devel feature) - this is only for if you are developing content for ansible

https://docs.ansible.com/ansible/2.9/installation_guide/intro_installation.html




COMPONENTS


To understand components first run ‘ansible --version’

ansible [core 2.12.0]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/Users/gaz/.ansible/plugins/modules', '/usr/local/etc/ansible/plugins/modules']
  ansible python module location = /Users/gaz/Library/Python/3.9/lib/python/site-packages/ansible
  ansible collection location = /usr/local/etc/ansible/collections
  executable location = /Users/gaz/Library/Python/3.9/bin/ansible
  python version = 3.9.9 (main, Nov 21 2021, 03:23:44) [Clang 13.0.0 (clang-1300.0.29.3)]
  jinja version = 3.0.1
  libyaml = True


ansible core			
- the core ansible code		
- found via ‘executable location’

config file 				
- configuration information for ansible 	
- found via ‘config file’, 	
- based on local dir OR env variable ANSIBLE_CONFIG OR default location of /etc/ansible/ansible.cfg

modules that came with ansible		
- found via ‘ansible python module location’ 
- look for sub-directory modules/

additional modules										
- found via ‘configured module search path’ 	
- based on ‘library’ key in config file OR env variable ANSIBLE_LIBRARY

plugins that came with ansible		
- found via ‘ansible python module location’		
- look for sub-directory plugins/ 

additional plugins						
- based on ‘<plugin-type>_plugins’ key in config file OR env variable ANSIBLE_<plugin_type>_PLUGINS	
- test_plugins is a special case

collections that came with ansible 				
- found via ‘ansible python module location’
- look for sub-directory collections/

additional collections				
- found via ‘ansible collection location’	
- based on ‘collections_path’ key in config file	
- look for sub-directory ansible_collections/
- playbook local directory (./collections)

NOTE: run the command ```ansible-galaxy collection list``` to see all collections that ansible knows about
	

roles	
- based on ‘roles_path’ key in config file


python interpreter
the version of python that ansible is going to use on the control node


jinja


libyaml





HOW ANSIBLE WORKS

Prereq
control node	-	python2.7 or python3.5 and higher
managed node	-	python2.6 or python3.5 and later

NOTE: if managed node does not have python then try this ```$ ansible myhost --become -m raw -a "yum install -y python2”```



Creates a small program on the control node	

- ~/.ansible/tmp/ansible-tmp-<timestamp>.<id>/*.py
- ~/.ansible/tmp/ansible-tmp-<timestamp>.<id>/ansiballz_cache/zip files

Connect to the managed node (managed node could be localhost as well, ansible_connection=local) and push the program to the managed node using either SFTP or SCP
- /tmp/		(i think it is tmp)


Connect to the managed node (over ssh OR local OR some other way?) and execute the program using the managed node’s python interpreter
- first ansible looks for /usr/bin/python on managed node





CONCEPTS

built-ins				-	available python functions

expressions			-	using JINJA

connection methods		-	local
						ssh			-	OpenSSH by default because it supports ControlPersist
										if no OpenSSH then ansible will use a python implementation of OpenSSH called paramiko
						winrm
						docker




SETUP ANSIBLE

Ansible management host/station
 - on the ansible management host allocate a user - let’s say ansadmin
   make sure this user has ssh keys
 - a hosts file is needed with the inventory of the servers
	- a host can have a specifice anisble_user
	- a host can have the address as ansible_host

Target host
 - on each target host there should be an account (can also be ansadmin) that has sudo privileges, preferably with no password requirement
 - the management node ansadmin ssh pub key must be in authorized keys of target host account (use ssh-copy-id for getting the public key across) 
 - on the target host the python interpreter must exist
 - on the target host any required python modules must exist, e.g docker module


Check
 - do a one time login so that target host becomes a known host
 - run the ping module




THE ANATOMY OF A RUN

In order for a playbook to have an effect on server/servers it should
- know the hosts it is running on
- be able to gather_facts if those facts will be used
- be able to get privileges on the hosts on which it will run
- have access to the variable data in some way
- correctly reference roles and includes




ANSIBLE ENVIRONMENT VARIABLES:

These are used for configuring the ansible process on the management station

ANSIBLE_HOST_KEY_CHECKING
ANSIBLE_COLLECTIONS_PATH
ANSIBLE_LIBRARY						-	colon separated path to the directory/s containing local modules, these modules will then become available to playbooks and roles
ANSIBLE_<plugin_type>_PLUGINS			-	colon separated path to the directory/s containing plugins of type <plugin_type>
ANSIBLE_CONFIG_*




CONFIGURING ANSIBLE 

Configuration precedence:

1. value of ANSIBLE_CONFIG environment variable
2. ansible.cfg in current directory 						
3. User local ansible configuration, i.e. ~/.ansible.cfg
4. Global ansible configuration, i.e. /etc/ansible/ansible.cfg



The configuration file - ansible.cfg

[default]
inventory
roles_path
collections_path
host_key_checking				-		e.g. true, false
retry_files_enabled
nocows
library						-		ansible looks for modules in the directories specified here, (the specified directory will contain a directory called modules?)
callback_whitelist
action_plugins
callback_plugins
filter_plugins
lookup_plugins
vars_plugins

[ssh_connection]
pipelining=True					-		ansible will send and execute commands for most ansible modules directly over SSH connection (make sure requiretty option is sudoers is commented)
ssh_args

[inventory]
enable_plugins = kubernetes.core.k8s


Commands:

Get the current configuration of ansible
ansible-config dump



VARIABLES 

Variables can be applied for:
  a particular host					-		in the inventory file, in a file of the hosts name in a host_vars/  directory in the same directory as the inventory file
  a group of hosts					-		in the inventory file, in a file or directory of the group name in group_vars/ directory alongside the inventory file
  all hosts
  OR
  to generally store information



Specifying variables:
In ini-style inventory file variable is assigned using var=value 
In other cases variables are assigned using var: value


Variables can be:
	undefined	or defined				-		{{ var is not defined }} or {{ var is undefined }}
	empty or have value				-		the variable is defined but empty
	true or false					-
	lists or dictionaries


Type of Variables:
	Special
		magic variables
		connection variables
		facts
	User specified
	Registered



Special:

	magic variables 		- 	These are variables related to runtime and apply to ansible:

		vars								-	all the variables that ansible knows about (for a given run at the point at which ‘vars’ is called)
		hostvars							-	an array of hosts
		hostvars[<hostname>]				-	the variables for <hostname>

		groups							-	a list of all group names in the inventory
											there are 2 default groups as well - ‘all’ and ‘ungrouped’
		group_names						-	a list of all groups of which the current host is a part of

	
		inventory_hostname					-	hostname of the current host according to inventory (different to ansible_hostname)
		inventory_hostname_short				-	the first part of the inventory_hostname, up to the first period
		inventory_dir						-	the directory in which the inventory file is defined
		play_hosts						-	all hosts on which the current play will be run

		role_name						-	the name of the role being run
		role_path							-	the absolute path to the role
		all								-	all hosts parsed through the inventory
		ungrouped

	connection variables 		- 	used to define how the machine that runs Ansible connects to remote hosts during the execution of tasks and playbooks

		connection


	facts					-	These are variables that are related to runtime and apply to hosts, ansible retrieves these from hosts at run time, e.g. ansible_distribution:

		how to collect or set facts:
		gather_facts
		-m setup
		set_fact						- 	this is a module, it is used to set a fact.
		local fact file					-	Local Facts can be incubated for a host by placing a fact file in /etc/ansible/facts.d/ on that target host


		ansible_python_interpreter
		ansible_connection					-	ssh, local
		ansible_user
		ansible_host						-	localhost
		ansible_hostname
		ansible_fqdn
		ansible_distribution
		ansible_ssh_port
		ansible_ssh_extra_args				-	e.g. ‘-o StrictHostKeyChecking=no’
		ansible_ssh_private_key_file
		ansible_playbook_python

		Note: During a play the facts for a host can be reloaded by calling the setup module


Note: the full list is at ‘https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html’



User specified:

“{{ variable_name }}”					-	debug/msg, with_items, value for any modules argument 
variable_name						-	debug/var, when					<— only a variable can be specified with these, so interpolation is not required



Registered:

These are variables formed from the output of tasks

Registered variables have these properties
	changed		-	true or false
 	failed		-	true or false

Then depending on the module that was more properties are added

 shell module
	cmd			-	the command that was ran
	msg
	rc			-	result code
	start
	delta
	end
	stdout
	stdout_lines	-	array
	stderr
	stderr_lines	-	array

 stat module
	stat




Precedence Order of variables - high to low:

extra vars at command line				 -e "user=my_user"				from command line
include params
role (and include_role) params				<role>/vars/main.yml::include_vars

task vars							set_facts module / registered vars
task vars							include_vars	module			Loads YAML/JSON variables dynamically from a file or directory, recursively, during task runtime
task vars 							vars: or include_vars:			for the specific task

block vars 							vars: or include_vars			for tasks within the block

role vars 							<role>/vars/main.yml

play 								vars_files:
play 								vars_prompt:
play 								vars:
play								gather_facts:					host facts / cached set_facts

playbook dir  						host_vars/<host_name>			the directory host_vars should be in the same directory as the playbook, one yaml file per host
inventory dir						host_vars/<host_name>			the directory host_vars should be in the same directory as the inventory file, then a file per host
inventory file [static or scripted] 				host vars 					these are specified alongside the host

playbook dir						group_vars/<group_name>
inventory dir						group_vars/<group_name>		the directory group_vars should be in the same directory as the inventory file, then a file per group
playbook dir						group_vars/all				the directory group_vars should be in the same directory as the playbook, one yaml file per group
inventory dir						group_vars/all

inventory file [static or scripted]				[<group>:vars]
role defaults 						role/defaults/main.yml


	Note: child groups variables have higher precedence than a parent groups variables

	ANSIBLE/ANSIBLE-PLAYBOOK command

	ROLE
		VARS/MAIN.YML
		DEFAULTS/MAIN.YML
		
	PLAYBOOK/
		PLAYBOOK FILE
			PLAY							
				BLOCK
					TASK
		GROUP VARS/
			GROUP FILE(S)
			ALL FILE

	HOST FACTS

	INVENTORY/
		INVENTORY FILE
		GROUP VARS/
			GROUP FILE(S)
			ALL FILE
		HOST VARS/




Commands:

I want to see the variables per host basis from an inventory
ansible-inventory -i <inventory_file> -y --list 



PRIVILEGE ESCALATION



ANSIBLE PLAYBOOKS

Options at command line to run a playbook

- - extra-vars						-	define variables at the command line for the playbook in the form “key=value, key=value” or 
									through a yaml/json file using “@<file-name>”
- - connection=TYPE 				-	default is ssh, you can also use ‘local’, ‘paramiko’
- - check							-	dry run
- - forks							-	number of concurrent processes, default is 5
- - become						-	get elevated privileges
- - become-user						-	become this user for privilege escalation rather than root
- - become-method					-	how to become a certain user, e.g. su, sudo, runas, etc  (see ‘ansible-doc -t become -l’)
- - ask-become-pass					-	prompt for the sudo password
- - ask-pass						-	ask for a password when connecting to the server (this assumes no key based authentication)
- - force-handlers					-	will make handlers run even if the play fails (so long as the notify is successful)
- - ask-vault-pass					-	prompt for the password to a vault file
- - vault-password-file				-	makes ansible pick up a vault password from a file
- - tags							-	only run the tasks with specific tags
- - skip-tags



Keywords in playbook:

https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html


Job Templates:

- a definition and set of parameters for running an Ansible playbook


Verbosity when running a playbook

- v								-	show the output of the modules that were run
-vv								-	above 
									show the output of ‘ansible --version’ at start									something about plugins
									PLAYBOOK - show number of plays discovered in playbook
									For each PLAY - show META: ran handlers





LOCAL CONNECTION

To make a ‘local’ connection to a host you can use
ansible_connection = local			-	this fact/var can be set at the host/group level or at the playbook level or at the task level (it is a good idea to use ansible_python_interpreter to be ansible_playbook_python along with ansible_connection)
connection: local				-	as an option to the playbook either at command line or as a keyword declaration


ANSIBLE TASKS

Looping on a task
  loop
  loop_control
	loop_var
  until
  with_<lookup>
	with_items 				-	takes a list
  	with_nested
	with_fileglob




ANSIBLE MODULES

What:
Modules are reusable standalone scripts that can be used by the Ansible API, the ansible command or the ansible-playbook command. 
Modules work as standalone scripts that Ansible executes in their own process outside of the controller, i.e. execute automation tasks on a ‘target’ (usually a remote system). 

Examples of modules:
	Variable related:
		 include_vars						-	Load variables from files, dynamically within a task
											can use a variable name for the file name from which to load the variables
		 set_fact							-	allows us to define variables at runtime

	Command related:
		 shell
		 command
		 raw								-	like shell but no STDERR, STDOUT and Return Code is returned
											Good for when there is no python on the target host

	Task related:
		 import_tasks						- 	provide a file name with tasks in it, tasks will be formatted as a flat list
											the file ideally contains a set of tasks (a play) that can be used across different playbooks
											imported tasks can reference other imported tasks
											ansible statically includes tasks as if they are part of the main playbook
											can’t use variables for task include file names
	 	include_tasks						-	used to include tasks dynamically - that is based on some condition
											can use variable for task include file names
 		import_playbook					-	statically imports an entire playbook

	Role related:
		import_role						-	statically include a role at the task level
 		include_role						-	dynamically include a role at the task level

	Waiting and Pausing:
 		wait_for
 		wait_for_connection
 		pause


	URL:
 		get_url
 		uri

	Testing:
		 assert
		 assert_defined
 		fail


	Inventory management:
		add_host							-	add host and group to ansible-playbook in-memory inventory
											maybe because the host/container came into existence after the playbook started running
											maybe because the playbook needs to run on the ‘localhost’ which is not inventory but needs access to group_vars and host_vars

	Scripting:
 		script

	Docker
 		docker_container


	TBS:
 		git
 		find
 		group_by
 		action
 		slurp



How can I add a local module to Ansible for use with all playbooks and roles:
1. set the ANSIBLE_LIBRARY environment variable to the colon separated path/s of the directories that contain your module
2. As specified by ‘library’ key in ansible.cfg - e.g library=/usr/local/etc/ansible/plugins/modules:plugins/modules (paths that start without / are relative to current dir)
3. ~/.ansible/plugins/modules/
4. /usr/share/ansible/plugins/modules

How can I add a module for use with specific playbooks :
1. In a selected playbook or playbooks: Store the module in a subdirectory called library in the directory that contains those playbooks.

How can I add a module for use with a specific role:
1. Store the module in a subdirectory called library within that role.



Note: A module called local_users.py will be referred to as local_users in the playbook




ANSIBLE PLUGINS

Types of plugins:

action					-	front ends to modules, can execute actions on the controller before calling the module themselves
become
cache					-	used to keep a cache of facts
callback					-	perform custom actions in response to ansible events
	stdout				-	control the format of output, in ansible.cfg/stdout_callback
	notification
	aggregate
cliconf
connection				-	define how to communicate with inventory hosts
filter						-	allow manipulation of data inside Ansible plays and/or templates (Jinja2 feature)
httpapi
inventory
lookup					-	used to pull data from external source
netconf
shell						-	deal with low level commands and formatting for different shells
strategy					-	control the flow of a play and execution logic
test						-	allow validation of data inside Ansible plays and/or templates
							rather than have a test/ dir in a plugin/ dir in a collection path, a playbook can also have a local test_plugins/ dir
vars						-	inject additional variable data into ansible runs that did not come from an inventory, role, playbook or command line


All plugins need to:
- be written in python
- have the ability to raise errors
- return strings in unicode
- conform to Ansible configuration and documentation standard

How do I add plugins to Ansible:
1. Colon separated list of directory/s to the ANSIBLE_<plugin_type>_PLUGINS environment variable
2. Using ansible.cfg and specifying the path using the plugin key, e.g. action_plugins, callback_plugins, filter_plugins, etc.
3. ~./ansible/plugins/<plugin_type>/
4. /usr/share/ansible/plugins/<plugin_type>/
5. test_plugins is a special case, e.g. test_plugins directory can be in the playbook local directory OR in the plugins directory of a collection

How can I add a plugin only for a playbook or role:
1. In a selected playbook or playbooks: Store the plugin in a subdirectory for the correct plugin_type (for example, callback_plugins or inventory_plugins) in the directory that contains the playbooks.
2. In a single role: Store the plugin in a subdirectory for the correct plugin_type (for example, cache_plugins or strategy_plugins) within that role. When shipped as part of a role, the plugin is available as soon as the role is executed.




PLUGINS::LOOKUP PLUGINS:

What:
Lookup plugins are a way to get data from external sources

var0: “{{ lookup(‘vars’, ‘item’) }}”								-	lookup the value of item1 and assign it to var0
var1: “{{ lookup(‘vars’, item) }}”										-	lookup the value of item1, use that value as a variable name and get that variables value and assign to var1
var2 : ”{{ lookup(‘file’, ‘/path/to/afile’) }}”								-	get the contents of afile and assign it to var2, file lookups only run in localhost (ansible controller)
var3: “{{ lookup(‘template’, template_file_name }}						-	lookup a template in a role by name template_file_name
var4: “{{ lookup(‘url’, ‘https://google.com/search/?q='' + item  | urlencode | join(‘ ‘) }}

Ansible enables all lookup plugins it can find. 

You can activate a custom lookup by 
 - either dropping it into a lookup_plugins directory adjacent to your play
 - inside the plugins/lookup/ directory of a collection you have installed
 - inside a standalone role
 - or in one of the lookup directory sources configured in ansible.cfg.




PLUGINS::FILTERS

4 type of filters:
- ansible specific
- Jinja2 built-in filters
- python methods
- custom ansible filters as plugins

How to use a filter:
“{{ <var_name> | <filter> [ | <filter [ | <filter> .....] ] ] }}”

Some common filters:
default(<default_value>) 					-	provide a default value to a variable
default(omit)							-	if the value is not defined it will not be sent to the module 
										e.g. chart_version: “{{ version_of_chart | default(omit) }}” - if version_of_chart is not defined then the chart_version parameter and value will not be sent to the helm module
default(mandatory) 
count 
dirname
basename
reverse
split
regex_search(“some string”)
regex_replace
selectattr								-	e.g. {{ result.json | selectattr(‘name’, ‘eq’, ‘first name’) }}
list
length
string

NOTE: i found that the filter ‘to_yaml’ worked inside a playbook and role BUT did not work inside a jinja template file





PLUGINS::TESTS

tests can be used with the following keywords
  until:
  when:
  changed_when:
  failed_when:

<var> is defined							-	there is a value for <var>
<var> is failed

<var> is <something>						-	<something> could be a test as defined by a user in a test plugin
not <var> is <something>
<var> is not <something>

<var> is match(“some string”)
<var> is search(“some string”)
<var> is regex(“some regex in quotes”)

<var> is truthy or <var> is truthy(conver_bool=True)	-	python truthy
<var> is falsy or <var> is truthy(conver_bool=True)	-	python falsy
<var> in <list>								-	e.g. status in [1, 2]
<var> not in <list>							-	e.g. status not in [1, 2]

<var> == <some int>
<var> != <some int>


How to use a test:
There are 3 ways of using tests
with variable		-		when: variable is defined
with string			-		when: ‘ “string” in variable’		- To use a string you need to quote the entire condition plus the string
with string			-		when: >
							not “string” in variable	- This way you dont have to quote




CONDITIONAL LOGIC IN VARIABLE INTERPOLATION


action if test

action if test else another_action		-	e.g. maven_version: "{{ lookup('vars', 'deployment_marketsystem_operator_version' + ('' if openshift_namespace_app in ('', 'mkt') else ('_' + openshift_namespace_app))) }}"
								-	e.g. "__marketsystem_secrets": "{{ lookup('vars', non_existent, default=([] if openshift_namespace_app not in ('', 'mds', 'mkt'))) }}"


FIT THIS SOMEWHERE

<registered_variable>.stdout.find(‘<string>’) != 1





TEMPLATING

Understand templating
- templating has expressions, filters, tests, 
- all templating happens on the Ansible control node before the task is sent and executed on the target machine

Where can I use templating
- with the template module using a template file
- directly in playbooks by templating task names and more
	e.g.  - name: Get openshift maven_artifact_id yml File from Maven
		  maven_artifact:
		  	repository_url: “{% if ‘SNAPSHOT’ in maven_version | string%}http://someurl.co.nz{%else%}http://someotherurl.co.nz{%endif%} 
 




ANSIBLE ROLES

The role directory can be in the playbook local directory (./roles) OR in the roles directory as configured in ansible.cfg

One way to build the scaffolding for a role is ‘ansible-galaxy role init <role_name>’

Role directory structure
role_name/
	defaults/
	files/
	handlers/
M	meta/			- dependencies, 
M	tasks/
	templates/
	vars/


The requirements for a role that is local or in galaxy are different to when the role is part of a collection


How can I add roles to Ansible:
1. In /etc/ansible/roles
2. Update the ‘roles_path” key in ansible.cfg with a colon separated list of paths 

1. Add your roles to the roles/ dir in the same directory as your playbook





ANSIBLE COLLECTIONS

a format for modules, plugins, roles and playbooks so that they can be distributed/used as a collective
 


in the collections directory as configured in ansible.cfg
OR
in ~/.ansible/collections/ansible_collections
OR 
The collections directory can be in the playbook local directory (./collections) 


FQCN:
Fully Qualified Collections Name		-	ansible builtin stuff is “ansible.builtin.*”

Collection directory structure
	README.md
	docs/
	galaxy.yml
	plugins/
	roles/


Namespaces:
Collections come in namespaces. 
Common namespaces are Community, Kubernetes, AWS, VMWare, etc.


https://docs.ansible.com/ansible/latest/collections_guide/index.html





ANSIBLE GALAXY

a repository of community maintained roles and collections


The idea of a requirements file:

a requirements file can be created which has a list of roles and collections in this format
- - - 
collections:
  - name: provider1.collection1
    version: 3.2.1
  - name: provider1.collection2
roles:
  - name: provider2.role1
  - name: provider2.role2
    version: 1.2.3

then ‘ansible-galaxy install -r requirements.yml’ will install the roles and collections required before a playbook run


Commands

I want to see a list of all collections
ansible-galaxy collection list



AUTOMATION HUB






ANSIBLE VAULT


Vault password based:
 - a yaml file encrypted with ansible-vault command OR a string encrypted with ansible-vault command
 - to decrypt the yaml file, the password that was used at time of encryption is required
 - this password is referred to as the vault password
 - to reference the encrypted object either use the ‘--ask-vault-pass(word)’ cli option OR use a vault password file  and use the ‘--vault-pass(word)-file’ cli option


Vault ID based:

What:
a vault-id is an identifier for one or more vault secrets


 - uses the ‘--vault-id’ flag on the ansible command line and this flag can be used multiple times
 - each use of ‘--vault-id’ ties a vault-id to a vault password file
  e.g. --vault-id corp@~/.corp-vault		the ‘corp’ id has been tied to ~/.corp-vault vault password file


!Vault tag
This can be used to encrypt a single value inside YAML






DYNAMIC INVENTORIES

Dynamic inventories are made available by scripts
- the script should be executable
- the script should return json to stdout
- the script should provide a ‘- -list’ command line option 
- json output format should be 
    - a dictionary of group(s) with each group having 
  	- a list of hosts called “hosts”
	- a dictionary of variables called “vars”
    - a dictionary called “_meta” having
	- a dictionary called “hostvars”
		- a dictionary for each host containing the variable and value pairs

Mixing static and dynamic inventories

- pass a directory to ansible or ansible-playbook and ansible will combine the output of all inventories
- make sure that the directory only contains inventory files either static or scripts






MOLECULE

drivers				-		backend provisioning systems like docker, podman, virtual machines, delegated (cloud)
Test Scenario			-		each directory in the molecule directory represents a test scenario


delegated driver
- molecule automatically runs a create.yml playbook before starting a test run
- molecule automatically runs a destroy.yml playbook when the test run is complete


default scenario test matrix: dependency, lint, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy