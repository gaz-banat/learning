\
\
COMMANDS\
\
\
sysctl - get or set kernel state\
e.g. I want to know if VT-x/AMD-v vitualization is enabled: 'sysctl -a
\| grep machdep.cpu.features \| grep VMX' - This should output
something\
\
\
\
\
\
STARTUP\
\
\
\
\
\
SHORTCUTS\
\
\
\
\
LOCATIONS\
\
System Locations:\
\
/System/Applications - System stuff\
/System/Developer\
/System/DriverKit\
/System/Library\
\
\
All User Location:\
\
/Applications - Application stuff\
/Library\
\
\
Per User Locations:\
\
\~/Applications\
\~/Library\
\
\
\
\
BREW\
\
\
Formulae - package definition written in Ruby\
\
tap - third party repository\
\
Cask - brew cask is an extension to brew that allows management of
graphical applications through the Cask project\
a way to command line manage the installation of graphical applications\
\
keg-only - a brew installation that is not symlinked to /usr/local
probably because the MacOS provides similar software
