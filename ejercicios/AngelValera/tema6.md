# **Ejercicios Tema 6**

###**Ejercicio 1:**Instalar chef en la máquina virtual que vayamos a usar

* Una forma rápida de instalarlo es usar:

```curl -L https://www.opscode.com/chef/install.sh | sudo bash```

* Para comprobar que la instalación se ha realizado:

```chef-solo -v```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_001_zpswsl7xzp1.png)


###**Ejercicio 2:**Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.

* Lo primero que debemos hacer es crear los directorios para las recetas:

```
mkdir -p chef/cookbooks/nginx/recipes
mkdir -p chef/cookbooks/nano/recipes
```

* Lo siguiente es crear los ficheros que contienen las recetas para instalar **nginx** y **nano**, por ejemplo. Dichos ficheros son los default.rb ubicados cada uno de ellos en los directorios creados en el paso anterior respectivamente.

* **default.rb** para nginx:

```
package 'nginx'
directory "/home/angel/Documentos/nginx"
file "/home/angel/Documentos/nginx/LEEME" do
    owner "angel"
    group "angel"
    mode 00544
    action :create
    content "Directorio para nginx de prueba"
end
```

* **default.rb** para nano:

```
package 'nano'
directory "/home/angel/Documentos/nano"
file "/home/angel/Documentos/nano/LEEME" do
    owner "angel"
    group "angel"
    mode 00544
    action :create
    content "Directorio para nano"
end
```
* Ahora creamos el fichero **node.json** en el directorio /chef con el siguiente contenido.

```
{
    "run_list":["recipe[nginx]", "recipe[nano]"]
}
```

* Por último debemos añadir el archivo **solo.rb** en el directorio /chef, en el que se incluyen referencias a los anteriores archivos.

```
file_cache_path "/home/angel/chef"
cookbook_path "/home/angel/chef/cookbooks"
json_attribs "/home/angel/chef/node.json"
```
Quedando un árbol de directorios de la siguiente manera:

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_005_zpswuassorh.png)

* Ahora se instalan los paquetes ejecutando la orden:

``` sudo chef-solo -c chef/solo.rb```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_003_zpssyici8ul.png)

* Comprobamos que ha funcionado.

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_006_zpsl6wft1gg.png)



###**Ejercicio 3:** Escribir en YAML la siguiente estructura de datos en JSON 
###{ uno: "dos",
###  tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }


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
###**Ejercicio 4:**Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.

* **Paso 1** Instalamos Ansible:

```sudo pip install paramiko PyYAML jinja2 httplib2 ansible```

* **Paso 2** Añadimos la máquina sobre la que desplegaremos la aplicación, en mi caso una máquina virtual de Azure que usé en el ejercicio 5 del tema 5, al fichero inventario:

```echo "maquina-angel-ej5.cloudapp.net" > ~/ansible_hosts```

* **Paso 3** Indicamos la ubicación del fichero inventario mediante una variable de entorno:

```export ANSIBLE_HOSTS=~/ansible_hosts```

* **Paso 4** Hechpo esto, nos identificamos en azure para arrancar la máquina virtual sobre la que desplegaremos la aplicación:

```
azure login
azure vm start maquina-angel-ej5
```
* **Paso 5** Ahora tenemos que configurar SSH para poder conectar con la máquina, para ello creamos una par de claves pública-privada:
```
ssh-keygen -t dsa 
ssh-copy-id -i .ssh/id_dsa.pub angel@maquina-angel-ej5.cloudapp.net
```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_007_zpsjfrwovex.png)

* **Paso 6**Ahora, comprobamos que hay conexión con la máquina Azure a través de Ansible con el comando ping:

```
ansible all -u angel -i ansible_hosts -m ping
```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_008_zpskl8ltscf.png)
ansible all -u angel -m command -a "sudo apt-get -y install python-setuptools python-dev build-essential python-psycopg2 git"
```
ansible all -u angel  -m command -a "sudo apt-get install python-setuptools python-dev build-essential python-psycopg2 git -y"
 ansible all -u angel -i ansible_hosts -m command -a "sudo easy_install pip" 
```
![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_012_zpsrxsxit6r.png)

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_010_zps9j7a62ei.png)

* **Paso 7** Ahora descargamos la aplicación desde nuestro repositorio, e instalamos los requisitos necesarios:

```
ansible all -u angel -m git -a "repo=https://github.com/AngelValera/bares-y-tapas-DAI  dest=~/aplicacionDAI version=HEAD"
ansible all -u angel -m command -a "sudo pip install -r aplicacionDAI/requirements.txt"
```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_013_zpscqgvtwnk.png)

* **Paso 7**Ahora antes de ejecutar la aplicación debemos tener en cuenta algunas cosas:

1- Saber que puerto vamos a usar para lanzar la aplicación, en mi caso uso el 80, por lo que debemos abrirlo.

```
ansible all -u angel -m command -a "sudo fuser -k 80/tcp"
```
2- Si teniamos nginx ocupando ese puerto , debemos pararlo:

```ansible all -u angel -m command -a "sudo service nginx stop"```

3- Abrir trafico de azure, (yo ya lo hice en el tema pasado)

```azure vm endpoint create maquina-angel-ej5 80 80```


Hechas estas comprobaciones, ejecutamos:

```
ansible all -u angel -m command -a "sudo python ~/aplicacionDAI/manage.py runserver 0.0.0.0:80"
```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Seleccioacuten_014_zpsq1otkyfy.png)

Por último apagar la máquina:

```
azure vm shutdown maquina-angel-ej5
```


###**Ejercicio 5.1:**Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.

El playbook que he usado para subir la aplicación a azure ha sido el siguiente:

```
- hosts: azure
  remote_user: angel
  become: yes
  become_method: sudo
  tasks:
  - name: Actualizar repositorios
    apt: update_cache=yes
  - name: Instalar dependencias de la aplicacionDAI
    apt: name=python-setuptools state=present
    apt: name=build-essential state=present
    apt: name=python-dev state=present
    apt: name=python-psycopg2 state=present
    apt: name=git state=present
  - name: easy_install
    easy_install: name=pip
  - name: Descargar fuentes de la aplicacionDAI
    git: repo=https://github.com/AngelValera/bares-y-tapas-DAI  dest=~/aplicacionDAI force=yes
  - name: Instalar requirements
    pip: requirements=~/aplicacionDAI/requirements.txt
  - name: Lanzar aplicacion
    command: nohup python ~/aplicacionDAI/manage.py runserver 0.0.0.0:80
```
Lo ejecutamos de la siguiente forma:

```ansible-playbook baresytapas.yml```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_015_zps6bkxqo6v.png)

![](http://i666.photobucket.com/albums/vv21/angelvalera/Seleccioacuten_014_zpsq1otkyfy.png)

###**Ejercicio 5.2:**¿Ansible o Chef? ¿O cualquier otro que no hemos usado aquí?.

Prefiero ansible, ya que me ha parecido mucho más sencillo que chef y su estructura de directorios específica para las recetas.


###**Ejercicio 6:**Instalar una máquina virtual Debian usando Vagrant y conectar con ella.

* Lo primero que debemos hacer es instalar vagrant:

```
sudo apt-get install vagrant
```
* Ahora descargo la imagen de Debian tal y como se explica en los apuntes:

```
vagrant box add debian https://github.com/holms/vagrant-jessie-box/releases/download/Jessie-v0.1/Debian-jessie-amd64-netboot.box
```
![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_016_zpsmqkz20qw.png)

* Lo siguiente es crear el fichero Vagranfile:

```
vagrant init debian
```
![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_017_zpsk92abimb.png)

* Ahora se inicializa la máquina usando:

```
vagrant up
```
![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_018_zpsz2kn26ou.png)

* Ahora conectamos con ella por ssh.

```
vagrant ssh
```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_019_zpsvunjnh9q.png)

###**Ejercicio 7:**Crear un script para provisionar `nginx` o cualquier otro servidorweb que pueda ser útil para alguna otra práctica


* Para provisionar nuestra máquina debian creada anteriormente con nginx, debemos modificar el vagrantfile creado anteriormente de la siguiente forma:

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_020_zpskexlzvzp.png)

* Arrancamos la máquina mediante vagrant up y se ejecuta la provisión mediante:

```
vagrant provision
```
* Ahora conectamos por ssh:

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_021_zpsgfxa3nfq.png)

```
vagrant ssh
```
* Comprobamos que efectivamente se ha instalado y ha arrancado el servicio:

```
sudo service nginx status
```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_022_zpsgvbwoi0t.png)


###**Ejercicio 8:**Configurar tu máquina virtual usando vagrant con el provisionador ansible
En este ejercicio vamos a configurar una máquina virtual de azure y provisionarla con la aplicación antes usada de bares y tapas de la asignatura de DAI.

De ejercicios anteriores ya tenemos instalados tanto vagrant como azure-cli, por tanto lo primero que tenemos que hacer es instalar es el provisionador azure para vagrant

```vagrant plugin install vagrant-azure```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_023_zpsrddsrwg7.png)


El siguiente paso es loguearse y conseguir información de las credenciales de Azure:

```
azure login
azure account download
```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_024_zpsdfxi5axe.png)

```azure account import Evaluación\ gratuita-2-5-2016-credentials.publishsettings```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_025_zpsqvr4qpfj.png)


Lo siguiente que debemos hacer es generar los certificados que se van a subir a Azure:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout azurevagrant.key -out azurevagrant.key
chmod 600 azurevagrant.key
openssl x509 -inform pem -in azurevagrant.key -outform der -out azurevagrant.cer
```

Lo siguiente es subir el archivo .cer a Azure:

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_026_zpslehlmd7g.png)

Para autenticar la maquina azure desde el Vagrantfile, necesitamos un archivo.pem. Para generarlo necesitamos hacer un truco, primero ejecutar:

```
openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out azurevagrant.pem
```

Para generarlo y después concatenarle el fichero.key. Esto es necesario para que el fichero.pem contenga tanto la clave publica como la privada.

```
cat azurevagrant.key > azurevagrant.pem 
```
Ahora lo que debemos hacer es crear el vagrantfile:

```
Vagrant.configure(2) do |config|
  config.vm.box = "azure"
  config.vm.box_url = 'https://github.com/msopentech/vagrant-azure/raw/master/dummy.box'  
  config.vm.network "public_network"
  config.vm.network "forwarded_port", guest: 80, host: 80
  config.vm.define "localhost" do |l|
          l.vm.hostname = "localhost"
  end
  config.vm.provider :azure do |azure, override|
    azure.mgmt_certificate = '/home/angel/Prueba/azurevagrant.pem'
    azure.mgmt_endpoint = 'https://management.core.windows.net'
    azure.subscription_id = '8afb40f4-4482-4a5c-832a-b7aab655fed1'
    azure.vm_image = 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_2-LTS-amd64-server-20150506-en-us-30GB'
    azure.vm_name = 'maquinaavm'
    azure.vm_password = 'Clave#Angel#1'
    azure.vm_location = 'Central US' 
        azure.ssh_port = '22'
        azure.tcp_endpoints = '80:80'
  end
  config.vm.provision "ansible" do |ansible|
        ansible.sudo = true
        ansible.playbook = "playbookIV.yml"
        ansible.verbose = "v"
  end
end
```
Ahora definimos el playbook de ansible:

```
- hosts: localhost
  remote_user: vagrant
  become: yes
  become_method: sudo
  tasks:
  - name: Actualizar repositorios
    apt: update_cache=yes        
  - name: Instalar dependencias
    apt: name={{ item }}
    with_items:
      - python-setuptools
      - python-dev
      - build-essential
      - python-psycopg2
      - git    
  - name: easy_install
    easy_install: name=pip    
  - name: Descargar fuentes
    git: repo=https://github.com/AngelValera/bares-y-tapas-DAI dest=~/appBaresyTapas force=yes    
  - name: Instalar requirements
    pip: requirements=~/appBaresyTapas/requirements.txt    
  - name: Lanzar app
    command: nohup python ~/appBaresyTapas/manage.py runserver 0.0.0.0:80
```
Además en el fichero ansible_host ponemos:

```
[localhost]
127.0.0.1              ansible_connection=local
```

Hecho esto, podemos ejecutar:

```
vagrant up --provider=azure
```

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_002_zpsc3tsonux.png)

Y vemos que también se ejecuta el playbook de ansible:

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_003_zpsylr2hhzb.png)

![](http://i666.photobucket.com/albums/vv21/angelvalera/Ejercicios%20tema%206/Seleccioacuten_004_zpskpbu6eux.png)

