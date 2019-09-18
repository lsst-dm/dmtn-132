:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

Background
==========

`Identity management for LSST services`_ is currently provided by multiple sources.

.. _Identity management for LSST services: https://docushare.lsstcorp.org/docushare/dsweb/Get/Document-32406/LSST%20Identity%20and%20Access%20Managment%20Design.docx

Staff services managed by Tucson IT such as project.lsst.org, Jira/Confluence, etc. have their identities hosted in a Tucson-based LDAP service.

Developer services managed by NCSA include the "lsst-dev" cluster (a remote-login "head node" and batch nodes) and attached filesystems, a VPN, and three Kubernetes clusters that host LSP instances and other services.
These all utilize an NCSA-based identity management system that includes the identity.lsst.org site, an LDAP service, and an underlying NCSA Kerberos service.

The Kubernetes clusters and other Web-based services can make use of `CILogon`_ to enable authentication using federated identities, including the use of NCSA Kerberos to authenticate the NCSA/LSST identity.

.. _CILogon: https://www.cilogon.org/

Duo two-factor authentication can also be configured into the authentication process.

Observatory machines, including those at the Summit and Base as well as those in the Tucson and NCSA test stands, do not currently use a central identity management service; they are using only local accounts at present.

Plans are being discussed to merge the Tucson-based LDAP into the NCSA-based LDAP, unifying these systems.
Plans are also being discussed to integrate the Observatory machines with LDAP.

The NCSA Kerberos service contains identities for all NCSA users.
It is highly secured and therefore difficult to deploy and replicate.
In addition, sharing this service means that NCSA users and LSST users cannot have the same name.
These collisions can force LSST users (or NCSA users) to have to select less-desirable usernames.

The internal `Identity Management Review`_ recommended that separation from the NCSA Kerberos be investigated.
The 2019 LSST Joint Status Review recommended that this separation occur.

.. _Identity Management Review: https://docushare.lsstcorp.org/docushare/dsweb/Get/Document-32503/Document32503_IDmgmtReviewReport_20190402.docx

Alternatives
============

In all cases, two-factor authentication should be usable on top of the base identity service.
This could use Duo, as in the existing system, via NOAO's, NCSA's, or a new LSST-specific configuration.
The ``pam_duo`` module allows two-factor authentication to be applied to ``ssh`` logins.
Another two-factor solution, such as hardware `YubiKey`_ with ``yubico-pam`` could also be used in certain contexts.
As an example, Summit and Base control systems and their bastion hosts and Kubernetes control nodes may want to use a hardware solution since the Summit needs to be able to operate without an external Internet connection.

.. _Yubikey: https://yubico.com/

In addition, Web-based services including deployments on the Kubernetes clusters can continue to use CILogon to provide federated identity.
The CILogon client configuration for LSST might need to be updated to point to a different LDAP, depending on the alternative chosen.


LSST Kerberos
-------------

LDAP in conjunction with Kerberos `is a`_ `recommended`_ `practice`_.
Using the NCSA Kerberos service is problematic, however, due to the presence of non-LSST identities and the inherent complexity of Kerberos.
Setting up a separate LSST Kerberos system would keep all interfaces to identity.lsst.org and NCSA-managed services the same.
Every user would need to create new passwords; by design, Kerberos does not allow extraction of credentials to create the new LSST service.

.. _is a: https://help.ubuntu.com/lts/serverguide/kerberos-ldap.html
.. _recommended: https://stackoverflow.com/questions/46183178/why-use-kerberos-when-you-can-do-authentication-and-authorization-through-ldap
.. _practice: https://security.stackexchange.com/questions/109565/kerberos-vs-ldap-for-authentication-which-one-is-more-secure

The LSP handles single sign-on via OAuth tokens that are only indirectly and not necessarily based on Kerberos, and ssh connections typically use public/private key pairs and an ssh agent to allow transferable authentication.
This means that we are rarely using Kerberos's ability to do single sign-on itself (e.g. via the ``kinit`` command).

NCSA Kerberos for NCSA Services
-------------------------------

In this alternative, the NCSA identity systems would be used unchanged for the "lsst-dev" cluster.
The Kubernetes clusters and all Tucson, Summit, and Base systems would use the LSST LDAP for authentication.
Because the identity spaces would be separate and different, the filesystems associated with the "lsst-dev" cluster could not be shared with Kubernetes.
Additional filesystems would need to be created to hold user home directories and shared datasets accessible from the LSP instances.
identity.lsst.org would have to be reconfigured and perhaps modified to use the LSST LDAP as its identity source and metadata repository.
Note that LSP authentication would still proceed via CILogon to allow federated identities to be used; it is only the underlying LSST identity provider that would be switched from Kerberos to LDAP.

With LDAP being the ultimate source of authentication authority, the security of passwords within and traveling to the LDAP server needs to be ensured.
All LDAP connections must use TLS, and the passwords within LDAP must be securely hashed.

LSST LDAP for All Services
--------------------------

The NCSA services, including "lsst-dev", filesystems, and Kubernetes clusters, could be transitioned to using the (presumably Tucson-based) LSST LDAP for authentication.
ssh would be configured via PAM to use LDAP (``libpam_ldapd`` appears to be the current recommendation).
Unix user ids and group ids would have to be loaded into the LSST LDAP.
VPN authentication and other NCSA services like Jira and Nebula would remain on NCSA Kerberos, but only a relatively small number of people need to use these.
Again, identity.lsst.org would have to be reconfigured and perhaps modified to use the LSST LDAP as its identity source and metadata repository.

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
