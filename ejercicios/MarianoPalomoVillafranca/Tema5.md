Tema 5
===
> Mariano Palomo Villafranca

#### Ejercicio 1. Instalar los paquetes necesarios para usar KVM. Se pueden seguir estas instrucciones. Ya lo hicimos en el primer tema, pero volver a comprobar si nuestro sistema está preparado para ejecutarlo o hay que conformarse con la paravirtualización.
Como hicimos en el primer tema, basta con ejecutar el comando `kvm-ok` para comprobar la compatibilidad.

![](http://i.imgur.com/Tms7Qvk.png?1)

A continuación, seguimos los pasos de la [página para la instalación de KVM](https://help.ubuntu.com/community/KVM/Installation) para nuestro sistema, en mi caso, Ubuntu. En primer lugar ejecutamos el comando siguiente para instalar los paquetes necesarios:

```
$ sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils
```

Comprobamos que la instalación ha ido correctamente con:

```
$ virsh -c qemu:///system list
 Id Name                 State
----------------------------------
```

Además, instalamos una interfaz gráfica para el manejo de las máquinas virtuales:

```
$ sudo apt-get install virt-manager
```

#### Ejercicio 2
##### 2.1. Crear varias máquinas virtuales con algún sistema operativo libre tal como Linux o BSD. Si se quieren distribuciones que ocupen poco espacio con el objetivo principalmente de hacer pruebas se puede usar CoreOS (que sirve como soporte para Docker) GALPon Minino, hecha en Galicia para el mundo, Damn Small Linux, SliTaz (que cabe en 35 megas) y ttylinux (basado en línea de órdenes solo).

- CoreOS:

En el caso de CoreOS, la propia [web](https://coreos.com/os/docs/latest/booting-with-qemu.html) nos provee de un script que automatiza todo el proceso, por lo que bastará con hacer lo siguiente:

```
$ mkdir coreos; cd coreos
$ wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_qemu.sh
$ wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_qemu.sh.sig
$ wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_qemu_image.img.bz2
$ wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_qemu_image.img.bz2.sig
$ gpg --verify coreos_production_qemu.sh.sig
$ gpg --verify coreos_production_qemu_image.img.bz2.sig
$ bzip2 -d coreos_production_qemu_image.img.bz2
$ chmod +x coreos_production_qemu.sh
```

A continuación, y asegurándonos de que nuestra clave ssh está añadida con `ssh-add`, ejecutamos:

```
$ ./coreos_production_qemu.sh -a ~/.ssh/authorized_keys -- -nographic
```

Y ya podemos acceder a la máquina con:

```
$ ssh -l core -p 2222 localhost
```

![](http://i.imgur.com/GNgJKvk.png)

- SliTaz:

En primer lugar, descargamos la ISO desde su [página web](http://www.slitaz.org/en/get/). Una vez descargada, creamos un disco virtual con la orden:

```
$ qemu-img create -f qcow2 slitaz.qcow2 500M
```

Tras esto, iniciamos el sistema con:

```
$ qemu-system-x86_64 -machine accel=kvm -hda slitaz.qcow2 -cdrom slitaz-4.0.iso -m 256M -boot d
```

![](http://i.imgur.com/G4PS8vT.png)

##### 2.2. Hacer un ejercicio equivalente usando otro hipervisor como Xen, VirtualBox o Parallels.
Probamos a cargar SliTaz, por ser este el más liviano, en Parallels. Para ello, iniciamos Parallels Desktop y seleccionamos "Archivo>Nuevo" para crear una nueva máquina virtual. Marcamos la opción de imagen de disco y arrastramos la ISO a la ventana del programa. Nos avisará de que no se ha podido reconocer la distribución, por lo que seleccionaremos que se trata de "Otros Linux>Otro Linux". Tras esto, configuramos el disco, CPU y RAM a asignar.

![](http://i.imgur.com/NyViKYK.png)

Y por último, iniciamos la máquina.

![](http://i.imgur.com/GMgqGWE.png)

#### Ejercicio 3. Crear un benchmark de velocidad de entrada salida y comprobar la diferencia entre usar paravirtualización y arrancar la máquina virtual simplemente con `qemu-system-x86_64 -hda /media/Backup/Isos/discovirtual.img`.

#### Ejercicio 4. Crear una máquina virtual Linux con 512 megas de RAM y entorno gráfico LXDE a la que se pueda acceder mediante VNC y ssh.

Instalaremos Lubuntu que, por defecto, viene con entorno gráfico LXDE. Para ello, descargamos la ISO correspondiente desde [aquí](https://help.ubuntu.com/community/Lubuntu/GetLubuntu#Lubuntu_LTS).

Una vez descargada, creamos el espacio de almacenamiento:

```
$ qemu-img create -f qcow2 lubuntu.img 8G
```

E iniciamos la instalación con:

```
$ qemu-system-x86_64 -hda lubuntu.img -cdrom lubuntu-15.10-desktop-amd64.iso -m 512M
```

Una vez instalado el sistema, indicamos que queremos arrancar la máquina utilizando vnc:

```
$ qemu-system-x86_64 -machine accel=kvm -hda lubuntu.qcow2 -m 512M -vnc :1
```

Y, ahora, desde el cliente `vinagre`, que podemos instalar con `sudo apt-get install vinagre`, nos conectamos a la máquina:

```
$ vinagre localhost:1
```

![](http://i.imgur.com/g04On8N.png)

Ahora, para acceder mediante SSH, arrancamos la máquina, pero haciendo NAT de los puertos 2222 de la host a la 22 de la virtual.

```
$ qemu-system-x86_64 -machine accel=kvm -hda lubuntu.img -m 512M -redir tcp:2222::22
```

Y accedemos con:

```
$ ssh -p 2222 mpvillafranca@localhost
```

#### Ejercicio 5. Crear una máquina virtual ubuntu e instalar en ella un servidor nginx para poder acceder mediante web.
Podemos realizar esta tarea mediante el panel de control web de Azure, pero en este caso lo haremos mediante terminal. En primer lugar, descargamos e instalamos el cliente de Azure:

```
sudo apt-get install nodejs-legacy
sudo apt-get install npm
sudo npm install -g azure-cli
```

Hacemos login en el cliente con `azure login`, lo cual nos da una dirección web y un código. Accedemos a la web y tras introducir el código y seleccionar la cuenta correspondiente de microsoft, ya estaremos logeados.

Buscamos qué imágenes de Ubuntu se encuentran disponibles con `azure vm image list | grep Ubuntu` y seleccionamos ver más detalles de la de Ubuntu 14.04 con:

```
azure vm image show b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_3-LTS-amd64-server-20160201-en-us-30GB
```
![](http://i.imgur.com/BAUIc7X.png)

Ahora, pasamos a instalarla con:

```
$ azure vm create mpvillafranca-t5-ej5 b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_3-LTS-amd64-server-20160201-en-us-30GB mpvillafranca MpvillafrancaClave --location "East Asia" --ssh
```

Una vez completado, la iniciamos y accedemos por ssh:

```
$ azure vm start mpvillafranca-t5-ej5
$ ssh mpvillafranca@mpvillafranca-t5-ej5.cloudapp.net
```

Ahora actualizamos el sistema e instalamos nginx:

```
$ sudo apt-get update
$ sudo apt-get install nginx
$ sudo service nginx start
```

Y, en otro terminal, abrimos el puerto 80 de la máquina para acceder por web:

```
$ azure vm endpoint create mpvillafranca-t5-ej5 80 80
```

![](http://i.imgur.com/bEGS26L.png)

#### Ejercicio 6. Usar juju para hacer el ejercicio anterior

Seguiremos los pasos indicados en la [web del paquete](https://jujucharms.com/docs/stable/getting-started). En primer lugar, instalamos Juju:

```
$ sudo add-apt-repository ppa:juju/stable
$ sudo apt-get update && sudo apt-get install juju-core
```

Y lo iniciamos con:

```
$ juju init
```

Si queremos revisar la configuración, la podemos encontrar en el archivo `/home/mpvillafranca/.juju/environments.yaml`.

Ahora, crearemos el certificado necesario para trabajar en Azure con juju.

```
$ openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout azure.pem -out azure.pem
$ openssl x509 -inform pem -in azure.pem -outform der -out azure.cer
$ chmod 600 azure.pem
```

Editamos la configuración de juju con los siguientes valores:

- `location`: la localización del espacio de almacenamiento. Se consigue también con `azure storage account list`.
- `management-subscription-id`: el ID de nuestra suscripción. Lo podemos consultar con `azure account list`.
- `management-certificate-path`: el camino hasta el fichero "azure.pem". En mi caso `/home/mpvillafranca/azure.pem`.
- `storage-account-name`: el nombre de la cuenta de almacenamiento. Lo obtenemos con `azure storage account list`.

Ahora, accedemos al [panel de administración de Azure](manage.windowsazure.com) para subir el certificado. Nos vamos a la opción Configuración > Certificados de administración > Cargar y seleccionamos el archivo `azure.cer`.

Ahora, indicamos a juju que queremos trabajar con azure:

```
$ juju switch azure
```

Y creamos el entrono Bootstrap que juju utiliza para manejar los servicios:

```
$ juju deploy cs:~imbrandon/precise/nginx-7
```

Por último, publicamos el servicio y accedemos a la web que nos indica `juju status`:

```
$ juju expose nginx
```
![](http://i.imgur.com/ptu3XGg.png)

#### Ejercicio 7. Instalar una máquina virtual con Linux Mint para el hipervisor que tengas instalado.

En primer lugar, descargamos la imagen de Linux Mint desde [aquí](http://www.linuxmint.com/download.php).

Una vez descargada, creamos el espacio de almacenamiento con:

```
$ qemu-img create -f qcow2 linuxmint.qcow2 8G
```

E instalamos el sistema con:

```
$ qemu-system-x86_64 -machine accel=kvm -hda linuxmint.qcow2 -cdrom linuxmint-17.3-cinnamon-64bit.iso -m 500M -boot d
```
