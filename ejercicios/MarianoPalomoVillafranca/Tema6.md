Tema 6
===
> Mariano Palomo Villafranca

#### Ejercicio 1. Instalar chef en la máquina virtual que vayamos a usar
En una máquina virtual con Ubuntu 14.04, instalamos `chef` de forma rápida mediante:

```
$ curl -L https://www.opscode.com/chef/install.sh | sudo bash
```

![](http://i.imgur.com/9ASaw8m.png)

#### Ejercicio 2. Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.
En primer lugar creamos los directorios para las recetas:

```
$ mkdir -p chef/cookbooks/nginx/recipes
$ mkdir -p chef/cookbooks/nano/recipes
```

Las recetas serán ficheros `default.rb`. A continuación se muestra la receta para instalar nginx y nano en el sistema:

- Nginx:
```
package 'nginx'
directory "/home/mpvillafranca/Documentos/nginx"
file "/home/mpvillafranca/Documentos/nginx/LEEME" do
    owner "mpvillafranca"
    group "mpvillafranca"
    mode 00544
    action :create
    content "Directorio para nginx"
end
```

- Nano:
```
package 'nano'
directory "/home/mpvillafranca/Documentos/nano"
file "/home/mpvillafranca/Documentos/nano/LEEME" do
    owner "mpvillafranca"
    group "mpvillafranca"
    mode 00544
    action :create
    content "Directorio para nano"
end
```

En el directorio `chef/`, creamos un fichero `node.json` con el siguiente contenido:

```
{
  "run_list":["recipe[nginx]", "recipe[nano]"]
}
```

Además, en el mismo directorio, añadimos un fichero `solo.rb`, que incluye referencias a los archivos anteriores, con el siguiente contenido:

```
file_cache_path "/home/pvillafranca/chef"
cookbook_path "/home/mpvillafranca/chef/cookbooks"
json_attribs "/home/mpvillafranca/chef/node.json"
```

El árbol de directorios quedaría entonces así:

![](http://i.imgur.com/cTSm8sD.png)

Por último, ejecutamos la orden:

```
$ sudo chef-solo -c chef/solo.rb
```

![](http://i.imgur.com/cPB8Uvw.png)

#### Ejercicio 3. Escribir en YAML la siguiente estructura de datos en JSON:

```
{ uno: "dos", tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }
```

La estuctura en YAML quedaría así:
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
#### Ejercicio 4. Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.
Lo primero será instalar `ansible`. Para ello, ejecutamos:

```
$ sudo pip install paramiko PyYAML jinja2 httplib2 ansible
```

Utilizaré la máquina del ejercicio 5 del tema anterior para ello. En primer lugar, añadimos la máquina de Azure al control de ansible (inventario).

```
echo "mpvillafranca-t5-ej5.cloudapp.net" > ~/ansible_hosts
```

Utilizaremos una variable de entorno para indicar a ansible la dirección ruta del fichero.

```
export ANSIBLE_HOSTS=~/ansible_hosts
```

Con la máquina iniciada (`azure vm start mpvillafranca-t5-ej5.cloudapp.net/`), configuramos ssh para poder acceder sin necesidad de usuario y contraseña.

```
$ ssh-keygen -t dsa
$ ssh-copy-id -i /home/mpvillafranca/.ssh/id_dsa.pub mpvillafranca-t5-ej5.cloudapp.net
```

Y comprobamos la conexión:

```
$ ansible all -u mpvillafranca -i $ ansible_hosts -m ping
```

**NOTA**: si nos diera el error `too long for Unix domain socket`, lo que debemos hacer es editar la configuración de Ansible en `/etc/ansible/ansible.cfg` y añadir:

```
[ssh_connection]
control_path = %(directory)s/%%h-%%p-%%r
```

Ya podemos desplegar la aplicación. En primer lugar, descargamos e instalamos todos los paquetes necesarios para realizar estas gestiones (Git, python, pip ...):

```
$ ansible all -u mpvillafranca -m command -a "sudo apt-get install python-setuptools python-dev build-essential git -y"
$ ansible all -u mpvillafranca -m command -a "sudo easy_install pip"
```

Ahora, descargamos los fuentes e instalamos las dependencias de la aplicación.

```
$ ansible all -u mpvillafranca -m git -a "repo=https://github.com/mpvillafranca/barestapas.git dest=~/AnsibleTest version=HEAD"
$ ansible all -u mpvillafranca -m command -a "sudo pip install -r AnsibleTest/requirements.txt"
```

Realizamos las migraciones y rellenamos la base de datos con:

```
$ ansible all -u mpvillafranca -m command -a "sudo python AnsibleTest/tango_with_django_project/manage.py migrate"
$ ansible all -u mpvillafranca -m command -a "sudo python AnsibleTest/tango_with_django_project/populate_rango.py"
```

Y para ejecutar la aplicación:

```
$  ansible all -u mpvillafranca -m command -a "sudo python AnsibleTest/tango_with_django_project/manage.py runserver 0.0.0.0:80"
```

![](http://i.imgur.com/d78pvmn.png)

#### Ejercicio 5
##### 5.1. Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.

Creamos el playbook `barestapas.yml`:

```
---
- hosts: azure-ej5
  sudo: yes
  remote_user: mpvillafranca
  tasks:
  - name: Instalacion de los paquetes necesarios
    apt: name=python-setuptools state=present
    apt: name=build-essential state=present
    apt: name=python-dev state=present
    apt: name=git state=present
  - name: Descarga de los fuentes desde Github
    git: repo=https://github.com/mpvillafranca/barestapas.git  dest=AnsibleTestEj5 clone=yes force=yes
  - name: Asignar permisos de ejecución al directorio
    command: chmod -R +x /home/mpvillafranca/AnsibleTestEj5
  - name: Instalacion de requisitos de la aplicacion con pip
    command: sudo pip install -r /home/mpvillafranca/AnsibleTestEj5/requirements.txt
  - name: Aplicar migraciones de la base de datos
    command: sudo python /home/mpvillafranca/AnsibleTestEj5/tango_with_django_project/manage.py migrate
  - name: Rellenar la base de datos
    command: sudo python /home/mpvillafranca/AnsibleTestEj5/tango_with_django_project/populate_rango.py
  - name: Ejecutar aplicacion
    command: nohup sudo python /home/mpvillafranca/AnsibleTestEj5/tango_with_django_project/manage.py runserver 0.0.0.0:80
```

Y desplegamos la aplicación con:

```
$ ansible-playbook -u mpvillafranca barestapas.yml
```

Con la orden `nohup` mantenenemos la ejecución del comando pese a salir del terminal, ya que hace que se ejecute de forma independiente a la sesión. Por lo tanto al terminar el proceso con el que desplegamos el playbook, la ejecución se mantiene persistente y, por tanto, el servidor corriendo.

![](http://i.imgur.com/HgdmRYB.png)
##### 5.2. ¿Ansible o Chef? ¿O cualquier otro que no hemos usado aquí?.
`Ansible`, puesto que es mucho más flexible que Chef y sus playbooks son mucho más sencillos y fáciles de entender y configurar.

#### Ejercicio 6. Instalar una máquina virtual Debian usando Vagrant y conectar con ella.
Instalamos `Vagrant` descargando el `.deb` correspondiente desde [aquí](https://releases.hashicorp.com/vagrant/1.8.1/vagrant_1.8.1_x86_64.deb) y ejecutando:

```
$ sudo dpkg -i vagrant_1.8.1_x86_64.deb
```

Ahora descargamos la imagen a utilizar:

```
$ vagrant box add ubuntu https://github.com/kraksoft/vagrant-box-ubuntu/releases/download/14.04/ubuntu-14.04-amd64.box
```

Y creamos un fichero Vagrant:

```
$ vagrant init ubuntu
```

E iniciamos la máquina:

```
$ vagrant up
```

Finalmente, comprobamos la conexión con ella:

```
$ vagrant ssh
```

![](http://i.imgur.com/nRUqHmu.png)

#### Ejercicio 7. Crear un script para provisionar `nginx` o cualquier otro servidor web que pueda ser útil para alguna otra práctica
Para ello, añadimos antes del final del fichero `Vagrantfile`:

```
config.vm.provision "shell",
inline: "sudo apt-get update && sudo apt-get install -y nginx && sudo service nginx start"
```

Y ejecutamos:

```
$ vagrant provision
```

#### Ejercicio 8. Configurar tu máquina virtual usando vagrant con el provisionador ansible
Este ejercicio es equivalente al último hito, por lo que se exiplacará detalladamente en el respectivo apartado dentro del repositorio.
