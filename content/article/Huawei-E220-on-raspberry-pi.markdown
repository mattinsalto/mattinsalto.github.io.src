---
title: Huawei E220 modem on Raspberry pi
date: 2017-12-01T18:10:14+01:00
featuredImage: "images/HuaweiRaspberry.jpg"
categories: []
tags: ['raspberry pi', 'huawei', '3g-modem', 'movistar', 'freedompop']
author: "Martin Garmendia"
noSummary: false
---
Linux kernel has drivers for this modem, so the installation is straight forward:

```sh
# apt-get install wvdial
```
Edit configuration
```sh
# nano /etc/wvdial.conf
```
Modem standard configuration:
```sh
[Dialer Defaults]
New PPPD = yes
Stupid Mode = 1
Modem Type = 3G Modem
Baud = 460800
Modem = /dev/ttyUSB0
ISDN = 0

[Dialer poweron]
Init1 = AT+CFUN=1

[Dialer poweroff]
Init1 = AT+CFUN=0
```
ISP specific config. Here you have different configurations for two ISPs from Spain.
Movistar:
```sh
[Dialer movistar]
Init2 = AT
Init3 = AT&F&D2&C1E0V1S0=0
Init4 = AT+IFC=2,2
Init5 = ATS0=0
Init6 = AT
Init7 = AT&F&D2&C1E0V1S0=0
Init8 = AT+IFC=2,2
Phone = *99***1#
Password = MOVISTAR
Username = MOVISTAR
```
FreedomPop (Uses masmovil configuration)
```sh
[Dialer freedompop]
Init1 = ATZ
Init2 = ATQ0 V1 E1 S0=0 &C1 &D2
Init3 =
Init4 = AT+CGDCONT=1,"IP","internetmas","",0,0
Baud = 3600000
Username = off
Password = off
New PPPD = yes
Phone = *99*#
Dial Command = ATDT
Stupid Mode = 1
Compuserve = 0
Force Address =
Idle Seconds = 0
Carrier Check = no
ISDN = 0
Auto DNS = 1
Remote Name = "*"
```
To dial, type:
```sh
# wvdial freedompop
```
I have deactivated SIM’s PIN for simplicity and I haven’t tryed to dial with it, but you could test with:
```sh
[Dialer pin]
Dial Command = ATDT
Init = ATZ
Init2 = AT+CPIN=XXXX
```
then:
```sh
# wvdial pin
# wvdial movistar
```
To test if it has worked sue an ifconfig to see if ppp0 has a valid IP
```sh
# ifconfig
```
To connect at startup edit
```sh
# nano /etc/network/interfaces
```
and add this line
```sh
iface ppp0 inet wvdial
	provider movistar
```
As the modem takes some time for initialization we could delay the dialing in
```sh
# nano /etc/rc.local
```
for 20 seconds
```sh
sleep 20
ifup ppp0
```

Credit goes to:

#### [Juan José Valera][movistar].

and

#### [Pablo Martín Medrano][freedompop].

[movistar]: http://www.juanjosevalera.com/archivos/huawei-e220-configuracion-en-raspberry-pi/
[freedompop]: http://odkq.com/debianx060s