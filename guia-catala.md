
Guia de creació d'un servidor Raspberry Pi Zero en català.

## Taula de continguts:
1. [Preparació](https://github.com/Pedroos46/raspberry-server/blob/master/guia-catala.md#1-preparaci%C3%B3)
2. [Recomanacions](https://github.com/Pedroos46/raspberry-server/blob/master/guia-catala.md#2-recomanacions)
3. [Configuració local](https://github.com/Pedroos46/raspberry-server/blob/master/guia-catala.md#3-comen%C3%A7ant-a-nivell-local)
4. [Obrint el servidor](https://github.com/Pedroos46/raspberry-server/blob/master/guia-catala.md#4-obrint-el-servidor)
5. [VPN, FTP, etc](https://github.com/Pedroos46/raspberry-server/blob/master/guia-catala.md#5-vpn-ftp-etc)



## 1. Preparació:
Abans de començar amb la creació del servidor hem d'instal·lar el sistema operatiu. Encara que [hi ha varies alternatives](https://www.raspberrypi.org/downloads/) jo faré servir la distro oficial Raspbian.

### ➡️ Instal·lació del sistema operatiu:
Podem descarregar Raspbian directament des de [aqui](https://www.raspberrypi.org/downloads/raspbian/).
La instal·lació no té secret, descarreguem la imatge que vulguem i fem servir els [passos proporcionats](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) a la pàgina oficial.

> -   Download  [Etcher](https://etcher.io/)  and install it.
> -   Connect an SD card reader with the SD card inside.
> -  Open Etcher and select from your hard drive the Raspberry Pi  `.img`  or  `.zip`  file you wish to write to the SD card.
> -  Select the SD card you wish to write your image to.
> -   Review your selections and click 'Flash!' to begin writing data to the SD card.

### ➡️ Configuració inicial: 


Un cop instal·lat el sistema operatiu començarem amb les configuracions.
En aquest punt ens interessa principalment tenir la connexió wifi configurada i el SSH habilitat.

La idea d'aquesta configuració inicial és minimitzar treball, si comencem amb aquestes dues configuracions evitem posteriorment tenir connectar a la nostra placa un ratolí, un teclat i un monitor.

Per **configurar la connexió WiFi** ens dirigim en el volum "boot" de la targeta SD on haguem instal·lat el sistema. Un cop a l'arrel del directori vam crear un arxiu anomenat **wpa_supplicant.conf**.

![Creamos el archivo wpa_supplicant.conf](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/1.png)

Aquest fitxer ha de contenir els següents detalls:
```bash
network={
    ssid="NETWORK_NAME"
    psk="NETWORK_PASSWORD"
    key_mgmt=WPA-PSK
}
```

Col·locant aquest arxiu aquí Raspbian el mourà a **/etc/wpa_supplicant/** quan el sistema arranqui.

Per **habilitar el SSH** crearem en el mateix directori "boot" un arxiu anomenat **ssh** sense cap extensió.

![enter image description here](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/2.png)


Aquest fitxer serà eliminat en arrencar el sistema, però habilita per defecte el servei SSH. Un cop tinguem això fet ja podem continuar.

Podem utilitzar la comanda `touch` des del terminal per agilitzar la creació dels arxius. Exemple:

    touch ssh

### ➡️ Començant amb SSH

Un cop iniciada la Raspberry i el sistema el primer que hem de saber és la IP que se li ha assignat per poder establir connexió.

Podem utilitzar un escàner de xarxa per obtenir-la. Si no tenim cap eina una simple [cerca a Google](http://bfy.tw/LSCo) o la botiga d'apps. Jo utilitzaré una aplicació anomenada `Fing`.

Un cop tinguem la IP comencem amb SSH, per a això necessitem un servidor SSH a la nostra Raspberry o placa (que ja hem activat en passos anteriors) i un client SSH al nostre ordinador.

Per al client SSH hem de tenir instal·lat un en el nostre ordinador. Si fem servir **Linux** o **MacOS** probablement no caldrà instal·lar cap, normalment ja l'inclouen

Si fem servir **Windows** podem utilitzar clients com el [Putty] (http://www.putty.org/) o podem fer servir el CMD o el PowerShell de Windows al juntament amb [OpenSSH](http: //www.mls-software .com/opensshd.html). D'altra banda Windows 10 Pro inclou un client SSH ja instal·lat.

El funcionament és exactament el mateix tant per a Linux i Mac com per a Windows.

Així que tornem a intentar la comanda `ssh`:

	usage: ssh [-1246AaCfGgKkMNnqsTtVvXxYy] [-b bind_address] [-c cipher_spec]
           [-D [bind_address:]port] [-E log_file] [-e escape_char]
           [-F configfile] [-I pkcs11] [-i identity_file]
           [-J [user@]host[:port]] [-L address] [-l login_name] [-m mac_spec]
           [-O ctl_cmd] [-o option] [-p port] [-Q query_option] [-R address]
           [-S ctl_path] [-W host:port] [-w local_tun[:remote_tun]]
           [user@]hostname [command]


De moment només realitzarem una simple connexió amb el servidor.

Per fer-ho necessitem **l'usuari** del nostre servidor, en el cas de la Raspberry l'usuari per defecte és `pi`, **la IP** de la nostra placa i **la contrasenya** l'usuari, que en el cas de les Raspberry la contrasenya per defecte per a l'usuari `pi` és` raspberry`.

	ssh [usuario]@[ip o hostname]
Per lo tant:

    ssh pi@[ip]
    password: raspberry
    

Un cop realitzada la connexió i el login podrem actuar com si estiguéssim en el propi dispositiu.

❗️A partir d'aquest punt tot el gestionarem via SSH i el terminal. Podem deixar la Raspberry en un altre costat.


## 2. Recomanacions:

Obrim una connexió SSH amb la nostra Raspberry amb l'usuari `pi` i ja podem començar a aplicar els punts que ens interessen.

### ➡️ Assignar una IP local estàtica:
Molt recomanat ✅

Un cop tinguem el nostre dispositiu arrencat i connectat a la xarxa local és molt recomanable  posar la nostra IP local en estàtica. El motiu pel qual fem això és perquè més tard, des del router, anem a apuntar la IP del nostre futur servidor de manera que necessitem que aquesta IP no canviï mai.

Per aconseguir-ho ens dirigirem a la següent ruta en el nostre sistema: `/etc/network/`

Dins de la carpeta ens trobarem amb un arxiu anomenat `interfaces`. El primer que farem amb aquest fitxer és fer una còpia per si cometem algun error. Farem servir la comanda `cp`:

	sudo cp /etc/network/interfaces interfaces.copy
    
Un cop feta la còpia, farem servir `nano` per editar l'arxiu.

	sudo nano -w /etc/network/interfaces

Haurem afegir les línies que hi ha més avall. És probable que `iface xxxx inet nanual/static` ja existeixi, en aquest cas només afegirem les línies que ens interessen. Al final hauria de tenir un aspecte com aquest.

    iface wlan0 inet static
      address 192.168.1.180
      gateway 192.168.1.1
      

On posa `address` posarem la IP estàtica que volem. No estaria de més entrar al nostre router per saber que rang d'IPs podem entrar-li i quin és el `gateway` ja que això sol estar subjecte al router que tinguem.

❗️Normalment `wlan0` és la interfície que s'usa amb connexió wifi. Si la nostra connexió és per ethernet normalment la interfície és `eth0`. Sempre podem mirar que interfície estem fent servir amb la comanda `ifconfig`.

Per a més informació i per a una altra versions de Linux es pot visitar la següent [pàgina](https://www.linode.com/docs/networking/linux-static-ip-configuration).

Un cop ho tinguem, reiniciem la nostra placa i ja hauria de començar a treballar amb la nova IP estàtica.

	sudo reboot now



### ➡️ Millorar el rendiment reassignant recursos:

La idea és tenir treballant la Raspberry sense necessitat de supervisió i sense cap perifèric connectat. Per tant, podríem reduir la quantitat de recursos que destinem al processament gràfic i fins i tot desactivar la interfície.


Per començar llançarem l'assistent de configuració de Raspbian.
	sudo raspi-config
    
En caso de no tenerlo lo instalamos:  

	sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 7FA3303E
	sudo apt-get update
	sudo apt-get install raspi-config
    
    
Un cop llançat, el primer que farem serà desactivar la interfície d'escriptori, per tant tampoc ens cal tanta capacitat GPU així que li reduirem la memòria.
 
L'assistent és bastant visual així i per desactivar la interfície serà una cosa així:
 		
   	Boot options > Desktop / CLI > Console. 
        
Posteriorment li reduirem la memòria a la GPU assignant-li `16`.
 
 	Advanced options > Memory Split. 
	
 

### ➡️ Crear un nou usuari i eliminar l'usuari per defecte:

Recomanat ✅

Per motius de seguretat no podem deixar el nostre usuari per defecte habilitat. Per tant en crearem un de nou i eliminarem el per defecte:

Per crear-los farem servir la següent comanda, el qual conté també els grups de sistema al qual volem que aquest.

	sudo useradd -m -G adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,netdev,input NUEVO-USUARIO
 
Posteriorment li donarem una nova contrasenya.
 
	sudo passwd NUEVO-USUARIO
    
    
Un cop, creat el nou usuari tanquem la connexió SSH actual escrivint `exit` i ens tornarem a connectar amb el nou usuari.

	ssh NUEVO-USUARIO@[ip o hostname]
    
Un cop dins eliminarem l'usuari per defecte i totes les seves dades. En aquest cas de la Raspberry és `pi`, si tenim un altre nom d'usuari hi ha posar dit usuari.
 
 	sudo deluser --remove-all-files pi	
	sudo userdel -r -f pi



### ➡️ Actualitzant el sistema operatiu:

Un altre punt important de seguretat és mantenir el sistema operatiu actualitzat, d'aquesta manera ens protegim dels últims errors de seguretat.

Els següents comandes s'actualitzaran, no solament el programari del nostre dispositiu, sinó també les seves llibreries i el propi sistema operatiu.

	sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    
Aquest procés pot trigar una estona. En acabar reiniciem el nostre servidor:

	sudo reboot now
    
### ➡️ Connexions SSH sense contrasenyes:
  Còmode 🔑
 
En aquest punt volem establir connexions SSH sense necessitat d'introduir la contrasenya. Per a això **crearem certificats d'autenticació** i els intercanviarem entre els dos dispositius.

Primer de tot crearem una clau RSA:

    ssh-keygen -t rsa -b 4096
    
![](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/3.png)

Seguidament  creem una carpeta en el dispositiu receptor.

    ssh pi@[ip] mkdir -p ssh


Sortim de la connexió actual amb un "exit" per procedir a copiar la nostra clau.

Ens dirigim mitjançant ordres (o obrim un terminal nou) a la carpeta on els hàgim generat i executem la següent comanda.

    cat .ssh/[file-name.pub] | ssh pi@[ip] 'cat >> .ssh/authorized_keys'

D'aquesta forma la nostra clau del dispositiu resideix en el dispositiu receptor, de manera que ja no caldrà fer ús de la contrasenya.


### ➡️ Desactivar connexions amb root via SSH:

Deshabilitarem les connexions a root via SSH per motius de seguretat, per fer-ho modificarem l'arxiu de configuració de SSH (`ssh_config`) allotjat en aquesta direcció:

	cd /etc/ssh/
 
Podem usar `nano` per alterar l'arxiu i canviar el` PermitRootLogin` per `no`.

	sudo nano sshd_config
  
L'arxiu a modificar hauria d'estar així:
    
	# Authentication:	
	...
	PermitRootLogin no
    
Guardem y reiniciem el servei SSH.  

    # service ssh restart
  o
  
    $ sudo service ssh restart

### ➡️ Firewall i Fail2Ban:

Una altra configuració extra que podem habilitar és el fail2ban.

Fail2ban bàsicament el que fa és detectar i bloquejar una IP quan aquesta, aquesta fent moltes peticions de connexió.

Aquesta mesura és completament opcional i relativament simple d'implementar. Per fer ús d'fail2ban es pot seguir l'explicació d'aquesta [pàgina](https://www.linode.com/docs/security/using-fail2ban-for-security).


## 3. Començant a nivell local.

➡️ Es podria dir que fins ara, del punt 1 al 2 tot el que hem fet és una posada a punt de la nostra placa. En aquest punt és quan vam començar a muntar el servidor, en aquest cas un servidor web basat en Apache en el qual podem allotjar pàgines web.

Un cop tinguem tot configurat, començarem creant i configurant el nostre servidor. Com ja tenim creat el servidor SSH, passarem a crear el lloc web.

Una de les solucions més conegudes per a servidors és Web és Apache, que és el que vaig a utilitzar en aquesta guia.
😮Existen altres solucions com Nginx, però que no són tan configurables ni fàcils d'instal·lar, encara que en determinats entorns són més recomanables que Apache.

Obrim una connexió SSH amb la nostra Raspberry amb l'usuari `pi`o` NOU-USUARIO` i ja podem començar amb la instal·lació.

Abans de passar a la instal·lació el que farem serà crear el grup "www-data". Per a això executarem les següents comandes:

    sudo groupadd www-data
	sudo usermod -a -G www-data www-data

Un cop ho tinguem actualitzem i reinciem el servei.

	sudo apt-get update
    sudo apt-get install apache2
    sudo service apache2 restat

Un cop el tinguem instal·lat podem per comprovar com el nostre servidor web aquesta funcionant inserint la IP del nostre servidor al navegador del nostre equip.

❗️ Aquesta connexió (HTTP) es realitza per defecte en el port 80. En cas d'afegir un certificat SSL i tenir una connexió HTTPS es faria pel port 443.

Fins aquí ja tindríem un gestor de connexions HTTP i ja podríem cagar pàgines web.

Avui dia Javascript és el llenguatge mes estès per crear contingut dinàmic en entorns web, a més permet una bona separació de client-servidor. Tot i aixó i hi ha altres alternatives com ara **PHP**.

❗️D'altra banda depenent del projecte que vulguem dur a terme necessitarem instal·lar o configurar un altre tipus de programari i maquinari: Bases de dades, discs o memòries externes, etc.

**MySQL** i **PhpMyAdmin** solen ser bones alternatives. **MongoDB** és una bona solució per a bases de dades escalables.

☢️ Apache accepta moltes configuracions i ofereix moltes possibilitats, com llocs virtuals, control d'accés, etc.


## 4. Obrint el servidor.
En aquest apartat obrirem el nostre dispositiu al món per poder accedir a la nostra pàgina web o al nostre servidor des de qualsevol part.

Aquest punt pot ser obviat si només volem un servidor local.

Farem servir un domini gratuït i el servei [**DynDNS**](https://es.wikipedia.org/wiki/DNS_din%C3%A1mico) ens facilita el poder tenir un nom de domini assignat a una IP pública dinàmica. El procediment seria similar amb un domini propi també.

**Com funciona DynDNS:** El servei DynDNS s'usa per associar uns serveis o ports d'un equip de la nostra xarxa amb un nom de domini.

> Bàsicament s'associa el **nom de domini** a la **IP pública dinàmica** que tenim. Instal·lem un client d'actualització, que notifica els canvis de la **IP pública** al servidor de nom de domini, perquè el nom de domini actualitzi sempre la IP pública a la qual està associat.

D'aquesta manera podrem accedir des de fora de la xarxa a l'equip de la nostra xarxa interna que s'associa a aquest nom de domini.

### ➡️ No-IP.
En el meu cas faré servir 1 domini gratuït de [No-IP](https://www.noip.com/).

❗️ No-IP ofereix serveis de pagament. Jo faré servir només en pla **Free Dynamic DNS** el qual em permet tenir fins a 3 dominos de forma gratuïta, tot i que no personalitzats.

❗️ En cas de tenir un domini propi aquest hauria de ser transferit a No-IP. També es poden comprar dominis. Totes aquestes gestions estan subjectes a cobraments, de la mateixa manera que ho està el gestor de correu, els certificats SSL, backups, etc.

#### Passos:
El primer que hem de fer és dirigir-nos a la [pàgina](https://www.noip.com/) de No-IP i registrar-nos. Un cop tinguem el nostre compte creat i accedim al panel del nostre compte (en dashboard) veurem un widget anomenat "Quick Add".

![enter image description here](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/4.png)


Podem triar el domini i el hostname que vulguem sempre que estigui disponible. Aquest hostname i domini serà la futura URL que introduirem en el nostre navegador per accedir al servidor apatxe que hem instal·lant anteriorment.

També serà el hostname que farem servir per establir les connexions SSH en cas que no estiguem a la xarxa local de la Raspberry. Ja que, recordo, que aquesta URL correspondrà a la IP publica de la nostra Raspberry.

Un cop tinguem el hostname creat, podem procedir a la instal·lació del servei DynDNS. Jo usi el client DynDNS que proporciona No-IP.

Les instruccions les trobem en [Dynamic Update Client(https://my.noip.com/#!/dynamic-dns/duc) dins de l'apartat **Dynamic DNS**

Els passos són bastant simples:

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


Tots aquestes comandes han de ser executats en el terminal en el qual tinguem iniciada la connexió SSH amb la nostra placa.

Per descarregar el paquet a la nostra Raspberry i no al nostre ordinador. Podem copiar l'enllaç de descàrrega i després usar la comanda `wget`.

![enter image description here](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/5.png)

Abans d'executar aquesta comanda seria interessant dirigir-se en el directori que se'ns demana que es descarregui per evitar després haver de moure-ho.
Per tant:
 

    cd /usr/local/src
    wget https://www.noip.com/client/linux/noip-duc-linux.tar.gz


D'aquesta manera via SSH descarreguem el paquet directament a la nostra Raspberry i en el directori que ens interessa.

Existeix també un vídeo proporcionat per No-IP on es pot seguir visualment els passos a seguir:

[![IMAGE ALT TEXT](http://img.youtube.com/vi/8xp4kkbsZi0/0.jpg)](http://www.youtube.com/watch?v=8xp4kkbsZi0 "How to Download and Configure the No-IP Dynamic Update Client on Linux")

En cas de no tenir  `wget` instalat:

    sudo apt-get install wget
	

Un cop acabat aquest procés cal procedir a obrir els ports que ens interessen del router. Per accedir al router generalment hem de teclejar en el navegador la IP del router, podem usar una altra vegada el escanner d'IPs o Fing per saber com és.

❗️ Cal assegurar que tenim **accés al panel de control de router** per a la redirecció de ports.

Un cop dins el tauler de control del router, ens dirigim l'opció que posi **Port Forwarding** o **Redirecció de Ports**.

Aquesta part pot ser diferent segons el router que estiguem configurant. Però bàsicament el que hem de fer redireccionar els ports.

La configuració que ens trobarem serà una cosa semblant a això:

![enter image description here](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/6.png)


El funcionament és simple. La connexions que entrin en un determinat port per `from` han de ser redirigides cap a la `IP Address` de la nostra Raspberry, amb el mateix port per `to`, sempre que aquest no hagi estat configurat per un altre, que no hauria de ser el nostre cas.

Exemple:

> Si nosaltres fem una petició a la nostra IP publica pel port 80
> (Fent servir el nostre hostname) aquesta quan 'entri' serà redirigida cap a la
> IP local al port 80, aquesta IP local serà la de la nostra Raspberry i
> El port 80, que és el port per defecte de navegació web, serà escoltada per Apache.

Com ja he dit el port 80 sol ser el port per defecte de navegació de manera que si ens entra una connexió pel port 80 (`Port from`) hem de redireccionar amb `Port to` cap a la `IP Address` estàtica de la nostra Raspberry .

El mateix succeeix amb les connexions SSH que es realitzen pel port 22.

Per tant després de configurar la connexions web i les connexions SSH hauria de quedar-nos alguna cosa així:
![enter image description here](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/7.png)


Un cop configurat tot, el nostre servidor ja pot ser accessible des d'on vulguem 🚀.

## 5. VPN, FTP, etc.


Un cop creat un servidor web amb SSH i haver-ho fet accessible fora de la xarxa local, podem extrapolar tot l'après a altres solucions, com ara un gestor de fitxers FTP per poder enviar les nostres pàgines web al servidor de manera còmoda o un servidor VPN per poder encapsular les nostres connexions.

### ➡️ FTP

Primer anem a descarregar el servidor `vsftpd`. Aquest serà l'encarregat de gestionar les connexions i la transferència d'arxius.

    sudo apt-get install vsftpd

Una vegada que aquest descarregat obrim el següent arxiu de configuració.

    sudo nano /etc/vsftpd.conf

S'ha de descomentar les següents línies. Aquestes línies permeten l'escriptura d'arxius als usuaris de la Raspberry Pi.

>    local_enable=YES
>  
>   write_enable=YES

Finalment reiniciem el servei.

    sudo service vsftpd restart

Ara ja tenim el servei de servidor instal·lat. Ara procedim a instal·lar el client.

Per sort nostra ha un genial programa open-source anomenat **FireZilla**, procedim a [descarregar-lo](https://filezilla-project.org/) i instal-lo.

Un cop instal·lat i obert veurem alguna cosa similar a això:

![](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/8.png)


-   servidor :  [IP de la Raspberry] o [URL]
-   nom d'usuari: [pi] o [nuevo-usuario]
-   contrasenya: la contraseña del usuario.


En quan haguem establert la connexió, transferir arxius és tan fàcil com arrossegar i deixar anar.

⚠️ Si volem realitzar connexions usant el nostre hostname o la nostra URL haurem de **habilitar al router la redirecció del port per defecte en connexions FTP, el 21**, ja que repeteixo, les connexions que realitzem usant el hostname es fan amb la IP publica i per tant requereixen una redirecció per arribar a la IP local de l'Raspberry.


👨‍👩‍👧‍👦 Si volem utilitzar el nostre servidor FTP amb diversos comptes, però no volem que aquestes tinguin accés total podem crear un usuari nou en el nostre sistema i atorgar-sol permisos en certes carpetes o en un directori principal. Una forma molt bàsica de fer-ho seria així:

    sudo useradd NUEVO-USUARIO-FTP

Creem una carpeta en la qual el geekyuser podrà crear tots els directoris que vulgui.

    sudo mkdir /home/NUEVO-USUARIO-FTP

Donem permisos a l'usuari creat.

    sudo chown NUEVO-USUARIO-FTP:users /home/NUEVO-USUARIO-FTP

Finalment vam crear una contrasenya per a l'usuari

    sudo passwd NUEVO-USUARIO-FTP

☢️ El servei FTP, igual que Apache, accepta moltes altres configuracions, de manera que és possible fer un ús més intensiu i personalitzat.

### ➡️ VPN
🚀 OpenVPN Easy-RSA:

[http://carlini.es/crear-un-servidor-vpn-en-una-raspberry-pi/](http://carlini.es/crear-un-servidor-vpn-en-una-raspberry-pi/)
