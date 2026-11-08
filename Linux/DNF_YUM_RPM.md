
--------
YUM
--------


PLUGINS

productid
subscription-manager
auto-update-debuginfo
langpacks
search-disabled-repos









============================

---------
RPM
---------


A package is a header structure on top of a CPIO archive



PARTS OF A PACKAGE

a header with a leading identifier (magic number)		-	this identifies the file as an RPM package
a signature									-	verifies the integrity of the package
header (tagged data)								-	package information, version numbers, copyright messaging, 
archive										-	actual program files




COMMANDS

List the files in a package
rpm -qpl <package>


List a packages pre-instal and post-install script
rpm -qp --scripts <package>









/bin/sh -c cd '/builds/Xvyx5ovn/0/elk/nagios-checks/tp-mkt-nagios-server/target/rpm/tp-mkt-nagios-server/SPECS' && \
 
'rpmbuild'
'-bb'
'--target' 'noarch-redhat-linux'
'--buildroot' '/builds/Xvyx5ovn/0/elk/nagios-checks/tp-mkt-nagios-server/target/rpm/tp-mkt-nagios-server/buildroot'
'--define' '_topdir /builds/Xvyx5ovn/0/elk/nagios-checks/tp-mkt-nagios-server/target/rpm/tp-mkt-nagios-server'
'--define' '_build_name_fmt %%{ARCH}/%%{NAME}-%%{VERSION}-%%{RELEASE}.%%{ARCH}.rpm'
'--define' '_builddir %{_topdir}/BUILD'
'--define' '_rpmdir %{_topdir}/RPMS'
'--define' '_sourcedir %{_topdir}/SOURCES'
'--define' '_specdir %{_topdir}/SPECS'
'--define' '_srcrpmdir %{_topdir}/SRPMS'
'--define' '_gpg_name C13B0395'
'--sign' 'tp-mkt-nagios-server.spec'
