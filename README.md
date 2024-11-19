# LDAP Authentication Testing for Dell Enterprise SONIC


[![Contributions welcome](https://img.shields.io/badge/contributions-welcome-orange.svg)](#-how-to-contribute)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/Dell-Networking/PoC-SONiC-template/blob/main/LICENSE.md)
[![GitHub issues](https://img.shields.io/github/issues/Dell-Networking/PoC-SONiC-template)](https://github.com/Dell-Networking/PoC-SONiC-template/issues)

Built and maintained by [Ben Goldstone](https://github.com/benjamingoldstone/) and [Contributors](https://github.com/Dell-Networking/PoC-SONiC-template/graphs/contributors)

------------------

## 📑 Contents

- [Description and Objective](#description-and-objective)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [LDAP Structure Details](#ldap-structure-details)
- [Troubleshooting](#troubleshooting)
- [How to Contribute](#how-to-contribute)

## 🚀 Description and Objective

This repository provides ansible playbooks to simplify LDAP authentication testing with Dell Enterprise SONIC (DES). It helps set up Linux and Active Directory (AD)-like structures using OpenLDAP, making it easier to test and validate your LDAP authentication configuration.

⚠️ **IMPORTANT DISCLAIMER**
This setup is intended for Proof of Concept (PoC) and testing environments only. It is NOT suitable for production use due to security limitations. For production environments, please follow your organization's security best practices and implement proper security controls.

### Limitations and Constraints
- No TLS/SSL encryption implemented
- Uses a single password for all users and LDAP bind operations
- Simplified LDAP schema without advanced security features
- Basic setup without high availability or redundancy
- No audit logging or advanced security controls

### Key Features
- Supports both Linux-style and Active Directory-style LDAP configurations
- No POSIX add-ons or overlays required for AD setup
- Compatible with real Active Directory server configurations
- Simplified approach without complex LDAP schema requirements

## 📋 Requirements

- Ubuntu 22.04 server or virtual machine
- Ansible 2.10 or higher
- Dell Enterprise SONIC (DES) instance
- Python pip (automatically managed by playbook)

## 💾 Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   ```

2. Update the password in your chosen playbook:
   - For Linux: Edit [src/setup_linux_opennldap.yaml](src/setup_linux_opennldap.yaml)
   - For Active Directory: Edit [src/setup_linux_opennldap.yaml](src/setup_ad_like_openldap.yaml)

3. Edit the `src/inventory` file to specify your LDAP host.

## ⚙️ Configuration

### Predefined Users and Roles

#### Users:
| Username | Description |
|----------|-------------|
| admin | Equivalent to built-in admin |
| testadmin | Test admin user |
| testnetadmin | Test network admin |
| testsecadmin | Test security admin |
| testoperator | Test operator |

#### Roles (Groups):
- sonic-admins
- sonic-netadmins
- sonic-secadmins
- sonic-operators

All users and groups are created with IDs that comply with DES built-in limits (`/etc/adduser.conf`) so any future local or LDAP users/groups won't cause any issues.

### Domain Structure
- Domain: `example.com`
- Users OU: `ou=Users,dc=example,dc=com`
- Groups OU: `ou=Groups,dc=example,dc=com`

Feel free to change the predefined users, group, organizational units or domain as you see fit.

## 🔧 Usage

1. Run the appropriate playbook:
   ```bash
   # For Linux-style LDAP
   ansible-playbook -i src/inventory src/setup_linux_opennldap.yaml --ask-become-pass

   # For AD-style LDAP
   ansible-playbook -i src/inventory src/setup_ad_like_openldap.yaml --ask-become-pass
   ```

2. Configure your switch:
   - Use the sample configuration from:
     - Linux: `src/sonic-linux-ldap-config`
     - Active Directory: `src/sonic-ad-ldap-config`
   - Set the LDAP bind password (this needs to be done manually on every DES instance as hashe dpassword will differ):
     ```bash
     ldap-server bindpw <your-password> encrypted
     ```

3. Test the authentication:
   ```bash
   ssh testadmin@<switch-ip>
   ```

## 📚 LDAP Structure Details

### Active Directory Required Attributes

#### User Attributes:
- `uidNumber`: User ID number
- `gidNumber`: Group ID number (must match group's gidNumber)
- `homeDirectory`: Format `/home/<user>`

#### Group Attributes:
- `gidNumber`: Group ID number

### Group Membership Differences

#### Linux-style LDAP
In Linux-style LDAP, group membership is stored at the group level using the `memberUid` attribute:

```ldif
# Group Entry Example
dn: cn=sonic-admins,ou=Groups,dc=example,dc=com
objectClass: posixGroup
cn: sonic-admins
gidNumber: 60000
memberUid: testadmin
memberUid: admin

# User Entry Example
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

Key characteristics:
- Group membership is stored in the group entry
- Users reference their primary group using `gidNumber`
- Simple but requires searching group entries to determine user's groups
- No built-in support for nested groups

#### Active Directory-style LDAP
In AD-style LDAP, group membership is bidirectional using `member` and `memberOf` attributes:

```ldif
# Group Entry Example
dn: cn=sonic-admins,ou=Groups,dc=example,dc=com
objectClass: top
objectClass: group
cn: sonic-admins
sAMAccountName: sonic-admins
groupType: -2147483646
gidNumber: 60000
member: cn=admin,ou=People,dc=example,dc=com
member: cn=testadmin,ou=People,dc=example,dc=com

# User Entry Example
dn: cn=testadmin,ou=People,dc=example,dc=com
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: testadmin
sn: testadmin
sAMAccountName: testadmin
distinguishedName: cn=testadmin,ou=People,dc=example,dc=com
uidNumber: 60100
gidNumber: 60000
homeDirectory: /home/testadmin
memberOf: cn=sonic-admins,ou=Groups,dc=example,dc=com
```

Key characteristics:
- Group membership is stored in both user and group entries
- `memberOf` attribute in user entry lists all groups
- `member` attribute in group entry lists all members
- Supports efficient group membership queries
- Enables nested group memberships (not used in this PoC)
- Requires `memberOf` overlay in OpenLDAP
- Uses full DNs instead of just usernames

Please note that current DES LDAP AD authentication is not able to use ``memberOf`` and ``member`` approach, it is showed here for completeness purposes only. This is why attribute and ObjectClass mapping is important for AD environements.

### Attribute Mapping

| DES Attribute | AD Attribute |
|---------------|--------------|
| memberUid | sAMAccountName |
| uniqueMember | member |
| pam-login-attribute | sAMAccountName |
| pam-member-attribute | member |
| posixAccount | user |
| shadowAccount | user |
| posixGroup | group |

### AD DES Configuration Impact

#### PAM and NSS Attribute Configuration:
```bash
ldap-server attribute pam-login-attribute sAMAccountName
ldap-server attribute pam-member-attribute member
```
#### LDAP Attribute and ObjectClass Configuration:
```bash
ldap-server map attribute memberUid to sAMAccountName
ldap-server map attribute memberOf to memberOf
ldap-server map attribute uid to sAMAccountName
ldap-server map attribute uniqueMember to member
ldap-server map objectclass posixAccount to user
ldap-server map objectclass shadowAccount to user
ldap-server map objectclass posixGroup to group
```

## 🔍 Troubleshooting

### Verifying LDAP Server Setup

#### For Both Linux and AD:
```bash
# Test LDAP server accessibility
ldapsearch -H ldap://<server-ip> -x -b "dc=example,dc=com" -LLL

# Test bind credentials
ldapwhoami -H ldap://<server-ip> -x -D "cn=admin,dc=example,dc=com" -W
```

#### For Linux-style LDAP:
```bash
# Verify user attributes
ldapsearch -x -LLL -b dc=example,dc=com '(&(objectClass=posixAccount)(uid=testadmin))'

# Check group membership
ldapsearch -x -LLL -b dc=example,dc=com '(&(objectClass=posixGroup)(memberUid=testadmin))'

# Verify password functionality (will prompt for password)
ldapwhoami -x -D "uid=testadmin,ou=People,dc=example,dc=com" -W
```

#### For Active Directory-style:
```bash
# Verify user attributes
ldapsearch -x -LLL -b dc=example,dc=com '(&(objectClass=user)(sAMAccountName=testadmin))'

# Check group membership
ldapsearch -x -LLL -b dc=example,dc=com "(memberOf=cn=sonic-admins,ou=Groups,dc=example,dc=com)"

# Verify group attributes
ldapsearch -x -LLL -b dc=example,dc=com '(&(objectClass=group)(cn=sonic-admins))'

# Test AD-style bind (will prompt for password)
ldapwhoami -x -D "cn=testadmin,ou=People,dc=example,dc=com" -W
```

### Verifying Switch Configuration

```bash
# Show LDAP configuration
show ldap-server

# Show LDAP server status
show ldap-server status

# Test authentication (will prompt for password)
ssh testadmin@<switch-ip>

# View authentication logs
sudo tail -f /var/log/auth.log
```

### Common Issues and Solutions

1. **Connection Refused**
   - Verify LDAP server is running:
     ```bash
     sudo systemctl status slapd
     ```
   - Check firewall settings:
     ```bash
     sudo ufw status
     ```

2. **Authentication Failures**
   - Verify bind DN and password
   - Check user DN format matches configuration
   - Ensure user exists in correct OU
   - Verify group membership

3. **Group Membership Issues**
   - For Linux: Check `memberUid` attribute
   - For AD: Verify `memberOf` attribute
   - Ensure group GID matches user's primary GID


## 👏 How to Contribute

We welcome contributions to the project. Please reference the [CONTRIBUTING](https://github.com/Dell-Networking/PoC-Index/blob/main/CONTRIBUTING.md) guide in the PoC-Index repo for more details (this guide is common across Dell Networking PoC projects).



