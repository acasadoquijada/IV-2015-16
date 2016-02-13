# Ejercicios Tema 6

## Ejercicio 1
**Instalar chef en la máquina virtual que vayamos a usar.**

Primero, vamos a conectarnos a nuestra máquina virtual de Azure, con ```azure login```, arrancamos la máquina y nos conectamos por ssh con:

```
jesmorc@jesmorc-PClaptop ~/Workinout $ sudo vagrant ssh
[sudo] password for jesmorc: 
==> localhost: Attempting to read state for iv-jesmorc-ubuntuserver in iv-jesmorc-ubuntuserver-service-euemq
==> localhost: VM Status: ReadyRole
==> localhost: Attempting to read state for iv-jesmorc-ubuntuserver in iv-jesmorc-ubuntuserver-service-euemq
==> localhost: VM Status: ReadyRole
    localhost: Looking for local port 22
    localhost: Found port mapping 22 --> 22
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.16.0-37-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Sat Feb 13 17:54:21 UTC 2016

  System load:  0.48              Processes:           232
  Usage of /:   4.3% of 28.80GB   Users logged in:     0
  Memory usage: 9%                IP address for eth0: 100.77.224.46
  Swap usage:   0%

  Graph this data and manage this system at:
    https://landscape.canonical.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud


*** System restart required ***
Last login: Sat Feb 13 17:54:22 2016 from 217.216.81.186.dyn.user.ono.com
vagrant@iv-jesmorc-ubuntuserver:~$ 
```


Dentro de la máquina, instalamos *Chef* con:
```
curl -L https://www.opscode.com/chef/install.sh | sudo bash
```

Esto descargará **chef-solo**, una versión aislada para trabajar en una máquina desde la misma.

Para comprobar la versión instalada:
```
vagrant@iv-jesmorc-ubuntuserver:~$ chef-solo -v
Chef: 12.7.2
```

## Ejercicio 2
**Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.**

Creamos los directorios donde irán las recetas para instalar nginx y el editor nano:

Desde el *home* :

```
vagrant@iv-jesmorc-ubuntuserver:~$ mkdir -p chef/cookbooks/nginx/recipes
vagrant@iv-jesmorc-ubuntuserver:~$ mkdir -p chef/cookbooks/nano/recipes

```

Ahora, vamos a configurar los ficheros que contendrán las recetas para instalar "nginx" y "nano".

[Tutorial Server chef-solo](http://www.mechanicalfish.net/configure-a-server-with-chef-solo-in-five-minutes/).

El fichero que contendrá la receta de instalación se llamará *default.rb*, que existirá uno en cada uno de los directorios creados antes.
Esto contiene ordenes que  instalarán el paquete especificado, creará un directorio para documentos y dentro de él un fichero que explica de qué se trata.

- Receta **default.rb** para nginx:

```
package 'nginx'
directory "/home/jesmorc/Documentos/nginx"
file "/home/jesmorc/Documentos/nginx/LEEME" do
	owner "jesmorc"
	group "jesmorc"
	mode 00544
	action :create
	content "Directorio para nginx"
end
```

- Receta **default.rb** para nano:

```
package 'nano'
directory "/home/jesmorc/Documentos/nano"
file "/home/jesmorc/Documentos/nano/LEEME" do
	owner "jesmorc"
	group "jesmorc"
	mode 00544
	action :create
	content "Directorio para nano"
end
```


Creamos directorios necesarios:
```
  mkdir -p Documentos/nano
  mkdir -p Documentos/nginx
```

El siguiente paso es crear un fichero llamado **node.json**, que irá en el directorio *chef* y va a tener la lista de recetas a ejecutar. Contiene:

```
{
        "run_list":["recipe[nginx]", "recipe[nano]"]
}
```

Creamos el archivo de configuración **solo.rb**, también en el directorio *chef*, que incluye referencias a los ficheros previos.

```
file_cache_path "/home/jesmorc/chef"
cookbook_path "/home/jesmorc/chef/cookbooks"
json_attribs "/home/jesmorc/chef/node.json"
```

Comprobamos que los paquetes  "nano" y "nginx" se instalan con la siguiente orden:
```
sudo chef-solo -c chef/solo.rb
```

## Ejercicio 3
**Escribir en YAML la siguiente estructura de datos en JSON "{ uno: "dos", tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }".**


```
---
- uno: "dos"
  tres:
    - 4
    - 5
    - "Seis"
    -
      - siete: 8
        nueve:
          - 10
          - 11
```

## Ejercicio 4
**Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.**

Para empezar, primero instalamos Ansible:
```
sudo pip install paramiko PyYAML jinja2 httplib2 ansible
```

Añado la máquina la máquina de Azure al "inventario" :
```
echo "iv-jesmorc-ubuntuserver.cloudapp.net" >> ~/ansible_hosts
```

Configuramos la variable de entorno:
```
export ANSIBLE_HOSTS=~/ansible_hosts
```

Arrancamos la máquina virtual:
```
azure vm start iv-jesmorc-ubuntuserver
```


Ahora debemos configurar SSH para poder conectar con la máquina:
```
ssh-keygen -t dsa

ssh-copy-id -i .ssh/id_dsa.pub jesmorc@iv-jesmorc-ubuntuserver.cloudapp.net
```

Ahora comprobaremos que tenemos acceso tanto por SSH como desde Ansible:

```
jesmorc@jesmorc-PClaptop ~/Workinout $ ssh jesmorc@iv-jesmorc-ubuntuserver.cloudapp.net
jesmorc@iv-jesmorc-ubuntuserver.cloudapp.net's password: 
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.16.0-37-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Sat Feb 13 18:25:20 UTC 2016

  System load:  0.24              Processes:           230
  Usage of /:   4.3% of 28.80GB   Users logged in:     0
  Memory usage: 10%               IP address for eth0: 100.77.224.46
  Swap usage:   0%

  Graph this data and manage this system at:
    https://landscape.canonical.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud


*** System restart required ***
Last login: Sat Feb 13 18:25:21 2016 from 217.216.81.186.dyn.user.ono.com
jesmorc@iv-jesmorc-ubuntuserver:~$
```

A continuación conectamos con Ansible:
```
jesmorc@jesmorc-PClaptop $ ansible all -u jesmorc -m ping
Enter passphrase for key'/home/jesmorc/.ssh/id_dsa':
iv-jesmorc-ubuntuserver.cloudapp.net | SUCCESS => {
	"changed": false,
	"ping": "pong"
}
```
)

Podemos ejecutar comandos desde nuestra máquina local:
)
```
jesmorc@jesmorc-PClaptop $ ansible all -u jesmorc -m ping
iv-jesmorc-ubuntuserver.cloudapp.net | SUCCESS => | rc=0 >> olakase 
```

Para el despliegue de la aplicación, primero instalamos los librerías básicos en la máquina:
```
ansible all -u jesmorc -a "sudo apt-get install -y python-setuptools python-dev build-essential git pkg-config libjpeg-dev zlib1g-dev"
ansible all -u jesmorc -m command -a "sudo easy_install pip"
```

Clonamos el repositorio en la máquina de Azure:

```
ansible all -u jesmorc -m git -a "repo=https://github.com/jesmorc/Workinout.git  dest=~/Workinout version=HEAD"
```

Instalamos lo necesario para ejecutar la aplicación:
```
ansible all -u jesmorc -m command -a "pip install -r Workinout/requirements.txt"
```

En local: 
```
azure vm endpoint create iv-jesmorc-ubuntuserver 80 80
```

Desactivamos el servidor web nginx en la máquina azure para que no ocupe el puerto 80:
```
 ansible all -u jesmorc -m command -a "sudo update-rc.d nginx disable;"
```

Hay 2 maneras de ejecutar la aplicación:

Primera: nos movemos al directorio de la aplicación, y luego ejecutaremos la app:
```
ansible all -m shell -a "cd ~/Workinout && sudo python manage.py runserver 0.0.0.0:80"
```

Segunda: con un script

*script.sh*
```
cd ~/Workinout/
sudo python manage.py runserver 0.0.0.0:80
```

Ahora, desde la máquina local podemos ejecutar la siguiente línea, y pondrá en funcionamiento nuestra apliación:
```
ansible all -u jesmorc -m command -a "sh ~/Workinout/script.sh"
```

*NOTA: necesario moverse al directorio de la apliación*

Ahora si entramos a *http://iv-jesmorc-ubuntuserver.cloudapp.net* veremos la aplicación corriendo.

## Ejercicio 5.1
**Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.**

Primero he editado el archivo ansible_hosts:
```
[iv-jesmorc-ubuntuserver]
iv-jesmorc-ubuntuserver.cloudapp.net
```

Editamos también el archivo */etc/ansible/hosts* con el mismo contenido que el archivo **ansible_hosts**.

Creamos el archivo **playbook.yml**:
```
---
- hosts: iv-jesmorc-ubuntuserver
  sudo: yes
  remote_user: jesmorc
  tasks:
  - name: Instalar paquetes necesarios
    apt: name=python-setuptools state=present
    apt: name=python-dev state=present
    apt: name=build-essential state=present
    apt: name=git state=present
    apt: name=libtiff4-dev state=present
    apt: name=libjpeg8-dev state=present
    apt: name=zlib1g-dev state=present
    apt: name=libfreetype6-dev state=present
    apt: name=liblcms1-dev state=present
    apt: name=libwebp-dev state=present
  - name: Clonando repositorio desde git
    git: repo=https://github.com/jesmorc/Workinout.git dest=Workinout version=HEAD force=yes
  - name: Instalar requisitos para la app
    shell: cd Workinout && make install
  - name: Ejecutar aplicacion
    shell: cd Workinout && make run
```

Y ejecutamos el playbook con:

```
jesmorc@jesmorc-PCLaptop:~$ ansible-playbook -u jesmorc playbook.yml
 __________________
< PLAY [localhost] >
 ------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||


 _________________
< GATHERING FACTS >
 -----------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||


 ____________________________________
< TASK: Instalar paquetes necesarios >
 ------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||


previous known host file not found
previous known host file not found
ok: [localhost] => {"changed": false}
 ______________________________________
< TASK: Clonando repositorio desde git >
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||


[...]
```


