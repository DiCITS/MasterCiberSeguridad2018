**Course 2017-2018**

[Master CyberSecurity -- University of Granada](http://ucys.ugr.es/master-propio-en-ciberseguridad/)

![LogoHeadMasterCES](https://sites.google.com/site/manuparra/home/logo_master_ciber.png)


[UGR](http://www.ugr.es) | [DICITS](http://dicits.ugr.es) | [SCI2S](http://sci2s.ugr.es) | [DECSAI](http://decsai.ugr.es)

Manuel J. Parra Royón (manuelparra@decsai.ugr.es) & José. M. Benítez Sánchez (j.m.benitez@decsai.ugr.es)


This is the repository for:

## Module: Network and Systems Protection
## Course: Security in Operating Systems

In this section you can consult the cluster structure to be used in the subject.

### Access to the servers

Two servers will be accessed:

- ```betatun.ugr.es```
- ```atcstack.ugr.es```

To access the clusters it is necessary to use SSH, if you use Windows, you will need the SSH Putty application or if you use Linux, the ssh command is already integrated.

The connection data to the clusters are:

- Server: ```betatun.ugr.es```   or    ```atcstack.ugr.es```
- Port: ```22```
- Username: ```<your assigned username>```
- Password: ```<your assigned password>```


To connect by ssh to the servers is used:

```ssh <your assigned username>@betatun.ugr.es```

or 

```ssh <your assigned username>@atcstack.ugr.es```

Use your assigned password when asked.


### Port diagram enabled for each user:

Each user has a range of 5 TCP ports. These ports will be used to make available services that are enabled from containers or virtual machines.

So ports that are used locally on ```betatun.ugr.es and``` or ```atcstack.ugr.es``` for services must always go in the range that has been set for each user.

This is the schema of the port range:

![Schema](../imgs/schema.png?raw=true)

Each user has 5 ports within the range ```153XXX```. The rank assigned to each user will be provided during the practice class, always being a contiguous range of ports.

To access the services, depending on the service we will use the corresponding application. For example, if it is a web service that has been installed/published in port 15367, to see the service, we can see it in the browser: ```http://bahia.ugr.es:15367/``` and this will redirect traffic to the service that is running in that port inside ```betatun.ugr.es```.

Table with port assignment (container external ports, container internal IP, VM external port, etc.):


| 	User      | betatun.ugr.es  | atcstack.ugr.es | betatun.ugr.es  | atcstack.ugr.es|
|-------------|-----------------|-----------------|-----------------|----------------|
| 	Name      |  external  Port |   external Port | internal WEB IP | internal WEB IP|
| CSXXXXXX88E |  15350 al 15354	| 	30200,30201   |10.2.0.18.50-54  | 192.168.10.141 |
| CSXXXXXX95W |  15355 al 15359	| 	30202,30203   |10.2.0.18.55-59  | 192.168.10.142 |
| CSXXXXXX03N |  15360 al 15364	| 	30204,30205   |10.2.0.18.60-64  | 192.168.10.143 |
| CSXXXXXX82R |  15365 al 15369	| 	30206,30207   |10.2.0.18.65-69  | 192.168.10.144 |
| CSXXXXXX57T |  15370 al 15374	| 	30208,30209   |10.2.0.18.70-74  | 192.168.10.145 |
| CSXXXXXX94X |  15375 al 15379	| 	30210,30211   |10.2.0.18.75-79  | 192.168.10.146 |
| CSXXXXXX88S |  15380 al 15384	| 	30212,30213   |10.2.0.18.80-84  | 192.168.10.147 |
| CSXXXXXX08Q |  15385 al 15389	| 	30214,30215   |10.2.0.18.85-89  | 192.168.10.148 |
| CSXXXXXX38V |  15390 al 15394	| 	30216,30217   |10.2.0.18.90-94  | 192.168.10.149 |
| CSXXXXXX21R |  15395 al 15399	| 	30218,30219   |10.2.0.18.95-99  | 192.168.10.150 |
| CSXXXXXX50G |  15400 al 15404	| 	30220,30221   |10.2.0.18.100-104| 192.168.10.151 |


Ports in ```atcstack.ugr.es```  are redirected to ```389``` and ```636``` (i.e: first ```30208``` -> ```389``` ; second ```30209``` -> ```636```).


### Next steps (first course)

Once understood the scheme of work and access to the servers, the next thing to do is work with Docker containers on ```bahia.ugr.es```

[Starting docker](../Docker/starting_docker.md)



