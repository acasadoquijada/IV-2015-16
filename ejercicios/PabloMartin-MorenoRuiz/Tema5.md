# Uso de sistemas de virtualización

Estas prácticas las he desarrollado en el ordenador de un compañero de 64 bits dónde creamos otra cuenta

## Ejercicio 1. Instalar los paquetes necesarios para usar KVM. Se pueden seguir estas instrucciones. Ya lo hicimos en el primer tema, pero volver a comprobar si nuestro sistema está preparado para ejecutarlo o hay que conformarse con la paravirtualización.

Primero tenemos que instalar el siguiente paquete: sudo apt-get install cpu-checker

Ejecutamos el siguiente comando para comprobar comprobar que tenemos paravirtualización:

![comprobacionkvm](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_001_zpsfstijrg7.png)

Instalamos los siguientes paquetes para tener KVM:
```
sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils
```

Para crear máquinas virtuales instalamos el siguiente paquete:
```
sudo apt-get install virtinst
```

Ahora añado mi usuario a los siguientes grupos:

![grupos](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_002_zpshlro1hrw.png) 

Reinicio

Y ahora compruebo que esta bien instalado:

![funciona](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_003_zpsmhcjt3x4.png)

Podemos instalar una interfaz gráfica con el siguiente comando:
```
sudo apt-get install virt-manager
```
Y aquí la tenemos:

![InterfazGraficaQemu](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_004_zpsu9d4rfqe.png)

## Ejercicio 2.

### 1. Crear varias máquinas virtuales con algún sistema operativo libre tal como Linux o BSD. Si se quieren distribuciones que ocupen poco espacio con el objetivo principalmente de hacer pruebas se puede usar CoreOS (que sirve como soporte para Docker) GALPon Minino, hecha en Galicia para el mundo, Damn Small Linux, SliTaz (que cabe en 35 megas) y ttylinux (basado en línea de órdenes solo).

Lo primero que hare será crear una máquina de Ubuntu.

Para ello lo primero es crear espacio en disco para la máquina:

![EspacioUbuntu](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_006_zps4pnodmef.png)

Lo siguiente será instalar ubuntu usando una imágen descargada en el ordenador y el anterior disco creado:

![CrearUbuntu](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_007_zpsmawtimbv.png)

Y ya tenemos instalado ubuntu:

![UbuntuInstalado](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_008_zpssieux13u.png)

Lo siguiente que voy a hacer es instalar otro sistema operativo en este caso debian, repitiendo los mismos pasos hechos anteriormente:

Primero creamos espacio en disco para debian:

![CrearDiscoParaDebian](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_009_zpsuogwwbzv.png)

Lo siguiente es comenzar con la instalación:

![InstalaciónDebian](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_011_zpsbdjrmvkx.png)

Y por último procedemos a la instalación de debian:

![InstalandoDebian](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_010_zpscxiwkqzj.png)

Y cómo no solo nos interesa instalar máquinas virtuales sino que también nos interesa usarlas, lo hacemos con el siguiente comando:

![UsoMaquinas](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_012_zpsglaujcgl.png)

![MaquinaActiva](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_013_zpsjimofhv1.png)

### 2. Hacer un ejercicio equivalente usando otro hipervisor como Xen, VirtualBox o Parallels. 

En este apartado simplemente usaremos VirtualBox para instalar la misma imágen de Debian usada antes:

![VirtualBox1](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_014_zpsudmcwklq.png)

![VirtualBox2](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_015_zpsitxnxtxg.png)


## Ejercicio 3. Crear un benchmark de velocidad de entrada salida y comprobar la diferencia entre usar paravirtualización y arrancar la máquina virtual simplemente con qemu-system-x86_64 -hda /media/Backup/Isos/discovirtual.img

Usare el siguiente benchmark:

![benchmark](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_018_zpss2gvxk2e.png)

Y podemos la diferencia entre usar paravirtualización ``` qemu-system-x86_64 -boot order=c -drive file=ubuntu.qcow2,if=virtio ```

![Paravirtualización](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_017_zpsvliefwao.png)

y no usarla ``` qemu-system-x86_64 -hda ubuntu.qcow2 ```  :

![NoParavirtualización](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_019_zpschgbrben.png)

## Ejercicio 4. Crear una máquina virtual Linux con 512 megas de RAM y entorno gráfico LXDE a la que se pueda acceder mediante VNC y ssh.

Primero creamos espacio en disco:

![CrearDisco](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_021_zps4ebiztug.png)

Instalamos el sistema operativo:

![InstalamosLUbuntu](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_022_zpswnrfacgf.png)

Ejecutamos la máquina con el siguiente comando:

![Comando](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_023_zpsegjeq2l9.png)

Instalamos vinagre ``` sudo apt-get install vinagre ```

Y ahora ejecutamos el entorno gráfico:

![EntornoGráfico](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_024_zpskt9niaaa.png)

A continuación instalamos ssh ```sudo apt-get install ssh```

Arrancamos la máquina con NAT del el puerto 2222 de la máquina host al puerto 22 de la invitada:

![2222:22](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_025_zpspf71lz7g.png)

Y ahora le hacemos un ssh:

![ssh](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_026_zpsm87dxwif.png)

## Ejercicio 5. Crear una máquina virtual ubuntu e instalar en ella un servidor nginx para poder acceder mediante web.

Para este ejercicio he usado Koding.com que registrandote o logeandote con github te permite usar máquinas virtuales. Cómo mi ordenador es de 32 bits al intentar instalar azure-cli tenía muchos de problemas (de hecho, creo que no se puede), he usado Koding con una máquina de 64 bits.

Instalo azure-cli con los siguientes comandos:
```
sudo apt-get install nodejs-legacy
sudo apt-get install npm
sudo npm install -g azure-cli
```

Lo primero es hacer es loguearse en azure. Con el siguiente comando que nos dará una url y un código. En la web ya introducimos
nuestra cuenta y contraseña:

![azureLogin](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_008_zpsktl5psni.png)

Ejecutamos ```azure vm image list``` para ver las imagenes disponibles en azure:

![AzureList](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_009_zpss8k0l5ts.png)

Podemos ver información de la imagen con el siguiente comando:

![ShowImagen](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_011_zps33xnypm1.png)

Creamos la máquina virtual seleccionada:

![CrearMáquina](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_012_zpsftxunuq1.png)

Ahora iniciamos la máquina y nos logueamos con ssh:
```
azure vm start maquina-ubuntu-pmmre
ssh pablo@maquina-ubuntu-pmmre.cloudapp.net
```

![inicio_ssh](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_013_zpswv39ay87.png)

Ahora actualizamos repositorios y instalamos nginx:
```
sudo apt-get update
sudo apt-get install nginx
```
Y ahora comprobamos el estado de nginx y iniciamos:

![nginx_status](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_014_zpsruai39mv.png)

Abrimos el puerto 80 fuera del ssh

![abrirPuerto80](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_016_zpsxkalepby.png)

Vemos en el navegador que funciona nginx:

![PruebaNavegador](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_015_zpsx7on0uif.png)

## Ejercicio 7. Instalar una máquina virtual con Linux Mint para el hipervisor que tengas instalado.

Cómo hemos hecho anteriormente primero creamos el disco:

![ReservamosEspacio](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_027_zpsldvqnwjj.png)

Después iniciamos la instalación con la imágen descargada en el espacio creado:

![InstalamosLinuxMint](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_028_zps7yqxpmor.png)

Y finalmente aquí tenemos Linux Mint:

![FunionandoLinuxMint](http://i393.photobucket.com/albums/pp14/pmmre/IVEjercicios5y6/Seleccioacuten_029_zpsn1vafrjh.png) 