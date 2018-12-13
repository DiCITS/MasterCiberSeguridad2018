# Working with LDAP. Starting with LDAP command-line utilities.


Manuel Parra (manuelparra@decsai.ugr.es) & José Manuel Benítez (j.m.benitez@decsai.ugr.es), December 2016
![DICITSlogo](http://sci2s.ugr.es/dicits/images/dicits.png)


Table of Contents
=================

   * [Training with LDAP](#training-with-ldap)
      * [LDAP Basics](#ldap-basics)
      * [Objects and Classes](#objects-and-classes)
      * [Attributes](#attributes)
      * [Entries](#entries)
      * [DIT](#dit)
      * [Distinguished name (dn)](#distinguished-name-dn)
      * [Structure of the LDAP example](#structure-of-the-ldap-example)
      * [Starting: check if everything is okay](#starting-check-if-everything-is-okay)
      * [Adding an user](#adding-an-user)
   * [Changing password of LDAP user](#changing-password-of-ldap-user)
   * [Modifying LDAP user account data: DELETE, MODIFY.](#modifying-ldap-user-account-data-delete-modify)
      * [Add a OU to the LDAP:](#add-a-ou-to-the-ldap)
      * [Searching into DIT](#searching-into-dit)


**This course must be done on ```docker.ugr.es```**


<center><img src="http://www.openldap.org/images/headers/LDAPworm.gif"></center>


Default password for LDAP admin user is `password` 


## Training with LDAP

The openldap-clients package installs tools which are used to add, modify, and delete entries in an LDAP directory. These tools include the following:

* ldapadd — Adds entries to an LDAP directory by accepting input via a file or standard input; ldapadd is actually a hard link to ldapmodify -a.

* ldapdelete — Deletes entries from an LDAP directory by accepting user input at a shell prompt or via a file.

* ldapmodify — Modifies entries in an LDAP directory, accepting input via a file or standard input.

* ldappasswd — Sets the password for an LDAP user.

* ldapsearch — Searches for entries in an LDAP directory using a shell prompt.

* ldapcompare — Opens a connection to an LDAP server, binds, and performs a comparison using specified parameters.

* ldapwhoami — Opens a connection to an LDAP server, binds, and performs a whoami operation.

* ldapmodrdn — Opens a connection to an LDAP server, binds, and modifies the RDNs of entries.


### LDAP Basics

LDAP is a lightweight protocol for accessing directory servers. Okay, so what is a directory server? It’s a hierarchical object orientated database.

### Objects and Classes

Data in LDAP are stored in objects. These objects contain a number of attributes, which are basically a set of key/value pairs. Because data in LDAP is structured, objects can only contain valid keys, and which keys are valid is dependant on what class the object is. Classes in LDAP can define mandatory attributes and optional attributes and their type.


*Classes are assigned to objects using the objectClass attribute. LDAP defines some basic classes, types and comparison methods by default, but you are free to define your own.*

### Attributes

The data itself in an LDAP system is mainly stored in elements called attributes. Attributes are basically key-value pairs. Unlike in some other systems, the keys have predefined names which are dictated by the objectClasses selected for entry. Furthermore, the data in an attribute must match the type defined in the attribute's initial definition.

```
...
home: /home/mparra
...
```


### Entries

Attributes by themselves are not very useful. To have meaning, they must be associated with something. Within LDAP, you use attributes within an entry. 

An entry is basically a collection of attributes under a name used to describe something.

```
# LDAP user entry example
dn: cn=mparra,ou=Users,dc=openstack,dc=org
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: mparra
...
```


### DIT

A DIT is simply the hierarchy describing the relationship of existing entries. 

Upon creation, each new entry must "hook into" the existing DIT by placing itself as a child of an existing entry.

This creates a tree-like structure that is used to define relationships and assign meaning.

### Distinguished name (dn)

It functions like a full path back to the root of the Data Information Trees, or DITs.

For example:

```
dn: cn=mparra,ou=Users,dc=openstack,dc=org
```

cn: Common name

ou: organizational segment / Organizational Unit

dc = Domain Component

The direct parent is an entry called ``ou=Users`` which is being used as a container for entries describing users.

The parents of this entry derived from the ``openstack.org``  domain name, which functions as the root of our DIT.

These are often used for the general categories under the top-level DIT entry, things like ```ou=people, ou=groups, and ou=inventory``` are common.


### Structure of the LDAP example

![LDAPstruct](https://sites.google.com/site/manuparra/home/Untitled.png)


### Starting: check if everything is okay

This is the schema of port and services in BETATUN.UGR.ES:



![LDAPstruct](../imgs/schema_ldap_betatun)


First, with your user account (logged) try out:

```
ldapsearch -H ldap://localhost -LL -b ou=Users,dc=openstack,dc=org -x
```

Is it not working?. Those are the parameters:

- We are using BETATUN.UGR.ES as server of LDAP
- If you are connected BETATUN.UGR.ES, use localhost. If you are connected outside of BETATUN.UGR.ES, use betatun.ugr.es
- Port by default: 389 -> LDAP  and 986 -> SLDAP
- BETATUN.UGR.ES contains LDAP service running locally and externally at port number 15040 (LDAP, no SLDAP).

So, how to connect from external node:

```
ldapsearch -H ldap://betatun.ugr.es:15040 -LL -b ou=Users,dc=openstack,dc=org -x
```

How to connect from internal BETATUN.UGR.es:

```
ldapsearch -H ldap://localhost:15040 -LL -b ou=Users,dc=openstack,dc=org -x
```



This command returns the list of LDAP users (something similar):


```
...
dn: ou=Users,dc=openstack,dc=org
objectClass: organizationalUnit
ou: Users

dn: cn=Robert Smith,ou=Users,dc=openstack,dc=org
objectClass: inetOrgPerson
cn: Robert Smith
cn: Robert J Smith
cn: bob  smith
sn: smith
uid: rjsmith
carLicense: HISCAR 123
homePhone: 555-111-2222
mail: r.smith@example.com
mail: rsmith@example.com
mail: bob.smith@example.com
description: swell guy
ou: Human Resources
...
```

The format for a simple query with ``ldapsearch`` is: 

``ldapsearch -H <host> -LL -b ou=<Organizational Unit> -x``

LDAP utility ``ldaputility`` cointains multiple options and configuration, please review: https://www.centos.org/docs/5/html/CDS/ag/8.0/Finding_Directory_Entries-Using_ldapsearch.html


### Adding an user

To add something to the LDAP directory, you need to first create a LDIF file.

The ldif file should contain definitions for all attributes that are required for the entries that you want to create.

With this ``ldif`` file, you can use ``ldapadd`` command to import the entries into the directory as explained in this tutorial.

Create a file, i.e. ``user.ldif`` and copy this skeleton, modify and include your data (i.e. ``cn=mparra`` to ``cn=<user>``, i.e. ``uid=mparra`` to ``uid=<uid>`` , ``gecos`` etc.)

```
dn: cn=mparra,ou=Users,dc=openstack,dc=org
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: mparra
uid: mparra
uidNumber: 16859
gidNumber: 100
homeDirectory: /home/mparra
loginShell: /bin/bash
gecos: mparra
userPassword: {crypt}x
shadowLastChange: 0
shadowMax: 0
shadowWarning: 0
```

To add this user to LDAP directory (at openstack/org == openstack.org):

```
ldapadd -x -D "cn=admin,dc=openstack,dc=org" -w password -c -f user1.ldif
```

Syntax:

```
ldapadd [options] [-f LDIF-filename]
```

If you remove ``-w password`` option, and change by ``-W`` it will ask you about LDAP admin password.


## Changing password of LDAP user

```
ldappasswd -s <new_user_password> -W -D "cn=admin,dc=openstack,dc=org" -x "cn=mparra,ou=Users,dc=openstack,dc=org"
```

In this case, using -W option, ``ldappasswd`` ask for LDAP admin password.

Syntax:

```
ldappasswd [ options ] [ user ]
```

Please check: https://www.centos.org/docs/5/html/CDS/cli/8.0/Configuration_Command_File_Reference-Command_Line_Utilities-ldappasswd.html for more information about this command.

Now, try out:

```
ldapsearch -H ldap://localhost:15040 -LL -b ou=Users,dc=openstack,dc=org -x
```

It returns LDAP directory with the last user added.

## Modifying LDAP user account data: DELETE, MODIFY.

The syntax of the ldapmodify tool on the command-line can take any of these forms:

```
ldapmodify [ options ]

ldapmodify [ options ] < LDIFfile

ldapmodify [ options ] -f LDIFfile
```

LDIF text file containing new entries or updates to existing entries on LDAP directory.

When modifying the contents of a directory, you must satisfy several prerequisite conditions. 

First, the bind DN and password used for authentication must have the appropriate permissions for the operations being performed.

Create an example LDIF Modify and save the file as i.e. ``mparra_modify.ldif``

```
dn: cn=mparra,ou=Users,dc=openstack,dc=org
changetype: modify
replace: loginShell
loginShell: /bin/csh
```

Then execute:

```
ldapmodify -x -D "cn=admin,dc=openstack,dc=org" -w password -H ldap:// -f mparra_modify.ldif
```

It will update ``cn=mparra`` with a new ``loginShell``, in this case ``/bin/csh``

Check if the change has been done:

```
...
dn: cn=mparra,ou=Users,dc=openstack,dc=org
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: mparra
uid: mparra
uidNumber: 16859
gidNumber: 100
homeDirectory: /home/mparra
gecos: mparra
shadowMax: 0
shadowWarning: 0
loginShell: /bin/csh
...

```

Adding a entry of LDAP user. Create a new file ``manu_add_descrip.ldif`` and add: 

```
dn: cn=mparra,ou=Users,dc=openstack,dc=org
changetype: modify
add: description
description: Manuel Parra
```

Execute next command:

```
ldapmodify -x -D "cn=admin,dc=openstack,dc=org" -w password -H ldap:// -f manu_add_descrip.ldif
```

Now, check changes:


```
ldapsearch -H ldap://localhost:15040 -LL -b ou=Users,dc=openstack,dc=org -x
```

And finally delete description. Create a new file i.e. ``manu_del_descr.ldif``

```
dn: cn=mparra,ou=Users,dc=openstack,dc=org
changetype: modify
delete: description
``` 

Then execute: 

```
ldapmodify -x -D "cn=admin,dc=openstack,dc=org" -w password -H ldap:// -f manu_del_descr.ldif
```

Now, check out:

```
ldapsearch -H ldap://localhost:15040 -LL -b ou=Users,dc=openstack,dc=org -x
```

Verify if entity description is not set.


### Add a OU to the LDAP:

Create a new ``ldif`` file. i.e. ``add_new_ou.ldif`` with:

```
dn: ou=People,dc=openstack,dc=org
ou: People
objectClass: top
objectClass: organizationalUnit
description: Parent object of all PEOPLE accounts
```

Then use:

```
ldapadd -x -D cn=admin,dc=openstack,dc=org -w password -c -f add_new_ou.ldif
```

### Searching into DIT

For instance, if we use ``ou=People``

```
ldapsearch -H ldap://localhost:15040 -LL -b ou=People,dc=openstack,dc=org -x
```

It shows everything under ``ou=People`` from ``dc=openstack,dc=org``

Any combination of ``ou``, ``dc``, ... is allowed to search into DIT.


