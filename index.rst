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

identity.lsst.org is a website that allows users to self-manage the linkages of external federated identities with their LSST identity and to self-manage the user profile values and group memberships stored in LDAP.

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

Drawing the Line
================

The separation between LSST-based authentication and NCSA-based authentication will initially occur at the NCSA boundary.
All systems in Tucson and Chile will use Tucson-LDAP-based authentication.
All systems at NCSA, including the ``lsst-dev`` cluster, the staff Kerberos clusters, and the LSP instances hosted on them, will continue to use the current NCSA-based authentication system.
The Tucson LDAP and the NCSA LDAP will *not* be merged.

With LDAP being the ultimate source of authentication authority for Tucson and Chile, the security of passwords within and traveling to the LDAP server needs to be ensured.
All LDAP connections must use TLS, and the passwords within LDAP must be securely hashed.

A Tucson LDAP replica will be installed at the Summit for use in case of network disconnection.
In addition, two-factor authentication will be used at the Summit and Base for all control systems and their bastion hosts.
The server for the extra factor must not require an Internet connection; this could possibly be implemented by Duo with passcodes (not push notifications) or by a hardware `YubiKey`_ with ``yubico-pam``.

.. _Yubikey: https://yubico.com/

ssh for Summit, Base, and Tucson nodes will be configured via PAM to use LDAP (``libpam_ldapd`` appears to be the current recommendation).

The Commissioning Cluster at the Base and any other Web-based authenticated Observatory services will also be authenticated via the Tucson-based LDAP.
It is not likely that a full-featured self-serve solution like identity.lsst.org will be needed for the Commissioning Cluster; by default, all work there should be freely shared amongst the Commissioning Team, and having to have an LSST account is not a great burden for non-staff members of that team.

LSST staff developers working on the NCSA resources will continue to have separate LSST and NCSA accounts with distinct logins.
They will continue to use identity.lsst.org to control federated identities, profile information, and group management for the NCSA identities.

Potential Future Work
=====================

The Tucson-based LDAP could be made into a full CILogon-compliant identity provider using the `SAML protocol`_.
`Shibboleth`_ provides an implementation that can use LDAP as a backend, but suitable processes would need to be in place to meet `REFEDS`_ and `SIRTFI`_ standards.
(NCSA already meets these standards for its identities.)
In addition, the LDAP schema would need to provide certain CILogon-required elements.

.. _SAML protocol: https://auth0.com/blog/how-saml-authentication-works/
.. _Shibboleth: https://www.shibboleth.net/products/identity-provider/
.. _REFEDS: https://wiki.refeds.org/display/ASS/REFEDS+Assurance+Framework+ver+1.0
.. _SIRTFI: https://aarc-project.eu/policies/sirtfi/

Becoming an identity provider would enable LSST identities to be used to login to CILogon-fronted services at NCSA.
Such services can include the future US Data Access Center, in addition to the current staff LSP instances.
Otherwise, staff members would always have to use separate logins for LSST systems in Tucson and Chile versus the US DAC.

A further step would be to have the US DAC identities provided by LSST rather than NCSA.
Then the LSST identity provider would not be merely a federation possibility but would become the primary provider.
In this case, a separate instance of the identity.lsst.org management system using the LSST LDAP as its identity source and metadata repository or another system with equivalent functionality would be required to enable DAC users to manage their own profiles and group memberships.
Note that sharing between the DAC systems and LDF production systems would be difficult or impossible in this scenario, however.

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
