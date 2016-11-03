Ejercicio 1 .
-------
* Instalamos los paquetes necesarios de kvm:
![Imagen 1.1](https://www.dropbox.com/s/zbrtc3kpfg46gpd/1.1.png?dl=1)

* Comprobamos:

 ![Imagen 1.2](https://www.dropbox.com/s/ac4eyqk4a3p160k/1.2.png?dl=1)


Ejercicio 2 .
-------
* Creamos una máquina para Slitaz:
![Imagen 2.1](https://www.dropbox.com/s/90wjl2e46b21pvu/2.1.png?dl=1)

* Instalamos la imagen de Slitaz:
![Imagen 2.2](https://www.dropbox.com/s/8hbb9cjemzbxvqh/2.2.png?dl=1)

* La probamos:

 ![Imagen 2.3](https://www.dropbox.com/s/lbrqdy4seofixbb/2.3.png?dl=1)

* Creamos una máquina para Ártabros:
![Imagen 2.4](https://www.dropbox.com/s/qx4zblopn3lmpte/2.4.png?dl=1)

* Instalamos la imagen de Ártabros:
![Imagen 2.5](https://www.dropbox.com/s/lv4883pn6bf6ol7/2.5.png?dl=1)

* La probamos:

 ![Imagen 2.6](https://www.dropbox.com/s/u4oemry4kukk46g/2.6.png?dl=1)

* Instalamos Trisquel  en VirtualBox:
![Imagen 2.7](https://www.dropbox.com/s/bbjpkr8jvkrvwvz/2.7.png?dl=1)


Ejercicio 3 .
-------
* Creamos un programa para medir tiempos:
![Imagen 3.1](https://www.dropbox.com/s/gdkrnio84okv817/3.1.png?dl=1)

* Observamos que la paravirtualización (la segunda) tarda más:

 ![Imagen 3.2](https://www.dropbox.com/s/7uphcklaiavdeo6/3.2.png?dl=1)


Ejercicio 4 .
-------
* Creamos una máquina para Trisquel:
![Imagen 4.1](https://www.dropbox.com/s/o46a6ce32lhxxt0/4.1.png?dl=1)

* Comprobamos que funcione:
![Imagen 4.2](https://www.dropbox.com/s/tboxnssjj7ftsqk/4.2.png?dl=1)

* Ahora, la arrancamos mediante VNC:
![Imagen 4.3](https://www.dropbox.com/s/c2xkdsfzvlb0grn/4.3.png?dl=1)

* Instalamos vinagre para conectarnos:
![Imagen 4.4](https://www.dropbox.com/s/th47q7tadhuv0wb/4.4.png?dl=1)

* Para conectarse usamos *vinagre localhost:5900* (o 5901 para acceso como administrador).
![Imagen 4.4](https://www.dropbox.com/s/18cvsfw5s64i3ak/4.5.png?dl=1)
* Para conectarnos por ssh usamos *qemu-system-x86_64 -hda trisquel -m 512 -redir tcp:7777::8001* y *ssh: ssh -p 8001 trisquel@localhost*.

Ejercicio 5 .
-------
* Instalamos Azure:

 ![Imagen 5.1](https://www.dropbox.com/s/6tbmey4lq69ey9k/5.1.png?dl=1)

[Guía](https://azure.microsoft.com/es-es/documentation/articles/xplat-cli-install/)


Ejercicio 7 .
-------
* Creamos una máquina para Linux Mint:
![Imagen 7.1](https://www.dropbox.com/s/v0scqictv4koi6a/7.1.png?dl=1)

* Comprobamos que funcione:

 ![Imagen 7.2](https://www.dropbox.com/s/m37f9jjgw3mz3eh/7.2.png?dl=1)
