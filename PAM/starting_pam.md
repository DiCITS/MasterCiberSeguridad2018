# Working with PAM.


Manuel Parra (manuelparra@decsai.ugr.es) & José Manuel Benítez (j.m.benitez@decsai.ugr.es), December 2016
![DICITSlogo](http://sci2s.ugr.es/dicits/images/dicits.png)



## Crear MVs desde Microsoft Azure

Para ello inicia la sesión con Azure y crea dos MVs sencillas con la imagen de CentOS e inicial el Shell de Azure.

Primero comprueba que existen una imagen para CentOS 7,5 disponible:

```
az vm image list --output table
```

Luego crea las dos VM con el comando:

```
az vm create --resource-group myResourceGroupVM --name myVM --image CentOS     --admin-username azureuser
```

Espera la creación de las dos MVs y guarda los datos de la IP externa de cada una de las MVs.

- IP MV 1: XXXXXXXXXXX
- IP MV 2: YYYYYYYYYYY

Tambíen puedes conocer la IP con:

```
az network public-ip list -g myResourceGroupVM
```

## Asignar nombre de servidor a cada VM

Para ello entra por SSH  en cada uno de las MVs que acabas de crear y escribe:

- Para el servidor 1 que será el servidor de LDAP: ```hostnamectl set-hostname "ldapserver"```
- Para el servidor 2 que sera el cliente de LDAP con PAM: ```hostnamectl set-hostname "ldapclient"```


## Instalar el servicio de LDAP en la VM Servidor

Para ello accedemos por ssh a la MV ```ldapserver``` y seguimos los siguientes pasos:


```
[root@dlp ~]# yum -y install openldap-servers openldap-clients
[root@dlp ~]# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG 
[root@dlp ~]# chown ldap. /var/lib/ldap/DB_CONFIG 
[root@dlp ~]# systemctl start slapd 
[root@dlp ~]# systemctl enable slapd 
```


### Set OpenLDAP admin password.

```
[root@dlp ~]# slappasswd 
New password:
Re-enter new password:
{SSHA}xxxxxxxxxxxxxxxxxxxxxxxx
```

### Add data to ldiff file:

```
[root@dlp ~]# vi chrootpw.ldif
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}xxxxxxxxxxxxxxxxxxxxxxxx
```

### Save data for admin user:

```
[root@dlp ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f chrootpw.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={0}config,cn=config"
```

### Import basic Schemas

```
[root@dlp ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=cosine,cn=schema,cn=config"
```

```
[root@dlp ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=nis,cn=schema,cn=config"
```

```
[root@dlp ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=inetorgperson,cn=schema,cn=config"
```

### Set your domain name on LDAP DB.

```
[root@dlp ~]# slappasswd 
New password:
Re-enter new password:
{SSHA}xxxxxxxxxxxxxxxxxxxxxxxx
```

```
[root@dlp ~]# vi chdomain.ldif
# replace to your own domain name for "dc=***,dc=***" section
# specify the password generated above for "olcRootPW" section
 dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
  read by dn.base="cn=Manager,dc=srv,dc=world" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=srv,dc=world

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=srv,dc=world

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}xxxxxxxxxxxxxxxxxxxxxxxx

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=Manager,dc=srv,dc=world" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=Manager,dc=srv,dc=world" write by * read
```

```
[root@dlp ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={1}monitor,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"
```

```
[root@dlp ~]# vi basedomain.ldif
# replace to your own domain name for "dc=***,dc=***" section
 dn: dc=srv,dc=world
objectClass: top
objectClass: dcObject
objectclass: organization
o: Server World
dc: Srv

dn: cn=Manager,dc=srv,dc=world
objectClass: organizationalRole
cn: Manager
description: Directory Manager

dn: ou=People,dc=srv,dc=world
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=srv,dc=world
objectClass: organizationalUnit
ou: Group
```


```
[root@dlp ~]# ldapadd -x -D cn=Manager,dc=srv,dc=world -W -f basedomain.ldif 
Enter LDAP Password: # directory manager's password
adding new entry "dc=srv,dc=world"

adding new entry "cn=Manager,dc=srv,dc=world"

adding new entry "ou=People,dc=srv,dc=world"

adding new entry "ou=Group,dc=srv,dc=world"
```

### If Firewalld is running, allow LDAP service. LDAP uses 389/TCP.

```
[root@dlp ~]# firewall-cmd --add-service=ldap --permanent 
success
```

```
[root@dlp ~]# firewall-cmd --reload 
success
```

### Open Port in Azure


Abre los puertos en el Shell de Azure para cada una de las MVs:

```
az vm open-port -g MyResourceGroup -n MyVm1 --port '389'
az vm open-port -g MyResourceGroup -n MyVm2 --port '389'
az vm open-port -g MyResourceGroup -n MyVm1 --port '636'
az vm open-port -g MyResourceGroup -n MyVm2 --port '636'
```

Y además debes ir al Panel de control de las MV desde el Dashboard de Azure para hacer lo siguiente:

- Dashboard > Virtual machines
- Click in your VMs (ldapserver and ldapclient) name.
- Select Networking.
- Add InBound RULE.
- Añade las reglas para los dos puertos que hay que abrir.






