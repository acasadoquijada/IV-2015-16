# Ejercicios Tema 6

## Ejercicio 1
**Instalar chef en la máquina virtual que vayamos a usar.**

Primero, vamos a conectarnos a nuestra máquina virtual de Azure, con ```azure login```, arrancamos la máquina y nos conectamos por ssh con:

```
jesmorc@jesmorc-PClaptop ~/Workinout $ sudo vagrant ssh
[sudo] password for jesmorc: 
==> localhost: Attempting to read state for iv-jesmorc-ubuntuserver in iv-jesmorc-ubuntuserver-service-euemq
==> localhost: VM Status: ReadyRole
==> localhost: Attempting to read state for iv-jesmorc-ubuntuserver in iv-jesmorc-ubuntuserver-service-euemq
==> localhost: VM Status: ReadyRole
    localhost: Looking for local port 22
    localhost: Found port mapping 22 --> 22
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.16.0-37-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Sat Feb 13 17:54:21 UTC 2016

  System load:  0.48              Processes:           232
  Usage of /:   4.3% of 28.80GB   Users logged in:     0
  Memory usage: 9%                IP address for eth0: 100.77.224.46
  Swap usage:   0%

  Graph this data and manage this system at:
    https://landscape.canonical.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud


*** System restart required ***
Last login: Sat Feb 13 17:54:22 2016 from 217.216.81.186.dyn.user.ono.com
vagrant@iv-jesmorc-ubuntuserver:~$ 
```


Dentro de la máquina, instalamos *Chef* con:
```
curl -L https://www.opscode.com/chef/install.sh | sudo bash
```

Esto descargará **chef-solo**, una versión aislada para trabajar en una máquina desde la misma.

Para comprobar la versión instalada:
```
vagrant@iv-jesmorc-ubuntuserver:~$ chef-solo -v
Chef: 12.7.2
```

