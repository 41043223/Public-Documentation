# PypI
Pypi is a package index that can host own pip repo server


## Run in Docker 

Save as Dockerfile
```Dockerfile
FROM ubuntu:20.04
MAINTAINER NFU_CCIS_41043223
RUN apt update
RUN apt upgrade -y
RUN apt install python3 python3-pip python3-venv screen -y
RUN cd /
RUN mkdir pypi_server
RUN cd pypi_server/
RUN python3 -m venv venv
RUN pip3 install pypiserver
RUN pip3 install pip2pi
RUN mkdir packages
```
build
```
docker build -t pypi .
```
run and forwarding port, 963 is external port and 8800 is internal 
```
docker run -it --name pypi -p 963:8800 pypi:latest bash
```

## Run directly
if you don't want install in docker, you can directly isntall in you OS use below commands

```
apt update
apt upgrade -y
apt install python3 python3-pip python3-venv screen -y
cd /
mkdir pypi_server
cd pypi_server/
python3 -m venv venv
pip3 install pypiserver
pip3 install pip2pi
mkdir packages
```

## Install

Download packages as you want
```pip3 install numpy```

Write installed package to txt
```pip3 freeze > requirements.txt```

Use installed packages to cache
```
pip2pi ./packages/. -r requirements.txt
```
Check packages

```sh
ls ./packages
pip3 install passlib
htpasswd -sc .htpasswd demouser
```
Start Server in background
```sh
screen -S pypi -dm bash -c "pypi-server -p 8800 ./packages/; exec sh"
```



## How to add packages

Download packages as you want
```
pip3 install PACKAGE
```
```
PACKAGE is your package name 
```
Write installed package to txt
```
pip3 freeze > requirements.txt
pip2pi ./packages/. -r requirements.txt
```
Check packages

```sh
ls ./packages
```

## How to use in client

```
pip3 install --index-url http://IP/simple/ PACKAGE [PACKAGE2...]
```

```
IP is Server IP
```