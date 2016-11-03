#Tema 5

##Ejercicios

###Ejercicio 1
**Instalar los paquetes necesarios para usar KVM. Se pueden seguir estas instrucciones. Ya lo hicimos en el primer tema, pero volver a comprobar si nuestro sistema está preparado para ejecutarlo o hay que conformarse con la paravirtualización.**
Compruebo que mi sistema está preparado para virtualizar a nivel de hardware.
El modelo de mi procesador es el [Intel Core i5 M430 @2.27GHz](http://ark.intel.com/products/43537/Intel-Core-i5-430M-Processor-3M-Cache-2_26-GHz). Lo he consultado con el comando *cat /proc/cpuinfo*, el cual muestra la siguiente información:

![Modelo de procesador](http://i1016.photobucket.com/albums/af281/raperaco/cpuinfo_zpsjbqaxxjk.png)

La salida que aparece al ejecutar la orden egrep '^flags.*(vmx|svm)' /proc/cpuinfo es:
![Orden flags virtualización](http://i1016.photobucket.com/albums/af281/raperaco/salidaFlagsVirtualizacion_zpscl5z8qgh.png)

*Aparece la misma salida repetida cuatro veces, porque es la recogida en cada uno de los "cores"*.
Vemos como efectivamente aparece el flag **vmx**, por lo que indica que mi procesador está preparado para la virtualización hardware.
Otra opción para saber si nuestro procesador soporta virtualización hardware es:
~~~
kvm-ok
~~~

![kvm-ok](http://i1016.photobucket.com/albums/af281/raperaco/kvm-ok_zpsqnlggvxm.png)


Para instalar KVM he seguido las [intrucciones de la página de Ubuntu](https://help.ubuntu.com/community/KVM/Installation) actualizada a día 20-9-2015.

* Instalar los siguientes paquetes:
~~~
sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils
~~~
Una descripción sobre para qué sirve cada paquete se puede encontrar en las [intrucciones de la página de Ubuntu](https://help.ubuntu.com/community/KVM/Installation).

* Como voy a crear máquinas virtuales desde linea de comandos, también hay que instalar **virtinst**.
~~~
sudo apt-get install virtinst
~~~

* Para administrar máquinas virtuales con usuarios normales, hay que añadir estos usuarios a los grupos *kvm* y *libvirt*. En mi caso:
~~~
adduser alex kvm
adduser alex libvirtd
~~~

* Para que el paso anterior tenga efecto, hay que *reloguearse* en el sistema con dicho usuario.

* Verificamos la instalación:
~~~
virsh -c qemu:///system list
~~~

![verificarInstalacion](http://i1016.photobucket.com/albums/af281/raperaco/verificarInstalacion_zpsvva9gbnk.png)

*Si se tienen errores en este paso, en las [intrucciones de la página de Ubuntu](https://help.ubuntu.com/community/KVM/Installation) se recoge como solucionar algunos de los posibles errores.*

* Si queremos tener una interfaz gráfica:
~~~
sudo apt-get install virt-manager
~~~
En las [intrucciones de la página de Ubuntu](https://help.ubuntu.com/community/KVM/Installation) también se recoge cómo usar dicha interfaz.

Tras abrirla, probablemente informe que necesita algunos paquetes adicionales por lo que los descargamos.

![virt-manager](http://i1016.photobucket.com/albums/af281/raperaco/virt-manager_zpssexgtazr.png)

Una vez finalizada la instalación, ya tenemos la interfaz lista para su uso:

![virt-manager1](http://i1016.photobucket.com/albums/af281/raperaco/virt-manager1_zps6o82yntt.png)


###Ejercicio 2
**1. Crear varias máquinas virtuales con algún sistema operativo libre tal como Linux o BSD. Si se quieren distribuciones que ocupen poco espacio con el objetivo principalmente de hacer pruebas se puede usar [CoreOS](https://coreos.com/) (que sirve como soporte para Docker), [GALPon Minino](http://minino.galpon.org/en), hecha en Galicia para el mundo, [Damn Small Linux](http://www.damnsmalllinux.org/download.html), [SliTaz](http://www.slitaz.org/en/) (que cabe en 35 megas) y [ttylinux](http://ttylinux.net/) (basado en línea de órdenes solo).**
* **SliTaz**
	* Creo el disco duro:
	~~~
    $ qemu-img create -f qcow2 slitaz.qcow2 8G
    ~~~

	* Creo la máquina virtual con la imagen de SliTaz:
	~~~
    $ qemu-system-x86_64 -machine accel=kvm -hda slitaz.qcow2 -cdrom /home/alex/Escritorio/slitaz-4.0.iso -boot d
    ~~~
    ![slitaz_install](http://i1016.photobucket.com/albums/af281/raperaco/slitaz_install_zpse5npnmnb.png)
    ![slitaz_install_finish](http://i1016.photobucket.com/albums/af281/raperaco/coreos_install_finish_zpsjhgdmgxm.png)
    
* **CoreOS**
	* Creo el disco duro:
	~~~
    $ qemu-img create -f qcow2 coreos.qcow2 8G
    ~~~
    
    * Creo la máquina virtual con la imagen de CoreOS e indicarle que tiene una memoria de 1G:
	~~~
    $ qemu-system-x86_64 -machine accel=kvm -hda coreos.qcow2 -cdrom /home/alex/Escritorio/coreos_production_iso_image.iso -m 1G -boot d
    ~~~
    ![coreos_install](http://i1016.photobucket.com/albums/af281/raperaco/coreos_install_zps8tfbnfmq.png)
    ![coreos_install_finish](http://i1016.photobucket.com/albums/af281/raperaco/coreos_install_finish_zpsjhgdmgxm.png)
    
**2. Hacer un ejercicio equivalente usando otro hipervisor como Xen, VirtualBox o
Parallels.**
Usaré VirtualBox e instalaré una imagen de SliTaz:
![slitaz_virtualbox](http://i1016.photobucket.com/albums/af281/raperaco/slitaz_virtualbox_zps0xgjnghz.png)
![slitaz_virtualbox_finish](http://i1016.photobucket.com/albums/af281/raperaco/slitaz_virtualbox_finish_zpslgcyams6.png)


###Ejercicio 3
**Crear un benchmark de velocidad de entrada salida y comprobar la diferencia entre usar paravirtualización y arrancar la máquina virtual simplemente con qemu-system-x86_64 -hda /media/Backup/Isos/discovirtual.img
**
Para realizar el testeo de entrada salida entre las dos posibles opciones, me he basado en el uso de la herramienta **dd** y en su explicación de la [wiki de Archlinux](https://wiki.archlinux.org/index.php/Benchmarking/Data_storage_devices)

La orden a usar para el testeo ha sido:
~~~
$ dd if=/dev/zero of=./temporal bs=1M count=1024 conv=fdatasync,notrunc
~~~
que devuelve el tiempo empleado en la lectura y escritura (copia) de 1.1GB de datos. 

* Sin usar la paravirtualización, los resultados obtenidos han sido:
	![benchmarkSINVirtio](http://i1016.photobucket.com/albums/af281/raperaco/benchmarkSINVirtio_zpsqwliqhnr.png)

* Usando la paravirtualización (virtio), los resultados obtenidos han sido:
	![benchmarkCONVirtio](http://i1016.photobucket.com/albums/af281/raperaco/benchmarkCONVirtio_zpseyrnl7g5.png)

Como puede verse, cuando se ha usado la paravirtualización se han transmitido 19.9 MB/s frente a los 5.1 MB/s que se transmitían sin la paravirtualización.


###Ejercicio 5
**Crear una máquina virtual ubuntu e instalar en ella un servidor nginx para poder acceder mediante web.**
1. Instalar el cliente de Azure para línea de comandos:
~~~
sudo apt-get install npm
sudo npm install -g azure-cli
~~~
2. Nos logueamos con nuestra cuenta en Azure:
~~~
azure login
~~~
Nos devolverá un código y una URL en la cual insertarlo para realizar el login.
![azureLogin](http://i1016.photobucket.com/albums/af281/raperaco/azureLogin_zpsnevylalg.png)
3. Listar todas las imágenes disponibles para ver cual usar. Cómo voy a instalar Ubuntu, veo las posibles imágenes de Ubuntu y selecciono la más reciente. (ya que el listado es muy extenso he optado por redirigir la salida a un fichero de texto plano en el cual buscar más fácilmente las imágenes de Ubuntu) 
~~~
azure vm image list > temporal
~~~
4. Me decido a usar la imagen *0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1 *
5. Creo la máquina virtual indicando el nombre de ella, la imagen a usar en la instalación, el superusuario y su clave (longitud a partir de 8 caracteres y que contenga mayúsculas,minúsculas,números y caracteres especiales). Le indico localización *Central US* (porque con la de Europa daba un error) y por último le indico que tendrá acceso por SSH.
~~~
azure vm create azureVM-ubuntu14-alex 0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1 alex PasswordAlex2016! --location "Central US" --ssh
~~~
![azureVMCreate](http://i1016.photobucket.com/albums/af281/raperaco/azureVMCreate_zpsbshd02eo.png)
6. Arrancar la máquina
~~~
azure vm start azureVM-ubuntu14-alex
~~~
7. Conectarnos a la máquina por SSH:
~~~
ssh alex@azureVM-ubuntu14-alex.cloudapp.net
~~~
![azureVMStartSSH](http://i1016.photobucket.com/albums/af281/raperaco/azureVMStartSSH_zpsiovdjehl.png)
8. Actualizar los repositorios de la imagen instalada:
~~~
sudo apt-get update
~~~
9. Instalar NGINX:
~~~
sudo apt-get install nginx
~~~
10. Abrir el puerto 80:
~~~
azure vm endpoint create azureVM-ubuntu14-alex 80 80
~~~
![azurePort80](http://s1016.photobucket.com/user/raperaco/media/azurePort80_zpsipzkm6il.png.html)
11. Listo los puertos y veo que ya aparece el puerto 80:
~~~
azure vm endpoint list azureVM-ubuntu14-alex
~~~
![azureListPorts](http://i1016.photobucket.com/albums/af281/raperaco/azureListPorts_zps5uilimtd.png)
12. En el navegador insertamos la URL de nuestra máquina. En mi caso **http://azurevm-ubuntu14-alex.cloudapp.net/** y obtenemos respuesta del servidor NGINX
![azureNginx](http://i1016.photobucket.com/albums/af281/raperaco/azureNginx_zpso7i6qu0d.png)
13. Apagar la máquina cuando no se vaya a utilizar más:
~~~
azure vm shutdown azureVM-ubuntu14-alex
~~~


###Ejercicio 7
**Instalar una máquina virtual con Linux Mint para el hipervisor que tengas instalado.**
Me dispongo a instalarla usando VMWare.

Para ello me descargo la [imagen de Linux Mint 17.3 64 bit](http://www.linuxmint.com/edition.php?id=204)

La instalo en VMWare con el procedimiento habitual.
![VMWareMint](http://i1016.photobucket.com/albums/af281/raperaco/linuxMintVMWare_zps3ih4xhye.png)
