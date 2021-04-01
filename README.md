
# guide

This has been tested on the raspberry pi 0w

So far I have only gotten it to work on the internal WiFi due to hostapd only haveing driver compatibility for that

### what you will need:

* raspberry pi 0w/3b/4
* if you have the 0w you will need a micro usb otg cable
* usb wifi dongle compatible with GNU/Linux
* power bank with at lease 4000mah
* an sd card at least 32GB

### Get started

first install raspberry pi os to the raspberry pi you are using. use the lite version so it will use less resources and power also your pi will not get as hot

now you want to make your raspberry pi a routed wifi access point

most of these instructions are from <https://www.raspberrypi.org/documentation/configuration/wireless/access-point-routed.md> So I suggest you have a look at them to get more of a understanding

get the pi ready for installing all the software

```
$ sudo apt update
```

```
$ sudo apt -y upgrade
```

```
$ sudo apt install -y wget git libmicrohttpd-dev zip unzip
```

### Now its time to install the access point software

First type or copy this into your terminal

```
$ sudo apt install hostapd
```

Now when it has installed we want to enable it to start when the pi boots with these commands

```
$ sudo systemctl unmask hostapd
```

```
$ sudo systemctl enable hostapd
```

now we install dnsmasq This is so we can give devices on our pis wifi a ip through a DHCP server

Type or copy this command into your terminal

```
$ sudo apt install dnsmasq
```

now install netfilter-persistent and its plugin iptables-persistent. This helps by saving firewall rules and restoring them when the Raspberry Pi boots

```
$ sudo DEBIAN_FRONTEND=noninteractive apt install -y netfilter-persistent iptables-persistent
```

### set up the router part

First we need to edit the dhcpcd config file

Type or copy this into your terminal

```
$ sudo nano /etc/dhcpcd.conf
```

Now add this to the end of the file

```
interface wlan0
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
```

then exit with ctrl + x press y then enter

Now we need to enabel routing to get data from our main network we are connected to through the usb wifi dongle (wlan1)

Type or copy this into your terminal

```
$ sudo nano /etc/sysctl.d/routed-ap.conf
```

Then add this to that file

```
# http://lagnet.glitch.me/guide/setup.html
# Enable IPv4 routing
net.ipv4.ip_forward=1
```

Exit with ctrl + x press y then enter

add the firewall rules

```
$ sudo iptables -t nat -A POSTROUTING -o wlan1 -j MASQUERADE
```

save the firewall rules

```
$ sudo netfilter-persistent save
```

## Configure the DHCP and DNS services for the wireless network

First make a backup of the default config file

```
$ sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
```

open the file to edit it

```
$ sudo nano /etc/dnsmasq.conf
```

Now copy and paste the content in this link https://github.com/thelagnet/dnsc/blob/main/Internet.conf then press ctrl + x then y then enter

### set up the interntet auto detect script

now say you are using your hotspot and you loose Internet or the connection to the main WiFi the login portal wont work so we need to set up a cron job to detect if we have Internet every 1 minute and edit the config then restart dnsmasq

First clone the git repo with the script

```
$ git clone https://github.com/thelagnet/dnsc.git
```

then move that folder to etc

```
$ sudo mv dnsc/ /etc/
```

now we want to setup a cron job for root, it has to be root so we don't need to supply a sudo password to edit the files.

edit the cron jops with this command

```
$ sudo crontab -e
```

then press 1 to select nano as the default text editor then press enter

now go to the bottom of the file and paste this

```
 */1 * * * * bash /etc/dnsc/check.sh
```

Now exit and save with ctrl + x press y then enter

that will run the file check.sh every one minute you might want to replcae the 1 with 3 for every three minute or or 5 for every five minutes

### Configure the access point software

first make sure wifi is unblocked

```
$ sudo rfkill unblock wlan
```

now edit the hostapd config file

```
$ sudo nano /etc/hostapd/hostapd.conf
```

go to this link <https://raw.githubusercontent.com/thelagnet/hostapd/main/hostapd.conf> and paste the content from it into the hostapd config file.\
a small note: make sure there are no spaces after each line or it will faill

you may need to chane your contery code to your contrey or it may not work see <https://en.wikipedia.org/wiki/ISO_3166-1> for the codes

now you can change (enter a name) to a cool or fun name something like "free games mobile games wifi" i don't know be creative.

### set up nodogsplash login page

Now that is now done your pi is now a open network so lets add a login page for this we will use nodogsplash

first clone the software

```
$ git clone https://github.com/nodogsplash/nodogsplash.git
```

  cd into that file 

```
$ cd nodogsplash/
```

 now we need to make it 

```
$ make
```

now install it with

```
$ sudo make install
```

now we will set up the config

first make a backup of the original one incase we need it

```
$ sudo mv /etc/nodogsplash/nodogsplash.conf /etc/nodogsplash/nodogsplash.conf.og
```

then make the new config file 

```
$ sudo nano /etc/nodogsplash/nodogsplash.conf
```

copy and paste the content of this url into it <https://raw.githubusercontent.com/thelagnet/nodogsplash/main/nodogsplash.conf>

the save it with ctrl + x press y then enter

now you want to setup what the login page will look like

clone the html code 

```
$ git clone https://github.com/thelagnet/htdocs.git
```

remove the old file

```
$ sudo rm -r /etc/nodogsplash/htdocs
```

now move the folder we just cloned to nodogsplash

```
$ sudo mv htdocs/ /etc/nodogsplash/
```

now to make nodogsplash start on boot

```
$ sudo nano /etc/rc.local
```

find exit 0 and place this above it

```
nodogsplash
```

restart the pi and the login page should come up when you connect to the wifi

```
$ sudo reboot
```

### set up apache2 webserver

now we need to setup apache2 i know it takes a lot of resorces but i know how to configer the vhosts you can use some other web server if you want

```
$ sudo apt-get install apache2
```

now you want to add all the game files and website files this will take a while

```
$ git clone https://github.com/thelagnet/www.git 
```

now remove the default web page

```
$ sudo rm -r /var/www/
```

then move the one we lust cloned to /etc where the default one was

```
$ sudo mv www/ /var/
```

now you want to set up the config file

```
$ sudo rm /etc/apache2/sites-available/000-default.conf
```

```
$ sudo nano  /etc/apache2/sites-available/000-default.conf
```

now copy the content from this url into there <https://raw.githubusercontent.com/thelagnet/sites-available/main/000-default.conf>

now we are done restart the pi and it should work if you have any problems please conteact us on out cotact page http://lagnet.glitch.me/contact.html

### things you could add

a kiwix wiki that you can access from your pi's access point

if you have a more powerfull pi like the pi 3 or 4 then you could install nextcloud 

### to do and things to add
* [x] github for better file tracking
* [ ] a guide for installing kiwix
* [ ] a guide for installing nextcloud
* [ ] better styled web pages (css)
* [ ] a domia
