
VERSIONS

helm version api version for charts

v3.0 v2
v3.5.0



USE-CASE

helm use-case is not just for installation but about distributing your
app and about discovery

Somewhere on the Internet:
Repo <--- access to this repo is by helm repo add <repo_name> <url_where_repo_file_is_stored>
Chart
Version 1
Version 2
Version 3
Chart
Version 1
Version 2
Repo
Chart
Version 1




ENVIRONMENT VARIABLES

KUBECONFIG - path to the kubeconfig file




REPOSITORY

- an index.yaml file with a list of chart versions that can be stored on
any web server
- looks like it is named index.yaml at a URL (so 'helm repo add <repo-name> <url>' is going to GET <url>/index.yaml)

local cache:
helm maintains a local cache of charts - e.g.
\~/Library/Caches/helm/repository
'helm repo update' will update the local cache of charts with latest
information sourced from repositories




BUILT IN OBJECTS

. - is a reference to the current scop, by default it represents all
data OR the root of all data, the current scope can be changed using
'with'

.Release - populated via install or upgrade command
.Name
.Namespace
.IsUpgrade
.IsInstall
.Revision
.Service - don't confuse service with a kubernetes service, this is the
service that is rendering the template

.Values - Values is populated via defaults in the chart/values.yaml, or
a parents chart/values.yaml or a values file at install time, or any
settings the user has overridden with \--set

.Chart - the contents of Chart.yaml
.Name
.Version

.Files - used to access non template files in the Chart
.Get - {{ .Files.Get "configuraitons/app.conf" }} will get the contents
of the app.conf file in the configurations/ directory
.GetBytes
.Glob - {{ .Files.Glob "pattern_string" }}, e.g. {{ .Files.Glob
"conf/\*\*" }}
.Lines
.AsSecrets - returns file contents as Base64 encoded strings
.AsConfig - returns a file body as a yaml map

NOTE: files in templates/, files excluded using .helmignore and files
outside of a chart root directory cannot be accessed (subchart cannot
access parent chart files)

.Capabilities - provides information about what capabilities the K8s
cluster supports
.APIVersions
.APIVersions.Has \$version
.KubeVersion
.KubeVersion.Version
.KubeVersion.Major
.KubeVersion.Minor

.Template - information about the current template that is being
executed
.Name
.BasePath - path up to dir of current template




WORKFLOW

Load chart and its dependencies - here templates get loaded

parse the values.yaml file

render the templates to yaml manifests

validate yaml manifests against kubernetes schema

send yaml manifests to kubernetes to create the resources


Note: \--dry-run goes through the first 4 phases but not the 5th one






CHART

a versioned application package
parameterized yaml files live in the templates folder
has a default set of values in the values.yaml file
charts are normally compressed archives but can be a root folder as
well
charts can include
tests - k8s job specs that run after installation to verify deployment
hooks - hooks let you run jobs at specific points in the installation
workflow
charts can be digital signed
can declare dependencies on subcharts but subcharts need to be
standalone



Structure of a Chart:
/root-dir/
Chart.yaml - specifies the chart metadata, including the apiVersion,
name, description, type, version of the chart, version of the app
values.yaml - sets the default values for parameters
crds - custom resource definitions
templates/ - contains the templated kubernetes manifests
deployment.yaml
pvc.yaml
secret.yaml
service.yaml
configmap.yaml
Notes.txt - display notes on release, helm does not treat this file as a
template
\_helpers.tpl - any file beginning with \_ is deemed not to be a k8s
manifest
tests/
job.yaml - this is the test for the chart, annotate in metadata with
"helm.sh/hook": test
pre-upgrade-checks/
job.yaml
configmap.yaml
.helmignore - a list of extensions that should be ignored when chart is
packaged
requirements.yaml - specify the dependencies for this chart



The 4 main ways to access variables are:
.Release - everything to do with the release
.\<var\> e.g. .Release.Name
.Chart - variables in the Chart.yaml file
.\<var\> e.g. .Chart.AppVersion
.Values - variables in the values.yaml file
.\<var1\>
.\<var2\>
.\<var3\>
.\<chart_name\>.\<var\> - a predefined function in the \_helpers.tpl
file, using \<chart_name\> is only a convention, not mandatory.



Chart.yaml Manifest:
apiVersion: v2 \# the version of the helm spec
name: data \# the name of the chart
description data \# a meaningful description that others would enjoy
reading
icon: url \# where to get the chart icon
keywords: \# list of keywords
- keyword1
- keyword2
sources:
- \<url\> \# url to the source of the chart, e.g.
https://github.com/gaz-banat/mychart
maintainers:
- name: *name*
** email: *email*
type: application or library
version: data \# the version of the chart
appVersion: data \# the version of the app itself that the chart
provides



requirements.yaml manifest:
dependencies: \# other charts this chart depends on - this chart will be
parent and the chart that it depends on will be child
- name: tomcat
version: 10.1.2
repository: http://charts.bitnami.com/bitnami
condition: tomcat.enabled \# tomcat.enabled is specified to true or
false in values.yaml
OR
tags:
- enabled
import-values:
- child: service
parent: tomcat.service \# service element in the child chart is brought
into this chart as tomcat.service, now you can access the service port
\# of the child chart as tomcat.service.port


Publish a Chart
- package into a zip archive
- upload the archive to a server
- update the repository (index.yaml) to add the new chart (btw
ChartMuseum will take care of this for you)


Release
- an instance of an installation of a chart
- every release has a name, you can install multiple instances of a
chart within a cluster as separate named releases


Installation
- can be atomic, which means there is an automatic rollback on failure
- can be set with a timeout, so that the update finishes in that time



Tests/Pre-Upgrades
- test and pre-upgrades run as jobs
tests
- annotate the job with test
- tests run on demand and not as part of an install
- best to configure a job as a test, but helm will not clean up the
completed Job
pre-upgrade
- to run a pre-upgrade check, i.e. a test before an upgrade, annotate
the job with pre-upgrade




CHART LIFECYCLE HOOKS

- predefined actions represented by kubernetes resources (deployments,
jobs, pods, etc.) to be run at specific times during the Chart
lifecycle

- hooks (the kubernetes resources with helm.sh/hook annotation) are in
the templates directory of the chart

- annotate the kubernetes resource with:
- "helm.sh/hook": \<hook-type\> \[, \<hook-type\>, \....\]
- "helm.sh/hook-weight": \<int\> \# ascending priority, i.e. 1 before 2
before 3
- "helm.sh/hook-delete-policy: \<delete-policy\>

hook types:

install hooks
pre-install - Executes after templates are rendered, but before any
resources are created in Kubernetes
post-install - Executes after all resources are loaded into Kubernetes
delete hooks
pre-delete - Executes on a deletion request before any resources are
deleted from Kubernetes
post-delete - Executes on a deletion request after all of the release's
resources have been deleted
upgrade hooks
pre-upgrade -  Executes on an upgrade request after templates are
rendered, but before any resources are updated
post-upgrade - Executes on an upgrade request after all resources have
been upgraded
rollback hooks
pre-rollback - Executes on a rollback request after templates are
rendered, but before any resources are rolled back
post-rollback - Executes on a rollback request after all resources have
been modified
test
test - Executes when the Helm test subcommand is invoked


delete policies:
before-hook-creation - if a similar hook resource exists then delete it
and create this one, this is the default value
hook-succeeded
hook-failed





DEFAULT CHART COMPONENTS


\_helpers.tpl

\<chart\>.name - Use .Values.nameOverride (with trunc 63 and trimSuffix
-) if defined,
else default to .Chart.Name from Chart.yaml

\<chart\>.fullname - Use .Values.fullnameOverride (with trunc 63 and
trimSuffix -) if defined,
else build a variable \$name from .Values.nameOverride if defined or use
.Chart.Name
and then check if \$name is in .Release.Name
if it is then use .Release.Name (with trunc 63 and trimSuffix -)
else use .Release.Name-\$name (with trunc 63 and trimSuffix -)

\<chart\>.chart

\<chart\>.selectorLabels

\<chart\>.labels

\<chart\>.serviceAccountName






DEPENDENCIES

- to specify values for a dependency chart put those values into
values.yaml of the main chart in a section with the name of the
dependency chart



TEMPLATES

looks to be like a mixture of text and templating language components

named templates - templates that have a name and can be included in
other templates



TEMPLATING LANGUAGE

Syntax:

{{ *function parameter* }} - the double curly braces are called
'actions', function is called with parameter
{{- "text" - remove all leading white space and new line *after 'text'
has been rendered*
"text" -}} - remove all trailing white space and new line *after 'text'
has been rendered*
{{ \$variable := value }} - a variable is assigned a value, to reference
use the \$ sign as well, the type of the value (boolean, string, int)
cannot be changed after the first assignment, the value itself can be
changed

{{- if and value1 value2 }} - if value1 and value2 are both true
{{- if or value1 value2 }} - if value1 or value2 are both true
{{- If eq value1 value2 }} - if value1 and value2 are equal
{{- with *list* }} - subsequent use of . in with function refers only to
list and not root of data
{{- end }} - end an if/with/range statement,
( ) - whatever is in parentheses will execute first
{{/\* comment \*/}} - comments



Special symbols:

\$ - is mapped to the root scope when template execution begins and does
not change during template execution
\$.Values.image - refers to image in the Value.


Keywords:


Actions:

Note: the output of actions cannot be passed to functions (i.e. piping
is not allowed)

define - e.g. {{- define "mychart.labels" }} , the 'labels' function
lives in the 'mychart' namespace, the 'mychart' namespace is at the root
of the data alongside '.'
template - render a pre-defined template
block


Functions:

Note: the output of functions can be passed to functions (i.e. piping is
allowed)

include - include a named template called my_template
e.g. {{ include "my_template" . }}

toYaml - convert data that was received to yaml format, mostly for
bringing a dictionary of values (say from a values file) into a yaml
template
e.g. {{- toYaml .Values.imagePullSecrets \| nindent 8 }}

with - scoping, once 'with' is entered . becomes local in scope, toYaml
maybe required to handle the data coming in

indent - e.g. {{ .Values.foo \| indent 4 }} - render .Values.foo and
indent it 4 spaces, remember that indent needs to work with a string

nindent - newline then indent

default - e.g. {{ default "foo" \$bar }} assign *foo* if no value was
found in \$bar, {{ \$value \| default 'aValue' }} is also a legitimate
syntax

contains - e.g. {{ contains "foo" .Values.bar }} - check if .Values.bar
contains foo

upper

lower

quote

if/else

tuple - make a list out of some items
{{ tuple "small" "medium" "large" }}

range - looping, once loop is entered . become local in scope, it
references the values that need to be looped over (think of range as a
for each style loop)
{{- range .Values.env }} - for each item (represented with .) in the env
list variable in the Values file
{{- range tuple "small" "medium" "large" }} - range works on the list
created by tuple
{{- range \$index, \$item := .Values.env }} - \$index goes from 0, 1, 2,
3 \...\... and \$item takes on the value of each item in the .Values.env
list
{{- range \$key, \$val := .Values.vars }} - \$key wil become key and
\$val will become value for each entry in the .Values.vars dictionary

contains *arg1 arg2* -

required - e.g. {{ required "default_value" .Values.foo }}

printf - e.g. {{ printf "%s%s" .username .password }}

tpl - this function evaluates its first argument as a template in the
context of its second argument and returns the rendered result
\# values
full_name: \"{{ .Values.name }} Sawyer"
name: \"Tom\"

\# template
{{ tpl .Values.full_name . }}

\# output
Tom Sawyer

b64enc


Full List of Functions:
https://helm.sh/docs/chart_template_guide/function_list


Categories of Functions:
- Cryptography and Security
- Date
- Dictionaries
- Encoding
- File Path
- Kubernetes and Chart
- Logic and Flow Control -
 [and](https://helm.sh/docs/chart_template_guide/function_list/#and), [coalesce](https://helm.sh/docs/chart_template_guide/function_list/#coalesce), [default](https://helm.sh/docs/chart_template_guide/function_list/#default), [empty](https://helm.sh/docs/chart_template_guide/function_list/#empty), [eq](https://helm.sh/docs/chart_template_guide/function_list/#eq), [fail](https://helm.sh/docs/chart_template_guide/function_list/#fail), [ge](https://helm.sh/docs/chart_template_guide/function_list/#ge), [gt](https://helm.sh/docs/chart_template_guide/function_list/#gt), [le](https://helm.sh/docs/chart_template_guide/function_list/#le), [lt](https://helm.sh/docs/chart_template_guide/function_list/#lt), [ne](https://helm.sh/docs/chart_template_guide/function_list/#ne), [not](https://helm.sh/docs/chart_template_guide/function_list/#not), [or](https://helm.sh/docs/chart_template_guide/function_list/#or),
and [required](https://helm.sh/docs/chart_template_guide/function_list/#required).
- Lists
- Math
- Network
- Reflection
- Regular Expressions
- Semantic Versions
- String
- Type Conversion
- URL
- UUID


Pipelines:

Pipelines are a way of organising functions similar to how bash
pipelines work
Pipelines are a chain of function calls.
The input/final argument of the function comes from the output of the
previous function


True/False
False - boolean false, numeric zero, empty string, nil/null, empty
collection (map, slice, tuple, dict, array)
True - everything else










COMMANDS


Get the templates rendered without actually installing
1. helm install \--debug \--dry-run \<release_name\> \<chart_name\> -
validates with kubernetes api server, summarises release info at top
2. helm template \<release_name\> \<chart_name\> - does not validate
with kubernetes api server, does not summarise release info, always
works like a new installation,
any functions that fetch data from kubernetes for rendering just produce
default values

Install
1. helm install
2. helm install \--atomic \[\--timeout \<XmYs\>\] - does an atomic
install, \--wait is activated and if pods don't finish coming up in the
wait timeout period then a rollback is done to the previous successful
release
3. helm install \--verify \--keyring /path/to/keyring *release
repo/chart* - verify a chart signature then install, e.g. helm install
\--verify \--keyring \~/.gnupg/secring.gpg temprelease
localrepo/firstchart


Upgrade
1. helm upgrade - restart those pods whose configurations have changed
2. helm upgrade \--force - the existing deployment will not be modified,
it will be deleted and recreated


Get values for a chart
helm show values \<repository\>/\<chart\>

Rollback
helm rollback


Uninstall
helm uninstall \[\--keep-history\] - \--keep-history if you desire to do
a rollback after uninstallation


Status of a release
helm status


Get information about a release
helm get manifest \<release_name\> \[\--revision \<number\>\] -
manifests for a release, optionally specify a revision number
helm get values \<release_name\> \[\--revision \<number\>\] - just the
user supplied values for the release (meaning the ones that were
changed), optionally target a particular revision
helm get values \<release_name\> \--all \[\--revision \<number\>\] - all
values that apply to the release, optionally target a particular
revision
helm get notes \<release_name\>
helm history \<release_name\> - installation history of a release, this
will include information on failed installations and upgrades


Rollback a release to a previous revision
helm rollback \<release_name\> \<revision_number\> - helm does not wipe
out release history, instead a new revision is created from the desired
\<revision_number\> and rollback is mentioned in the description


Package a chart
helm package \<dir_name\>


Linting
helm lint \<chart_dir\> - INFO,WARN,ERROR helm also lints against a
schema.json for the values file if a schema.json is present


Searching for a chart
helm repo update
helm search repo *chartname*


Pull a chart from a repo
helm repo update
helm pull \<repo\>/\<chart\>


Sign and package a chart
helm package \--sign \--key \<key-id\> \--keyrring /path/to/keyring
\<chart-name\> \[-d \<path/to/directory/to/store/chart.tgz\]

helm package \--sign \--key \'gaz@banat.com\' \--keyring
\~/.gnupg/secring.gpg firstchart -d chartsrepo/


See the value of a helm environment variable
helm env \[\<variable\> \] - e.g. helm env HELM_DATA_HOME


See the available plugins
helm plugin list

Install a plugin

helm plugin install *path-to-plugin*  - path to plugin can be a URL like
https://github.com/salesforce/helm-starter.git OR a folder on local disk

What can a plugin do?
helm \<plugin\> \--help - e.g. helm starter \--help

How to update a plugin
helm plugin update \<plugin\> - e.g. helm plugin update starter

Run a plugin
helm \<plugin\>
