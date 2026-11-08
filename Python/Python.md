

PYTHON INTERPRETER TO MACHINE CODE

Python Code 	--------------	>	Interpreter		-------------- Interpreted code  --------------------->    CPython VM  ----------- > Machine/Binary Code										0 LOAD_GLOBAL.     0 (print)
										2 LOAD_CONST        1(‘Hello World)’
										4 CALL_FUNCTION   1
										6 POP_TOP
										8 LOAD_CONST         0 (None)
										10 RETURN VALUE	
											

DISCOVER PYTHON3 ON YOUR SYSTEM

What python interpreters are available:
which -a python3				-	/usr/local/bin/python3, /usr/bin/python3


What are the versions of those interpreters:
/usr/local/bin/python3 -V			-	Python 3.13.2
/usr/bin/python3 -V				-	Python 3.9.6


Our first interpreter is /usr/local/bin/python3:
ls -l /usr/local/bin/python3
lrwxr-xr-x  1 gaz  admin  40  1 Mar 16:23 /usr/local/bin/python3 -> ../Cellar/python@3.13/3.13.2/bin/python3


What is the site information for python3:
python3 -m site
sys.path = [
    '/Users/gaz/Learning/Learning_Python',					(THIS IS JUST THE PWD)
    '/Users/gaz/Learning/Learning_Python/pymods',
    '/Users/gaz/Learning/Learning_Python/Pytest',
    '/usr/local/Cellar/python@3.13/3.13.2/Frameworks/Python.framework/Versions/3.13/lib/python313.zip',
    '/usr/local/Cellar/python@3.13/3.13.2/Frameworks/Python.framework/Versions/3.13/lib/python3.13',
    '/usr/local/Cellar/python@3.13/3.13.2/Frameworks/Python.framework/Versions/3.13/lib/python3.13/lib-dynload',
    '/usr/local/lib/python3.13/site-packages',
    '/usr/local/opt/python@3.13/Frameworks/Python.framework/Versions/3.13/lib/python3.13/site-packages',
]
USER_BASE: '/Users/gaz/Library/Python/3.13' (doesn't exist)
USER_SITE: '/Users/gaz/Library/Python/3.13/lib/python/site-packages' (doesn't exist)
ENABLE_USER_SITE: True


Which pip3 is available
which -a pip3					-	/usr/local/bin/pip3


What is the version of pip3
pip3 -V						-	pip 25.0 from /usr/local/lib/python3.13/site-packages/pip (python 3.13)
python3 -m pip -V				 -	pip 25.0 from /usr/local/lib/python3.13/site-packages/pip (python 3.13)
pip3 show pip
Name: pip
Version: 25.0
Summary: The PyPA recommended tool for installing Python packages.
Home-page: https://pip.pypa.io/
Author: 
Author-email: The pip developers <distutils-sig@python.org>
License: MIT
Location: /usr/local/lib/python3.13/site-packages
Requires: 
Required-by: 


What packages are available for the interpreter
pip3 freeze



---
BE WARY OF PIP, I THINK IT IS NOT AVAILABLE AFTER PYTHON3.10!!
Which pip is available:
which -a pip					-	/usr/local/bin/pip

What is the version of pip
pip -V						-	pip 23.0.1 from /usr/local/lib/python3.10/site-packages/pip (python 3.10)
pip show pip
Name: pip
Version: 24.3.1
Summary: The PyPA recommended tool for installing Python packages.
Home-page: https://pip.pypa.io/
Author: 
Author-email: The pip developers <distutils-sig@python.org>
License: MIT
Location: /usr/local/lib/python3.10/site-packages
Requires: 
Required-by: 
---





ENVIRONMENT VARIABLES FOR PYTHON PROCESS

PYTHONPATH				-		path to add additional directories that the user wants Python to add to the sys.path directory list, this will help find modules
PYTHONOPTIMIZE			-		set to True to turn of __debug__ system variable




PYTHON INTERPRETER OPTIONS

-o or -oo					-		start python interpreter with __debug__system variable turned off




PYTHON OBJECTS

In python everything is an object, so the following are examples of objects

int
float
string
list
namespace



NAMESPACES

A namespace is essentially a dictionary of identifiers (aka names) available to a block, function, class, module, and so on
the key is the identifier and the value is the object the identifier points to (the key is a binding to the object)
OR
a namespace is mapping of identifiers to objects

Note: Each module has its own namespace

Order in which Python interpreter checks namespaces for ‘identifiers’ for a block of code:

1. Local namespace				( use the locals() command)
	local functions
	local variables


2. Enclosing namespace


3. Global namespace			(use the globals() command)
	module functions
	module variables


example of global namespace
{
'__name__': '__main__', 
'__doc__': None, 
'__package__': None, 
'__loader__': <class '_frozen_importlib.BuiltinImporter'>, 
'__spec__': None, 
'__annotations__': {}, 
'__builtins__': <module 'builtins' (built-in)>					the ‘__builtins__’ identifier is a reference to the ‘builtins’ module in the ‘built-in’ namespace
}


4. built-in namespace				

	Built-in functions		
	Constants				-	False, True, __debug__, None, Ellipsis



NOTE: 
for a module, a script from file or a command in interactive session the global and local namespaces are the same
for a function a local namespace is created different to global namespace



Commands:

show bindings in the local namespace
locals()

show bindings in the global namespace
globals()

get the identifiers/names of the module
dir(<module_name>)		-

Get help on an identifier
print(<identifier>.__doc__)



Scope for variables:

1. local scope
2. enclosing (parent) scope
3. global scope
4. built-in 




Some key identifiers in the global namespace

__name__			-	name of the module (__name__ identifier in a namespace has the value __main__ for a python file called directly rather than as a module) which is used for
						a script run from file OR for an interactive session

__file__				-	contains the path to the module that is currently being imported
						



MODULES

What:
a module is a python object with arbitrarily named attributes that you can bind and reference
in english - a module is a file consisting of python code that can define functions, classes and variables which you can call in your program by using their names

Where is a module on disk:
- Where a python program will look for modules is defined in ‘sys.path’
	python3 -c "import sys; print(*sys.path,sep='\n')"
- Generally speaking this will be one of 3 locations 
	- the current directory of the python program
	- those paths that are defined by PYTHONPATH
	- python installation dependent module locations

To confirm for a module just run these 2 lines
import <module>
print(<module>.__file__) or print(module)




Module Attributes:
<module>.__name__					-		the module that is being executed has the name ‘__main__’  (a module being executed is not to be confused with a module that is imported)
											a module being imported has its __name__ set to the name of its file
<module>.__doc__						-		documentation string
<module>.__file__
<module>.__dict__						-		shows available objects in the module, the builtins module function dir(<module>) does the same





PACKAGES

What:
packages are a collection of modules


Location of packages:
Global site-packages					-	run ‘python3 -m site’ and look at the list in sys.path
User site-packages					-	run ‘python3 -m site’ and look at the list in USER_SITE


Types of packages:
regular							-	implemented as a directory containing an __init__.py
namespace						-	implemented without an __init__.py


Package directory:

__init__.py			-		this file makes the Python interpreter aware that the directory it is in will contain Python modules (modules are files ending in .py)
							it is the first file to be loaded when package is imported, so you can use it to execute code that you want to run each time the module is loaded
							think of the code in this file as being at the package level, e.g. variables, functions

	__all__			-		this attribute is a list that identifies the names of objects that need to be loaded
	version


__pycache__			-		a directory containing bytecode-compiled and optimized bytecode-compiled versions of python modules in the same directory in which __pycache__ is present
							files will be of type .pyo or .pyc



__main__.py			-		this file is executed if the package directory is passed as an argument to the python3 interpreter, i.e. if you want to run a package dir as a python program





THE CONFUSION OF built-in, builtins and __builtins__

built-in 		-	this is a namespace, it comes with python
__builtins__	-	this is an identifier in the built-in namespace and it points to the builtins module
builtins		-	this is a module, available in the built-in namespace and accessed via the identitfier __builtins__, (this module does not need to be imported)





PYTHON STANDARD LIBRARY -  MODULES AND PACKAGES

Type:

	buillt-in - those which are builtin to python (obtained from sys.builtin_module_names)

		_abc
		_ast
		_codecs
		_collections
		_functools
		_imp
		_io
		_locale
		_operator
		_signal
		_sre
		_stat
		_string
		_symtable
		_thread
		_tokenize
		_tracemalloc
		_typing
		_warnings
		_weakref
		atexit
		builtins					- 	this module does not need to be imported
		errno
		faulthandler
		gc
		itertools
		marshal
		posix
		pwd
		sys
		time

	external library (as seen from pycharm)

		argsparse
		copy
		getpass
		io
		json
		logging
		os
		pathlib
		pickle
		re
		requests
		shelve
		shlex
		shutil
		socket
		struct
		subprocess
		fileinput






builtins MODULE (referenced by __builtins__ identifier)

functions:
	bin
	input
	int
	len
	locals and globals
	max and min
	open
	print
	range
	reversed
	sorted
	str
	sum
	list									-		takes a tuple and returns a list
	tuple
	dir



sys MODULE

sys.argv				-	a list of command line arguments
sys.stdin				-	.read()
sys.stdout





IMPORTING MODULE OR PACKAGE

import <module>						-		a module comes from a file
from <module> import <object>

import <package>						-		a regular package can be imported like a module and its __init__.py file is implicitly executed
import <package>.<module>				-
from <package> import *					-		there is no recursive loading, only that which was identified in __all__ attribute of __init__.py will be loaded
from <package>.<module> import <object>	-		
from <package>.<module> import *			-




PIP/PIP3

pip freeze	or pip list					-	show the globally installed packages in a requirements format
pip freeze --user					-	show the locally installed packages for the user
pip show <package>					-	information of the package including location
pip -V or pip3 -V or pip --version		-	Which version of pip am I using, pay attention to the version of python
pip install <package>[==<version>]		-	install a package optionally with a specific version





PYTHON PACKAGE INDEX (PyPI)






ITERABLES

- an object that has the __iter__ method and the __next__ method
- an iterable need not be a generator, meaning all items in the iterable are present in memory when the iterable is created


list					=	[ ]						slice, .append(), .extend(), .insert(), .index(), .sort(),
										a list can be created by either specifying the elements in [ ] OR by specifying an expression in [ ] which is list comprehension
tuple					=	( )
dict					=	{ }						.keys(), .items()
										referencing the value of a key in a dictionary is by <dict_name>[<key_name>]  - this is similar to an array reference
string				=	“<characters>”				slice, .upper(), .lower(), .replace(), .startswith()
range(x,y,z)
enumerate(iterable)




GENERATORS

- generator object does not produce the entire sequence in memory, but rather a way of accessing each sequenced item one at a time
- generator objects are iterable objects because they have __iter__ method


REGULAR EXPRESSIONS

any one character, including whitespace, not including new line 		-		.


group												-		( )
or													-		|
range												-		[ ]		[a-z] is a through z, put hyphen first if you want to match hyphen, [-A-Z] matches any upper case character and hyphen

Quantifiers
zero or more preceding characters							-		*		This is a greedy match
one or more preceding sequence							-		+		enclose the sequence in ( )
zero or one preceding sequence							-		?		enclose the sequence in ( )
specified number of preceding sequence						-		{n}, {n,}, {n, x}
non greedy pattern matching								-		*?		e.g. match ‘<div>’ from '<div>hello world</div>' by using pattern ‘<.*?>’

Markers
starts with												-		^
ends with												-		$

Unprintable characters
tab													-		\t


Sequences
digits													-		\d
whitespace											-		\s	don’t confuse with one press of space key


Naming a pattern										-		(?P<pattern_name>pattern)



raw strings:
A way of treating a string as literal, no special sequences

The four backslash problem when raw strings are not used:
When a string is supplied to a re.compile(), 2 things happen
- python interpreter evaluates the string
- then re.compile evaluates the string

example1:
Look at this string “\\ten”
python interpreter evaluates the “\\” as one literal “\”
re.compile gets the string “\ten” and says you are looking for a tab followed by “en”

example2:
the string is “\\\ten”
python interpreter evaluates “\\” as one literal “\” and produces  ........


example3:
the string is “\\\\ten”
python interpreter evaluates the first two “\\” as “\” and the second two “\\” as “\” and produces the string “\\ten”
re.compile evaluates the two “\\” as one literal “\” and says you are looking for a literal “\t” followed by “en” 








CLASSES

_var			-		private variable starts with _


methods


Calling methods
1. When calling instance methods from outside the class there is no need to pass self
2. When calling instance methods from inside the class there is a need to pass self
3. When referring to instance variables from inside the class there should be preceded by self






VIRTUAL ENVIRONMENT

Multiple python providers
	pyenv				-	this is slightly different I think, it allows for different python versions

Virtual environment providers
	virtualenv				-	third party, works with python 2 and 3, 
	virtualenvwrapper		-	set of extensions to virtualenv
	venv					-	built-in module to python3	
	conda
	pyvenv				-	deprecated since 3.6

Package management
	pipenv				-	a tool that combines virtual environments with package management



Using venv:

First way
	Create a project directory
	mkdir my_project

	cd to project directory
	cd my_project

	Create a virtual environment with venv 
	python3 -m venv venv

	Activate the virtual environment
	source venv/bin/activate

	Upgrade pip if you want
	pip install --upgrade pip

	Create a requirements.txt with packages you need
	vi requirements.txt

	Install packages from requirements.txt
	pip3 install -r requirements.txt

	Start coding


Second way
	create the project directory as the virtual environment directory
	python3 -m venv <project_directory>

	Continue to activate, upgrade pip, add requirements file and install requirements

	Start coding


NOTE: It seems that the python virtual environment will get its own python interpreter




PYTHON DEBUGGING

pdb.set_trace()			-		start pdb at this point
<var>				-		value of var
step					-		execute the line of code the arrow is pointing to, executes one line at a time
next					-		executes one line at a time, but if doing a function call it executes the entire function and comes back
continue				-		continue till return
clear					-		clear all brakes
a					-		show all arguments
list					-		list the code
w					-		context for the current line







LINTING

pylint
PEP 8





TESTING





REQUESTS

Concepts:

- Requests are blocking
- Requests should be written as generic functions as much as possible


Types of Data:

URL Parameters		-		params=
Form Data			-		data=
JSON				-		json=
Files					-		files=
XML



Request Components:

Method
	GET			give me some data
	POST		I give you some data (you did not have)
	PUT			update some information (you most likely have) 
	DELETE		delete some data

Headers

Data

Authentication		can be part of headers of data
	Basic




Response Components:

Status Code
	1XX			informational
	2XX			success
	3XX			redirect
	4XX			client error, e.g. 404 not found
	5XX			server error

Headers

Data



Environment variables:
REQUESTS_CA_BUNDLE	-		path to ssl ca certs to be used by requests library





JSON TO PYTHON


	JSON											PYTHON


object
dict
array
list
string
str
number (int)
int
number (real)
float
true
True
false
False
null
None

