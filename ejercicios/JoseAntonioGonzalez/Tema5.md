#Ejercicio 1
#### Instalar los paquetes necesarios para usar KVM. Se pueden seguir estas instrucciones. Ya lo hicimos en el primer tema, pero volver a comprobar si nuestro sistema está preparado para ejecutarlo o hay que conformarse con la paravirtualización.
Empezaremos por el principio, y comprobaremos si nuestra CPU soporta la tecnología KVM.Para saber si tenemos KVM activado, utilizamos la orden

	egrep -c "vmx" /proc/cpuinfo
    
En nuestro caso, nos devuelve 4, que son el número de procesadores que tienen la tecnología KVM activada.

Al ser compatible, vamos a instalar los paquetes necesarios, siguiendo los pasos de [este enlace](https://wiki.debian.org/KVM#Installation)

	sudo apt-get install qemu-kvm

	sudo apt-get install libvirt-bin
    
#Ejercicio 2
#### Crear varias máquinas virtuales con algún sistema operativo libre tal como Linux o BSD. Si se quieren distribuciones que ocupen poco espacio con el objetivo principalmente de hacer pruebas se puede usar CoreOS (que sirve como soporte para Docker) GALPon Minino, hecha en Galicia para el mundo, Damn Small Linux, SliTaz (que cabe en 35 megas) y ttylinux (basado en línea de órdenes solo).

#### 2.Hacer un ejercicio equivalente usando otro hipervisor como Xen, VirtualBox o Parallels.
Primero creamos el disco duro virutal, de tipo QCOW2. Lo haremos de 6 gigas aproximadamente (6.000 MB):

	qemu-img create -f qcow2 fichero-cow.qcow2 6000M

Después, bajamos la imagen de una distribución, como por ejemplo Debian, desde [este enlace](cdimage.debian.org/debian-cd/8.2.0/i386/iso-cd/debian-8.2.0-i386-netinst.iso).

Cargamos el módulo kernel KVM, mediante la siguiente orden (ya que el equipo en el que se realizan estas prácticas, cuenta con un procesador Intel):

	sudo modprobe kvm-intel

Y levantado el módulo, y con todo listo, lanzamos la máquina virtual con el archivo de disco duro virtual creado y la imagen descargada:

	qemu-system-x86_64 -hda fichero-cow.qcow2 -cdrom ./Descargas/debian-8.2.0-i386-netinst.iso 


![](https://www.dropbox.com/s/l3pka2eob4qap9e/ejercicio2.png?dl=1)

Seguimos las instrucciones que aparecen en pantalla para completar la instalación de Debian

De esta forma, hemos instalado Debian usando QEMU en una máquina virtual.

#### 2.2

Podemos replicar el ejercicio anterior usando virtualbox. Lo instalamos y lanzamos mediante:
	sudo apt-get install virtualbox-qt
    virtualbox
 
Hecho eso, seguimos el asistente de nueva creación de máquina vritual. Por intentar hacerlo lo más parecido posible, se escoge como tipo de archivo de disco duro firtual QCOW, como en la imagen se ve:

![](https://www.dropbox.com/s/8ebab0lj18js0mj/Ejercicio5-2?dl=1)

Una vez finalizado el asistente,  configuramos la máquina para que el CD sea la imagen descargada de Debian.

![](https://www.dropbox.com/s/wov01bmj54zmonz/Ejercicio2-3.png?dl=1)

GUardamos, lanzamos la máquina, y seguimos el asistente de instalación igual que en el apartado 2.1


#Ejercicio 4
#### Crear una máquina virtual Linux con 512 megas de RAM y entorno gráfico LXDE a la que se pueda acceder mediante VNC y ssh.
Descargaremos la imagen de Lubuntu, que cumple los requisitos del enunciado, del siguiente [enlace](cdimage.ubuntu.com/lubuntu/releases/14.04/release/lubuntu-14.04-desktop-i386.iso)

DEspués, crearemos el fichero de disco duro virtual:

	qemu-img create -f qcow2 hdd_lubuntu.qcow2 6000M
    
Instalamos Lubunu:
	qemu-system-x86_64 -hda hdd_lubuntu.qcow2 -cdrom ./Descargas/lubuntu-14.04-desktop-i386.iso -m 512M

![](https://www.dropbox.com/s/gr68ofn74qt7zde/Ejercicio4_1.png?dl=1)
Con la máquina ya creada y con el SO instalado, necesitaremos descargar un cliente VNC. Ya que los apuntes del temario recomiendan Vinagre, vamos a seguir el ejemplo:

	sudo apt-get install vinagre

![](https://www.dropbox.com/s/f4lsptqerzga7m3/Ejercicio4.png?dl=1)
Una vez instalada la máquina, la apagamos, y la volvemos a encender, ésta vez con una opción de sevidor VNC:

	qemu-system-i386 -hda hdd_lubuntu.qcow2 vnc :1

Con esto, podemos conectarnos con vinagre a nuestra máquina, con la siguiente orden:

	vinagre localhost:5901

El puerto 5901 viene del puerto estandar (5900) +1 por enumerarla con un 1 en nuestro servidor VNC (opción vnc: 1 al arrancar la máquina)

Si quisieramos conectar mediante SSH a nuestra máquina, debemos de modificar las opciones de lanzamiento de la misma, añadiendo un puerto de conexión, y un nombre:

	qemu-system-x86_64 -boot order=c -drive file=hdd_lubuntu.qcow2,if=virtio -m 512M -name usuario -redir tcp:3000::22
    
Lo que decimos con las últimas opciones de la órden, es que exista un usuario llamado "usuario", y que redirija las peticiones tcp que se hagan por el puerto 3000 de la máquina anfitrión, al puerto 22 de la máquina anfitrion (recordemos que el puerto 22 es el utilizado por defecto para las conexiones SSH)

De esta forma, nos conectamos a la máquina mediante la órden:
	
    ssh -p 3000 usuario@
    
#Ejercicio 5
#### Crear una máquina virtual ubuntu e instalar en ella un servidor nginx para poder acceder mediante web.
Para poder utilizar Azure, haremos uso de los pases que se dieron a principio de curso. Creamos nuestra cuenta y asociamos el código proporcionado (fue necesario para el hito 4 de la asignatura).

Para poder hacer uso de la máquina Azure mediante consola (es más cómodo y nos permitirá ganar experiencia para automatizar scripts de despliegue), instalamos Azure:

	sudo apt-get install nodejs-legacy#Necesario para instalar azure
	sudo apt-get install npm #Necesario para instalar azure
    sudo npm install -g azure-cli
    
Instalado todo, conectaremos con la cuenta de azure en nuestra terminal. Para ello, primero configuramos asm de azure:

	azure config mode asm
    azure account download

Esta última orden nos devolverá un enlace que debemos de seguir, y tras loguearnos en nuestra cuenta, nos redirigirá a una página de descarga de un archivo.

![](https://www.dropbox.com/s/nn42dcblgzgcf3y/ejercicio5.png?dl=1)

Con este archivo descargado, lo importamos en azure:

	azure account import ./Descargas/Pase\ para\ Azure-1-24-2016-credentials.publishsettings 

Y nos devolverá un OK si todo va bien en la importación de credenciales

![](https://www.dropbox.com/s/mlkwkp6hooby7cc/Ejercicio5-2.png?dl=1)

Hecho todo, crearemos el sitio con la siguiente orden:

	azure site create --location "Central US" ejercicio5-IV-JAGonz
    
Donde ejercicio5-IV-JAGonz es el nombre del sitio web que estamos creando. Ya deberíamos poder acceder en el navegador [con el siguiente enlace](http://ejercicio5-iv-jagonz.azurewebsites.net/)

![](https://www.dropbox.com/s/6mszvxb3nd8z0pz/ejercicio5-3.png?dl=1)

El sitio web está creado, pero no cumple las condiciones del ejercicio aún.Instalaremos Ubuntu server en la máquina. Primero debemos saber la imagen a instalar, con la siguiente orden:

	azure vm image list "Central US" ubuntuserver
    
Nos devolverá una lista con todas las imágenes de ubuntu server disponibles. Nosotros instalamos una con nombre **b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_3-LTS-amd64-server-20160119-en-us-30GB**

Creamos la máquina virtual:

	azure vm create ejercicio5-iv-jagonz b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_3-LTS-amd64-server-20160119-en-us-30GB jagonz --location "West US" --ssh"

Una vez creada, la arrancamos con la siguiente orden:

	azure vm start ejercicio5-iv-jagonz

![](https://www.dropbox.com/s/pbbeetiejzetvdw/Ejercicio5-4-recortar%20solo%20la%20imagen%20de%20abajo.png?dl=1)
Nos conectamos por ssh a la máquina creada:

	ssh joseantonio@ejercicio5-iv-jagonz.cloudapp.net

Instalamos nginx:

	sudo apt-get update
    sudo apt-get install nginx	
    sudo fuser -k 80/tcp
    sudo service nginx start
    
Por último abrimos el puerto 80 en la máquina:

	azure vm endpoint create ejercicio5-iv-jagonz 80 80

Y al conectarnos a la página web que sirve nginx a través del navegador, veremos la página de bienvenida si se ha configurado todo correctamente:

![](https://www.dropbox.com/s/lbeuv6qv9m26t17/ejercicio5-5.png?dl=1)
#Ejercicio 7
#### Instalar una máquina virtual con Linux Mint para el hipervisor que tengas instalado.
Tenemos que descargarnos la imagen de LinuxMint. En mi caso, descargamos la imagen de 32 bit de [aqui](http://www.linuxmint.com/edition.php?id=203) (Se ubicará en la carpeta de descargas de la carpeta personal).

Ahora, creamos la imagen del disco duro virtual:

	qemu-img create -f qcow2 hdd_linuxmint.qcow2 6000M

Acto seguido, iniciamos la instalación con la siguiente orden:
	qemu-system-x86_64 -hda hdd_linuxmint.qcow2 -cdrom ./Descargas/linuxmint-17.1-cinnamon-32bit.iso 
    

![](https://www.dropbox.com/s/yytsqj6soltm4f2/ejercicio7.png?dl=1)

También podríamos hacerlo desde VIrtualbox, configurando una nueva máquina (siguiendo el asistente), y añadiendo la imagen virtual del disco de instalación, tal y como se aprecia en la imagen:

![](https://www.dropbox.com/s/yku84cnjusv31kn/Ejercicio7-1.png?dl=1)
