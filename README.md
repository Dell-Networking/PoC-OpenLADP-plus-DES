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
- Run the playbook
- Configure your switch using stock configuration for [Linux](src/sonic-linux-ldap-config) or [Active Directory](src/sonic-ad-ldap-config)
- Test the login process

## 👏 How to Contribute

We welcome contributions to the project. Please reference the [CONTRIBUTING](https://github.com/Dell-Networking/PoC-Index/blob/main/CONTRIBUTING.md) guide in the PoC-Index repo for more details (this guide is common across Dell Networking PoC projects).



