#Ejercicios Tema 6

##Ejercicio 1
###Instalar chef en la máquina virtual que vayamos a usar

En mi caso voy a instalar chef en una máquina virtual Ubuntu.
Una posibilidad para instalar chef es la siguiente:

`curl -L https://www.opscode.com/chef/install.sh | bash`

![ejercicio1](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/tema%206/ejercicio6_zps9nfurups.png)

##Ejercicio 2
###Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual

Voy a instalar nginx y como editor de texto vim

Comenzamos generando el directorio donde irán las recetas.

* mkdir -p chef/cookbooks/nginx/recipes
* mkdir -p chef/cookbooks/vim/recipes

Una vez creados los directorios el siguiente paso es crear las recetas en ellos.

Esta sería la receta para nginx

~~~
package 'nginx'
directory '/home/alejandro/Nginx'
file "/home/alejandro/Nginx/LEEME" do
owner "alejandro"
group "alejandro"
mode 00544
action :create
content "Directorio para nginx"
end
~~~

Y en el caso de vim es

~~~
package 'vim'
directory '/home/alejandro/vim'
file "/home/alejandro/vim/LEEME" do
owner "alejandro"
group "alejandro"
mode 00544
action :create
content "Directorio para vim"
end
~~~

Ahora hay que crear el fichero `json` incluyendo en él las referencias a las recetas, este fichero debe guardarse en `/home/alejandro/chef`

~~~
{
"run_list": [ "recipe[nginx]", "recipe[vim]" ]
}
~~~

Para terminar los preparativos necesitamos el fichero de configuración, que ha de guardarse en `home/alejandro/chef/solo.rb

~~~
file_cache_path "/home/alejandro/chef"
cookbook_path "/home/alejandro/chef/cookbooks"
json_attribs "/home/alejandro/chef/node.json"
~~~

Finalmente hemos de ejecutar

`sudo chef-solo -c chef/solo.rb`

![ejercicio2](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/tema%206/ejercicio2_zps34d54tsi.png)


##Ejercicio 3
###Escribir en YAML la siguiente estructura de datos en JSON `{ uno: "dos",tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }`

Para asegurarme de que he escrito la sintaxis de JSON correctamente he usado [jsonlint](jsonlint.com)

La representación correcta de la estructura de datos en JSON es:

~~~

{
	"uno": "dos",
	"tres": [4, 5, "Seis", {
		"siete": 8,
		"nueve": [10, 11]
	}]
}
~~~

Y para pasarla a YAML he usado un convertidor online llamado [jsontoyaml](http://jsontoyaml.com/)

Esta es la estructura de datos representada en YAML

~~~
---
  uno: "dos"
  tres: 
    - 4
    - 5
    - "Seis"
    - 
      siete: 8
      nueve: 
        - 10
        - 11


~~~


##Ejercicio 4
###Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.

Voy a desplegar los fuentes de mi aplicación [bares](https://github.com/acasadoquijada/IV) en una máquina virtual local con ubuntu 14.

Lo primero es instalar el software necesario en caso de que no dispongamos de él

`sudo pip install paramiko PyYAML jinja2 httplib2 ansible`

Después, generar el llamado fichero `inventario`

`echo "172.16.248.130" > ~/ansible_hosts`

Indicamos a Ansible que se trata de una variable de entorno

`export ANSIBLE_HOSTS=~/ansible_hosts`

Una vez hecho lo anterior comprobamos si Ansible funciona

`ansible all -u alejandro -m ping`

[ejercicio6-1](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/tema%206/ejercicio4-1_zpsuuimoubn.png)

Tras las configuraciones iniciales iniciamos el proceso de aprovisionamiento:

* ansible all -u alejandro -m command -a "sudo apt-get install -y python3-setuptools"

* ansible all -u alejandro -m command -a "sudo apt-get -y install python3-dev"

* ansible all -u alejandro -m command -a "sudo apt-get -y install python-psycopg2"

* ansible all -u alejandro -m command -a "sudo apt-get -y install libpq-dev"

* ansible all -u alejandro -m command -a "sudo easy_install3 pip"

* ansible all -u alejandro -m command -a "sudo pip install --upgrade pip"

* ansible all -u alejandro -m command -a "sudo apt-get install git"

Descargamos la app del repositorio

* ansible all -u alejandro -m git -a "repo=https://github.com/acasadoquijada/IV.git dest=~/prueba"

![ejercicio6-2](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/tema%206/ejercicio6-2_zpspe9h0hvw.png)

Instalamos la app

* ansible all -u alejandro -m command -a "sudo pip install -r prueba/requirements.txt"

La lanzamos

* ansible all -u alejandro -m command -a "sudo python3 prueba/manage.py runserver 0.0.0.0:80"

![ejercicio6-3](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/tema%206/ejercicio6-3_zpskcjacwcu.png)


##Ejercicio 6
###Instalar una máquina virtual Debian usando Vagrant y conectar con ella.

Como es natural, comenzamos instalando `Vagrant`

`sudo apt-get install vagrant`

Ahora descargamos la imagen de Debian

`vagrant box add debian https://github.com/holms/vagrant-jessie-box/releases/download/Jessie-v0.1/Debian-jessie-amd64-netboot.box`

![ejercicio5-1](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/tema%206/ejercicio5-1_zpsh5dcbisj.png)

Generamos el fichero vagranfile

`vagrant init debian`

![ejercicio5-2](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/tema%206/ejercicio5-2_zps7cxfhspc.png)

Lanzamos la máquina

`vagrant up`

![ejercicio5-3](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/tema%206/ejercicio5-3_zps6eypdx2a.png)

Finalmente comprobamos que podemos conectarnos via ssh a la máquina 

`vagrant ssh`

![ejercicio5-4](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/tema%206/ejercicio5-4_zpsyivzsruu.png)

##Ejercicio 7
###Crear un script para provisionar nginx o cualquier otro servidor web que pueda ser útil para alguna otra práctica

Debemos modificar nuestro fichero Vagrantfile generado en el ejercicio anterior añadiendo:

~~~
# Every Vagrant virtual environment requires a box to build off of.
config.vm.box = "debian"
config.vm.provision "shell",
inline: "sudo apt-get update && sudo apt-get install -y nginx && sudo service nginx start"
~~~

Después levantamos la máquina `vagrant up` y la provisionamos `vagrant provision`

Una vez hecho esto nos conectamos via ssh y comprobamos el estado de nginx

![ejercicio7](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/tema%206/ejercicio5-4_zpsyivzsruu.png)


##Ejercicio 8
###Configurar tu máquina virtual usando vagrant con el provisionador ansible

Lo primero que debemos hacer es crear un fichero `.yml`, el mio se llama `aprovisionamiento.yml` cuyo contenido es:

~~~
- hosts: localhost
  sudo: yes
  remote_user: vagrant
  tasks: 
  - name: Actualizamos 
    apt: update_cache=yes
  - name: Instalamos los paquetes necesarios
    apt: name=python3-setuptools state=present
    apt: name=python3-dev state=present 
    apt: name=build-essential state=present
    apt: name=python-psycopg2 state=present
    apt: name=libpq-dev state=present
    apt: name=python3-pip state=present
    apt: name=git state=present
  - name: Instalamos pip para python3
    action: apt pkg=python3-pip
  - name: Descargamos la aplicacion
    git: repo=https://github.com/acasadoquijada/IV.git  dest=proyectoIV clone=yes force=yes
  - name: Damos permisos de ejecución a la app
    command: chmod -R +x proyectoIV
  - name: Instalamos la app
    shell: cd proyectoIV && make install
  - name: Ejecutamos la app
    shell: cd proyectoIV && make run
~~~

En este fichero indicamos los paquetes a instalar necesarios para la app, tambien indicamos que se descargue la aplicación, se instale y se ejecute.

Con host `remote_user:` establecemos el nombre del usuario de la máquina

Ahora debemos crear un fichero llamado `ansible_hosts` con lo siguiente

~~~
[localhost]
192.168.10.175
~~~

La ip es la ip privada de la máquina de azure que se creará con Vagrant

Una vez realizados los preparativos de ansible, procedemos con Vagrant para crear la máquina en Azure.

Como no he usado Azure antes tengo que instalar el software necesario

* sudo apt-get install nodejs-legacy
* sudo apt-get install npm
* sudo npm install -g azure-cli

Tambien tenemos que instalar el plugin de Azure para Vagrant

`vagrant plugin install vagrant-azure`

Tras instalar todo lo necesario, obtenemos nuestros credenciales de Azure

* azure login
* azure account download

Al ejecutar los comandos nos pedira que visitemos unas webs, introduzcamos un codigo y descarguemos las credenciales.

Para importarlas a Azure usamos

`azure account import Free\ Trial-2-5-2016-credentials.publishsettings`

![credenciales](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/Practica%205/credenciales_zpsfgkdjyqk.png)

Lo siguiente que debemos hacer es subir nuestros certificados a Azure, primero hay que generarlos:

* `openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout azurevagrant.key -out azurevagrant.key`  
* `chmod 600 azurevagrant.key`
* `openssl x509 -inform pem -in azurevagrant.key -outform der -out azurevagrant.cer`

Una vez generados, los subimos, para ello accedemos a esta [direccion](https://manage.windowsazure.com)

![certificado](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/certificado_zps7s1pnop9.png)

Para terminar la configuración de Vagrant con Azure hay que generar un `.pem`

* `openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out azurevagrant.pem`

* `cat azurevagrant.key > azurevagrant.pem`

Tras terminar toda la configuración de Vagrant y Azure, creamos un fichero `Vagrantfile`

~~~
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.
  config.vm.box = 'azure'
  config.vm.network "public_network"
  config.vm.network "private_network",ip: "192.168.10.175", virtualbox__intnet: "vboxnet0"
  config.vm.network "forwarded_port", guest: 80, host: 80
  config.vm.define "localhost" do |l|
          l.vm.hostname = "localhost"
  end
   
  config.vm.provider :azure do |azure, override|
      azure.mgmt_certificate = File.expand_path('/home/alejandro/azure/azurevagrant.pem') 
      azure.mgmt_endpoint = 'https://management.core.windows.net'
      azure.subscription_id = '6f02263d-1401-4a4c-8cc9-88bf3d4b9877'
      azure.vm_image = 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_2-LTS-amd64-server-20150506-en-us-30GB'
      azure.vm_name = 'maquina-iv' 
      azure.vm_password = 'MiClave27'
      azure.vm_location = 'Central US' 
      azure.ssh_port = '22'
      azure.tcp_endpoints = '80:80'
  end
  
  config.vm.provision "ansible" do |ansible|
      ansible.sudo = true
        ansible.playbook = "aprovisionamiento.yml"
        ansible.verbose = "v"
        ansible.host_key_checking = false
  end

end  
~~~

Se puede ver que este fichero esta dividido en tres bloques.

El primer bloque indicamos que el nombre de la máquina a usar es azure, con ip privada indicada por nosotros, es la misma ip que la que es encuentra en el fichero `ansible_hosts`. Tambien tenemos que indicar donde se encuentra nuestro certicifado `.pem`, colocar nuestro id de azure, este id se puede encontrar en el mismo lugar donde subimos nuestras credenciales. Por ultimo se le indica la imagen a usar, un ubuntu server 14_04

En el segundo bloque establecemos parámetros de la máquina, como su nombre, contraseña, localización, puerto de ssh y endpoints, muy importantes, ya que sin ellos no se puede acceder a la maquina desde fuera.

En el ultimo bloque indicamos que el aprovisionamiento se va a realizar con ansible, el fichero de aprovisionamiento a usar y que queremos opciones de depuracion.

Tras redactar correctamente el Vagrantfile, realizamos

* `vagrant box add azure https://github.com/msopentech/vagrant-azure/raw/master/dummy.box`


Finalmente ya podemos levantar nuestra máquina con 

* `vagrant up --provider=azure`

![vagrant-up](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/vagrant-up_zpsatf5knak.png)


![vagrant-provision](http://i1045.photobucket.com/albums/b460/Alejandro_Casado/vagrant-provider_zpsi3dvxw1c.png)


