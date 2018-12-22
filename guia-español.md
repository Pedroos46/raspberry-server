Guia de creación de un servidor Raspberry Pi Zero en español.

## Tabla de contenidos: 
1. [Preparación](https://github.com/Pedroos46/raspberry-server/blob/master/guia-espa%C3%B1ol.md#1-preparaci%C3%B3n)
2. [Recomendaciones](https://github.com/Pedroos46/raspberry-server/blob/master/guia-espa%C3%B1ol.md#2-recomendaciones)
3. [Configuración local](https://github.com/Pedroos46/raspberry-server/blob/master/guia-espa%C3%B1ol.md#3-empezando-a-nivel-local)
4. [Abriendo el servidor](https://github.com/Pedroos46/raspberry-server/blob/master/guia-espa%C3%B1ol.md#4-abriendo-el-servidor)
5. [VPN, FTP, etc](https://github.com/Pedroos46/raspberry-server/blob/master/guia-espa%C3%B1ol.md#5-vpn-ftp-etc)


## 1. Preparación: 
Antes de empezar con la creación del servidor tenemos que instalar el sistema operativo. Aunque [existen varias alternativas](https://www.raspberrypi.org/downloads/) yo usaré la distro oficial Raspbian.

### ➡️ Instalación del sistema operativo:
Podemos descargar Raspbian directamente des de [aqui](https://www.raspberrypi.org/downloads/raspbian/).
La instalación no tiene secreto, descargamos la imagen que queramos y usamos los [pasos proporcionados](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) en la pagina oficial. 

> -   Download  [Etcher](https://etcher.io/)  and install it.
> -   Connect an SD card reader with the SD card inside.
> -  Open Etcher and select from your hard drive the Raspberry Pi  `.img`  or  `.zip`  file you wish to write to the SD card.
> -  Select the SD card you wish to write your image to.
> -   Review your selections and click 'Flash!' to begin writing data to the SD card.

### ➡️ Configuración inicial: 

Una vez instalado el sistema operativo empezaremos con las configuraciones.
En este punto nos interesa principalmente tener la conexión wifi configurada y el SSH habilitado.

La idea de esta configuración inicial es minimizar trabajo, si empezamos con estas dos configuraciones evitamos posteriormente tener conectar a nuestra placa un ratón, un teclado y un monitor. 

Para **configurar la conexión WiFi** nos dirigimos en el volumen "boot" de la tarjeta SD donde hayamos instalado el sistema. Una vez en la raiz del directorio creamos un archivo llamado **wpa_supplicant.conf**. 

![Creamos el archivo wpa_supplicant.conf](https://i.imgur.com/KhmhpAz.png)

Este archivo debe contener los siguientes detalles: 
```bash
network={
    ssid="NETWORK_NAME"
    psk="NETWORK_PASSWORD"
    key_mgmt=WPA-PSK
}
```
Colocando este archivo aquí Raspbian lo moverá  a **/etc/wpa_supplicant/** cuando el sistema arranque.

Para **habilitar el SSH** crearemos en el mismo directorio "boot" un archivo llamado **ssh** sin ninguna extensión. 
![enter image description here](https://i.imgur.com/UwIQnPL.png)

Este archivo será eliminado al arrancar, pero habilita por defecto el servicio SSH. Una vez tengamos esto hecho podemos arrancar el sistema. 

### ➡️ Empezando con SSH

Una vez iniciado el sistema lo primero que tenemos que saber es la IP que se le ha asignado para poder establecer conexión. 

Podemos usar un escaner de red para obtenerla. Si no tenemos ninguna herramienta una simple [busqueda en Google](http://bfy.tw/LSCo) o en la tienda de apps. Yo usare una aplicación llamada `Fing`. 

Una vez tengamos la IP empezamos con SSH, para ello necesitamos un servidor SSH en nuestra Raspberry o placa (que ya hemos activado en pasos anteriores) y un cliente SSH en nuestro ordenador. 

Para el cliente SSH debemos tener instalado uno en nuestro ordenador. Si usamos **Linux** o **MacOS** probablemente no hará falta instalar uno, normalmente ya lo incluye.

Si usamos **Windows** podemos usar clientes como [Putty](http://www.putty.org/) o podemos usar el CMD o el PowerShell de Windows junto [OpenSSH](http://www.mls-software.com/opensshd.html). Por otro lado Windows 10 Pro incluye un cliente SSH ya instalado. 

El funcionamiento es exactamente el mismo tanta para Linux y Mac como para Windows. 

Así que volvemos a intentar el commando `ssh`:

	usage: ssh [-1246AaCfGgKkMNnqsTtVvXxYy] [-b bind_address] [-c cipher_spec]
           [-D [bind_address:]port] [-E log_file] [-e escape_char]
           [-F configfile] [-I pkcs11] [-i identity_file]
           [-J [user@]host[:port]] [-L address] [-l login_name] [-m mac_spec]
           [-O ctl_cmd] [-o option] [-p port] [-Q query_option] [-R address]
           [-S ctl_path] [-W host:port] [-w local_tun[:remote_tun]]
           [user@]hostname [command]

Demomento solo realizaremos una simple conexión con el servidor. 

Para hacerlo necesitamos **el usuario** de nuestro servidor, en el caso de la Raspbery el usuario por defecto es `pi`, **la IP**  de nuestra placa y **la contraseña** del usuario, que en el caso de las Raspberry la contraseña por defecto para el usuario `pi` es `raspberry`.

	ssh [usuario]@[ip o hostname]
Por lo tanto:s

    ssh pi@[ip]
    password: raspberry
    
Una vez realizada la conexión y el login podremos actuar como si estubieramos en el propio dispositivo.


## 2. Recomendaciones: 

### ➡️ Asignar una IP local estatica:
Muy recomendado  ✅

Una vez tengamos nuestro dispositivo arrancado y conectado a la red local es muy recomendable que poner nuestra IP local estatica. El motivo por el cual hacemos esto es porque mas tarde, desde el router, vamos a apuntar la IP de nuestro futuro servidor por lo que necesitamos que esa IP no cambie nunca. 

Para lograrlo nos dirigiremos a la seguiente ruta en nuestro sistema: `/etc/network/`

Dentro de la carpeta nos encontraremos con un archivo llamado `interfaces`. Lo primero que haremos con este archivo es hacer una copia por si cometemos algun error. Usaremos el comando `cp`:

	sudo cp /etc/network/interfaces interfaces.copy
    
Una vez hecha la copia, usaremos `nano` para editar el archivo. 

	sudo nano -w /etc/network/interfaces

Deberemos añadir las lineas que hay más abajo. Es probable que `iface xxxx inet nanual/static` ya exista, en este caso solo añadiremos las lineas que nos interesan. Al final debería tener un aspecto como este.

    iface wlan0 inet static
      address 192.168.1.180
      gateway 192.168.1.1
      
En donde pone `address` podremos la IP estatica que queremos. No estaría de más entrar en nuestro router para saber que rango de IPs podemos entrarle y cual es el `gateway` ya que esto suele estar sujeto al router que tengamos. 

Nota:  Normalmene `wlan0` es la interfaz  que se usa con conexión wifi. Si nuestra conexión es por ethernet normalmente la interficie es `eth0`. Siempre podemos mirar que interficie estamos usando con el comando `ifconfig`.

Para más información y para otra versiones de Linux se puede vistar la siguiente [pàgina](https://www.linode.com/docs/networking/linux-static-ip-configuration).

Una vez lo tengamos, reiniciamos nuestra placa y ya deberia empezar a trabajar con la nueva IP estatica. 

	sudo reboot now


### ➡️ Mejorar el rendimiento:

La idea es tener trabajando la Raspberry sin necesidad de supervisión y sin ningún periférico conectado. Por lo tanto, podriamos reducir la cantidad de recursos que destinamos al procesamiento gráfico e incluso desactivar la interfaz. 

Para empezar lanzaremos el asistente de configuración de Raspbian.

	sudo raspi-config
    
En caso de no tenerlo lo instalamos:  

	sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 7FA3303E
	sudo apt-get update
	sudo apt-get install raspi-config
    
    
Una vez lanzado, lo primero que haremos será desactivar la interfaz de escritorio, por ende tampoco necesitamos tanta capacidad GPU así que le reduciremos la memoria.
 
El asistente es bastante visual aun así para desactivar la interfaz será algo así:
 		
   	Boot options > Desktop / CLI > Console. 
        
Posteriormente le reduciremos la memoria a la GPU asignadole `16`. 
 
 	Advanced options > Memory Split. 
	
 
### ➡️ Crear un nuevo usuario y eliminar el usuario por defecto:

Recomendado ✅

Por motivos de seguridad no podemos dejar nuestro usario por defecto habilitado. Por lo que crearemos uno nuevo y lo eliminaremos:

Para crearlos usaremos el siguiente comando, el cual contiene también los grupos de sistema al cual queremos que este. 

	sudo useradd -m -G adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,netdev,input NUEVO-USUARIO
 
Posteriormente le daremos una nueva contraseña.
 
	sudo passwd NUEVO-USUARIO
    
    
Una vez, creado el nuevo usuario cerramos la conexión SSH actual escribiendo `exit` y nos volveremos a conectar con el nuevo usuario. 

	ssh NUEVO-USUARIO@[ip o hostname]
    
Una vez dentro eliminaremos el usario por defecto y todos sus datos. En el caso de la Raspberry es `pi` por lo que si tenemos otro nombre de usuario por defecto hay poner dicho usuario. 
 
 	sudo deluser --remove-all-files pi	
	sudo userdel -r -f pi


### ➡️ Actualizando el sistema operativo:

Otro punto importante de seguridad es mantener el sistema operativo actualizado, de esta forma nos protegemos de los últimos fallos de seguridad.

Los siguientes comandos actualizarán, no solo el software de nuestro dispositivo, si no también sus librerias y el propio sistema operativo. 

	sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    
Este proceso puede tardar un rato. Cuando haya terminado reinciamos nuestro servidor:

	sudo reboot now
    
### ➡️ Certificados de autenticación en SSH:
 Comodo 🔑
 
En este punto queremos establecer conexiones SSH sin necesidad de introducir la contraseña.  Para ello crearemos certificados de autenticación y los intercambiaremos entre los dos dispositivos. 

Primero de todo crearemos una clave RSA:

    ssh-keygen -t rsa
![](https://i.imgur.com/QF14MXD.png)

Seguidamente creamos una carpeta en el dispositivo receptor.

    ssh pi@[ip] mkdir -p ssh

Salimos de la conexión actual con un "exit" para proceder a copiar nuestra llave.

Nos dirigimos mediante comandos (o abrimos un terminal nuevo) a la carpeta donde los hayamos generado y ejecutamos el siguiente comando.

    cat .ssh/[file-name.pub] | ssh pi@[ip] 'cat >> .ssh/authorized_keys'

De esta forma nuestra llave del dispositivo reside en el dispositivo receptor, por lo que ya no hará falta hacer uso de la contraseña.

### ➡️ Desactivar conexiones con root via SSH:

Deshabilitaremos las conexiones a root via SSH por motivos de seguridad, para hacerlo modificaremos el archivo de configuracion de SSH (`ssh_config`) alojado en esta direccion:

	cd /etc/ssh/
 
Podemos usar `nano` para alterar el archivo y cambiar el `PermitRootLogin` por `no`. 

	sudo nano sshd_config
  
El archivo a modificar tendría que estar así: 
    
	# Authentication:	
	...
	PermitRootLogin no
    
Guardamos y reiniciamos el servicio SSH.  

    # service ssh restart
  o
  
    $ sudo service ssh restart

### ➡️ Firewall y Fail2Ban:

Otra configuración extra que podemos habilitar es el Fail2Ban.

Fail2Ban basicamente lo que hace es detectar y bloquear una IP cuando esta, esta haciendo muchas peticiones de login. 

Esta medida es completamente opcional y relativamente simple de implementar. Para hacer uso de Fail2Ban se puede seguir la explicación de esta [página](https://www.linode.com/docs/security/using-fail2ban-for-security).

## 3. Empezando a nivel local.

Se podría decir que hasta ahora, del punto 1 al 2 todo lo que hemos hecho es una puesta apunto de nuestra placa. En este punto es cuando empezamos a montar el servidor, en este caso un servidor web basado en Apache en el cual podemos alojar paginas web. 

Una vez tengamos todo configurado, empezaremos creando y configurando nuestro servidor. Como ya tenemos creado el servidor SSH, pasaremos a crear el servidor web.

Una de las soluciones más conocidas para servidores es Web es Apache, que es el que voy a usar en esta guia. 

Existen otras soluciones como Nginx, pero que no son tan configurables ni faciles de instalar, aunque en determinados entornos son más recomendables que Apache. 

Antes de pasar a la instalación lo que haremos será crear el grupo "www-data". Para ello ejecutaremos los siguientes comandos:

    sudo groupadd www-data
	sudo usermod -a -G www-data www-data

Una vez lo tengamos actualizamos e instalamos: 

	sudo apt-get update
    sudo apt-get install apache2
    sudo service apache2 restat

Una vez lo tengamos instalado podemos para comprobar como nuestro servidor web esta funcionando insertando la IP de nuestro servidor en el navegador de nuestro equipo. 

Esta conexión se realiza por defecto en el puerto 80.

Hasta aquí ya tendríamos un gestor de conexiones HTTP y ya podríamos cagar paginas web. 

Hoy en día Javascript es el lenguaje mas extendido para crear contenido dinámico en entornos web, ademas permite una buena separación de cliente-servidor. Aun así existen otras alternativas como **PHP**. 

Por otro lado dependiendo del proyecto que queramos llevar acabo vamos a necesitar instalar o configurar otro tipo de software y hardware: Bases de datos, discos o memorias externas, etc. 

**MySQL** y **PhpMyAdmin** suelen ser buenas alternativas. **MongoDB** es una buena solución para bases de datos escalables.

Este software suele actualizarse con frecuencia y considero que ya hay buenas guias y foros donde explican su instalación. 

☢️Apache acepta muchas configuraciones y ofrece muchas posibilidades, como sitios virtuales, control de acceso, etc. 


## 4. Abriendo el servidor. 
En este apartado abriremos nuestro dispositivo al mundo para poder acceder a nuestra pagina web o a nuestro servidor desde cualquier parte.

Este punto puede ser obviado si solo queremos un servidor local. 

Usaremos un dominio gratuito y el servicio [**Dyndns**](https://es.wikipedia.org/wiki/DNS_din%C3%A1mico) nos facilita el poder tener un nombre de dominio asignado a una IP pública dinámica. El procedimiento seria similar con un dominio propio también.

### ➡️ No-IP. 
En mi caso usaré uno dominio gratuito de [No-IP](https://www.noip.com/) .

❗️ No-IP ofrece servicios de pago. Yo usaré solo en plan **Free Dynamic DNS** el cual me permite tener hasta 3 dominos de forma gratuita, aunque no personalizados. 

❗️ En caso de tener un dominio propio este tendría que ser transferido a No-IP. También se pueden comprar dominios mediante ellos. Todas estas gestiones están sujetas a cobros, de la misma forma que lo esta el gestor de correo, los certificados SSL, backups, etc.

**Como funciona Dyndns:** El servicio Dyndns se suele utilizar sobre todo para asociar unos servicios o puertos de un equipo de nuestra red con un nombre de dominio;

El funcionamiento no tiene secreto, básicamente se asocia el **nombre de dominio** a la **IP pública dinámica** que tenemos. Instalamos un cliente de actualización, que notifica los cambios de la **IP pública** al servidor de nombre de dominio, para que el nombre de dominio actualice siempre la IP pública a la que está asociado.

De esta forma podremos acceder desde fuera de la red al equipo de nuestra red interna que se asocia a ese nombre de dominio. 

#### Pasos: 
Lo primero que tenemos que hacer es dirigirnos a la [página](https://www.noip.com/) de No-IP i registrarnos. Una vez tengamos nuestra cuenta creada y accedamos al panel de nuestra cuenta (en dashboard) veremos un widget llamado "Quick Add". 

Podemos elegir el dominio y el hostname que queramos siempre que este disponible. Este hostname y dominio será la futura URL que introduciremos en nuestro navegador para acceder al servidor apache que hemos instalando anteriormente. 

También será el hostname que usaremos para establecer las conexiones SSH en caso de que no estemos en casa. Ya que recuerdo que esta URL corresponderá a la IP publica de nuestra Raspberry. 

Una vez tengamos el hostname creado, podemos proceder a la instalación del servicio Dyndns. Yo usare el cliente Dyndns que proporciona No-IP. 

Las instrucciones las encontramos en [Dynamic Update Client](https://my.noip.com/#!/dynamic-dns/duc)   dentro del apartado **Dynamic DNS**

Los pasos son bastante simples: 

> 1.  Download the DUC and save the file to:  /usr/local/src
>
> 2.  Open terminal and execute the following:
    -   cd /usr/local/src
    -   tar xzf noip-duc-linux.tar.gz
    -   cd no-ip-2.1.9
    -   make
    -   make install
 >   
> 3.  Create the configuration file:  /usr/local/bin/noip2 -C
>4.  You will be prompted to enter your username and password for No-IP, and for the hostnames you wish to update.
>
>5.  Launch the DUC:  /usr/local/bin/noip2


❗️ Asegurarse que tenemos **acceso al panel de control de router** para la redirección de puertos.  

Para acceder al router generalmente tenemos que teclear en el navegador la IP del router, podemos usar otra vez el escanner de IPs o Fing para saber cual es. 


🚀

## 5. VPN, FTP, etc.
https://www.sitepoint.com/setting-up-a-home-vpn-using-your-raspberry-pi/


	sudo apt-get install openvpn easy-rsa
http://carlini.es/crear-un-servidor-vpn-en-una-raspberry-pi/




