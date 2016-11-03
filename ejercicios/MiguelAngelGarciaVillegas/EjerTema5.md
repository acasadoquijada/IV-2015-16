# Asignatura: IV Grupo de Practicas Miercoles 8:30h a 10:30h
## Miguel Ángel García Villegas

# Ejercicios Tema 5

## Ejercicio 1.
### Instalar los paquetes necesarios para usar KVM. Se pueden seguir estas instrucciones. Ya lo hicimos en el primer tema, pero volver a comprobar si nuestro sistema está preparado para ejecutarlo o hay que conformarse con la paravirtualización.

Para instalarlo hay que ejecutar en el terminal la instrucción, ***sudo apt-get install cpu-checker***
![sudo apt-get install cpu-checker](https://www.dropbox.com/s/j3vplffz19hl3iv/01.1.png?dl=1)

Podemos comprobar que nuestro sistema esta preparado para ejecutarlo mediante la orden ***kvm-ok*** o la orden ***egrep -c '(vmx|svm)' /proc/cpuinfo*** si sale un número diferente a 0, indica que la virtualización está soportada.

![kvm-ok](https://www.dropbox.com/s/dg3jvt3yzftdjow/01.2.png?dl=1)

Para instalar los paquetes necesarios, podemos hacerlo mediante la orden ***sudo apt-get install qemu-kvm libvirt-bin virtinst kvm virt-viewer***

![sudo apt-get install qemu-kvm libvirt-bin virtinst kvm virt-viewer](https://www.dropbox.com/s/7tx0a1hh566szrq/01.3.png?dl=1)

Una vez instalados, es necesario indicar el usuario que queremos que utilice la virtualización ***sudo adduser `id -un` libvirtd***

A continuación se tiene que añadir el usuario a los grupos libvirtd y kvm con los siguientes comandos, ***sudo adduser `id -un` libvirtd*** y  ***sudo adduser `id -un` kvm***

![comandos](https://www.dropbox.com/s/coo1a4szqxl3i2i/01.4.png?dl=1)


## Ejercicio 2.1
### Crear varias máquinas virtuales con algún sistema operativo libre tal como Linux o BSD. Si se quieren distribuciones que ocupen poco espacio con el objetivo principalmente de hacer pruebas se puede usar CoreOS (que sirve como soporte para Docker) GALPon Minino, hecha en Galicia para el mundo, Damn Small Linux, SliTaz (que cabe en 35 megas) y ttylinux (basado en línea de órdenes solo).

Comenzamos activando KVM con: ***sudo modprobe kvm-intel***

Primero vamos a crear una máquina virtual con SliTaz: Necesitamos una imagen Iso.

Creamos un disco duro virtual en formato QCOW2:
***qemu-img create -f qcow2 fichero-cow.qcow2 200M***

![qemu-img create -f qcow2 fichero-cow.qcow2 200M](https://www.dropbox.com/s/9yoeeropfngufw2/2.01.png?dl=1)

Instalamos el sistema con el fichero de almacenamiento virtual creado y la ISO descargada, usando KVM mediante: ***qemu-system-x86_64 -hda fichero-cow.qcow2 -cdrom slitaz-4.0.iso***

![qemu-system-x86_64 -hda fichero-cow.qcow2 -cdrom slitaz-4.0.iso](https://www.dropbox.com/s/nglrj2pqfzj7yai/2.02.png?dl=1)

![Qemu](https://www.dropbox.com/s/gu77jgm66ibccrn/02.3.png?dl=1)

Ahora vamos a crear uno en Ubuntu

Creamos un disco duro virtual en formato QCOW2:

***qemu-img create -f qcow2 ubuntu.qcow2 8G***

Instalamos el sistema con el fichero de almacenamiento virtual creado y la ISO descargada, usando KVM mediante:
***qemu-system-x86_64 -machine accel=kvm -hda ubuntu.qcow2 -cdrom ubuntu-14.04.1-desktop-amd64.iso -m 1G -boot d***

![Qemu Ubuntu](https://www.dropbox.com/s/y7mz9mbmptt9b9v/2.4.png?dl=1)

Voy a realizar también la instalación de un lubuntu, desde Qemu.

añadimos disco virtual: ***qemu-img create -f qcow2 lubuntu.qcow2 4G***

Ejecutamos ***qemu-system-x86_64 -hda lubuntu.qcow2 -cdrom ~/Escritorio/Curso15-16/IV/EjerTema5/ejer4/lubuntu-12.04-desktop-i386.iso -m 512M*** para llevar a cabo la instalación.

![Lubuntu ](https://www.dropbox.com/s/zanjre1p4mla3da/04.1.png?dl=1)

![Instalación Lubuntu ](https://www.dropbox.com/s/z5muccqzxfqxrku/04.3.png?dl=1)

![Instalación Lubuntu ](https://www.dropbox.com/s/68nygdpivu19xmy/04.4.png?dl=1).

## Ejercicio 2.2
### Hacer un ejercicio equivalente usando otro hipervisor como Xen, VirtualBox o Parallels.

Vamos a utiliza VirtualBox y vamos a volver a instalar Slitaz.

Abrimos VirtualBox y le damos a Nueva, ponemos el nombre, la configuración, tamaño en la memoria, disco duro, tipo de archivo de disco duro, almacenamiento, ubicación de archivo y tamaño.

![VirtualBox Nueva](https://www.dropbox.com/s/szs3fexm0uqpkqy/3.1.png?dl=1)


![VirtualBox Disco Duro y tipo archivo](https://www.dropbox.com/s/g0522c6j3gzpyeq/3.02.png?dl=1)

Ahora iniciamos

![Corriendo](https://www.dropbox.com/s/nnaujdmwcmjchax/3.3.png?dl=1)

![Corriendo](https://www.dropbox.com/s/mvbwkhsvx3vcqds/3.4.png?dl=1)

## Ejercicio 3.
### Crear un benchmark de velocidad de entrada salida y comprobar la diferencia entre usar paravirtualización y arrancar la máquina virtual simplemente con qemu-system-x86_64 -hda /media/Backup/Isos/discovirtual.img

Para realizar éste ejercicio vamos a hacer uso de ***sysbench*** que permite evaluar la potencia de I/O de los discos

Para ello vamos a instalarlo.

![instalación](https://www.dropbox.com/s/sbee7z2skl0c1j3/03.1.png?dl=1)

Una vez instalado, en nuestro caso vamos a crear los ficheros para el test de 8GB (mayor que la RAM), ejecutando en la terminal el siguiente comando ***sysbench --test=fileio --file-total-size=8G prepare***

![Creando ficheros de test ](https://www.dropbox.com/s/o2yj6pskd8uls2g/03.2.png?dl=1)
Una vez creados, ya sólo nos queda ejecutar el test y esperar por lo resultados.

Vamos a ejecutar para paravirtualización:
***qemu-system-x86_64 -boot order=c -drive file=fichero-cow.qcow2,if=virtio***

## Ejercicio 4.
### Crear una máquina virtual Linux con 512 megas de RAM y entorno gráfico LXDE a la que se pueda acceder mediante VNC y ssh.

Arrancamos ahora mediante servidor VNC ejecutando en el terminal: ```qemu-system-i386 -hda lubuntu.img vnc :1```

Y con la versión de Lubuntu descargada, vamos a iniciar el SO con Qemu: ```qemu-system-x86_64 -hda lubuntu.img -cdrom lubuntu-15.10-desktop-amd64.iso -m 512M``` e instalamos.

![Instalación Lubuntu Ejer4](https://www.dropbox.com/s/nub41un21r0iio0/ejer%204.5.png?dl=1)

![Instalación Lubuntu Ejer4](https://www.dropbox.com/s/pv8bqv0869wic9o/Ejer4.6.png?dl=1)


Para acceso VNC:

Ahora vamos a conectar a la máquina virtual usando algún cliente de VNC tal como vinagre, para ello ejecutamos en el terminal, ***sudo apt-get install vinagre***

Ejecutando ``qemu-system-x86_64 -boot order=c -drive file=lubuntu.img,if=virtio -m 512M -name debian -vnc :1.``

Se queda en ejecución sin abrir ninguna ventana ni nada.

Abrimos otro terminal, y ejecutamos ***ifconfig*** y buscamos la dirección de la interfaz a la que nos conectaremos mediante ***192.168.122.1:5901***

![ejecutando](https://www.dropbox.com/s/dgaac6f404zcpxm/4.11.png?dl=1)

![ejecutando](https://www.dropbox.com/s/ayynmemvoymfufe/4.12.png?dl=1)


Para conectar por SSH:

Primero arrancamos la máquina aunque redireccionamos el puerto: ***qemu-system-x86_64 -boot order=c -drive file=lubuntu.img,if=virtio -m 512M -name Lubuntu -redir tcp:2222::22***

Una vez arranque la máquina virtual instalamos ***ssh*** ejecutando en su terminal, ***apt-get update && apt-get install ssh***

Y conectamos a través del terminal anfitrión ejecutando: ***ssh -p 2222 magv@localhost***.

## Ejercicio 5.
### Crear una máquina virtual ubuntu e instalar en ella un servidor nginx para poder acceder mediante web.


Para realizar este ejercicio, y crear una máquina virtual en ubuntu he usado Koding, que nos ofrece una máquina virtual.

Nos logueamos con GitHub y se crea nuestra máquina virtual

![koding](https://www.dropbox.com/s/wrm1bk30juykhnh/5.png?dl=1)

![koding ubuntu](https://www.dropbox.com/s/1p6xt1oio3p3nhs/5.1.png?dl=1)


Si vamos a la parte ***YourVMS*** podemos ver las características de nuestra máquina.
![Nuestra Máquina](https://www.dropbox.com/s/pfwfxwykfg55q9t/5.6.png?dl=1)

Ahora si ejecutamos en el propio terminal de Koding, ***lsb_release -a*** también podemos ver las características de nuestra máquina, como podemos observar nuestra máquina es Ubuntu.

![koding ubuntu](https://www.dropbox.com/s/pw8hz654hicfv8d/5.5.png?dl=1)

Ahora lo que vamos a hacer es instalar nginx, en la máquina Ubuntu de Koding, ejecutando en el terminal ***sudo apt-get install nginx***

![koding ubuntu](https://www.dropbox.com/s/d9csl769wwldsib/5.2.png?dl=1)

Una vez instalado nginx, hay que parar apache de kogind que está corriendo por defecto, y a continuación ejecutamos ***sudo service nginx start*** para arrancar el servicio nginx. Una vez arrancado nginx vamos y pulsamos a la opciones de nuestra máquina koding-vm-0.

![(..)](https://www.dropbox.com/s/fjgen6og7wc60n9/5.3.png?dl=1)
Vamos a introducir la url que nos ofrece en el navegador para comprobar que nginx está funcionando.

![Nginx en Koding](https://www.dropbox.com/s/3r4m5vxx4fqbb79/5.4.png?dl=1)

## Ejercicio 6.
### Usar juju para hacer el ejercicio anterior.

Vamos a instalar juju, ejecutando en el terminal

  sudo apt-get install juju

![Instalar Juju](https://www.dropbox.com/s/ieqpjcgy8xgznb1/eje6.1.png?dl=1)

Vamos a crear un fichero de configuración ejecutando en la terminal, ***juju generar - config***

Y vamos a editar el fichero de configuración, con nuestros datos en el tipo azure del fichero de configuración.

  location: ***North Europe***

  management-subscription-id: ***cc8baa98-9461-4225-866e-6b3e41c51556***
  management-certificate-path: /home/me/azure.pem

  storage-account-name: abcdefghijkl

![Instalar Juju](https://www.dropbox.com/s/qmsrnix31m5goko/eje6.2.png?dl=1)

Vamos a ejecutar los diferentes comandos para hacer un certificado.

  - openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout azure.pem -out azure.pem
  - openssl x509 -inform pem -in azure.pem -outform der -out azure.cer
  - chmod 600 azure.pem

Ahora tenemos que cargarlo en Azure, ***Configuración, Certificados de Administración y Cargar***, donde elegiremos nuestro azure.cer:

![azure.cer](https://www.dropbox.com/s/v3u4qmnnratom0r/06.10.png?dl=1)

![azure.cer](https://www.dropbox.com/s/7jl6n5x6nyi98hb/06.11.png?dl=1)

Ahora ajustamos de nuevo la configuración de environments.yml

![environments](https://www.dropbox.com/s/54stx62b13vwgln/06.14.png?dl=1)

Ejecutamos en el terminal:

***sudo juju switch azure***


![juju switch azure](https://www.dropbox.com/s/t8v0a8sgsuvv1br/06.15.png?dl=1)

***sudo juju bootstrap***

![sudo juju bootstrap](https://www.dropbox.com/s/nme1qb9n6ryynu3/06.16.png?dl=1)

***sudo juju deploy --to 0 juju-gui***

***sudo juju status***

![juju switch azure](https://www.dropbox.com/s/6mcsnq6d23ldztb/06.17.png?dl=1)

Si todo está correcto vemos la dirección pública que tenemos que poner en el navegador, en mi caso ***juju-azure-13akhf1jmn.cloudapp.net***

Nos muestra una imagen de login, que en debajo del formulario, podemos ver en dónde esta la contraseña.

Para instalar ahora nginx tenemos que ejecutar en el terminal,
`sudo juju deploy --to 0 cs:~hp-discover/trusty/nginx-4` y a continuación ``sudo expose nginx``


## Ejercicio 7.
### Instalar una máquina virtual con Linux Mint para el hipervisor que tengas instalado.

Vamos a utiliza VirtualBox y vamos a volver a instalar Linux Mint.

Abrimos VirtualBox y le damos a Nueva, ponemos el nombre, la configuración, tamaño en la memoria, disco duro, tipo de archivo de disco duro, almacenamiento, ubicación de archivo y tamaño.

![VirtualBox Nueva](https://www.dropbox.com/s/f9a74wjonrsep9i/7.0.png?dl=1)

Ahora iniciamos

![Corriendo](https://www.dropbox.com/s/rtkapkwur0ei0z5/7.1.png?dl=1)


Ahora vamos a realizar con Qemu.

Añadimos disco virtual: ***qemu-img create -f qcow2 linuxmint.qcow2 6G***

Ejecutamos ***qemu-system-x86_64 -hda linuxmint.qcow2 -cdrom ~/Escritorio/Curso15-16/IV/EjerTema5/ejer7/linuxmint-17.1-cinnamon-32bit.iso*** para llevar a cabo la instalación.

![comandos](https://www.dropbox.com/s/vw413w4t8w1qdhb/7.png?dl=1)
