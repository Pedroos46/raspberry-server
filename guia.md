Guia de creación de un servidor Raspberry Pi Zero español, catalán e Inglés. 

## Tabla de contenidos: 
1. [Preparación.](https://github.com/Pedroos46/raspberry-zero-server#1-preparación)
2. [Tips.](https://github.com/Pedroos46/raspberry-zero-server#2-seguridad)
3. [Configuración local.](https://github.com/Pedroos46/raspberry-zero-server#3-empezando-a-nivel-local)
4. Abriendo el servidor.  
5. VPN. 
6. 


## 1. Preparación: 
Antes de empezar con la creación del servidor tenemos que instalar el sistema operativo. Aunque [existen varias alternativas](https://www.raspberrypi.org/downloads/) yo usaré la distro oficial Raspbian.

### Instalación del sistema operativo:
La podemos descargar Raspbian directamente des de [aqui](https://www.raspberrypi.org/downloads/raspbian/).
La instalación no tiene secreto, descargamos la imagen que queramos y usamos los [pasos proporcionados](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) en la pagina oficial. 

> -   Download  [Etcher](https://etcher.io/)  and install it.
> -   Connect an SD card reader with the SD card inside.
> -  Open Etcher and select from your hard drive the Raspberry Pi  `.img`  or  `.zip`  file you wish to write to the SD card.
> -  Select the SD card you wish to write your image to.
> -   Review your selections and click 'Flash!' to begin writing data to the SD card.

### Configuración inicial: 

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
Colocando este archivo aquí, Raspbian lo moverá  a **/etc/wpa_supplicant/** cuando el sistema arranque.


Para **habilitar el SSH** crearemos en msomo


### Empezando con SSH:

Una vez tenemos nuestra IP local estatica y conexión a Internet empezaremos a trabajar con SSH.

Que es y que nos permite hacer? Para simplificar, SSH es una forma segura de comunicarse remotamente con otro ordenador via comandos y basicamente nos permitira administrar nuestro servidor de forma remota sin necesidad de tener acceso fisico a la maquina.

Usaremos SSH primero de todo por comododidad, con SSH podemos gestionar nuestro servidor desde nuestro ordendor personal sin necesidad de acceso fisico a este. 

Ademas las conexiones por SSH están cifradas, es rapido y ligero y no esta sujeto a los ataques que afectan a HTTP. 

Y finalmente, porque en la vida real se trabaja así. Hoy en dia es poco frequente que una empresa o institución tenga sus propios servidores y menos acceso fisico a estos. Hoy en dia, los servidores se suelen alquilar a otras grandes empresas, las cuales poseen grandes infraestructuras por lo que el coste es inferior al que supondria la misma infraestructura pero siendo propia, a parte de que ofrecen mucha mas flexibilidad, rapida escalabilidad, servicios complementarios, etc.

Para empezar a usar SSH necesitamos un servidor SSH en nuestro dispositivo y un cliente SSH en nuestro ordenador. 

Para el servidor, muchas distribuciones (incluido Raspbian) ya lo llevan instalado por defecto. Aun así no esta de mas asegurarnos que lo tenemos probando el comando `ssh`.
 
En caso de que no haya respuesta, lo instalaremos usando: 

	sudo apt-get install ssh

Nota: Raspbian como he dicho ya lo lleva instalado o deberia, pero es posible que este desactivado. En caso de que diese problemas se tendria que activar.

Para el cliente SSH debemos tener intalado uno en nuestro ordenador. Si usamos **Linux** probablemente no hará falta instalar uno, normalmente las ditribuciones ya incluyen un cliente SSH. Podemos probar, tambien desde el terminal el comando `ssh` y en caso de no tenerlo usar el comando anteriormente.

Si usamos **Windows** podemos usar clientes como [Putty](http://www.putty.org/) o podemos usar el CMD o el PowerShell de Windows junto [OpenSSH](http://www.mls-software.com/opensshd.html). Por otro lado Windows 10 Pro incluye un cliente SSH ya instalado. 

En mi caso usare OpenSSH sobre Windows (por comodidad), si decides usar también OpenSSH, recomiendo que durante la instalación se desmarque la opción de servidor, ya que solo usaremos el cliente. 

Una vez tengamos el cliente instalado en nuestro ordenador el funcionamiento es exactamente el mismo tanta para Linux como para Windows. 

Así que volvemos a intentar el commando `ssh`, esta vez si tendría que responder con un mensaje similar a este: 

	usage: ssh [-1246AaCfGgKkMNnqsTtVvXxYy] [-b bind_address] [-c cipher_spec]
           [-D [bind_address:]port] [-E log_file] [-e escape_char]
           [-F configfile] [-I pkcs11] [-i identity_file]
           [-J [user@]host[:port]] [-L address] [-l login_name] [-m mac_spec]
           [-O ctl_cmd] [-o option] [-p port] [-Q query_option] [-R address]
           [-S ctl_path] [-W host:port] [-w local_tun[:remote_tun]]
           [user@]hostname [command]

Como se puede suponer esta es la guia de uso de SSH, por el momento nosotros no necesitamos hacer uso estas opciones asi que demomento solo realizaremos una simple conexión con el servidor. 

Para hacerlo necesitamos **el usuario** de nuestro servidor, que en el caso de la Raspbery es `pi`, **la IP** estatica que hemos introducido antes y **la contraseña** del usuario.

	ssh [usuario]@[ip o hostname]
    
Una vez entrado este comando se establecerá la conexión y nos pedirá la contraseña del usuario. Una vez dentro podremos actuar como si estubieramos en el propio dispositivo pero de forma remota, por lo que podemos dejar nuestro dispositivo a un lado y trabajar desde nuestro ordenador. 

### Haciendo nuestra IP local estatica:

Una vez tengamos nuestro dispositivo arrancado y conectado a Internet tenemos que poner nuestra IP local estatica. El motivo por el cual hacemos esto es porque mas tarde, desde el router, vamos a apuntar la IP de nuestro futuro servidor por lo que necesitamos que esa IP no canvie nunca. 

Para lograrlo nos dirigiremos a la seguiente ruta en nuestro sistema: `/etc/network/`

Dentro de la carpeta nos encontraremos con un archivo llamado `interfaces`. Lo primero que haremos con este archivo es hacer una copia por si cometemos algun error. Usaremos el comando `cp`:

	sudo cp /etc/network/interfaces interfaces.old
    
Una vez hecha la copia, usaremos `nano` para editar el archivo. 

	sudo nano -w /etc/network/interfaces

Deberemos añadir las lineas que hay más abajo. Es probable que `iface xxxx inet nanual/static` ya exista, en este caso solo añadiremos las lineas que nos interesan. Al final deberia tener un aspecto como este.

    iface wlan0 inet static
      address 192.168.1.180
      gateway 192.168.1.1
      
En donde pone `address` podremos la IP estatica que queremos. No estaría de más entrar en nuestro router para saber que rango de IPs podemos entrarle y cual es el `gateway` ya que esto suele estar sujeto al router que tengamos. 

Nota:  Normalmene `wlan0` es la interfaz  que se usa con conexión wifi. Si nuestra conexión es por ethernet normalmente la interficie es `eth0`. Siempre podemos mirar que interficie estamos usando con el comando `ifconfig`.

Para más información y para otra versiónes de Linux se puede vistar la siguiente [pàgina](https://www.linode.com/docs/networking/linux-static-ip-configuration).

Una vez lo tengamos, reiniciamos nuestro servidor y ya deberia empezar a trabajar con la nueva IP estatica. 






