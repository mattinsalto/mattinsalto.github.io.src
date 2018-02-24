---
title: Run net core apps on raspberry pi
date: 2018-02-24T20:26:14+01:00
featuredImage: "images/netcoreraspi.png"
categories: ['Tutorials', 'Code']
tags: ['raspberry pi', 'net core']
author: "Martin Garmendia"
noSummary: false
---

There is no SDK that runs on ARM32 but you can publish an application that will run on a Raspberry Pi.

First we need to install the prerequisite packages for net core 2 on raspbian.
Open a terminal session on the raspberry pi and type
```sh
sudo apt-get update
sudo apt-get install curl libunwind8 gettext apt-transport-https
```

Now we will create a sample app on our desktop pc:

```sh
mkdir helloworld
cd helloworld
dotnet new console
```
The last command creates an example helloworld console program that we will use to test if it runs on the raspberry pi. For this we need to publish it for linux-arm

```sh
dotnet publish -r linux-arm -c Release
``` 
and copy the publish folder to the raspberry pi. If you are on a windows machine you could use [winscp](https://winscp.net).
```sh
cd bin/Release/netcoreapp2.0/linux-arm
scp -r publish pi@raspberry-pi-ip-address:/home/pi/helloworld
```

Open a terminal on the raspberry and type:
```sh
cd helloworld
./helloworld
``` 
and voilá
```sh
Hello World!
```