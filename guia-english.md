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


This file must contain the following details:
```bash
network={
    ssid="NETWORK_NAME"
    psk="NETWORK_PASSWORD"
    key_mgmt=WPA-PSK
}
```
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
