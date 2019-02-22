# practica_sambaldap_bkend

## roberto@edt asix m06 curs 2018-2019

### descripcion:

> repositiorio que contiene 3 containers:

* ldap
* host
* samba 

#### objetivo:

implementacion de las tres tecnologias  en una sola red donde se puedan comunicar, de tal manera que se monten los homes provenientes del servidor samba.
la base de la informacion sera dada por el ldap con la configuracion de samba con backend ldap.

### Arquitectura

* sambanet: la red que implica a los 3 containers 

* servidor ldap: robert72004/ldapserver_18roberto:ldapsmb contiene los usuarios ldap y el schema de samba para el backend 

* servidor host: robert72004/hostpam:18homesamba el cliente que hace la conectividad con el servidor ldap y puede loguear a los usuarios unix locales y proveniente del ldap

* servidor samba: robert72004/samba:smbldap con los paquetes smbldap-toolspara poder realizar el backend



#### configuracion:

* 1 configurar el schema samba  del servidor ldap

* 2 activar el container host (el mismo que de la practica de samba:18homes)

* 3 configurar el servidor samba que ahora incorpora smbldap-tools y poder hacer el backend con ldap, ficheros principales smbldap.conf smbldap_bind.conf y el intall.sh donde haremos instalaciones necesarias para el funcionamiento correcto del montaje de los homes del los usuarios.

	+ unix users samba necesitan la existencia de usuarios unix por tanto hay que disponer de un usuario unix locales o via ldap, con lo cual hay que configurar nscd y nslcd para la conectividad con ldap
	+ homes, los usuarios locales ya lo tienen, los usuarios ldap no por tanto hay que creales el home, la propiedad y el grupo correcto.


### ejecucion:

#### creacion de la red 
```
docker network create --subnet 172.99.0.0/16 --gateway 172.99.0.1 sambanet
```
arrancamos los containers 

``` 
docker run --rm --name ldap -h ldap --network sambanet -d robert72004/ldapserver_18roberto:ldapsmb 

docker run --rm --name host -h host --privileged --network sambanet -it robert72004/hostpam:smbldap

docker run --rm --name samba -h samba --network sambanet -it robert72004/samba:smbldap 
```


 


