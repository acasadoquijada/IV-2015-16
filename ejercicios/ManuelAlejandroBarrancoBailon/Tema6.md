#Tema 6

##Ejercicios

###Ejercicio 1
**Instalar chef en la máquina virtual que vayamos a usar**
1. He procedido descargándolo directamente de la página web:
~~~
curl -L https://www.opscode.com/chef/install.sh | sudo bash
~~~
![ChefLinuxMint](http://i1016.photobucket.com/albums/af281/raperaco/ChefLinuxMint_zpshvbevwah.png)


###Ejercicio 2
**Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.**
* Crear los directorios para las recetas:
~~~
mkdir -p chef/cookbooks/nginx/recipes
mkdir -p chef/cookbooks/geany/recipes
~~~

* Crear los ficheros para las recetas en su correspondiente directorio. Estos serán los ficheros **default.rb**:
- Para NGINX
En chef/cookbooks/nginx/recipes/default.rb:
~~~
package 'nginx'
~~~
    
- Para Geany
En chef/cookbooks/geany/recipes/default.rb:
~~~
package 'geany'
directory '/home/alex/Documentos/geany'
file "/home/alex/Documentos/geany/LEEME" do
	owner "alex"
    group "alex"
    mode 00644
    action :create
    content "Directorio para documentos diversos de Geany"
end
~~~

* Crear el fichero **node.json** con las recetas a ejecutar (en mi caso lo creo en la ruta /home/alex/chef):
~~~
{
	"run_list":["recipe[nginx]", "recipe[geany]"]
}
~~~

* En el directorio chef (en mi caso /home/alex/chef) incluimos el fichero de configuración **solo.rb** que incluye referencias a los ficheros de recetas y al fichero de recetas a ejecutar (node.json):
~~~
file_cache_path "/home/alex/chef"
cookbook_path "/home/alex/chef/cookbooks"
json_attribs "/home/alex/chef/node.json"
~~~

* Ejecutamos chef para que lleve a cabo sus tareas (en este caso instalación de nginx, emacs y creación de directorio y fichero):
~~~
sudo chef-solo -c chef/solo.rb
~~~
![ChefSolo](http://i1016.photobucket.com/albums/af281/raperaco/ChefSolo_zpsvwdydnur.png)

* Comprobamos el estado de nginx y la creación del fichero LEEME abriéndolo con el editor Geany:
![ChefSoloNginx](http://i1016.photobucket.com/albums/af281/raperaco/ChefSoloNginx_zps9lowtnfg.png)
![ChefSoloGeany](http://i1016.photobucket.com/albums/af281/raperaco/ChefSoloGeany_zpsh3h7sbuv.png)


###Ejercicio 3
**Escribir en YAML la siguiente estructura de datos en JSON**
~~~
{ uno: "dos",
  tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }
~~~

En YAML, sería:
~~~
-uno : "dos"
-tres :
	- 4
	- 5
	- "Seis"
	- 
		-siete : 8
        -nueve :
        	- 10
        	- 11
~~~


###Ejercicio 4
**Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.**
* Instalar Ansible:
~~~
sudo pip install paramiko PyYAML jinja2 httplib2 ansible
~~~

* Añadir la máquina sobre la que desplegaremos la aplicación, en mi caso una máquina virtual de Azure, al fichero inventario:
~~~
echo "azurevm-ubuntu14-alex.cloudapp.net" > ~/ansible_hosts
~~~

* Indicar la ubicación del fichero inventario mediante una variable de entorno:
~~~
export ANSIBLE_HOSTS=~/ansible_hosts
~~~

* Me identifico en azure para arrancar mi máquina virtual sobre la que desplegaré la aplicación:
~~~
azure login
azure vm start azureVM-ubuntu14-alex
~~~
![azureStart](http://i1016.photobucket.com/albums/af281/raperaco/azureStart_zpsjnzt5qdw.png)

* Creo el par clave pública/privada para poder acceder por SSH sin necesidad de introducir cada vez la contraseña:
~~~
ssh-keygen -t dsa 
ssh-copy-id -i ~/.ssh/id_dsa.pub alex@azurevm-ubuntu14-alex.cloudapp.net
~~~
![azureKeySSH](http://i1016.photobucket.com/albums/af281/raperaco/azureKeySSH_zps4ov8ycj5.png)

* Compruebo que tengo conexión con la máquina Azure a traves de Ansible con el comando ping:
~~~
ansible all -u alex -m ping
~~~
![ansiblePing](http://i1016.photobucket.com/albums/af281/raperaco/ansiblePing_zpsdbbtqqhb.png)

* Instalo los paquetes necesarios en la máquina:
~~~
ansible all -u alex -m command -a "sudo apt-get -y install python-setuptools python-dev build-essential python-psycopg2 git"
ansible all -u alex -m command -a "sudo easy_install pip"
~~~
![ansibleDependencias](http://i1016.photobucket.com/albums/af281/raperaco/ansibleDependencias_zpslvhhsh6q.png)
![ansibleDependenciaPIP](http://i1016.photobucket.com/albums/af281/raperaco/ansibleDependenciaPIP_zps1z8xpiwy.png)

* Descargo la aplicación del repositorio de GitHub:
~~~
ansible all -u alex -m git -a "repo=https://github.com/mabarrbai/TusPachangas.git  dest=~/appDAI"
~~~
![ansibleGitClone](http://i1016.photobucket.com/albums/af281/raperaco/ansibleGitClone_zps2sbtjkxo.png)

* Instalar los requisitos de la app:
~~~
ansible all -u alex -m command -a "sudo pip install -r ~/appDAI/requirements.txt"
~~~
![ansibleRequirements](http://i1016.photobucket.com/albums/af281/raperaco/ansibleRequirements_zpsq5jw6rxt.png)

* Saber que puerto vamos a usar para lanzar la aplicación. Yo usaré el puerto 80 (que ya lo abrí en el ejercicio 5 del tema 5) y además asegurarse de que no esté el puerto en uso (por el servidor nginx del ejercicio 5 del tema 5, por ejemplo; si es así, parar el servicio nginx). Lanzar la aplicación (lo haré con el servidor de desarrollo de Django el cual se queda ocupado y no muestra mensaje por pantalla. Otra opción será lanzarlo en segundo plano con &):
~~~
ansible all -u alex -m command -a "sudo python ~/appDAI/manage.py runserver 0.0.0.0:80"
~~~

* Comprobamos que se ha desplegado la app correctamente:
![ansibleAppDesplegada](http://i1016.photobucket.com/albums/af281/raperaco/ansibleAppDesplegada_zps5agmlryv.png)


###Ejercicio 5
####1. Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.
El playbook que he creado para el despliegue de mi aplicación ha sido:
~~~
- hosts: azure
  remote_user: alex
  become: yes
  become_method: sudo
  tasks:
  - name: Actualizar repositorios
    apt: update_cache=yes
    tags: 
    - apt-update
        
  - name: Instalar dependencias
    apt: name={{ item }}
    with_items:
      - python-setuptools
      - python-dev
      - build-essential
      - python-psycopg2
      - git
    tags:
    - dependencias
    
  - name: easy_install
    easy_install: name=pip
    tags:
    - pip
    
  - name: Descargar fuentes
    git: repo=https://github.com/mabarrbai/TusPachangas.git dest=~/appDAI force=yes
    tags:
    - fuentes
    
  - name: Instalar requirements
    pip: requirements=~/appDAI/requirements.txt
    tags:
    - requirements
    
  - name: Lanzar app
    command: nohup python ~/appDAI/manage.py runserver 0.0.0.0:80
    tags:
    - app
~~~

Para ejecutarlo:
~~~
ansible-playbook playbookTusPachangas.yml
~~~
*No hace falta que indique el usuario puesto que ya se lo he indicado en el playbook (remote_user).*

![ansiblePlaybook](http://i1016.photobucket.com/albums/af281/raperaco/ansiblePlaybook_zps75il8nmt.png)

Y se puede comprobar como la aplicación está desplegada
![ansibleAppDesplegada](http://i1016.photobucket.com/albums/af281/raperaco/ansibleAppDesplegada_zps5agmlryv.png)

####2. ¿Ansible o Chef? ¿O cualquier otro que no hemos usado aquí?.
Aunque las dos tienen funcionalidades similares y fáciles de comprender, personalmente me ha gustado más Ansible, ya que veo una mejor estructura y un mejor control desde un único fichero **playbook** que desde múltiples recetas en Chef. Además Ansible se conecta a los nodos mediante SSH para realizar las configuraciones teniendo la posibilidad de organizar los nodos en grupos y dirigirte a unos específicos, a un grupo o a todos, mientras que con Chef, la estructura de directorios y las recetas tienen que estar en las máquinas a configurar.
Por tanto, **Ansible**.


###Ejercicio 6
**Instalar una máquina virtual Debian usando Vagrant y conectar con ella.**
* Usando VirtualBox 5.x, descargar Vagrant de [vagrant_1.8.1_x86_64.deb](https://releases.hashicorp.com/vagrant/1.8.1/) e instalarlo:
~~~
sudo dpkg -i vagrant_1.8.1_x86_64.deb
~~~

* Descargo la imagen de Debian de [http://www.vagrantbox.es/](http://www.vagrantbox.es/). (elegir una imagen que contenga los guest additions para evitar futuros problemas)
~~~
vagrant box add debian8 https://github.com/holms/vagrant-jessie-box/releases/download/Jessie-v0.1/Debian-jessie-amd64-netboot.box
~~~
![VagrantAddBox](http://i1016.photobucket.com/albums/af281/raperaco/VagrantAddBox_zpsu3jv7pkq.png)

* Creo el fichero Vagrantfile:
~~~
vagrant init debian8
~~~
![VagrantInit](http://i1016.photobucket.com/albums/af281/raperaco/VagrantInit_zpsrszu0pfw.png)

* Arranco la máquina
~~~
vagrant up
~~~
![VagrantUp](http://i1016.photobucket.com/albums/af281/raperaco/VagrantUp_zpsjd5lppjx.png)

* Nos conectamos a ella:
~~~
vagrant ssh
~~~
![VagrantSSH](http://i1016.photobucket.com/albums/af281/raperaco/VagrantSSH_zpsy4bpdfue.png)


###Ejercicio 7
**Crear un script para provisionar `nginx` o cualquier otro servidor
web que pueda ser útil para alguna otra práctica**
* Al fichero Vagrantfile que se creó en el ejercicio anterior tras ejecutar *vagrant init*, le he añadido el siguiente código que instala **nginx** y lo arranca:
~~~
config.vm.provision "shell",
  	inline: "sudo apt-get -y install nginx && service nginx start"
~~~
El fichero completo queda:
~~~
Vagrant.configure(2) do |config|
	config.vm.box = "debian8"
	config.vm.provision "shell",
  		inline: "sudo apt-get -y install nginx && service nginx start"
end
~~~

* Arrancamos la máquina, mandamos la máquina a provisionar (con vagrant up no lo hace).
~~~
vagrant up
vagrant provision
~~~
![VagrantProvision](http://i1016.photobucket.com/albums/af281/raperaco/VagrantProvision_zpsjob9kwwr.png)

* Nos conectamos a ella y comprobamos que nginx está en ejecución:
~~~
vagrant ssh
sudo service nginx status
~~~
![VagrantNginx](http://i1016.photobucket.com/albums/af281/raperaco/VagrantNginx_zpsumh1dpq3.png)


###Ejercicio 8
**Configurar tu máquina virtual usando vagrant con el provisionador
ansible**
Configuraré una máquina virtual Azure con vistas al último hito. 
Me han sido de gran ayuda la [documentación del plugin vagrant-azure](https://github.com/Azure/vagrant-azure) y la web [https://unindented.org/articles/provision-azure-boxes-with-vagrant/](https://unindented.org/articles/provision-azure-boxes-with-vagrant/)

* Instalo el provisionador de Azure para Vagrant:
~~~
vagrant plugin install vagrant-azure
~~~

* Login en Azure y obtener las credenciales de la cuenta:
~~~
azure login
azure account download
~~~
Tras ejecutar azure account download, nos proporciona un enlace sobre el que pinchar y descargar dichas credenciales en el directorio actual.

* Importamos este fichero de credenciales a nuestro CLI de Azure:
~~~
azure account import ./*.publishsettings
~~~
![azureAccountImport](http://i1016.photobucket.com/albums/af281/raperaco/azureAccountImport_zpsnau5ww1e.png)

* Este fichero ya importado tiene información confidencial, por lo que lo borramos
~~~
rm ./*.publishsettings
~~~

* Generamos los certificados que nos permitirán interaccionar con Azure:
~~~
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout azurevagrant.key -out azurevagrant.key
chmod 600 azurevagrant.key
openssl x509 -inform pem -in azurevagrant.key -outform der -out azurevagrant.cer
~~~

* Subir el certificado (fichero .cer) a Azure. Para ello hacerlo desde https://manage.windowsazure.com en el menú Configuración, en la pestaña Certificados de administración:
![azureCer](http://i1016.photobucket.com/albums/af281/raperaco/azureCer_zpsxqfr6spb.png)

* Crear el fichero .pem que es el que acepta Azure. Dicho fichero .pem contiene tanto la clave privada como la pública:
~~~
openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out azurevagrant.pem
cat azurevagrant.key > azurevagrant.pem
~~~

* Crear el Vagrantfile:
~~~
Vagrant.configure(2) do |config|
  config.vm.box = "azure"
  config.vm.box_url = 'https://github.com/msopentech/vagrant-azure/raw/master/dummy.box'
  
  config.vm.network "public_network"
  config.vm.network "private_network",ip: "192.168.56.10", virtualbox__intnet: "vboxnet0"
  config.vm.network "forwarded_port", guest: 80, host: 80
  config.vm.define "localhost" do |l|
          l.vm.hostname = "localhost"
  end
  
  config.vm.provider :azure do |azure, override|
  	azure.mgmt_certificate = '/home/alex/Escritorio/trabajandoIV/pruebaVagrant/azurevagrant.pem'
  	azure.mgmt_endpoint = 'https://management.core.windows.net'
  	azure.subscription_id = '3a08b9a8-264a-49cc-b566-9bf11055fbae'
  	azure.vm_image = 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_2-LTS-amd64-server-20150506-en-us-30GB'
  	azure.vm_name = 'prueba'
  	azure.vm_password = 'Alex2016!'
  	azure.vm_location = 'Central US' 
        azure.ssh_port = '22'
        azure.tcp_endpoints = '80:80'
  end
  
  config.vm.provision "ansible" do |ansible|
        ansible.sudo = true
        ansible.playbook = "/home/alex/Escritorio/trabajandoIV/pruebaVagrant/playbookTusPachangas.yml"
        ansible.verbose = "v"
  end
end
~~~

* Definir la variable de entorno para Ansible en la que indicaremos dónde se encuentra el fichero ansible_hosts:
~~~
export ANSIBLE_HOSTS=~/Escritorio/trabajandoIV/pruebaVagrant/ansible_hosts
~~~

* Añadimos al fichero ansible_hosts, la nueva máquina que vamos a usar, que será localhost:
~~~
[localhost]
127.0.0.1              ansible_connection=local
~~~

* Crear el playbook a usar de Ansible(playbookTusPachangas.yml):
~~~
---
- hosts: localhost
  remote_user: vagrant
  become: yes
  become_method: sudo
  tasks:
  - name: Actualizar repositorios
    apt: update_cache=yes
    tags: 
    - apt-update
        
  - name: Instalar dependencias
    apt: name={{ item }}
    with_items:
      - python-setuptools
      - python-dev
      - build-essential
      - python-psycopg2
      - git
    tags:
    - dependencias
    
  - name: easy_install
    easy_install: name=pip
    tags:
    - pip
    
  - name: Descargar fuentes
    git: repo=https://github.com/mabarrbai/TusPachangas.git dest=~/appDAI force=yes
    tags:
    - fuentes
    
  - name: Instalar requirements
    pip: requirements=~/appDAI/requirements.txt
    tags:
    - requirements
    
  - name: Lanzar app
    command: nohup python ~/appDAI/manage.py runserver 0.0.0.0:80
    tags:
    - app
~~~

* Procedemos al despliegue completo. No es necesario crear una 'box' de Vagrant puesto que el Vagrantfile si no encuentra la 'box' especificada, la crea con la imagen indicada en el mismo Vagrantfile.
~~~
vagrant up --provider=azure
~~~
![VagrantUp1](http://i1016.photobucket.com/albums/af281/raperaco/VagrantUp1_zps3uwxshbw.png)

* Una vez alcanzado el final, podemos comprobar como la máquina virtual está en funcionamiento con la aplicación desplegada:
![VagrantUp2](http://i1016.photobucket.com/albums/af281/raperaco/VagrantUp2_zpsjhjqjo0a.png)

![VagrantfileAppDesplegada](http://i1016.photobucket.com/albums/af281/raperaco/VagrantfileAppDesplegada_zpsc8dw7dym.png)