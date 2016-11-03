#Ejercicios - Jesús Prieto López
##TEMA 6

###Ejercicio 1: Instalar chef en la máquina virtual que vayamos a usar.

Voy a utilizar una máquina virtual que tenía creada con VMware Workstation. La máquina virtual tiene el SO Ubuntu Server 14.04 de 64 bits.

Podemos instalar **chef** con la siguiente orden:

`$ curl -L https://www.opscode.com/chef/install.sh | sudo bash`

Para comprobar que se ha instalado correctamente:

`$ chef-solo -v`

![Instalación de chef y versión instalada](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap1_zpsklxvq5mk.png)

###Ejercicio 2: Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.

Creamos las recetas para instalar **nginx** y el editor seleccionado, en este caso **gedit**. 

Preparamos los directorios necesarios para poder utilizar las recetas creadas para **chef**. Lo principal será un directorio llamado *chef* del que colgará un directorio *cookbooks* y este a su vez tendrá tantos directorios como recetas hagamos. Dentro de cada directorio de la receta deberá haber un directorio llamado *recipes*. Tal que así

	chef/cookbooks/nginx/recipes
	chef/cookbooks/gedit/recipes

Necesitamos configurar un archivo para cada uno que actuará como receta. Este archivo deberá indicarse con el nombre de **default.rb**.

- Receta nginx

```ruby
package 'nginx'
directory '/home/jes/PruebaRecetas/nginx'
file '/home/jes/PruebaRecetas/nginx/LEEME' do
	owner 'jes'
	group 'jes'
	mode 00544
	action :create
	content 'Directorio de nginx creado por receta'
end
```

- Receta gedit

```ruby
package 'gedit'
directory '/home/jes/PruebaRecetas/gedit'
file '/home/jes/PruebaRecetas/gedit/LEEME' do
	owner 'jes'
	group 'jes'
	mode 00544
	action :create
	content 'Directorio de gedit creado por receta'
end
```

Estos dos archivos, deberá ir cada uno en su carpeta correspondiente dentro de *recipes*.

Tenemos que crear ahora el fichero que indica las recetas que deseamos que se ejecuten. Este archivo deberá llamarse **node.json** y lo crearemos en el directorio *chef*. Al querer ejecutar las recetas creadas anteriores el archivo quedaría:

```
{
	"run_list":[ "recipe[nginx]", "recipe[gedit]"]
}
```

El archivo encargado de indicar las referencias a los archivos anteriores será **solo.rb**, teniendo que indicar el camino a los mismos para poder lanzarlos.

```ruby
file_cache_path "/home/jes/chef"
cookbook_path "/home/jes/chef/cookbooks"
json_attribs "/home/jes/chef/node.json"
```

Una vez hecho todo esto, podemos proceder a ejecutar nuestras recetas y comprobar el funcionamiento.

`$ chef-solo -c solo.rb`

![Ejecución de las recetas de chef](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap2_zpsx0j7ndxl.png)

###Ejercicio 3: Escribir en YAML la siguiente estructura de datos en JSON
`{ uno: "dos",  tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }`

Correspondiente en YAML:

```
- uno: "dos"
  tres:
    - 4
    - 5
    - "Seis"
    -
      - siete: 8
      - nueve:
        - 10
        - 11
```


###Ejercicio 4: Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.

Voy a utilizar la aplicación que realicé para algunos ejercicios de temas anteriores. La aplicación web está en [este repositorio](https://github.com/JesGor/califpy-empresa). Y como alternativa a Azure utilizaré [Koding](https://koding.com).

Procedemos al ejercicio. Instalamos ansible.

`$ pip install paramiko PyYAML jinja2 httplib2 ansible`

La máquina que voy a usar podemos encontrala con este enlace `jesgor.koding.io`. Esto nos servirá para añadirla al fichero inventario que utiliza ansible.

`$ echo "jesgor.koding.io" > ~/ansible_hosts`

Y ahora configuramos la variable de entorno de ansible para indicar el fichero.

`$ export ANSIBLE_HOSTS=~/ansible_hosts`

Creamos una clave ssh, por ejemplo de tipo RSA, con el comando:

`$ ssh-keygen -t rsa`

>Por defecto creará las claves en el directorio *~/.ssh* y con el nombre *id_rsa*. La que necesitamos es la .pub

Y añadimos la clave al *ssh-agent* para trabajar desde la terminal.

	$ eval "$(ssh-agent -s)"
	$ ssh-add id_rsa.pub

Debemos subir este archivo la máquina virtual y agregar su contenido al archivo **authorized_keys** dentro de la carpeta *~/.ssh*

![Configuración de la clave pública en la máquina virtual](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap15_zpsqowxijif.png)

Una vez configurado esto, verificamos que ansible tenga conexión a la máquina virtual creada en Koding.

`$ ansible all -u jesgor -m ping`

![Ping realizado desde ansible a la máquina virtual de koding](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap3_zpswan8edtb.png)

Antes de desplegar la aplicación necesitamos instalar lo necesario. En este caso python y las dependencias para poder ejecutarla.

	ansible all -u jesgor -m command -a "sudo apt-get update"
	ansible all -u jesgor -m command -a "sudo apt-get install -y python3-setuptools python3-dev build-essential libpq-dev"
	ansible all -u jesgor -m command -a "sudo easy_install3 pip"

Descargamos en la máquina virtual, mediante ansible, la aplicación desde el enlace de nuestro repositorio.

`$ ansible all -u jesgor -m git -a "repo=https://github.com/JesGor/califpy-empresa.git dest=~/califpy-empresa version=HEAD"`

![Descarga de la aplicación del repositorio con ansible](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap4_zps8fqchasq.png)

Ahora podemos instalar las dependencias.

`$ ansible all -u jesgor -m command -a "sudo pip3 install -r ~/califpy-empresa/web/requirements.txt"`

Y ejecutamos la aplicación

`$ ansible all -u jesgor -m command -a "sudo python3 ~/califpy-empresa/web/manage.py runserver 0.0.0.0:80"`

Y si nos conectamos desde el navegador a la dirección de la máquina virtual podemos ver que funciona.

![Aplicación web funcionando en koding desplegada a través de ansible](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap5_zps16ix0d74.png)


###Ejercicio 5:
####Ejercicio 5.1: Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.

Voy a utilizar la aplicación del ejercicio anterior, y voy a hacer sobre Koding también. Para realizar este ejercicio, como bien indica, hay que crearse un archivo playbook que será la receta para nuestro objetivo.

Para este caso, quizás sería recomendable indicar en el archivo **ansible_hosts** la dirección de la máquina virtual como antes pero acompañada de un nombre, al cual luego se hará referencia en el playbook. Deberíamos insertar en el archivo lo siguiente:

	[koding]
	jesgor.koding.io

Tenemos que crearnos un archivo *.yml*, con el nombre que deseemos, yo le he puesto *prueba.yml*. Y dentro del archivo indicar el host objetivo y las operaciones a realizar.

Para conseguir algo como en el ejercicio anterior mediante el playbook, el contenido quedaría tal que así:

```
- hosts: koding
  sudo: yes
  remote_user: jesgor
  tasks:
  - name: Instalar python3
    apt: name=python3-setuptools state=present
    apt: name=python3-dev state=present
    apt: name=build-essential state=present
    apt: name=python3-pip state=present
    apt: name=libpq-dev state=present
  - name: Descargar repositorio
    git: repo=https://github.com/JesGor/califpy-empresa.git dest=~/califpy-empresa-playbook clone=yes force=yes
  - name: Instalar dependencias
    command: sudo pip install -r ~/califpy-empresa-playbook/web/requirements.txt
  - name: Ejecutar app
    command: sudo python3 ~/califpy-empresa-playbook/web/manage.py runserver 0.0.0.0:80

```

Cuando tengamos creado el playbook lo ejecutamos.

`$ ansible-playbook -u jesgor prueba.yml`

![Ejecución del playbook con ansible](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap6_zpsebxrjxph.png)

Podemos ver que todos los pasos los ha completado exitosamente y se queda parado en el último (ejecución). Desde el navegador comprobamos conectandonos a la dirección si de verdad ha funcionado todo.

![Ejecución de la aplicación desplegada mediante playbook](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap7_zpsxpyuuozx.png)

####Ejercicio 5.2:
¿Ansible o Chef? ¿O cualquier otro que no hemos usado aquí?.

Personalmente prefiero Ansible, es menos "engorroso" a la hora de crear recetas. He aprendido y lo he usado de forma más rápida que Chef. 

Chef permite menos agilidad al tener que crear un árbol de directorios para las recetas, con ansible podemos tenerlo todo en un solo archivo y además nos permite que esas recetas se realicen remotamente.


###Ejercicio 6: Instalar una máquina virtual Debian usando Vagrant y conectar con ella.

Instalamos Vagrant para realizar el ejercicio.

`$ apt-get install vagrant`

He elegido, de la [esta lista](www.vagrantbox.es), una imagen de Ubuntu, concretamente [esta](https://github.com/kraksoft/vagrant-box-ubuntu/releases/download/14.04/ubuntu-14.04-amd64.box). Para descargarla he utilizado el siguiente comando:

`$ vagrant box add ubuntu https://github.com/kraksoft/vagrant-box-ubuntu/releases/download/14.04/ubuntu-14.04-amd64.box`

Después creamos el fichero *Vagrantfile* para poder trabajar con esta imagen e iniciamos la máquina virtual, para luego conectarnos a ella mediante ssh.

	$ vagrant init ubuntu
	$ vagrant up
	$ vagrant ssh

![Conexión a máquina virtual creada con vagrant](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap8_zpseifyhnqp.png)


### Ejercicio 7: Crear un script para provisionar nginx o cualquier otro servidor web que pueda ser útil para alguna otra práctica.

He modificado el archivo *Vagrantfile* generado en el ejercicio anterior y le he añadido las ordenes necesarias para proveerlo de *nginx*. El contenido del archivo es el siguiente:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu"
  config.vm.provision "shell",
  inline: "sudo apt-get update && sudo apt-get install -y nginx && sudo service nginx start"
end

```

Después he ejecutado el provisionamiento.

`$ vagrant provision`

Y he comprobado que se hiciera todo correctamente conectandome a ella y comprobando el servicio.

![Provosionado de nginx para la máquina virtual con Ubuntu mediante vagrant](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap9_zpsnlmmq6ak.png)


###Ejercicio 8: Ejercicio 8: Configurar tu máquina virtual usando vagrant con el provisionador ansible.

Voy a configurar la máquina virtual del ejercicio anterior provisinándola con ansible a través de vagrant. Tenemos que utilizar un archivo *.yml*, usaré el que usé en el ejercicio 5 pero con algunas modificaciones: cambiamos *hosts* a *all*, quitamos *remote_user* y cambiamos el puerto en la ejecución de la aplicación.

```
- hosts: all
  sudo: yes
  tasks:
  - name: Instalar python3 y git
    apt: name=python3-setuptools state=present
    apt: name=python3-dev state=present
    apt: name=build-essential state=present
    apt: name=libpq-dev state=present
    apt: name=git state=present
    apt: name=python3-pip state=present
  - name: Descargar repositorio
    git: repo=https://github.com/JesGor/califpy-empresa.git dest=~/califpy-empresa-playbook clone=yes force=yes
  - name: Instalar dependencias
    command: sudo pip3 install -r ~/califpy-empresa-playbook/web/requirements.txt
  - name: Ejecutar app
    command: sudo python3 ~/califpy-empresa-playbook/web/manage.py runserver 0.0.0.0:8000 
```

> Si ponemos la opción *hosts: all* hay que tener cuidado con lo que hay en el archivo *ansible_hosts* ya que ejecutará todos los hosts que tengamos en él.

Tenemos que cambiar el archivo *ansible_hosts* también, con el siguiente contenido:

```
[localhost]
localhost           ansible_connection=local
```

Y utilizar la variable de entorno para indicar el fichero.

`$ export ANSIBLE_HOSTS=~/ansible_hosts`

Modificamos el archivo *Vagrantfile* para indicar que el provisionamiento se realizará a través de ansible, y además vamos a indicar una ip privada manualmente, para poder luego conectarnos a ella.

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "ubuntu"
  config.vm.network "private_network",ip: "192.168.33.10"
  
  config.vm.provision "ansible" do |ansible|
  	ansible.playbook = "prueba.yml"
  end
end
```

Tan solo queda ejecutar el siguiente comando:

`$ vagrant provision`

Y ya tendremos a nuestra máquina virtual provisionada como hemos indicado.
	
![Ejecutando tareas de provisionamiento](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap10_zpsqhj7g2p1.png)

> Si ves que tienes problemas porque da fallo y ni te cambia la ip de la máquina virtual, utiliza el siguiente comando: `$ vagrant reload` y vuelve a ejecutar el `$ vagrant provision`

Comprobamos desde el navegador, introduciendo la ip que configuramos en el *Vagrantfile* que podemos acceder a la aplicación.

![Aplicación ejecutandose en máquina virtual provisionada con vagrant y ansible](http://i1175.photobucket.com/albums/r628/jesusgorillo/cap11_zpsds1gwwkj.png)





