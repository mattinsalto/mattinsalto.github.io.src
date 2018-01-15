---
date: 2017-12-14T20:31:27+01:00
title: Pyhton remote debugging on a Raspberry Pi
categories: ['Code', 'Tutorials']
tags: ['python', 'raspberry pi', 'debugging']
author: "Martin Garmendia"
featuredImage: "images/python-remote-debugging.jpg"
noSummary: false
---
Although is posible to program in the Raspberry Pi (rpi onwards), is a little bit rudimentary due to it's low power and lack of powerfull IDEs. This is specially true if you don't install a desktop environment as in Raspbian lite.

There are some options for Python remote debugging on a rpi, but only few of them are free and posible from a Mac. The option I'm gonna write about, is the one provided by Wingware Wing Personal (free editon) Python IDE. You can find the information about rpi Python remote debugging in it's [website][wingware-remote-dbg], but it has some broken links and there are areas not covered by specific solutions that let the reader choose her preferred one. I'll try to give a complete solution here.


* Download Wing Python IDE

http://wingware.com/downloads/wing-personal

* Enable ssh remote access in the rpi

If you don't have enabled it yet, follow the instructions at
[Raspberry pi official docs][raspberry-ssh]

Once enabled, to know your rpi IP type below command in the rpi terminal
```sh
$ ifconfig
```
You will see the IPs for the different network interfaces:

```sh
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.91  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::178e:1188:54a:c522  prefixlen 64  scopeid 0x20<link>
        ether b8:27:eb:e9:67:5b  txqueuelen 1000  (Ethernet)
        RX packets 343823  bytes 26399285 (25.1 MiB)
        RX errors 0  dropped 100  overruns 0  frame 0
        TX packets 54392  bytes 3054249 (2.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 48  bytes 6280 (6.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 48  bytes 6280 (6.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

When using a wired connection it will be eth0, if wireless, should be wlan0

If everything is OK, you can connect to rpi with:
```sh
$ ssh pi@your-rpi-ip
```

In order to avoid discovering what IP your rpi has taken every time it reboots, you could set a static IP editing /etc/dhcpcd.conf

```sh
$ nano /etc/dhcpcd.conf
```
and adding this lines to the end of the file
```sh
interface eth0

static ip_address=192.168.1.91/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8 8.8.4.4
```

then reboot your rpi for the configuration to take effect
```sh
$ reboot
```
or 
sue a 
```sh
$ sudo service networking restart
```

* Generate ssh public / private key pair 

We don't want to be promted for a password every time we want to execute a command in the rpi. We can generate a public and private key pair and put the public key in the rpi to trust the connection without promting for password.

In Mac Terminal type (leave it empty when asks for password protection)
```sh
$ cd ~/.ssh
$ ssh-keygen -t rsa
```

Now copy the contents of id_rsa.pub to /home/pi/.ssh/authorized_keys as a new line in the rpi. If authorized_keys doesn't exist you can create it and copy the contents of id_rsa.pub.

For now on, you can ssh your rpi from the Mac without password. This also means you can copy files from the Mac to the rpi via scp without password promting.

* Download Wingware remote debugger to the raspberry pi

At the time of writing the last version of Wing Python IDE is 6.0.9. If it has changed, [here you have][wingware-remote-dbg] the root of the download directory for the different versions. Copy the link to your version of wingide-debugger-raspbian and ssh your rpi:

```sh
$ ssh pi@rpi-ip
$ wget http://wingware.com/pub/wingide/6.0.9/wingide-debugger-raspbian-6.0.9-1.tar.bz2
$ tar xaf wingide-debugger-raspbian-6.0.9-1.tar.bz2
$ mv wingide-debugger-raspbian-6.0.9-1 wing-rmt-dbgr
```

We also need to copy a file (wingdebugpw) from Settings directory of the Wing installation on the Mac to the recently decompressed directory in the rpi. To see where this directory is, go to Wings about box (Wingpersonal menu -> About Wingpersonal -> Settings directory).

Once located, copy to rpi via scp (don't forget to change YourUser and your-rpi-ip)
```sh
$ scp "/Users/YourUser/Library/Application Support/Wing Personal/v6/wingdebugpw" pi@your-rpi-ip:/home/pi/wing-rmt-dbgr
```

* Create a new Wing Pyhton IDE project

In this example we will name it PythonRemoteDebugTest 
and locate it at 
*/Users/YourUser/src/PythonRemoteDebugTest*
Add a new python script to your project: 

*main.py*
{{<highlight python "linenos=table">}}
name = input('What is your name? ')
print('Hello {}'.format(name))
{{</highlight>}}

Run it locally to ensure it works as expected

* Map local directory with a remote one

Create a directory in the rpi to host your python proyects.
You can execute a remote command via ssh from the Mac Terminal without the need of an open ssh sesssion

```sh
$ ssh pi@192.168.1.91 "mkdir /home/pi/python"
```
In Wing Python IDE go to 

Preferences -> Debugger -> Advanced

Scroll down to *Location Map*, select de existing line and pulse *edit button*.

Leave remote IP as 127.0.0.1 because we will set a reverse ssh tunnel so will look like the connection comes from localhost. 

Specify a mapping and set remote to

/home/pi/python 

and local to 

/Users/mattinsalto/src/Python

* Create a reverse ssh tunnel between your Mac and Rpi

```sh
$ ssh -N -R 50005:localhost:50005 pi@192.168.1.91
```
* Copy your project files to rpi

```sh
$ scp -r /Users/mattinsalto/src/Python/PythonRemoteDebugTest pi@192.168.1.91:/home/pi/python
```
* Start remote debugging setting desired python version and calling wingdb script 

```sh
$ ssh pi@192.168.1.91 "export WINGDB_PYTHON=/usr/bin/python3 && sudo -E /home/pi/wing-rmt-dbgr/wingdb /home/pi/python/PythonRemoteDebugTest/main.py"
```

[wingware-download]: http://wingware.com/downloads/wing-personal
[wingware-remote-dbg]: http://wingware.com/pub/wingide/
[raspberry-ssh]: https://www.raspberrypi.org/documentation/remote-access/ssh/
