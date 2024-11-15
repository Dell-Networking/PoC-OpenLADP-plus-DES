# Template repo for SONiC PoC Repos


[![Contributions welcome](https://img.shields.io/badge/contributions-welcome-orange.svg)](#-how-to-contribute)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/Dell-Networking/PoC-SONiC-template/blob/main/LICENSE.md)
[![GitHub issues](https://img.shields.io/github/issues/Dell-Networking/PoC-SONiC-template)](https://github.com/Dell-Networking/PoC-SONiC-template/issues)

Built and maintained by [Ben Goldstone](https://github.com/benjamingoldstone/) and [Contributors](https://github.com/Dell-Networking/PoC-SONiC-template/graphs/contributors)

------------------

## Contents

- [Description and Objective](#-description-and-objective)
- [Requirements](#-requirements)
- [How to Contribute](#-how-to-contribute)


## 🚀 Description and Objective

This repository's main aim is to simplify testing of LDAP authentication in Dell Enterprise SONIC (DES) by providing ansible playbooks to setup  Linux and Active Directory (AD) like structures using OpenLDAP (mostly for licensing and ease of setup reasons). 

In both cases, the easiest approach was taken: 

- no groups to be stored in LDAP
- no sudo information to be stored in LDAP
- no group named after system groups (i.e. sudoers)

Therefore following users and groups/roles will be created:

### Users:

- admin
- testadmin
- testnetadmin
- testsecadmin
- testoperator

(user admin is identical to buil-in user ``admin``)

### Groups/roles

- sonic-admins 
- sonic-netadmins
- sonic-secadmins
- sonic-operators

User and group IDs are created with DES built-in limits in mind (see ``/etc/adduser.conf`` for details) - so LDAP users and groups/roles (current or future) don't overlap with built-in ones.


## 📋 Requirements

- Ubuntu 22.04 server or a virtual machine
- ansible 2.10+
- an instance of DES


## 👏 How to Contribute

We welcome contributions to the project. Please reference the [CONTRIBUTING](https://github.com/Dell-Networking/PoC-Index/blob/main/CONTRIBUTING.md) guide in the PoC-Index repo for more details (this guide is common across Dell Networking PoC projects).



