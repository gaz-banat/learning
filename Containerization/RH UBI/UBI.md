
UBI

Image Types:

Micro image			(ubi8/ubi-micro)		-	for the absolute smallest use
										no package manager


Minimal image			(ubi8-minimal)		-	for a minimalized pre-installed content set
										micro-dnf
										only English (en) locale
										Initialization and Service management - NO
										no procps-ng package	

Standard/Platform image	(ubi8)			-	for general use
										yum or dnf
										all locales to addess internationalization and localization
										Initiialization and Service Management - YES based on systemd
										no procps-ng package
										SIGNAL is SIGTERM and SIGKILL
										CMD is set to /bin/bash

Multi-service image		(ubi8-init)			-	for multiple processes inside a container managed by systemd
										yum or dnf
										all locales
										Initiialization and Service Management - 
										procps-ng is avaliable
										SIGNAL is SIGRTMIN+3
										CMD is set to /sbin/init  (this will start systemd)


Licencsing:
Red Hat UBI EULA

UBI7

language runtimes images are based on Red Hat Software Collections (rhscl)


UBI8

language runtime images are based on Application Streams packaging

Associated Repositories (dont confuse this with registry/repository/image)
ubi-8-baseos		-	repository holds the redistributable subset of RHEL packages you can include in your container
ubi-8-appstream	-	repository holds Application streams packages that you can add to a UBI image to help you standardize the environments you use with applications that require particular runtimes.

