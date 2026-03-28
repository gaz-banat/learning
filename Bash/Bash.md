



talhleper gencommand | sh



IMPORTANT FILES FOR BASH

System wide
	•	/etc/profile: system wide environment setup for login shells, will execute scripts in /etc/profile.d/
	•	/etc/profile.d/*: initialization scripts
	•	/etc/bashrc: meant for functions and aliases

User specific
	•	~/.bash_profile: initialization for all interactive (bash-)shells
	•	~/.bash_login
	•	~/.bashrc: initialization for login (bash-)shells
	•	~/.profile: used for all shells
	•	~/.cshrc, ~/.tcshrc, ~/.zshrc, ~/.tcshrc: similar for non-bash shellsBash





HOW BASH STARTS UP


Background info login program					-	getty runs on terminal device, collects username and gives it to login.
											login reads the password database and decides whether it needs to ask for a password or not.
											login reads /etc/pam.d/login (or equivalent) and acts upon it
											login execs the relevant login shell with a hyphen in front indicating the shell is a login shell

Background info sshd						-	sshd runs at a port (mostly TCP/22), listens for incoming requests
											sshd responds based on its own configuration /etc/sshd and then reads and acts upon /etc/pam.d/ssh
											stdin is connected to network connection


When Invoked as bash

Interactive and Login	(-/bin/bash)			-	/etc/profile  (/etc/profile.d/*.sh)	 ~/.bash_profile (~/.bashrc ( /etc/bashrc ) ) ELSE ~/.bash_login ELSE ~/.profile
											then on logout - ~/.bash_logout

Interactive and Non Login						-	/etc/bashrc and then	~/.bashrc


Non-interactive shell 						- 	looks for $BASH_ENV in environment, expand its value, use the value as name of a file to read and execute


Non-login non-interactive remote (rshd, sshd) 		-	~/.bashrc	( /etc/bashrc)		This has to be enabled on a per vendor basis 





When Invoked as sh

Interactive and Login						-	/etc/profile  (/etc/profile.d/*.sh)  ~/.bash_profile (~/.bashrc ( /etc/bashrc ) )			

Interactive and Non Login						-	looks for $ENV variable, expands its value, uses the value as name of file to read and execute

Non-interactive							-	does not attempt to read any startup files

Invoked in POSIX mode (—posix) as interactive	-	looks for ENV variable, expands its value, uses the value as name of file to read and execute





SHELL AND ENVIRONMENT VARIABLES

Environment variables		-		are setup by the operating system for the bash process, type env to get the environment variables, these get passed to a process started by the shell
Shell variables				-		belong to the shell, these dont get passed to a process started by the shell

to make a shell variable available in the list of environment variables ‘export’ it.
HELLO=hi
env | grep HELLO
export HELLO
env | grep HELLO



COMMON ENVIRONMENT VARIABLES

PATH
USER			-	the currently logged in user
HOME
LANG
LOGNAME
PWD
SHLVL
SHELL			-	the default shell
TERM




COMMON SHELL VARIABLES

IFS





REDIRECTION

> or 1>				- redirect stdout
2>					- redirect stderr
&>					- redirect stdout and stderr
>&2					- redirect stdout to stderr
2>&1				- redirect stderr to stdout
<>					- redirect stdin and stdout
>|					- force redirection even if no clobber option is set


<<					- read from heredoc
<<-					- read from heredoc
<<<					- redirect the stdin of a single process to a string, e.g. read -ra CERTS <<< ${CERTS_VARIABLE}, this is a here-string
< <(process)			- process substitution, redirect the output of a process, treat the inner process as if it were a file, this way the output of a process can be fed to a while loop for e.g.
>(process)			- process substitution, lets the outer process write to inner process as if the inner process were a file




BASH KEYWORDS/RESERVED WORDS

Note: run the command compgen -k
They are used to begin and end the shell’s compound commands. 
The following words are recognized as reserved when unquoted and the first word of a command. 
‘in’ is recognized as a reserved word when it is the third word of a case or select or for command
‘do’ is recognized as a reserved word when it is the third word of a for command

if
then
else
elif
fi
case
esac
for
select
while
until
do
done
in
function
time
{					-	string expansion and concatenation, extended expansion, block of code for function, place holder for text in xargs, pathname in find constructs
				  	it is possible to redirect input and output from {} code block
}
!					- 	invert the exit status of a command (give space between the ! and the command), invert the meaning of a test operator, recall command with number in history, 
						!command  invokes the history status of bash and runs the most recent instance of that <command>
[[					-  	test, more flexible than [ operator
				    	No filename expansion or word splitting takes place between [[ and ]], but there is parameter expansion and command substitution
				    	the &&, ||, <, and > operators work within a [[ ]] test, despite giving an error within a [ ] construct
				  	can do octal and hex evaluations, [ cannot
]]					-	close a [[ test
coproc
local				-	define a variable local to a function





BASH SPECIAL CHARACTERS

Note: Switch off the meaning of special characters of bash by escaping them or quoting them

#				- mostly a comment, however it is not a comment in certain parameter subs and base conversions
;				- command seperator
;;				- terminator in case option
.				- current directory, prefix of hidden file
,				- links together a series of arithmetic operations. All are evaluated, but only the last one is returned. 
				  can concatenate strings (try ls {,~}/bin)
*				- wild card for file name globbing
()				- creates a subshell
				  also () are used to initialise an array
				  <string>() indicates that <string> is a function
[ ]				- can be used to reference an array element
(( ))				- integer expansion
				  returns 0 or 1 based on the numeric expression this construct evaluates
				  if the expression evaluates to 0 it returns 0 as a value and an exit status of 1
				  if the expression evaluates as anything else it returns 1 as a value and an exit status of 0
&				- run job in background
|				- pipe, starts a child bash
|&				- connects stdout and stderr of left side command to right side command
||				- conditional or
&&				- conditional and
- -				- long option to a command, used with commands it can signal end of options for that command
-				- prefix for option flags and operators, 
				  certain utilities like tar, cat, file recognise this as stdin or stdout
				  previous working directory with cd -
~				- home directory, can also be used with ~<username>
~+				- current working directory
~-				- previous working directory
“ “				- partial quotes, do not effect $, ` ` and \  {note that if $, ` ` and \ do not produce any meaning for bash in double quotes they will passed as is for execution}
‘ ‘				- full quotes, the special meaning of all special characters will get switched off
\				- escape the character after it, can also mean continue to the next line
	




BASH SPECIAL VARIABLES/CONSTRUCTS

$$				- the process ID of the bash shell, if you run in this a script you will get the process of the bash shell running the script and not the script itself
$#				- number of command line arguments
${@}			- all the positional parameters to a script (all the command line arguments)
	${@:n}		- all the positional parameters starting from the nth position, where counting begins with 1 (and not zero), e.g. ${@:3} will give omit the first two parameters and give the parameters from the third onwards
${#@}			- the length of the array (@) that holds all the positional parameters
$*				- all the positional parameters
$?				- exit status of the last command executed, 0 is success and any other number is failure
$0				- the script itself
$1, $2, $3 .. 		- positional parameters, arguments
${!var}			- indirect reference to var, e.g. var1=“var2”; var2=“alpha”; var3=${!var1}; echo ${var3} gives alpha
				  think of it as take the value of var1 and treat it as a variable and get me that variables value
$_				- the last argument passed to the last command that was run
$-				- returns the current shells flags
$!				- the last part of the previous command that was run
$’....’			- a way to expand ANSI escape sequences
	$’\…’			- can be octal or hex values which can be referenced as ASCII or Unicode characters
	$’\0’ 			- is the nil byte
	$’\n’ 			- is the newline
	$’hello\nhi’		- is 2 lines, the first line is hello and the second line is hi
$[expression]		- evaluate expression
$((expression))	- evaluate expression
$(expression)		- run expression as a command in a sub shell and produce an output, then the $ sign is helping to preserve that output so that it can be reused on the existing command line





BUILTINS

Note: builtins do not fork new processes

.					- 		same as source
:					- 		does nothing and returns true
[ 					- 		test
declare				-		declare can be used to create various types of variables (arrays, associative arrays, integer, etc) and give those variables attributes (readonly, enforce lower case, etc.) before assignment happens
test
shift					- 		moves positional parameters to a script leftwards
true
let “expression”		- 		evaluates expression as a numerical expression
					 		similar to (( )) it returns 0 or 1 based on the result of expression, if the expression evaluates to 0 it returns 0, if the expression evaluates to anything else it returns 1
				 			a return value of 0 is equated to false per bash logic therefore making $? non-zero, a return value of 1 is equated as true per bash logic therefore making $? zero
export				-		make a shell variable available in the environment
printf
getopts
shopt				-		specific to Bash, enable or disable options for the current shell
echo
exec				- 		used to execute a command from bash itself, does not create a new process, replaces the calling bash with the command to be executed, therefore no return to the calling process
eval					-		a way of providing a second level of bash/shell expansion
read
set					-		set allows you to change the values of shell options and set positional parameters OR display the names and values of shell variables
trap
which
help
command				-		run a command with arguments suppressing shell function lookup
type					-		available options for a command





CONDITIONALS

if
if expression			- 	test a command or expression, proceed if true, don’t proceed if false
if ! expression			-	test a command or expression, proceed if true (the command or expression returned a non zero value)


then				- 	follows if, elif
elif
else


Ex 1
expression is a command 
- if the command succeeds it will return with exit status 0 and bash will interpret that 0 as true
- if the command fails it will return with exit status non 0 and bash will interpret non 0 as false


Ex 2
expression is a logical expression and tested with “[“ command or “test” command
- if the logical expression is true then proceed
- if the logical expression is not true then don’t proceed


Ex 3
expression is a logical expression and tested with [[ keyword
- if the logical expression is true then proceed
- if the logical expression is not true then don’t proceed


Ex 4
expression is a numerical expression and tested with (( )) construct
- if (( )) returned 1 then that is true
- if (( )) returned 0 then that is false




Case



OPERATORS

file test
-e				- exists
-f				- regular file
-s				- has some size
-d				- is a directory
-b				- is a block device
-c				- is a character device
-p				- is a pipe
-S				- is a socket
-t				- is a terminal, applies only to file descriptors 0,1 and 2
-h/L				- file is a symbolic link
-r				- readable
-w				- writable
-x				- executable
-g				- set-group-id flag is set
-u				- set-user-id flag is set
-k				- sticky bit is set
-O				- you are the owner of the file
-G				- group-id is the same as yours
f1 -nt f2			- f1 is newer than f2
f1 -ot f2			- f1 is older than f2
f1 -ef f2			- f1 and f2 are hard links to the same file


variable test
-n				- has some value, i.e. is not null
-z				- has zero value, either because it is unset or because it is of zero length


integer comparison
-eq
-ne
-gt
-ge


logical operators
&&				- logical and
II				- logical or






PATTERN MATCHING


Globbing:

Globbing is the shell’s way of providing regular expression pattern matching, specifically used for finding files and content in files

where can globbing be used
- in filename expansion, i.e. in command line processing after commands like ls, rm, cp, etc. 
- after the =~ in a bash [[ expr ]] 
- after the == in a bash [[ expr ]]
- in the patterns to a case command
- in parameter expansion wherever a pattern needs to be specified


Without any operators: 

Examples:

var=“funny bunny”
[[ “${var}” =~ “funn” ]]		-		true, the pattern ‘funn’ occurs in $var
! [[ “${var}” =~ “funk” ]]		-		true, funk does not appear in $var (pay attention to the space between ! and [[)
! [[ “${var}” =~ “fun” ]]		-		false, ‘not fun appears in var’ is not true
[[ “${var}” == “funny” ]]		-		$var is ‘funny’



With the following simple matching operators:

.	-		i think dot matches a single character
*	-		match any sequence of characters except slashes
?	-		match a single character without case
[ ]	-		match a character sequence
!	-		negate the meaning
\	-		escape the meaning, so \? would match a literal question mark
**	-		match any number of path segments including none


Examples:

ls *.txt							-			returns files that end in .txt

rm ab?.yaml						-			removes .yaml files that start with ab followed by one more character

VAR=“alpha-beta-gama”
echo ${VAR/g?ma/theta} 				-		 	produces alpha-beta-theta


var=“funny bunny”
[[ "$var" =~ "fun"* ]]		-		true, fun is followed by a sequence of characters, keep the * out of the quotes
[[ “$var” =~ “fun*” ]]		-		false, fun is not followed by the asterisk character
[[ "$var" =~ fun* ]]		-		true, fun is followed by a sequence of characters
[[ "$var" =~ fun? ]]		-		true, fun is followed by a character
[[ "$var" =~ fun[anx] ]]		-		true, fun is followed by ’n’ which is in the sequence anx 
[[ "$var" =~ fun[a-z] ]]		-		true, fun is followed by a character between a to z including
[[ "$var" =~ fun[a-e] ]]		-		false, fun is not followed by a character between a to e including




Extended Globbing:

Extended Globbing is done with extended pattern matching operators:

Note: need extglob shell option which can be set with ‘shopt -s extglob’
Note: use the == operator and not the =~ operator

where can extended globbing be used
- in filename expansion, i.e. in command line processing after commands like ls, rm, cp, etc. 
- after the == in a bash [[ expr ]]
- in the patterns to a case command
- in paramete expansion where a pattern needs to be specified


with the following operators

?(pipe separated pattern-list)	-		Matches zero or one occurrence of the given patterns
*(pipe separated pattern-list)	-		Matches zero or more occurrences of the given patterns
+(pipe separated pattern-list)	-		Matches one or more occurrences of the given patterns
@(pipe separated pattern-list)	-		Matches one of the given patterns
!(pipe separated pattern-list)	-		Matches anything except one of the given patterns



Examples:

ls +(.bash)*		-		will show all files and directories starting with .bash


VAR=“alpha-beta-gama”
echo ${VAR/+(gama)/theta}. # produces alpha-beta-theta


var=“funny bunny”
[[ “${var}” == +(fun)* ]]		-		true, $var matches “starting with fun followed by some characters”
[[ “${var}” == +(fun) ]]		-		false, $var does not match ‘starting with fun and then no characters follow’
[[ "$var" == +(gaz|fun)* ]]					-			true, $var matches “starting with fun followed by some characters”







LOOPS


General Syntax

while expr
do
	..do something..
done  < input_object 


Ex 1. Read a file line by line
while read line
do
	echo $line
done < /path/to/file


Ex 2. Read from the output of a process (using process substitution)
while read line
do
	grep ‘[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}’ $line
done < <(ifconfig)


Ex 3 While true do (: is a placeholder that is always true)
while :
do
	something
	if something
	then
		exit or break
	fi
done


Ex 4. Pipe the output of a command to a while loop with read, read is receiving the line and reading it into tokens

kubectl get es -A | while read ns name x; do echo $ns $name; echo done






ARRAYS

Regular:
arr1=(‘alpha’, ‘beta’, ‘gama’) 
arr1=( $(echo happy birthday to you) )
declare -a arr1=(cat dog cow elephant) OR declare -a arr1; arr1=(cat dog cow elephant)

$arr1 or ${arr1}		-		first element of array
${arr1[0]}			-		first element of array
${#arr1[0]}			-		length of first element of array
${arr1[@]}			-		all the elements of the array
${#arr1[@]}			-		number of elements in the array
${!arr1[@]}			-		the list of indices in the array, e.g. 0 1 2 3 for arr1
arr1+=(tiger leopard)	-		add to array


Associative:
declare -A assArr1=( [mother]=Roze [daughter1]=Dilkash [daughter2]=Dilara) 
OR 
declare -A assArr1; assArr1=( [mother]=Roze [daughter1]=Dilkash [daughter2]=Dilara)


${assArr1[<key>]}	-		<value> for <key>

${!assArr1[@]}				-			all the keys of the associative array
${assArr1[@]}				-			all the values in the array

${assArr1[<key>]+_}			-			returns true if the key exists 

assArr1[father]=gaz

assArr1+=( [uncle}=Junaid)
unset assArr1[uncle]




PARAMETER EXPANSION

${VAR:-default}			-	set default value for var if var is not set, e.g. ${TEST_MODE:-0}, set TEST_MODE to 0 if it has no value
${VAR:=default}			-	if VAR is unset or null then set its value to default, e.g. ${PREFIX:=/usr/local/share}, set PREFIX to /usr/local/share if it is not set or null

${VAR/pattern1/pattern2}		-	search and replace first instance of pattern1 with pattern2
${VAR//pattern1/pattern2}		-	search and replace all instances of pattern1 with pattern2
${VAR//pattern1}			-	this is a technique for searching pattern1 and replacing with nothing

${VAR#pattern}			-	removes shortest matching prefix from expanded value that matches the pattern
${VAR##*pattern}			-	removes longest matching prefix from expanded value that matches the pattern
${VAR%pattern}			-	removes shortest matching suffix from expanded value that matches the pattern
${VAR%%pattern}			-	removes longest matching suffix from expanded value that matches the pattern
${VAR::-n}				-	removes last ’n’ characters from variable

${VAR:offset:length}			-	extracts substring from expanded value, counting starts at 0
${VAR:offset}				-	extracts substring from offset to end of string
${VAR:$((-offset))}			-	extracts last offset number of characters, so if offset is 7 then last 7 characters
${VAR:$((-offset)):length}		-	extracts substring from offset counting from end

${VAR+<string>}			-	will substitute value of ${VAR} to <string> if ${VAR} is set
${#VAR}					-	returns the length of VAR





HOW A COMMAND RUNS

A simple command is a sequence of:
optional variable assignments 
followed by blank-separated words 
and redirections 
and lastly terminated by a control operator (; or \n)


A command goes through bash/shell expansion in this order
- brace expansion
- tilde expansion
- parameter and variable expansion
- command substitution
- arithmetic expansion
- word splitting
- filename expansion
- quote removal

The first word after the optional assignments is the command to be executed  and is passed as argument zero. The remaining words are passed as arguments to the invoked command. 

If any of the remaining words are variables then they are resolved before being passed to the command BUT not if the command is a builtin. 
The environment of a builtin cannot be modified at runtime. The builtin will inherit the enviroment of the current bash process


Examples

echo ‘hello how are you’											-	echo is the command and hello how are you are the arguments

echo $var1													-	echo is a builtin bash first resolves $var1 to its value and calls echo with it. var1 MUST exist in the environment already

echo $’#!/bin/bash\necho “hello $var1”\n’ > scripty.sh
chmod +x scripty.sh
var1=gaz ./scripty.sh											-	var1 is set to gaz and supplied to the environment of /bin/bash in scripty.sh

var1=gaz echo hello $var1										-	does not work, var1 is set to gaz but not supplied to echo (because echo is a builtin) 
																so when var1 is resolved for echo it is blank because the current environment did not have var1

var1=gaz bash -c ‘echo hello $var1’									-	works, the command ‘bash’ is not a builtin and therefore its environment can be set


IFS=‘;’ while read xx yy zz; do echo $xx $yy $xx; done < file.txt				-	does not work, again like echo, read is a builtin and its environment cannot be changed in this way
(look for the proper way below in Tips and Tricks section)




HOW BASH CAN BE INVOKED

NOTE: in each case a seperate instance of bash than the one you might be in is being invoked!!

Put shebang into script and make the script executable
#!/bin/bash		

Use -c and single quotes
bash -c ‘for i in {1..10}; do echo $i; done’

Put the commands into a file and call the file with bash (the file need not be executable)
echo ‘for i in {1..10}; do echo $i; done’ > scripty.txt
bash scripty.txt
or
cat scripty.txt | bash

Send a command to bash stdin
echo ‘for i in {1..10}; do echo $i; done’ | bash






OPTIONS FOR INVOKING BASH

Possible ways of invoking bash

bash [long-opt] [-ir] [-abefhkmnptuvxdBCDHP] [-o option] [-O shopt_option] [argument …]

bash [long-opt] [-abefhkmnptuvxdBCDHP] [-o option] [-O shopt_option] -c string [argument …]

bash [long-opt] -s [-abefhkmnptuvxdBCDHP] [-o option] [-O shopt_option] [argument …]



Meaning of flags:

-D
-l				- run a login shell
-r				- run a restricted shell
-i 				- run an interactive shell\
-e				- if a pipeline ends with non-zero (error) then exit the script, so a grep that does not match anything at the end of a pipeline will cause the script to exit
-u 				- error on unset variable
-c command
-O shopt_option
-o option
- -				- double dash signals end of options


SHELL FLAGS 	

echo $-			-	get the current flags
set -<flag>			-	set the shell flag, eg. set -H will restore histexpand
set +<flag>			-	remove the shell flag, e.g. set +H will remove histexpand

h				- 	hashall, tells bash to remember the locations of commands it has found through querying your PATH
i				- 	interactive
m				- 	monitor, enables job control so you can send jobs to the background via bg
B				- 	braceexpand, enables brace expansion in bash2
H				- 	histexpand, enables you to re-run a command from history by prefacing its number with an exclamation point
s
r				- 	indicates a restricted shell
e				- 	errexit, causes bash to exit as soon as a command fails
C				- 	noclobber, stops you from overwriting files via redirection
u				- 	error on unset variable




SHELL OPTIONS (think $SHELLOPTS)

set -o 			-	will display options
set -o <option> 		-	will set the option 
set +o <option> 		-	will disable that option

allexport	
braceexpand	
emacs		
errexit				-	exit on error immediately, i.e. if any command in the script produces a non-zero code
errtrace
functrace
hashall			
histexpand		
history			
ignoreeof		
interactive-comments	
keyword			
monitor			
noclobber				-	a feature to prevent accidental overwriting of files
noexec			
noglob			
nolog			
notify			
nounset				-	terminate the shell/script if a variable that has not been initialized is trying to be used
onecmd			
physical
pipefail				-	causes a pipeline of commands to produce a failure return code if any command within the pipeline errors, normally pipelines only return a failure if the last command errors
posix			
privileged		
verbose			
vi			
xtrace			



SHOPT OPTIONS	(think $BASHOPTS)

shopt			-	show all the options
shopt -s <option>	-	set option
shopt -u <option>	-	unset option	

autocd
cdable_vars
cdspell
checkhash
checkjobs
checkwinsize
cmdhist
compat31
compat32
compat40
compat41
direxpand
dirspell
dotglob
execfail
expand_aliases
extdebug
extglob
extquote
failglob
force_fignore
globstar
gnu_errfmt
histappend
histreedit
histverify
hostcomplete
huponexit
interactive_comments
lastpipe
lithist
login_shell
mailwarn
no_empty_cdm_completion
nocaseglob
nocasematch
nullglob
progcomp
promtvars
restricted_shell
shift_verbose
sourcepath
syslog_history
xpg_echo




SUBSHELL

subshell					-	a subshell is (command1; command2; …..; command n)
							this is like a fork+wait
							there is no new process
							the calling bash shell is forking and then waiting for commands in the subshell to finish
							the environment of the calling shell is available in the subshell

fork						-	to fork a different process use a subshell with &
							(command1; command2; …..; command n)&
							The subshell will run asynchronously
							There is a new process
							Note: this is not the same as fork() system call
		
child shell				-	a child shell is a new process created with ‘bash’ or ‘sh’
							it will get its own environment but not shell vars from parent shell unless those shell vars were exported into the parents environment
							it is a new process




COMMAND AND PROCESS SUBSTITUTION

$(command)		-	this is command substitution
					a command is being run in a subshell and used to generate a command or argument (mostly used for generating arguments) for the calling shell

<(command)		-	this is process substitution
					the command is being run in a subshell then the output of that subshell will be presented as a filename in the calling shell from which the output of the command may be read





PROCESS MANAGEMENT

CTRL-z			-	suspend a process (sends a TSTP signal)
fg 				-	bring a suspended process back to running

nohup






BASHIMS

Key Sequences:

CTRL-l			- 	clear screen
CTRL-r			-	reverse search







HEREDOC





RULES

To avoid headaches in your life with bash when referencing a variable always quote it

Quoting
1. referencing a variable in double quotes will preserve the white space sequences as is as part of the variables value
2. bash uses the value of IFS variable as field separators and converts multiple continuous field separators to single white spaces. Partial quoting (double quotes) prevents this from happening.

Command line interpretation
- bash removes quotes

Logic
- for bash 0 is false and false is false, anything not 0 is true and true is true





INTRICACIES OF THE UTILITIES


**echo**

- e				- use this option to turn on the meaning of escaped characters





**grep**

options
-n			-	print line number
-o 			- 	print only the matched text


Regular Expressions with grep

match a single character except end of line		-	.
start of line		-	^
end of line		-	$
one or a range of characters		-	[ ]
negate one or a range of characters		-	[^]
switches off the meaning of a special character. 		-	\
Gives special meaning to an otherwise ordinary character		-	\
match the previous character zero or more times		-	*
match the previous character one or more times		-	\+
match x to y occurrences of the preceding character		-	\{x,y\}
match exactly x occurrences of the preceding character		-	\{x\}	
match x or more occurrences of the preceding character		-	\{x,\}
word boundary (quotes are important) 		-	‘\<‘ and ‘\>’




**egrep**

options
-c 			-	count matching lines
-e			-	Allows using hyphen at the beginning of a pattern


Extended Regular Expressions with grep (use grep -e OR egrep)

allows for this character or this character or this character and so on		-	|		‘A|B|C’
match the previous character zero or more times		-	*
match the previous character one or more times		-	+
previous character may or may not appear		-	?
match x to y occurrences of the preceding character		-	{x,y}
match exactly x occurrences of the preceding character		-	{x}	
match x or more occurrences of the preceding character		-	{x,}
word boundary (quotes are important) 		-	‘\<‘ and ‘\>’



**sed**

options


regular expressions with sed

.				- one character
*				- match the previous character zero or more times
&				- refers to the pattern found
[ ] 				- range of characters

\(…\)			- makes sed remember the pattern and can be referenced as a number, sed remembers up to 9 patterns

\n				- newline
\s				- space character

\{x,y\}			- match x to y occurrences of the preceding character
\{x\}				- match exactly x occurrences of the preceding character
\{x,\}			- match x or more occurrences of the preceding character


extended regular expressions with sed

-r				- turn on extended regular expressions (for e.g. meaning of +)


Solutions:

I want sed to match on a pattern
sed -e ‘/pattern/s|one pattern|another pattern|g’ 




**perl**

options

- e				- evaluate
- p				- print
- i				- in place edit with extension for backing up original



regular expressions with perl


.				- one character
*				- match the previous character zero or more times
?				- less greedy match
{x,y}			- match x to y occurrences of the preceding character
{x}				- match exactly x occurrences of the preceding character
{x,}				- match x or more occurrences of the preceding character
(…)				- makes perl remember the pattern and can be referenced as \1 or \2 or \3 depending 


\d				digits
\s				space






							PCRE 		vs 		Grep/Sed

Backrefrencing 				(…)					\(……\)
Number of occurances			{x,y}					\{x,y\}
Less greedy					.*?					not supported




**sort**

-u 		-	unique
-t		-	field delimiter, e.g. -t, means the comma is the delimiter
-k		-	key field to sort on, e.g. -k1,1





**env**





**eval**

the arguments given to eval are read and concatenated together into a single command. This command is then read and executed by the shell.
the exit status of this command is returned as the value of eval. eval executes the command in the current shell environment rather than creating a child shell process.
Without any arguments eval will return zero/0
							
think that eval takes a string as its argument and evaluates it as if the string was typed on the command line so helps to turn a command into a variable and later on use that command

e.g. 1 - simple eval
c="echo"; a1="Hello, "; a2="World!"; eval $c $a1 $a2


e.g. 2 - eval with parameter expansion
cmd1="cmd2"; cmd2="echo Hi!"; eval \${$cmd1}
1. bash removes the backslash and then runs ‘eval ${$cmd1}’
2. eval evaluates the inner $cmd1 to cmd2 and arrives at ${cmd2}
3. eval evaluates ${cmd2} to “echo Hi!”
4. eval gives bash the command “echo Hi!” to run
5. bash runs ‘echo Hi!’ which gives the output ‘Hi!’

e.g. 3 - eval with command substitution
eval $(ssh-agent)						-	think of it this way, ssh-agent will produce some output, that output should be run through bash command line, 
										eval will ensure that the output is run through bash command line

e.g. 4 - eval with a variable
cmd="ls | less"
$cmd								-	this fails, because bash tries to run the command ls with the arguments ‘|’ and ‘less’. Obviously files named ‘|’ and ‘ls’ are not there
eval “$cmd” 							-	this works, eval tells bash to run the ls command and pipe its output to the less command



https://stackoverflow.com/questions/11065077/the-eval-command-in-bash-and-its-typical-uses

a "typical" use of eval is for running commands that generate shell commands to set environment variables.




**read**
reads one line at a time either from stdin (or file descriptor fd supplied to -u), splits the line into fields and assigns this fields to the given variables 
removes the backslash character ( \ ) unless -r is used. 

	-p				-		prompt
	-s				-		silent, do not display the input on screen
	-n <int>			-		limit input to <int> number of characters
	-a				-		store input into an array based on IFS 
	-r				-		If this option is given, backslash does not act as an escape character. The backslash is considered to be part of the line. In particular, a backslash-newline pair may not then be used as a line continuation.
	 -d ‘’	or d $’\0’		-		tells read to stop reading at the nil byte, note the space between -d and its argument





**set**

set [-abefhkmnptuvxBCEHPT] [-o <shell_option>] [--] [-] [argument ...]
set [+abefhkmnptuvxBCEHPT] [-o <shell_option>] [--] [-] [argument ...]

setting shell options:
-a or -o allexport		-		each variable or function that is created or modified will be exported to environment of subsequent commands
-b
-x 					-		show the command that is to run as well as it output
-e or -o errexit	 		-		make the whole script exit if the command fails
-E 					-		any trap on ERR is inherited by shell functions, command substitutions and commands executed in a subshell environment)
-u or -o nounset 		-		treat unset variables as an error and immediately exit
-f or -o noglob 			-		disable filename expansion (globbing)

 -o <shell_option> 		-		will set the shell option


setting positional parameters:

Ex 1 
set -- one two three
echo $1 $2 $3





**trap**
trap command allows you to manage signals effectively. By using trap you can gracefully handle interruptions, perform necessary cleanup tasks, execute custom code based on the signal received

Ex 1
cleanup() {
  shred -u secret_file > /dev/null
}
# when receiving SIGINT SIGTERM run the cleanup function
trap cleanup SIGINT SIGTERM



**type**
type is good way to find out all the possible alternatives for a given command

Ex1
type -a echo 
this will show that echo is first available as a bash builtin and then as /usr/bin/echo 




**which **
which shows alternatives for a command in the PATH (so builtin commands will not be shown)




**help**
help can be used to get help on a bash builtin

Ex 1 
help read









TIPS AND TRICKS

Check if a variable is set
if [[ -z ${!var_name+z} ]]; then
    echo "  \$$var_name is not set"
    has_unset_vars=1
  fi

---

Build an array from a file, each line is an element of the array
readarray myarray < file.txt

---

Build an array from a process using process substitution
readarray -t  myarray < <(process_that_produces_array)


---

Build an array from a comma separated list in a variable

list=“hello,hi,salaam”
IFS=“,” read -r -a GREETINGS <<< “$list”
# now GREETINGS is an array
echo ${GREETINGS[@]}

---

Change IFS for read command to tokenize words 
Ex. - a file has words per line that are semicolon seperated
while IFS=‘;’ read xx yy zz
do 
	echo $xx $yy $zz
done < somefile.txt

---

Build and array from a command that gives output on multiple lines (you dont have readarray)
my_array=()
while IFS= read -r line
do
	my_array+=( “$line” )
done < <(my_command)

---

Build an array from a command line output 
IFS=$'\n' read -r -d '' -a my_array < <( my_command && printf '\0' )


---

Read a pem file with many certs

while read line
do
           if [ "${line//END}" != "$line" ]; then
	             txt="$txt$line\n"
            	 printf -- "$txt" | openssl x509 -subject -issuer -dates -fingerprint -noout; echo;
            	 txt=""
           else
            	 txt="$txt$line\n"
           fi
done < $1


---

Output in a colour:

GREEN='\033[0;32m'
BLUE='\033[0;34m'
RED='\033[0;31m'
NC='\033[0m'
 
red() {
  printf "${RED}$1${NC}\n"
}
 
green() {
  printf "${GREEN}$1${NC}\n"
}
 
blue() {
  printf "${BLUE}$1${NC}\n"
}


---

change the stdout of the current script
exec > <new_stdout>

---


Set positional parameters for a script with GNU getopt (dont confuse with bash builtin getopts)

eval set -- $(getopt -o <short-option>:<short-option>:....  -l <long_option>:..... -n “$PROG” -- “$@“)

getopt will get the options as one string
eval will tokenize the string into words
set will set the parameters to the running shell
