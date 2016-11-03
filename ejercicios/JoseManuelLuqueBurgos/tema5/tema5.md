# Ejercicios Tema 5

## Ejercicio 4
### Crear una máquina virtual Linux con 512 megas de RAM y entorno gráfico LXDE a la que se pueda acceder mediante VNC y ssh.

He elegido [debian](http://cdimage.debian.org/debian-cd/8.3.0/multi-arch/iso-cd/debian-8.3.0-amd64-i386-netinst.iso), através de su instalador podemos seleccionar lxde como entorno gráfico.

- Creamos el disco

  ```
   jose@tux2duo ⮀ ~ ⮀ qemu-img create -f qcow2 debian.img 10G

  ```
- Procedemos a instalarlo

  ```
  jose@tux2duo ⮀ ~ ⮀ qemu-system-x86_64 -hda debian.img -cdrom /home/jose/debian-8.3.0-amd64-CD-1.iso -m 512M
  ```

- Arrancamos el servicio VNC para qemu indicandole la salida (1)

  ```
  jose@tux2duo ⮀ ~ ⮀ qemu-system-i386 -hda debian.img vnc :1
  ```
- Por último nos conectamos con un cliente VNC al servicio

  ```
  jose@tux2duo ⮀ ~ ⮀ gtk-vnc localhost:5901
    ```
- Para hacerlo mediante SSH

  ```
  jose@tux2duo ⮀ ~ ⮀ qemu-system-x86_64 -boot order=c -drive file=debian.img,if=virtio -m 512M -name josedebian -redir tcp:2222::22
    ```

## Ejercicio 5
### Crear una máquina virtual ubuntu en Azure e instalar en ella un servidor nginx para poder acceder mediante web.

En primer lugar nos descargamos azure-cli através de npm

```
sudo npm install -g azure-cli
```

Nos descargamos el archivo para vincular nuestra cuenta

```
 jose@tux2duo ⮀ ~ ⮀ azure config mode asm
zsh: correct 'config' to '.config' [nyae]? n
info:    New mode is asm
 jose@tux2duo ⮀ ~ ⮀ azure account download
info:    Executing command account download
info:    Launching browser to http://go.microsoft.com/fwlink/?LinkId=254432
help:    Save the downloaded file, then execute the command
help:      account import <file>
info:    account download command OK
 jose@tux2duo ⮀ ~ ⮀

```

Y lo importamos

```
jose@tux2duo ⮀ ~ ⮀ azure account import /home/jose/Desktop/ivfinal/Azure\ Pass-
2-6-2016-credentials.publishsettings
info:    Executing command account import
info:    account import command OK
```

Ahora vamos a crear el sitio web, por alguna razón debe ser con la localización West US

```
azure site create --location "West US" pruebasite
```

Con ésto ya podemos acceder a él através de la URL [pruebasite.azurewebsites.net](http://pruebasite.azurewebsites.net)

El siguiente proceso podría hacerse mediante el cliente web, pero en este caso seguiremos usando el cliente de la línea de órdenes.

Vamos a elegir una imagen ubuntu de las disponibles en azure

```
 jose@tux2duo ⮀ ~ ⮀ azure vm image list westus ubuntuserver | more
info:    Executing command vm image list
info:    Fetching VM images
data:    Name                                                                   
                                               Category  OS       Publisher     

data:    -----------------------------------------------------------------------
---------------------------------------------  --------  -------  --------------
-------------------------------
data:    03f55de797f546a1b29d1b8d66be687a__CoreCLR-x64-Beta5-Linux-PartsUnlimite
d-Demo-App-201504.29                           Public    Linux    Microsoft Visu
al Studio Group                
data:    03f55de797f546a1b29d1b8d66be687a__Team-Foundation-Server-2013-Update4-W
S2012R2                                        Public    Windows  Microsoft Visu
al Studio Group                
data:    03f55de797f546a1b29d1b8d66be687a__Team-Foundation-Server-2013-Update4-W
S2012R2-201503.20                              Public    Windows  Microsoft Visu
al Studio Group                
data:    03f55de797f546a1b29d1b8d66be687a__Team-Foundation-Server-2013-Update4-W
S2012R2-201503.27                              Public    Windows  Microsoft Visu
al Studio Group                
data:    03f55de797f546a1b29d1b8d66be687a__VS-2013-Comm-VSU5-AzureSDK-2.8-WS2012
R2                                             Public    Windows  Microsoft Visu
al Studio Group                
data:    03f55de797f546a1b29d1b8d66be687a__VS-2013-Comm-VSU5-Cordova-CTP3.2-Azur
--More--


```

Creamos la máquina con el id de la imagen escogida

```
jose@tux2duo ⮀ ~ ⮀ azure vm create testvm1212 0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1
0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1 jose jose --location "North Europe" --ssh

```

Por último la lanzamos y nos conectamos por ssh

```
jose@tux2duo ⮀ ~ ⮀ azure vm start testvm
```

```
jose@tux2duo ⮀ ~ ⮀ ssh jose@testvm1212.cloudapp.net
```

Para instalar Nginx procedemos de la manera habitual tal y como lo hicimos en prácticas anteriores.

## Ejercicio 6: Usar juju para hacer el ejercicio anterior.

Ahora podremos observar como juju añade una capa de abstracción sobre azure, lo cual nos facilitará (tras configurarlo) las tareas sobre azure y podremos realizar los pasos del ejercicio 5 con menos cantidad de trabajo.
Los repositorios de linux están disponibles para Ubuntu y Mac por tanto he usado una máquina virtual para ahorrarme quebraderos de cabeza al instalar un paquete de ubuntu en Arch.

En primer lugar añadimos los repositorios de juju
```
    sudo add-apt-repository ppa:juju/stable
    sudo apt-get update && sudo apt-get install juju-core
```

Ahora creamos el archivo de configuración para Azure

```
jose@ubuntu:~$: juju init

```

Generamos los certificados necesarios para que juju se conecte a azure
```
    jose@ubuntu:~$: openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout azure.pem -out azure.pem
    jose@ubuntu:~$: openssl x509 -inform pem -in azure.pem -outform der -out azure.cer
    jose@ubuntu:~$: chmod 600 azure.pem
```

Necesitaremos la id de nuestra cuenta en azure,otra id de almacenamiento y la localización , para ello usamos

```
jose@ubuntu:~$: azure account list

```
El id de suscripción lo introducimos en **management-subscription-id** del envirouments.yml perteneciente a la configuración de juju.

El id de almacenamiento lo introducimos en **storage-account-name** del mismo archivo.

La última información requerida que nos proporciona el comando anterior es la localización, la situamos en el campo **location**.

En **management-certificate-path** establecemos la ruta hacia el archivo .pem generado anteriormente.


Ahora es el momento de añadir el certificado .cer en Azure, *Settings > Manage Certificates*


Ahora ya tenemos juju configurado para interactuar con nuestra cuenta de azure, es el momento de lanzar e instalar Nginx

- Cambiamos juju a azure

  ```
  jose@ubuntu:~$: juju switch azure
  ```
- Preparamos el **contenedor**

  ```
  jose@ubuntu:~$: juju bootstrap
  ```
- Instalamos Nginx

  ```
  jose@ubuntu:~$: juju deploy cs:~imbrandon/precise/nginx-7
  ```
- Lanzamos Nginx

  ```
  jose@ubuntu:~$: juju expose nginx
  ```
## Ejercicio 7
### Instalar una máquina virtual con Linux Mint para el hipervisor que tengas instalado.

Tras descargarnos la imagen de Linux Mint realizamos dos pasos:

- Creamos el disco que va a usar la máquina (qcow2 indica el tipo de imagen para la vm)

```
jose@tux2duo ⮀ ~ ⮀ qemu-img create -f qcow2 mymint.qcow2 8G
```

- Ahora procedemos a la instalación:

```
jose@tux2duo ⮀ ~ ⮀ qemu-system-x86_64 -machine accel=kvm -hda mymint.qcow2 -cdrom /home/jose/Desktop/linuxmint-17.3-cinnamon-64bit.iso -m 512M -boot d
```
