Guide to create a Raspberry Pi Zero server in English.

## Table of Contents:
1. [Preparation](https://github.com/Pedroos46/raspberry-server/blob/master/guia-espa%C3%B1ol.md#1-preparaci%C3%B3n)
2. [Recommendations](https://github.com/Pedroos46/raspberry-server/blob/master/guia-espa%C3%B1ol.md#2-recomendaciones)
3. [Local Settings](https://github.com/Pedroos46/raspberry-server/blob/master/guia-espa%C3%B1ol.md#3-beginning-to-level-local)
4. [Opening the server](https://github.com/Pedroos46/raspberry-server/blob/master/guia-espa%C3%B1ol.md#4-abriendo-el-servidor)
5. [VPN, FTP, etc](https://github.com/Pedroos46/raspberry-server/blob/master/guia-espa%C3%B1ol.md#5-vpn-ftp-etc)


## 1. Preparation:
Before starting with the creation of the server we have to install the operating system. Although [there are several alternatives](https://www.raspberrypi.org/downloads/) I will use the official Raspbian distro.

### ➡️ Operating system installation:
You can download Raspbian directly from [here](https://www.raspberrypi.org/downloads/raspbian/).

The installation has no secret, download the image you want and use the [steps provided](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) on the official page.

> -   Download  [Etcher](https://etcher.io/)  and install it.
> -   Connect an SD card reader with the SD card inside.
> -  Open Etcher and select from your hard drive the Raspberry Pi  `.img`  or  `.zip`  file you wish to write to the SD card.
> -  Select the SD card you wish to write your image to.
> -   Review your selections and click 'Flash!' to begin writing data to the SD card.


### ➡️ Initial configuration:

Once the operating system is installed, we will start with the configurations.
At this point we are mainly interested in having the Wi-Fi connection configured and the SSH enabled.

The idea of this initial configuration is to minimize work, if we start with these two configurations we avoid having to connect a mouse, a keyboard and a monitor to our board.

To **configure the WiFi connection** we go to the "boot" volume of the SD card where we have installed the system. Once in the directory root, we create a file called **wpa_supplicant.conf**.

![Creamos el archivo wpa_supplicant.conf](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/1.png)


We can also use terminal: 

```bash
    cd /Volumes/boot  
    nano `wpa_supplicant.conf`
```
This file must contain the following details:

 ```bash
 country=COUNTRY
 ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
 update_config=1
 
 network={
        ssid="SSID"
        psk="PASSWORD"
        key_mgmt=WPA-PSK
     }
```
Where COUNTRY should be set the two letter ISO country name, for example: ES, US, GB, FR, etc.
Placing this file here, Raspbian will move it to **/etc/wpa_supplicant/** when the system boots.

To **enable the SSH** we will create in the same directory "boot" a file called **ssh** without any extension.

![enter image description here](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/2.png)


This file will be deleted when booting, but it enables the SSH service by default. Once we have this done we can start the system.

We can use the `touch` command from the terminal to speed up the creation of the files. Example:

    touch ssh

### ➡️ Starting with SSH

Once the Raspberry and the system are started, the first thing we need to know is the IP that has been assigned to establish a connection.

We can use a network scanner to obtain it. If we do not have any tool a [simple Google search](http://bfy.tw/LSCo) or in the app store, can help us. I will use an application called `Fing`.

Once we have the IP we start with SSH, for this we need an SSH server on our Raspberry or board (which we have already activated in previous steps) and an SSH client on our computer.

For the SSH client we must have one installed on our computer. If we use **Linux** or **MacOS** you probably will not need to install one, normally it already includes it.

If we use **Windows** we can use clients like [Putty](http://www.putty.org/) or we can use the CMD or the Windows PowerShell together [OpenSSH](http://www.mls-software.com/opensshd.html). On the other hand Windows 10 Pro includes an SSH client already installed.

The operation is exactly the same for Linux and Mac as for Windows.

So we try the `ssh` command again:

	usage: ssh [-1246AaCfGgKkMNnqsTtVvXxYy] [-b bind_address] [-c cipher_spec]
           [-D [bind_address:]port] [-E log_file] [-e escape_char]
           [-F configfile] [-I pkcs11] [-i identity_file]
           [-J [user@]host[:port]] [-L address] [-l login_name] [-m mac_spec]
           [-O ctl_cmd] [-o option] [-p port] [-Q query_option] [-R address]
           [-S ctl_path] [-W host:port] [-w local_tun[:remote_tun]]
           [user@]hostname [command]


At the moment we will only make a simple connection with the server.

To do this we need **the username** of our server, in the case of the Raspberry the user by default is `pi`, **the IP** of our board and **the password** of the user, which in the case of the Raspberry the default password for the user `pi` is `raspberry`.

	ssh [usuario]@[ip o hostname]
	
Therefore:

    ssh pi@[ip]
    password: raspberry
    

Once the connection and the login, we can act as if we were on the device itself.

❗️ From this point we will manage everything via SSH and the terminal. We can leave the Raspberry elsewhere.


## 2. Recommendations:

Open an SSH connection with our Raspberry with the user `pi` and we can start applying the points that interest us.

### ➡️ Assign a static local IP:
Highly recommended ✅

Once we have our device booted and connected to the local network it is highly recommended that we put our local IP static. The reason why we do this is because later, from the router, we will point the IP of our future server so we need that IP never change.

To achieve this we will go to the following route in our system: `/etc/network/`

Inside the folder we will find a file called `interfaces`. The first thing we will do with this file is to make a copy in case we make an error. We will use the `cp` command:

	sudo cp /etc/network/interfaces interfaces.copy
    
Once the copy is made, we will use `nano` to edit the file.

	sudo nano -w /etc/network/interfaces

We will have to add the lines below. It is probable that `iface xxxx inet manual/static` already exists, in this case we will only add the lines that interest us. In the end it should look like this.

    iface wlan0 inet static
      address 192.168.1.180
      gateway 192.168.1.1
      

Where it says `address` we will put the static IP that we want. It would not hurt to enter our router to know what range of IPs we can enter and what is the `gateway` as this is usually subject to the router we have.

❗️ Normally `wlan0` is the interface used with Wi-Fi connection. If our connection is via ethernet, the interface is usually `eth0`. We can always see what interface we are using with the `ifconfig` command.

For more information and for other versions of Linux you can view the following [page](https://www.linode.com/docs/networking/linux-static-ip-configuration).

Once we have it, we reboot our board and should start working with the new static IP.

	sudo reboot now



### ➡️ Improve performance by reallocating resources:

The idea is to have the Raspberry working without supervision and without any connected peripheral. Therefore, we could reduce the amount of resources we allocate to graphic processing and even disable the interface.

To start we will launch the Raspbian configuration wizard.

	sudo raspi-config
    
In case of not having it, install it:

	sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 7FA3303E
	sudo apt-get update
	sudo apt-get install raspi-config
    
    

Once launched, the first thing we will do is disable the desktop interface, therefore we do not need as much GPU capacity so we will reduce the memory.
 
The assistant is quite visual yet to deactivate the interface it will be something like this:
 		
   	Boot options > Desktop / CLI > Console. 
        
Later we will reduce the memory to the GPU by assigning `16`.
 
 	Advanced options > Memory Split. 
	
 

### ➡️ Create a new user and delete the user by default:

Recommended ✅

For security reasons we can not leave our user by default enabled. So we will create a new one and delete the default one:

To create them we will use the following command, which also contains the system groups to which we want it to be.

	sudo useradd -m -G adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,netdev,input NUEVO-USUARIO
 
Later we will give it a new password.
 
	sudo passwd NUEVO-USUARIO
    
    
Once, created the new user we closed the current SSH connection by typing `exit` and we will reconnect with the new user.

	ssh NUEVO-USUARIO@[ip o hostname]
    
Once inside we will eliminate the user by default and all your data. In this case the Raspberry is `pi`, if we have another username there we'll put that user.
 
 	sudo deluser --remove-all-files pi	
	sudo userdel -r -f pi



### ➡️ Updating the operating system:

Another important security point is to keep the operating system updated, in this way we protect ourselves from the latest security flaws.

The following commands will update, not only the software of our device, but also its libraries and the operating system itself.

	sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    
This process may take a while. When finished, we restart our server:

	sudo reboot now
    

### ➡️ SSH connections without passwords:
  Convenient 🔑
 
At this point we want to establish SSH connections without the need to enter the password. To do this **we will create authentication certificates** and exchange them between the two devices.

First of all we will create an RSA key:

    ssh-keygen -t rsa -b 4096
    
![](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/3.png)

Next, we create a folder on the receiving device.

    ssh pi@[ip] mkdir -p ssh


We leave the current connection with an "exit" to proceed to copy our key.

We go through commands (or open a new terminal) to the folder where we generated them and execute the following command.

    cat .ssh/[file-name.pub] | ssh pi@[ip] 'cat >> .ssh/authorized_keys'


In this way, our device key resides in the receiving device, so it will no longer be necessary to use the password.

### ➡️ Disable root connections via SSH:

We will disable the connections to root via SSH for security reasons, to do so we will modify the SSH configuration file (`ssh_config`) hosted at this address:

	cd /etc/ssh/
 
We can use `nano` to alter the file and change the` PermitRootLogin` to `no`.

	sudo nano sshd_config
  
The file to modify should be like this:
    
	# Authentication:	
	...
	PermitRootLogin no
    
Save and restart the SSH service.

    # service ssh restart
  or
  
    $ sudo service ssh restart

### ➡️ Firewall and Fail2Ban:


Another extra configuration that we can enable is the Fail2Ban.

Fail2Ban basically what it does is detect and block an IP when it is, is making many requests for login.

This measure is completely optional and relatively simple to implement. To use Fail2Ban you can follow the explanation of this [page](https://www.linode.com/docs/security/using-fail2ban-for-security).


## 3. Beginning at the local level.

It could be said that until now, from point 1 to 2, all we have done is a quick start of our plate. At this point is when we start to mount the server, in this case a web server based on Apache in which we can host web pages.

Once we have everything configured, we will start creating and configuring our server. As we have created the SSH server, we will create the web server.

One of the best known solutions for servers is Web Apache, which is the one I will use in this guide.

😮 There are other solutions such as Nginx, but they are not as configurable or easy to install, although in certain environments they are more recommendable than Apache.

We open an SSH connection with our Raspberry with the user `pi`o` NEW-USER` and we can start with the installation.

Before moving on to the installation, what we'll do is create the "www-data" group. For this we will execute the following commands:

    sudo groupadd www-data
	sudo usermod -a -G www-data www-data

Once we have it, we update and install:

	sudo apt-get update
    sudo apt-get install apache2
    sudo service apache2 restat


Once we have it installed we can check how our web server is working inserting the IP of our server in the browser of our team.

❗️ This connection (HTTP) is done by default on port 80. In case of adding an SSL certificate and having an HTTPS connection it would be done through port 443.

So far we would have an HTTP connection manager, and we could already store web pages on it.

❗️ Nowadays Javascript is the most extended language to create dynamic content in web environments, besides it allows a good separation of client-server. Even so, there are other alternatives such as **PHP**.

On the other hand depending on the project that we want to carry out we will need to install or configure another type of software and hardware: Databases, disks or external memories, etc.

**MySQL** and **PhpMyAdmin** are usually good alternatives. **MongoDB** is a good solution for scalable databases.


☢️ Apache accepts many configurations and offers many possibilities, such as virtual sites, access control, etc.


## 4. Opening the server.
In this section we will open our device to the world to access our website or our server from anywhere.

This point can be ignored if we only want a local server.🌍

We will use a free domain and the service [**Dyndns**](https://es.wikipedia.org/wiki/DNS_din%C3%A1mico) makes it easier for us to have a domain name assigned to a dynamic public IP. The procedure would be similar with a domain of its own as well.

**How Dyndns works:** The Dyndns service is used to associate services or ports of a computer in our network with a domain name.

> Basically the **domain name** is associated with the **dynamic public IP** that we have. We install an update client, which notifies the changes of the **public IP** to the domain name server, so that the domain name always updates the public IP to which it is associated.

In this way we can access from outside the network the team of our internal network that is associated with that domain name.

### ➡️ No-IP.
In my case I will use a free domain of [No-IP](https://www.noip.com/).

❗️ No-IP offers payment services. I will use only the **Free Dynamic DNS** plan, which allows me to have up to 3 domains for free, although not personalized.

❗️ In case of having a domain of its own this would have to be transferred to No-IP. You can also buy domains through them. All these procedures are subject to charges, in the same way as the mail manager, SSL certificates, backups, etc.


#### Steps:
The first thing we have to do is go to the [page](https://www.noip.com/) of No-IP and register. Once we have our account created and access the panel of our account (in dashboard) we will see a widget called "Quick Add".

![enter image description here](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/4.png)


We can choose the domain and the hostname that we want whenever it is available. This hostname and domain will be the future URL that we will introduce in our browser to access the apache server that we have previously installed.

It will also be the hostname that we will use to establish SSH connections in case we are not on the local network of the Raspberry. Since, I remember, that this URL will correspond to the public IP of our Raspberry.

Once we have the hostname created, we can proceed to the installation of the Dyndns service. I will use the Dyndns client that provides No-IP.

The instructions can be found in [Dynamic Update Client](https://my.noip.com/#!/dynamic-dns/duc) within the section **Dynamic DNS**

The steps are quite simple:
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


All these commands have to be executed in the terminal in which we have initiated the SSH connection with our board.

To download the package in our Raspberry and not in our computer. We can copy the download link and then use the `wget` command.

![enter image description here](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/5.png)

Before executing this command it would be interesting to go to the directory that we are asked to download to avoid having to move it later.
Therefore:
 

    cd /usr/local/src
    wget https://www.noip.com/client/linux/noip-duc-linux.tar.gz


In this way via SSH download the package directly in our Raspberry and in the directory that interests us.

There is also a video provided by No-IP where you can visually follow the steps to follow:

[![IMAGE ALT TEXT](http://img.youtube.com/vi/8xp4kkbsZi0/0.jpg)](http://www.youtube.com/watch?v=8xp4kkbsZi0 "How to Download and Configure the No-IP Dynamic Update Client on Linux")

In case of not having `wget` installed:

    sudo apt-get install wget
	

Once this process is finished, we must proceed to open the ports that interest us on the router. To access the router we usually have to type in the router's IP address, we can use the IP or Fing scanner again to know what it is.

❗️ Make sure we have **access to the router control panel** for port redirection.

Once inside the control panel of the router, we go to the option that says **Port Forwarding**.

This part may be different depending on the router we are configuring. But basically what we have to do is redirect the ports.

The configuration that we will find will be something similar to this: 

![](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/6.png)


The operation is simple. The connections that enter a certain port from `from` have to be redirected to the `IP Address` of our Raspberry, with the same port by `to`, as long as it has not been configured by another, which should not be our case.

Example:

> If we make a request to our public IP by port 80
> (using our hostname) is when 'in' will be redirected to the
> Local IP on port 80, that local IP will be that of our Raspberry and
> port 80, which is the default web browsing port, will be
> heard by Apache.

As I said port 80 is usually the default port for navigation so if we enter a connection through port 80 (`Port from`) we must redirect it with `Port to` to the static IP address of our Raspberry .

The same would happen with the SSH connections that are made by port 22.

Therefore, after configuring the web connections and the SSH connections, we should have something like this:

![](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/7.png)


Once everything is configured, our server can now be accessed from wherever we want 🚀.

## 5. VPN, FTP, etc.

Once you have created a web server with SSH and have made it accessible outside your local network, we can extrapolate everything learned to other solutions, such as **for example** an FTP file manager to be able to send our web pages to the server in a comfortable way or a VPN server to encapsulate our connections.

### ➡️ FTP

First let's download the vsftpd server. This will be responsible for managing connections and file transfer.

    sudo apt-get install vsftpd

Once it is downloaded, we open the following configuration file.

    sudo nano /etc/vsftpd.conf

The following lines must be uncommented. These lines allow the writing of files to the users of the Raspberry Pi.

>    local_enable=YES
>
>    write_enable=YES

Finally we restart the service.

    sudo service vsftpd restart


Now we have the server service installed. Now we proceed to install the client.
Luckily, there is a great open-source program called **FireZilla**, we proceed to [download it](https://filezilla-project.org/) and install it.

Once installed and open we will see something similar to this:

![](https://raw.githubusercontent.com/Pedroos46/raspberry-server/master/resources/8.png)


Fill in the server fields, username and password:

- server: [Raspberry IP] or [URL]
- username: [pi] or [NUEVO-USUARIO]
- password: the user's password.

Once we have established the connection, transferring files is as easy as drag and drop.

⚠️ If we want to make connections using our hostname or our URL we will have to **enable in the router the default port redirection in FTP connections, the 21**, since I repeat, the connections we make using the hostname are made with the IP publishes and therefore require a redirection to reach the local IP of the Raspberry.


👨‍👩‍👧‍👦 If we want to use our FTP server with several accounts, but we do not want them to have full access, we can create a new user in our system and grant them only permissions in certain folders or in a main directory. A very basic way to do it would be like this:

    sudo useradd NUEVO-USUARIO-FTP

We create a folder in which the geekyuser will be able to create all the directories that he wants.

    sudo mkdir /home/NUEVO-USUARIO-FTP

We give permissions to the created user.

    sudo chown NUEVO-USUARIO-FTP:users /home/NUEVO-USUARIO-FTP

Finally we create a password for the user

    sudo passwd NUEVO-USUARIO-FTP

☢️ The FTP service, like Apache, accepts many other configurations, so it is possible to make more intensive and personalized use.


### ➡️ VPN
🚀 OpenVPN Easy-RSA:

[http://carlini.es/crear-un-servidor-vpn-en-una-raspberry-pi/](http://carlini.es/crear-un-servidor-vpn-en-una-raspberry-pi/)
