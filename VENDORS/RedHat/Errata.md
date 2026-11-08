


RH CDN   ---------> Satellite (on prem) ----------------> System



Satellite makes Errata available as
- Applicable (errata applies to packages installed on servers but is yet not synchronized into the subscribed content view)
- Installable (errata applies to packages installed on servers, is synchronized into the subscribed content view, ready to be deployed)




# BUGS

https://bugzilla.redhat.com



# ERRATA (ADVISORY)

The Red Hat Errata system is an advisory publishing and package management framework used to track and deliver 
- software updates
- bug fixes
- and security patches for Red Hat Enterprise Linux (RHEL) and related products.

It categorizes updates so administrators can quickly prioritize critical vulnerabilities




ADVISORY        -   the published text
ERRATA          -   the packaged release


Types of Advisory/Errata

RHSA        -       Red Hat Security Advisory                   Low, Moderate, Important, Critical
RHBA        -       Red Hat Bug Fix Advisory
RHEA        -       Red Hat Enhancement Advisory

An advisory advises which CVE's are addressed


Links

Errata              -   https://access.redhat.com/errata/#/, https://access.redhat.com/errata-search/
Notifications       -   https://www.redhat.com/wapps/ugc/protected/notif.html
Satellite Server    -   https://access.redhat.com/documentation/en-us/red_hat_satellite/6.3/html-single/errata_management_guide/index
YUM/DNF             -   https://access.redhat.com/solutions/258973

Security Advisories -   https://access.redhat.com/security/security-updates/security-advisories
CVE Database        -   https://access.redhat.com/security/security-updates/#/cve
Security Labs       -   https://access.redhat.com/security/security-updates/#/security-labs