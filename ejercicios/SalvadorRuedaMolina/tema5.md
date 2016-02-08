## Ejercicio 1 
## Instalar los paquetes necesarios para usar KVM. Se pueden seguir estas instrucciones. Ya lo hicimos en el primer tema, pero volver a comprobar si nuestro sistema está preparado para ejecutarlo o hay que conformarse con la paravirtualización.  

Para comprobar si mi sistema esta preparado lo compruebo de la siguiente manera:  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/1_zpskn6cs20h.png)

Para instalar los paquetes necesarios:  

``sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils``   
Ahora indicamos los usuarios que pueden utilizar dicha virtualización:  
``sudo adduser usuario kvm``  

## Ejercicio 2
##1. Crear varias máquinas virtuales con algún sistema operativo libre tal como Linux o BSD. Si se quieren distribuciones que ocupen poco espacio con el objetivo principalmente de hacer pruebas se puede usar CoreOS (que sirve como soporte para Docker) GALPon Minino, hecha en Galicia para el mundo, Damn Small Linux, SliTaz (que cabe en 35 megas) y ttylinux (basado en línea de órdenes solo).  
CoreOS:  
Primero creo el disco:  
``qemu-img create -f qcow2 coreos.qcow2 10G``  
Me descargo la imagen en el siguiente [enlace](https://coreos.com/os/docs/latest/booting-with-iso.html). 
Cargo la imagen:  
``$ qemu-system-x86_64 -machine accel=kvm -hda coreos.qcow2 -cdrom coreos_production_iso_image.iso -m 1G -boot d
``  
Funcionamiento:  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/CoreOS_zpsv60oqeyj.png)   
Slitaz:   
Creamos el disco para el SO:   
``qemu-img create -f qcow2 slitaz.qcow2 3G
``  
Nos descargamos la iso del siguiente [enlace](http://www.slitaz.org/en/) y la  arranco:  
``qemu-system-x86_64 -machine accel=kvm -hda slitaz.qcow2 -cdrom slitaz-4.0.iso -m 2G -boot d
``  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/Slitaz_zps6foremon.png) 

##2. Hacer un ejercicio equivalente usando otro hipervisor como Xen, VirtualBox o Parallels.  

He usado VMWare Player porque ya lo tenía instalado y conocía su funcionamento. He utilizado la imagen que previamenta me habia descargado para el apartado anterior, como es la de Slitaz. Creo la máquina virtual le asigno espacio en el disco y cargo la iso.  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/2_1_zps6tdr4wlz.png) 

![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/2_2_zpsvgo69glu.png) 

## Ejercicio 4
## Crear una máquina virtual Linux con 512 megas de RAM y entorno gráfico LXDE a la que se pueda acceder mediante VNC y ssh.

Me he bajado la imagen de lubuntu que viene con entorno gráfico LXDE de serie. La iso me la he bajado del siguiente [enlace](https://help.ubuntu.com/community/Lubuntu/GetLubuntu).

Creo un disco virtual  para poder instalar posteriormente lubuntu  con el siguiente comando:  
``qemu-img create -f qcow2 espacio_lubuntu.img 9G``

Ahora procedo a instalar la iso con el siguiente comando:  
``qemu-system-x86_64 -hda espacio_lubuntu.img -cdrom lubuntu-15.10-desktop-amd64.iso -m 512M ``

Con este comando se arranca la máquina virtual:  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/4_zpsviteobkl.png)  
Procedemos a instalar lubuntu siguiendo todos los pasos.  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/4_2_zps6j2sumi7.png)  
Ahora si queremos acceder por VNC o ssh primero deberemos instalar en la máquina virtual vinagre y ssh.  
``sudo apt-get install vinagre``  
``sudo apt-get install ssh``  
Ahora arrancamos la máquina mediante VNC:  
``qemu-system-i386 -hda lubuntu.img vnc :1``  
Ahora podemos conectarnos a VNC de la siguiente manera:  
``vinagre localhost:1``  
Para ssh procemos arrancar la máquina de la siguiente forma:  
``qemu-system-x86_64 -machine accel=kvm -hda espacio_lubuntu.img -m 512M -redir tcp:2222::22
``  
Y ya podremos acceder a través de ssh con el siguiente comando:  
``ssh nombredeusuario@localhost -p 2222``
##Ejercicio 5  
## Crear una máquina virtual ubuntu e instalar en ella un servidor nginx para poder acceder mediante web.

He usado mi máquina virtual de ubuntu que ya tenía instalado  nginx aunque su instalación es tan sencilla como:  
``sudo apt-get install nginx``  
Ahora simplemente lanzamos el servicio con:
``sudo service nginx start``  
El servicio sale por el puerto 80 por lo que accedemos a la dirección de nuestra máquina virtual en el navegador.  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/55_zpswahhg5rd.png)  

## Ejercicio 7
## Instalar una máquina virtual con Linux Mint para el hipervisor que tengas instalado.  
Me he bajado la iso del página web de [Linux Mint](http://www.linuxmint.com/edition.php?id=204) y la he instalado en VMware Player que es el programa donde tengo todas las máquinas virtuales. Su puesta en marcha es muy sencilla.  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/7_zpsduaaoyk3.png)   
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/7_2_zpsjfhahdf1.png)