---
layout: single
title: ¿Cómo administrar dispositivos?
excerpt: ¿Cuántas veces hemos buscado alguna forma de recordar las IP's de cada uno de los dispositivos de la red?
date: 2024-07-15
classes: wide
header:
    teaser: /assets/images/post_thumbnails/01_CLI.png
---
# ¿Tenemos alternativas a la hora de recordar la **IP** de cada dispositivo?
Este post trata justamente de ese tema. Muchas veces nos hemos enfrentado a que nuestro empleador, no nos permite instalar software que no tenga una licencia y/o que no cumpla con ciertas caracteristicas. entoces esto nos deja fuera de la mano software como, ***SecureCRT, Putty, MobaXterm, Termux o Xshell7**, seguramente deje afuera muchas alternativas.
Para mi este fue un caso, dentro de las normar corporativas, no podemos hacer uso de software que no tuviera una licencia y/o que no cumpliera con esos requerimientos.
Elgo complicado cuando estas acostumbrado a usar ciertas aplicaciones. Pero al final encontre una forma de hacerlo comodo, simple y sin gastar.
En mi caso, siendo usuario de Linux, fue utilizar las herramientas existentes, basicamente SSH (Openssh), telnet, sshpass, y un script que cree para aprovechar el archivo **/etc/hosts**

Vallamos al inicio de este entorno de trabajo.
## Preparativos para el entorno de trabajo.
No entrare en detalle de como instalar paquetes, eso dependera de cada usario y de cada distribución.
Pero seria facilmente aplicable para entornos basados en Linux y basados en Unix.

### Archivo ssh_config
El primer paso seria estructurar el archivo ssh_config. Aunque mucha gente no lo hace, creo que es una herramienta poderosa si se sabe usar.

Dentro del archivo ssh_config, modificaremos al final lo siguiente:
```
#Todos los hosts Cifrados
Host *
    Ciphers 3des-cbc,aes128-cbc,aes192-cbc,aes256-cbc,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com,chacha20-poly1305@openssh.com
#Todos los hosts MACs
    MACs hmac-sha1,hmac-sha1-96,hmac-sha2-256,hmac-sha2-512,hmac-md5,hmac-md5-96,umac-64@openssh.com,umac-128@openssh.com,hmac-sha1-etm@openssh.com,hmac-sha1-96-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-md5-etm@openssh.com,hmac-md5-96-etm@openssh.com,umac-64-etm@openssh.com,umac-128-etm@openssh.com
#Todos los hosts HostKeyAlgorithms
HostKeyAlgorithms ssh-ed25519,ssh-rsa,ssh-ed25519,ssh-ed25519-cert-v01@openssh.com,sk-ssh-ed25519@openssh.com,sk-ssh-ed25519-cert-v01@openssh.com,ecdsa-sha2-nistp256,ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp384,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp521,ecdsa-sha2-nistp521-cert-v01@openssh.com,sk-ecdsa-sha2-nistp256@openssh.com,sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,ssh-rsa-cert-v01@openssh.com
#Todos los hosts KexAlgorithms
KexAlgorithms diffie-hellman-group1-sha1,diffie-hellman-group14-sha1,diffie-hellman-group14-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group-exchange-sha1,diffie-hellman-group-exchange-sha256,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,curve25519-sha256,curve25519-sha256@libssh.org,sntrup761x25519-sha512@openssh.com
```
Estos cambios, nos ayudaran a no tener provemas con los diferentes vendors, por ejemplo cisco con sus algoritmos **diffie-hellman**, basicamente estamos preparando a ssh para poder trabajar con lo que se le pida.

La siguiente parte es a consideración de cada uno.
dentro de las caracteristicas del archivo ssh_config, nos permite jugar con perfires, por ejemplo, si tus dispositivos existen en un loque de ips (publicas o privadas), puedes crear un perfir de esa manera, pongamos un ejemplo.
```
Host 192.168.30.*
    User usuario_local_1
```
>Esta configuración, nos dice que todas las IP's del segmento 192.168.30.0/24, tendran el **usuario_local_1** ya definido, por lo que a la hora de ejecutar el comando solo habria que incluir la IP. "ssh 192.168.30.20"

En Caso de que los dispositivos que busques administrar, tienen nomenclaturas definisas, como por ejemplo P,PE,BR,CE,RR,N,H,SW,etc. puedes definirlo en este momento.
```
Host SW_*
    User usuario_1 
```
## Archivo /etc/hosts 
¿Alguna vez has pensado en que grandioso seria que los dispositivos tuvieran un DNS o similar?
Este archivo, nos permite hacer eso, crear una relación entre un dominio y una IP.

En base a ese consepto vamos a ingresar en este archivo esa relación.

```
192.168.30.20 SW_CISCO_1
```

Cuando son pocos dispositivos, no tenemos problemas de ir ingresando y modificando uno a uno, pero cuando ya se manejan redes grandes esto se complica, es facil cometer errores. Por eso cree un script, `mgmt-etc-host`.

Les dejo el link [mgmt-ect-hosts](https://github.com/syncophat/mgmt-etc-hosts).

 ### **Instalar la el script**
 La aplicación esta pensada para ejecutar como root, pero si tu necesidad es poder 
 ejecutarlo como usuario, necesitaras realizar los siguientes cambios.
 ```
 chmod o+w /etc/hosts
 ```
Para poder ejecutarlo desde cualquier parte, tendrás que agregarlo al **PATH**, podrías agregarlo en el 
**.local/bin/**, o en la ruta de tu preferencia. 
Para consultar el PATH basta con ejecutar ```echo $PATH"```, posteriormente puedes ejecutar la siguiente secuencia de comandados, para dejarlo funcional
```
chmod u+x mgmt-etc-hosts.sh

sudo mv mgmt-etc-hosts.sh ~/.local/bin/mgmt-etc-hosts.sh  
mgmt-etc-hosts.sh -h
```
### Forma de uso
|Script 		   |Argumento 									| Descripción			|
|------------------|--------------------------------------------|-----------------------|
|mgmt-etc-hosts.sh |-h 		  									|Panel de ayuda			| 
|mgmt-etc-hosts.sh |-e busqueda_IP -i 192.168.100.2 			|Búsqueda por IP 		| 
|mgmt-etc-hosts.sh |-e busqueda_IP -j SW_TEST_2 				|Búsqueda por HOSTS		|
|mgmt-etc-hosts.sh |-a alta -b 192.168.100.2 -c SW_TEST_2		|Alta de un Hosts 		|
|mgmt-etc-hosts.sh |-a baja -d 192.168.100.2 -f SW_TEST_2		|Baja de un Hosts 		|
|mgmt-etc-hosts.sh |-a cambio -b IP -c hostname -d IPn -f hostN	|Cambio de nombre o IP 	|

### Screenshots

![](/assets/images/post_thumbnails/01_Panel_Ayuda.png )

**Panel de Ayuda**

![](/assets/images/post_thumbnails/01_busqueda_IP.png )

**Busqueda por IP**

![](/assets/images/post_thumbnails/01_busqueda_host.png)

**Busqueda Por Host**

![](/assets/images/post_thumbnails/01_alta.png)

**Alta de un Host**

![](/assets/images/post_thumbnails/01_baja.png)

**Baja de un Hosts**

![](/assets/images/post_thumbnails/01_remplazo.png)

**Cambio de IP o Hosts a una entrada existente**

##### Espero que te sea de utilidad este script, si es el caso apoyame con haciendo "★ Star" en este repositorio.¡Gracias!





