# Working with PAM.


Manuel Parra (manuelparra@decsai.ugr.es) & José Manuel Benítez (j.m.benitez@decsai.ugr.es), December 2016
![DICITSlogo](http://sci2s.ugr.es/dicits/images/dicits.png)

Tabla de contenido:

  * [Crear MVs desde Microsoft Azure](#crear-mvs-desde-microsoft-azure)
  * [Asignar nombre de servidor a cada VM](#asignar-nombre-de-servidor-a-cada-vm)
  * [Instalar el servicio de LDAP en la VM Servidor](#instalar-el-servicio-de-ldap-en-la-vm-servidor)
    + [Set OpenLDAP admin password.](#set-openldap-admin-password)
    + [Add data to ldiff file:](#add-data-to-ldiff-file-)
    + [Save data for admin user:](#save-data-for-admin-user-)
    + [Import basic Schemas](#import-basic-schemas)
    + [Set your domain name on LDAP DB.](#set-your-domain-name-on-ldap-db)
    + [If Firewalld is running, allow LDAP service. LDAP uses 389/TCP.](#if-firewalld-is-running--allow-ldap-service-ldap-uses-389-tcp)
    + [Open Port in Azure](#open-port-in-azure)
  * [Verificación de la instalación](#verificaci-n-de-la-instalaci-n)
  * [Breve información sobre PAM](#breve-informaci-n-sobre-pam)
    + [**Grupos de gestión**](#--grupos-de-gesti-n--)
    + [**Banderas de control**](#--banderas-de-control--)
    + [**Orden de los módulos**](#--orden-de-los-m-dulos--)
    + [Modulos](#modulos)
  * [Instalación y configuración de PAM + LDAP access en Ubuntu](#instalaci-n-y-configuraci-n-de-pam---ldap-access-en-ubuntu)
  * [Instalación y configuración de PAM + LDAP access en CentOS](#instalaci-n-y-configuraci-n-de-pam---ldap-access-en-centos)



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

## Verificación de la instalación

Para ello, debes instalar en la VM ```ldapserver``` y ```ldapclient``` el paquete de utilidades de LDAP:

```
yum -y install openldap-clients nss-pam-ldapd
```

Conecta con ```ldapserver``` por ssh y ejecuta el comando de busqueda en el directorio LDAP:

```
ldapsearch .....
```

Haz lo mismo en ```ldapclient```, pero usa como ```HOST``` la IP externa del servidor de LDAP ```ldapserver```.

```
ldapsearch -H <IP de ldapserver>
```

Para el caso de que esta segunda conexión al servicio de LDAP NO  funcione, probablemente no has abierto los puertos de forma correcta.

## Breve información sobre PAM

Los cuatro tipos de servicios PAM:

- Módulos de servicio de autenticación.
- Módulos de administración de cuentas.
- Módulos de gestión de sesiones.
- Módulos de gestión de contraseñas.

Cualquier aplicación que requiere autenticación puede registrarse en PAM utilizando un nombre de servicio. Puedes listar los servicios de Linux que usan Linux-PAM de la siguiente forma:

```
ls /etc/pam.d/
```

Si abres un archivo de servicio, verás que está dividido en tres columnas. La primera columna es el grupo de administración, la segunda columna es para las banderas de control y la tercera columna es el módulo (por lo tanto, el archivo) utilizado.

```
cat /etc/pam.d/sshd
...
account    required     pam_nologin.so
...

```

La cuenta es el grupo de administración, required es la bandera de control y el módulo utilizado es ```pam_nologin.so```. Se puede observar una cuarta columna que es para los parámetros del módulo.

### **Grupos de gestión** 

Hay cuatro grupos de administración que verás en los archivos de servicios de PAM:

- GrupoAuth: puede validar usuarios
- Grupo Account: controla el acceso al servicio, por ejemplo, cuántas veces deberías un servicio determinado.
- Grupo Session: responsable del entorno de servicio.
- Grupo Password: se utilizar para la actualización de contraseñas.

### **Banderas de control**

Hay cuatro banderas de control en los archivos de servicios:

- Requisite: la bandera de más precedencia.Si el requisito no se encuentra o no se puede cargar, dejará de cargar otros módulos y retornará un error.
- Required: lo mismo que el requisite, pero si el módulo no se pudo cargar por algún motivo, continúa cargando otros módulos y retorna el error al final de la ejecución.
- Sufficient: si el módulo retorna éxito, el procesamiento de otros módulos ya no es necesario.
- Optional: en caso de falla, la pila de módulos continúa la ejecución y el código de retorno se ignora

### **Orden de los módulos**

El orden es importante porque cada módulo depende del módulo anterior en la pila.

Si intentas una configuración como la siguiente para iniciar sesión:


```
auth required pam_unix.so
auth optional pam_deny.so
```
Funcionará correctamente, pero ¿qué pasará si cambiamos el orden de esta manera?

```
auth optional pam_deny.so
auth required pam_unix.so
```
Nadie puede iniciar sesión, por lo que el orden importa.

### Modulos

**Módulo pam_succeed_if**

Este módulo permite el acceso a grupos especificados. Puedes validar cuentas de usuario de esta manera:

```auth required pam_succeed_if.so gid=1000,2000```

La línea anterior indica que solo los usuarios en el grupo cuyo ID este entre 1000 o 2000 pueden iniciar sesión.

Puedes usar uid como id de usuario.

```auth requisite pam_succeed_if.so uid >= 1000```

En este ejemplo, cualquier id de usuario mayor o igual a 1000 puede iniciar sesión.

**Módulo pam_access**

Este módulo funciona como el módulo pam_succeed_if, excepto que el módulo pam_access verifica el registro de los hosts en red, mientras que al módulo pam_succeed_if no le importa.

**Módulo pam_deny**

Este módulo se usa para restringir el acceso. Siempre devolverá un no OK.

Puedes usarlo al final de tu pila de módulos para protegerte de cualquier configuración incorrecta.

**Módulo pam_unix**

Este módulo se utiliza para verificar las credenciales del usuario contra el archivo ```/etc/shadow```.

```auth required pam_unix.so```

**Módulo pam_mysql**

En lugar de verificar las credenciales del usuario contra ```/etc/shadow```, puedes usar una base de datos MySQL como back-end utilizando el módulo pam_mysql.

Se puede usar como se muestra a continuación:

```
auth sufficient pam_mysql.so user=myuser passwd=mypassword host=localhost db=mydb table=users usercolumn=username passwdcolumn=password
```

**pam_cracklib module**

Las contraseñas seguras son imprescindibles en estos días. Este módulo garantiza que usarás contraseñas seguras.

```password required pam_cracklib.so retry=4 minlen=12 difok=6```

Este ejemplo se asegura que la longitud de la contraseña = 12 y cuatro veces para elegir una contraseña, de lo contrario, saldrá.



## Instalación y configuración de PAM + LDAP access en Ubuntu


Install client packages on ```ldapclient```

```
sudo apt-get update
sudo apt-get install libpam-ldap nscd
```

And follow the next questions:

````
LDAP server Uniform Resource Identifier: ldap://LDAP-server-IP-Address

Change the initial string from "ldapi:///" to "ldap://" before inputing your server's information
Distinguished name of the search base:

   This should match the value you put in your LDAP service
   
   Our example was "dc=ugr,dc=es"

LDAP version to use: 3

Make local root Database admin: Yes

Does the LDAP database require login? No

LDAP account for root:

   This should also match the value in your LDAP SERVER
   Our example was "cn=Manager,dc=ugr,dc=es"

LDAP root account password: Your-LDAP-root-password
````

if you make a mistake:

```
sudo dpkg-reconfigure ldap-auth-config
```

Edit ``/etc/nsswitch.conf``

```
passwd:         ldap compat
group:          ldap compat
shadow:         ldap compat
```

Edit ``/etc/pam.d/common-session``

Add to the bottom:

```
session required    pam_mkhomedir.so skel=/etc/skel umask=0022
```

And then:

```
/etc/init.d/nscd restart
```

**Now we should verify the PAM configuration**:

Edit file ``/etc/pam.d/common-auth``

```
vi /etc/pam.d/common-auth
```

It must to contain:

```
...
auth    [success=2 default=ignore]      pam_unix.so nullok_secure try_first_pass
auth    [success=1 default=ignore]      pam_ldap.so use_first_pass
...
auth    requisite                       pam_deny.so
...
auth    required                        pam_permit.so
...
```

Edit file ``/etc/pam.d/common-account``

```
vi /etc/pam.d/common-account
```

```
...
account [success=2 new_authtok_reqd=done default=ignore]        pam_unix.so
account [success=1 default=ignore]      pam_ldap.so
...
account requisite                       pam_deny.so
...
account required                        pam_permit.so
...
```

Edit file  ``/etc/pam.d/common-password``,

```
vi /etc/pam.d/common-password
```

```
...
password        [success=2 default=ignore]      pam_unix.so obscure sha512
password        [success=1 user_unknown=ignore default=die]     pam_ldap.so use_authtok try_first_pass
...
password        requisite                       pam_deny.so
...
password        required                        pam_permit.so
...
```

Edit ``/etc/pam.d/common-session``

```
vi /etc/pam.d/common-session
```

```
...
session  required                                         pam_mkhomedir.so
...
```

It will create a HOME directory for LDAP users who does not have home directory when login to LDAP server.

Edit file ``/etc/pam.d/common-session-noninteractive``


```
vi /etc/pam.d/common-session-noninteractive,
```

```
...
session [default=1]                     pam_permit.so
...
session requisite                       pam_deny.so
...
session required                        pam_permit.so
...
session required        pam_unix.so
session optional                        pam_ldap.so
```


Restart nscd service:

```
/etc/init.d/nscd restart
```


Try login in with ssh:

```
ssh  myuser@IP
```


## Instalación y configuración de PAM + LDAP access en CentOS

Install the next packages:

```
yum install -y openldap-clients nss-pam-ldapd
```

or 

```
yum group install "Directory Client"
```

Follow the next steps: 

```
yum install authconfig
```

```
authconfig --enableldap --enableldapauth --ldapserver='ldap://LDAPServer' --ldapbasedn='dc=ugr,dc=es' --enablemkhomedir --enableshadow --enablelocauthorize --passalgo=sha256 --update
```

If Pluggable Authentication Module (PAM) for LDAP is missing. You can find out what package provides this command with:

```
yum install pam_ldap
```

and

``
authconfig --test
``


Pay attention to are the “LDAP server” and the “LDAP base DN” lines, making sure they match with your LDAP server.

Finally:


```
yum install nss-pam-ldapd
chkconfig nslcd on
service nslcd start
```

Name Service Switch (NSS) allows your LDAP server to provide user account, group, host name, alias, netgroup, etc. It also provides a Pluggable Authentication Module (PAM) to do authentication to an LDAP server based on the configuration of the nsswitch file.


```
ssh myuser@IP
```


