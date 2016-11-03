Ejercicio 1 .
-------
* Instalamos chef:
![Imagen 1.1](https://www.dropbox.com/s/n1rzvo1ucv1nb51/1.1.png?dl=1)
 

Ejercicio 2 .
-------
* Creamos los directorios:

 ![Imagen 2.1](https://www.dropbox.com/s/88doiq7ikyvypzx/2.1.png?dl=1)

* Creamos el archivo *solo.rb* dentro de la carpeta *chef*:

 ![Imagen 2.2](https://www.dropbox.com/s/8v0rkeua5lwet8i/2.2.png?dl=1)

* Creamos el archivo *default.rb* tanto de nginx como del editor gedit dentro de la carpeta *recipes* correspondiente:
![Imagen 2.3](https://www.dropbox.com/s/m7vmvnwhdjod9sr/2.3.png?dl=1)

 ![Imagen 2.5](https://www.dropbox.com/s/g5pec5di3wkuh8v/2.5.png?dl=1)

* Creamos el archivo *node.json* dentro de la carpeta *chef*:
![Imagen 2.4](https://www.dropbox.com/s/wpi3blfr0b51276/2.4.png?dl=1)

* Ejecutamos y observamos que todo se instala:
![Imagen 2.6](https://www.dropbox.com/s/kolmc5yx3h4zv6s/2.6.png?dl=1)

Ejercicio 3 .
-------
~~~
--- 
- 
  tres: 
    - 4
    - 5
    - seis
    - 
      - 
        siete: 8
      - "nueve:[10,11]"
  uno: dos
~~~


Ejercicio 6 .
-------
* Instalamos vagrant:

 ![Imagen 6.1](https://www.dropbox.com/s/c3ecosf12059wrh/6.1.png?dl=1)

* Creamos la máquina con Debian:

 ![Imagen 6.2](https://www.dropbox.com/s/knmqloqaa2m1l6o/6.2.png?dl=1)

* Creamos el archivo Vagrantfile:

 ![Imagen 6.3](https://www.dropbox.com/s/qyzkfd93m1l9gdz/6.3.png?dl=1)

* Para inicializar la máquina virtual usaremos *vagrant up*.

* Para conectarnos usaremos *vagrant ssh*.


Ejercicio 7
-------
* Creamos el script:

 ![Imagen 7.1](https://www.dropbox.com/s/s4n4qt3l4hrkcul/7.1.png?dl=1)


Ejercicio 8
-------
* Añadimos al final del script anterior:

 ![Imagen 8.1](https://www.dropbox.com/s/oj11dfisilztdeg/8.1.png?dl=1)

 [Manual](https://www.vagrantup.com/docs/provisioning/ansible.html)
