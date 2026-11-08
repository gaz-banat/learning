
=======================
PYTHON FOR MAC SYSTEM
=======================

--------------
PYTHON
--------------

which python
/usr/bin/python

python —version
2.7.16

ls -l /usr/bin/python
/usr/bin/python@ -> ../../System/Library/Frameworks/Python.framework/Versions/2.7/bin/python2.7

python -m site
sys.path = [
    '/Users/gaz',
    '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python27.zip',
    '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7',
    '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/plat-darwin',
    '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/plat-mac',
    '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/plat-mac/lib-scriptpackages',
    '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/lib-tk',
    '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/lib-old',
    '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/lib-dynload',
    '/Library/Python/2.7/site-packages',
    '/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python',
    '/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/PyObjC',
]
USER_BASE: '/Users/gaz/Library/Python/2.7' (doesn't exist)
USER_SITE: '/Users/gaz/Library/Python/2.7/lib/python/site-packages' (doesn't exist)
ENABLE_USER_SITE: True




PYTHONS IN /Library/Python
===========================

ls -l /Library/Python/
total 0
drwxr-xr-x  3 root  wheel   96 10 Nov  2019 2.7/	[THIS IS THE SYSTEM ONE DON’T TOUCH]
drwxr-xr-x  5 root  wheel  160  7 Oct 15:05 3.7/





=======================
PYTHON FOR USERS
=======================

----------
PYTHON 3.9.6 aka /usr/bin/python3 (this seems to be not via brew) and is lower in the PATH
----------


/usr/bin/python3 — version						 			- 3.9.6
/Library/Developer/CommandLineTools/usr/bin/python3 			- 3.9.6
/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/Resources/Python.app/Contents/MacOS/Python 		- 3.7.3


sys.path = [
    '/Users/gaz/Learning_Terraform/udemy_hcta/Other/Practice-7/webserver',
    '/Users/gaz/Learning_Python/pymods',
    '/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python39.zip',
    '/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9',
    '/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9/lib-dynload',
    '/Users/gaz/Library/Python/3.9/lib/python/site-packages',
    '/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9/site-packages',
]
USER_BASE: '/Users/gaz/Library/Python/3.9' (exists)
USER_SITE: '/Users/gaz/Library/Python/3.9/lib/python/site-packages' (exists)
ENABLE_USER_SITE: True




/usr/bin/pip3 --version
pip 21.2.4 from /Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9/site-packages/pip (python 3.9)



-----------
PYTHON 3.12.4 aka /usr/local/bin/python3 (installed via brew) and is higher in the PATH
-----------

Which python3 are we using 
which python3 						-		/usr/local/bin/python3
python3 —version					-		3.12.4
/usr/local/bin/python3			-		3.12.4


Where does it come from
ll /usr/local/bin/python3
lrwxr-xr-x  1 gaz  admin  40  1 Aug 09:11 /usr/local/bin/python3 -> ../Cellar/python@3.12/3.12.4/bin/python3
OR
ll /usr/local/opt/python3
lrwxr-xr-x  1 gaz  admin  28  1 Aug 09:11 /usr/local/opt/python3 -> ../Cellar/python@3.12/3.12.4


This is what a python3 installation looks like
 ll /usr/local/Cellar/python@3.12/3.12.4
total 96
drwxr-xr-x   3 gaz  admin     96  7 Jun 04:26 Frameworks
drwxr-xr-x   3 gaz  admin     96  7 Jun 04:26 IDLE 3.app
-rw-r--r--   1 gaz  admin   4101  1 Aug 09:11 INSTALL_RECEIPT.json
-rw-r--r--   1 gaz  admin  13936  7 Jun 04:26 LICENSE
drwxr-xr-x   3 gaz  admin     96  7 Jun 04:26 Python Launcher 3.app
-rw-r--r--   1 gaz  admin   8703  7 Jun 04:26 README.rst
drwxr-xr-x  16 gaz  admin    512  1 Aug 09:11 bin
drwxr-xr-x   3 gaz  admin     96  7 Jun 04:26 lib
drwxr-xr-x   4 gaz  admin    128  7 Jun 04:26 libexec
-rw-r--r--   1 gaz  admin   8914  1 Aug 09:11 sbom.spdx.json
drwxr-xr-x   3 gaz  admin     96  7 Jun 04:26 share




Let’s investigate a little bit on the actual binary

ll /usr/local/Cellar/python@3.12/3.12.4/bin/python3
lrwxr-xr-x  1 gaz  admin  56  7 Jun 04:26 /usr/local/Cellar/python@3.12/3.12.4/bin/python3 -> ../Frameworks/Python.framework/Versions/3.12/bin/python3

ll /usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/bin/python3
lrwxr-xr-x  1 gaz  admin  10  7 Jun 04:26 /usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/bin/python3 -> python3.12


ll /usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/bin/python3.12
-rwxr-xr-x  1 gaz  admin  13592  1 Aug 09:11 /usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/bin/python3.12       <-- The real executable




brew info python3
==> python@3.12: stable 3.12.4 (bottled)
Interpreted, interactive, object-oriented programming language
https://www.python.org/
Installed
/usr/local/Cellar/python@3.12/3.12.4 (3,297 files, 62.7MB) *
  Poured from bottle using the formulae.brew.sh API on 2024-08-01 at 09:11:27
From: https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/p/python@3.12.rb
License: Python-2.0
==> Dependencies
Build: pkg-config ✔
Required: mpdecimal ✔, openssl@3 ✔, sqlite ✔, xz ✔
==> Caveats
Python has been installed as
  /usr/local/bin/python3

Unversioned symlinks `python`, `python-config`, `pip` etc. pointing to
`python3`, `python3-config`, `pip3` etc., respectively, have been installed into
  /usr/local/opt/python@3.12/libexec/bin

See: https://docs.brew.sh/Homebrew-and-Python
==> Analytics
install: 289,529 (30 days), 905,571 (90 days), 2,275,944 (365 days)
install-on-request: 103,476 (30 days), 302,835 (90 days), 605,827 (365 days)
build-error: 1,347 (30 days)



Where will our python look for its modules before site module is loaded
python3 -c "import sys; print(*sys.path,sep='\n')"

/Users/gaz/Learning_Python/pymods
/usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/lib/python312.zip
/usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/lib/python3.12
/usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/lib/python3.12/lib-dynload
/usr/local/lib/python3.12/site-packages


env | grep PYTHONPATH
PYTHONPATH=/Users/gaz/Learning_Python/pymods



Where will our python look for modules after site module is automatically loaded when python3 starts up
python3 -m site
sys.path = [
    ‘<the current directory>’,
    '/Users/gaz/Learning_Python/pymods',

    '/usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/lib/python312.zip',
    '/usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/lib/python3.12',
    '/usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/lib/python3.12/lib-dynload',
    '/usr/local/lib/python3.12/site-packages',

    '/usr/local/opt/python@3.12/Frameworks/Python.framework/Versions/3.12/lib/python3.12/site-packages',
]
USER_BASE: '/Users/gaz/Library/Python/3.12' (doesn't exist)
USER_SITE: '/Users/gaz/Library/Python/3.12/lib/python/site-packages' (doesn't exist)
ENABLE_USER_SITE: True





PIP

which pip
/usr/local/bin/pip

pip -V
pip 24.1.2 from /usr/local/lib/python3.10/site-packages/pip (python 3.10)




PIP3

which pip3
/usr/local/bin/pip3

pip3 -V
pip 24.0 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)


