# OpenLDAP PoC with Dell Enterprise SONiC


[![Contributions welcome](https://img.shields.io/badge/contributions-welcome-orange.svg)](#-how-to-contribute)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/Dell-Networking/PoC-SONiC-template/blob/main/LICENSE.md)
[![GitHub issues](https://img.shields.io/github/issues/Dell-Networking/PoC-SONiC-template)](https://github.com/Dell-Networking/PoC-SONiC-template/issues)

Built and maintained by [Ben Goldstone](https://github.com/benjamingoldstone/) and [Contributors](https://github.com/Dell-Networking/PoC-SONiC-template/graphs/contributors)

------------------

## Contents

- [Description and Objective](#-description-and-objective)
- [Requirements](#-requirements)
- [How to Use](#-how-to-use)
- [How to Contribute](#-how-to-contribute)


## 🚀 Description and Objective

This repository's main aim is to simplify testing of LDAP authentication setup with Dell Enterprise SONIC (DES) by providing easy to use ansible playbooks to simplify the setup  of Linux and Active Directory (AD) like structures using OpenLDAP instance. 

Active Directory part doesn't rely on any  ``POSIX`` add-ons or overlays to be installed, and switch [example](src/sonic-ad-ldap-config) configuration will work against a real Active Directory server too.

In both cases, the easiest approach was taken: 

- no group names to be stored in LDAP
- no sudo information to be stored in LDAP
- no groups named after system groups (i.e. sudoers)

Following users and roles (groups) will be created:

### Users:

- admin
- testadmin
- testnetadmin
- testsecadmin
- testoperator

(user admin is identical to buil-in user ``admin``)

### Roles (groups)

- sonic-admins 
- sonic-netadmins
- sonic-secadmins
- sonic-operators

User and group IDs are created with DES built-in limits in mind, see ``/etc/adduser.conf`` for details. No LDAP users and roles/groups (current or future) will therefore overlap with built-in ones. Used role names closely match role names used in DES.

The oft used ``example.com`` test domain was used, together with ``organizational units (OU)`` ``Users`` for users and ``Groups`` for roles/groups. Same password is used for OpenLDAP for base ``distinguished name (DN)`` and all users.

The ansible playbook takes care of all ``pip`` dependencies needed for a smooth run.

### Sample switch configuration

Sample switch configuration is provided in here for [Linux](src/sonic-linux-ldap-config) and [Active Directory](src/sonic-ad-ldap-config) backends.


## 📋 Requirements

- Ubuntu 22.04 server or a virtual machine
- ansible 2.10+
- an instance of DES

## How to Use

- Clone the repository to your machine
- Set the password on line 7 of the [Linux](src/setup_linux_opennldap.yaml) or [Active Directory](src/setup_ad_like_openldap.yaml) playbook and save
- Create minimal ansible ``inventory`` [file](src/inventory) and define LDAP host where OpenLDAP will worh
- Run the playbook (with ``--ask-become-pass`` option or )
- Configure your switch using stock configuration for [Linux](src/sonic-linux-ldap-config) or [Active Directory](src/sonic-ad-ldap-config) (don't forget to set the LDAP ``bindpw`` password manually: ``ldap-server bindpw <redacted> encrypted``, it should match the password you set in your ansible playbook)
- Test the login process

## Note on Active Directory structures

### Required attributes

For DES to succesfuly authenticate against Active Directory LDAP backend, one needs to add these attributes (ansible playbook takes care of it) to AD structure:

#### User

- uiDNumber (User ID number)
- gidNumber (Group ID number) // this needs to match the gidNumber below
- homeDirectory (Home directory in /home/<user> format)

#### Group

- gidNumber (Group ID number)

If you use an actual Active Directory, then these attributes are not populated by default.

### Attribute mapping

Following attributes need to be mapped on DES side in order to LDAP authentication work:



### Group membership

The [Active Directory](src/setup_ad_like_openldap.yaml) playbook will setup all important AD structures including ``memberOf`` and ``refint`` (referential integrity) overlays, even though DES in its current LDAP implementation doesn't use them. Please note that group information (such as ``groupType``) has different meaning in Active Directory world. It is not used to infer group membership, ``gidNumber`` mentioned above is used to do that.

The main difference between ``memberOf`` and Linux approach is that Linux stores group membership separately, AD stores it together with user information:

``ldapsearch -x -LLL -b dc=example,dc=com "(memberOf=cn=sonic-admins,ou=Groups,dc=example,dc=com)"``:

```
dn: cn=admin,ou=People,dc=example,dc=com
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: admin
sn: admin
userPrincipalName: admin@example.com
sAMAccountName: admin
distinguishedName: cn=admin,ou=People,dc=example,dc=com
uidNumber: 1000
gidNumber: 60000
**bold**memberOf: cn=sonic-admins,ou=Groups,dc=example,dc=com**bold**

dn: cn=testadmin,ou=People,dc=example,dc=com
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: testadmin
sn: testadmin
userPrincipalName: testadmin@example.com
sAMAccountName: testadmin
distinguishedName: cn=testadmin,ou=People,dc=example,dc=com
uidNumber: 60100
gidNumber: 60000
homeDirectory: /home/testadmin
**bold**memberOf: cn=sonic-admins,ou=Groups,dc=example,dc=com**bold**
```

and

``ldapsearch -x -LLL -b dc=example,dc=com '(&(objectClass=group)(cn=sonic-admins))'``:

```
dn: cn=sonic-admins,ou=Groups,dc=example,dc=com
objectClass: top
objectClass: group
cn: sonic-admins
sAMAccountName: sonic-admins
groupType: -2147483646
gidNumber: 60000
**bold**member: cn=admin,ou=People,dc=example,dc=com**bold**
**bold**member: cn=testadmin,ou=People,dc=example,dc=com**bold**
```

versus

``ldapsearch -x -LLL -b dc=example,dc=com '(&(objectClass=posixGroup)(cn=sonic-admins))'``:

```
dn: cn=sonic-admins,ou=Groups,dc=example,dc=com
objectClass: posixGroup
cn: sonic-admins
gidNumber: 60000
**bold**memberUid: cn=admin,ou=People,dc=example,dc=com**bold**
**bold**memberUid: cn=testadmin,ou=People,dc=example,dc=com**bold**
```

and

``ldapsearch -x -LLL -b dc=example,dc=com '(&(objectClass=posixAccount)(uid=testadmin))'``:

```
dn: cn=testadmin,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: testadmin
cn: testadmin
sn: testadmin
uidNumber: 60100
gidNumber: 60000
homeDirectory: /home/testadmin
```



## 👏 How to Contribute

We welcome contributions to the project. Please reference the [CONTRIBUTING](https://github.com/Dell-Networking/PoC-Index/blob/main/CONTRIBUTING.md) guide in the PoC-Index repo for more details (this guide is common across Dell Networking PoC projects).



