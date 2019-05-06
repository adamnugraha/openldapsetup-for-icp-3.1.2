# openldapsetup-for-icp-3.1.2

Prerequisites :
1. Make sure LDAP server are accessible

2. Make an host entry on LDAP Server machine in /etc/hosts for name resolution.

Install OpenLDAP Packages

Install the following LDAP RPM packages on LDAP server :

yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel

Start the LDAP service and enable it for the auto start of service on system boot

--> systemctl start slapd

--> systemctl enable slapd

Verify the LDAP

--> netstat -antup | grep -i 389

Output:

tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN      1520/slapd          
tcp6       0      0 :::389                  :::*                    LISTEN      1520/slapd

Setup LDAP admin password
Run below command to create an LDAP root password. We will use this for LDAP admin (root) passwor

Put in with your password.

--> slappasswd -h {SSHA} -s (yourpassword)

The above command will generate an encrypted hash of entered password which you need to use in LDAP configuration file. So make a note of this and keep it aside.

Output:

{SSHA}d/thexcQUuSfe3rx3gRaEhHpNJ52N8D3


Configure OpenLDAP server

OpenLDAP servers configuration files are found in /etc/openldap/slapd.d/. To start with the configuration of LDAP, we would need to update the variables “olcSuffix” and “olcRootDN“.

olcSuffix – Database Suffix, it is the domain name for which the LDAP server provides the information. In simple words, it should be changed to your domain
name.

olcRootDN – Root Distinguished Name (DN) entry for the user who has the unrestricted access to perform all administration activities on LDAP, like a root user.

olcRootPW – LDAP admin password for the above RootDN.

The above entries need to be updated in /etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif file. Manually edit of LDAP configuration is not recommended as you will lose changes whenever you run ldapmodify command.
Please create a .ldif file.


vi db.ldif

Add the below entries.

Replace the encrypted password ({SSHA}d/thexcQUuSfe3rx3gRaEhHpNJ52N8D3) with the password you generated in the previous step.

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=mycluster,dc=icp

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldapadm,dc=mycluster,dc=icp

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}d/thexcQUuSfe3rx3gRaEhHpNJ52N8D3

Once you are done with the ldif file, send the configuration to the LDAP server.

--> run this command : ldapmodify -Y EXTERNAL  -H ldapi:/// -f db.ldif

Make a changes to /etc/openldap/slapd.d/cn=config/olcDatabase={1}monitor.ldif (Do not edit manually) file to restrict the monitor access only to ldap root (ldapadm) user not to others.

vi monitor.ldif

Use the below information.

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external, cn=auth" read by dn.base="cn=ldapadm,dc=mycluster,dc=icp" read by * none
Once you have updated the file, send the configuration to the LDAP server.

--> run this command : ldapmodify -Y EXTERNAL  -H ldapi:/// -f monitor.ldif

Set up LDAP database

Copy the sample database configuration file to /var/lib/ldap and update the file permissions.

run this command : cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG

run this command : chown ldap:ldap /var/lib/ldap/*

Add the cosine and nis LDAP schemas.

--> run this : ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif

--> also this : ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 

--> and this : ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif

Next, Generate base.ldif file for your domain.

vi base.ldif

Use the below information. You can modify it according to your requirement.

dn: dc=mycluster,dc=icp
dc: mycluster
objectClass: top
objectClass: domain

dn: ou=people,dc=mycluster,dc=icp
objectClass: organizationalUnit
description: All people in organization
ou: people

dn: ou=groups,dc=mycluster,dc=icp
objectClass: organizationalUnit
objectClass: top
ou: groups

dn: uid=auser01,ou=people,dc=mycluster,dc=icp
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: top
cn: auser01
sn: auser01
uid: auser01
userPassword: auser01

dn: uid=auser02,ou=people,dc=mycluster,dc=icp
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: top
cn: auser02
sn: auser02
uid: auser02
userPassword: auser02

dn: uid=auser03,ou=people,dc=mycluster,dc=icp
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: top
cn: auser03
sn: auser03
uid: auser03
userPassword: auser03

dn: uid=auser04,ou=people,dc=mycluster,dc=icp
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: top
cn: auser04
sn: auser04
uid: auser04
userPassword: auser04

dn: uid=auser05,ou=people,dc=mycluster,dc=icp
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: top
cn: auser05
sn: auser05
uid: auser05
userPassword: auser05

dn: uid=auser06,ou=people,dc=mycluster,dc=icp
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: top
cn: auser06
sn: auser06
uid: auser06
userPassword: auser06

dn: cn=clusteradmin,ou=groups,dc=mycluster,dc=icp
objectClass: groupOfUniqueNames
objectClass: top
cn: clusteradmin
uniquemember: uid=auser01,ou=people,dc=mycluster,dc=icp
uniquemember: uid=auser02,ou=people,dc=mycluster,dc=icp
uniquemember: uid=auser03,ou=people,dc=mycluster,dc=icp
uniquemember: uid=auser04,ou=people,dc=mycluster,dc=icp
uniquemember: uid=auser05,ou=people,dc=mycluster,dc=icp
uniquemember: uid=auser06,ou=people,dc=mycluster,dc=icp
Build the directory structure.

--> run this command : ldapadd -x -W -D "cn=ldapadm,dc=itzgeek,dc=local" -f base.ldif

The ldapadd command will prompt you for the password of ldapadm (LDAP root user).
Output:

Enter LDAP Password: 

adding new entry "dc=itzgeek,dc=local"

adding new entry "cn=ldapadm ,dc=mycluster,dc=icp"

adding new entry "ou=People,dc=mycluster,dc=icp"

adding new entry "ou=Group,dc=mycluster,dc=icp"
