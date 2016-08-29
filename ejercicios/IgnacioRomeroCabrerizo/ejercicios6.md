###Ejercicio 1: Instalar chef en la máquina virtual que vayamos a usar.


1. Descargamos el script de instalación de chef:

	```curl -L https://www.opscode.com/chef/install.sh | bash```

2. Comprobamos que se ha instalado chef correctamente:

![img](https://github.com/nachobit/ETSIIT/blob/master/backup/IV1516/ejercicios/tema6/chef.png)


###Ejercicio 2: Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.

1. Crear los directorios necesarios:

	- ```mkdir -p chef/cookbooks/nginx/recipes```
	- ```mkdir -p chef/cookbooks/nano/recipes```
	
2. Crear los ficheros de configuración con las recetas para instalar nginx y nano:

	- default.rb (nginx):
	
	```
	package 'nginx'
	directory "/home/nacho/Documentos/nginx"
	file "/home/nacho/Documentos/nginx/LEEME" do
    	owner "nacho"
    	group "nacho"
    	mode 00544
    	action :create
    	content "Directorio nginx"
	end
	```
	
	
	- default.rb (nano):
	
	
	```
	package 'nano'
	directory "/home/nacho/Documentos/nano"
	file "/home/nacho/Documentos/nano/LEEME" do
    	owner "nacho"
    	group "nacho"
    	mode 00544
    	action :create
    	content "Directorio nano"
	end
	```

3. Crear el archivo **node.json**:

	```
{
    "run_list":["recipe[nginx]", "recipe[nano]"]
}
	```

4. Crear el archivo **solo.rb** con las referencias a los otros archivos creados:

	```
file_cache_path "/home/nacho/chef" 
cookbook_path "/home/nacho/chef/cookbooks" 
json_attribs "/home/nacho/chef/node.json" 
	```

![img](https://github.com/nachobit/ETSIIT/blob/master/backup/IV1516/ejercicios/tema6/chef1.png)

5. Ejecutar la orden **chef**:
	```chef-solo -c solo.rb ```

![img](https://github.com/nachobit/ETSIIT/blob/master/backup/IV1516/ejercicios/tema6/chef2.png)



###Ejercicio 3: Escribir en YAML la siguiente estructura de datos en JSON


{ uno: "dos", tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }

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


###Ejercicio 4: Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.


 1. Instalar Ansible:

	``` sudo pip install paramiko PyYAML jinja2 httplib2 ansible ```
	
	* En el caso de no tener previamente la herramienta pip instalada:
	
		```sudo apt-get install python-pip python-dev build-essential ```
	
 2. Crear el fichero host añadiendo la máquina de Azure:

	```echo "maquina-nacho-ubu.cloudapp.net" > ~/ansible_hosts```

 3. Definir la variable de entorno con la ubicación del fichero:

	```export ANSIBLE_HOSTS=~/ansible_hosts```
	
 4. Iniciar la máquina virtual:

	```azure vm start maquina-nacho-ubu```
	
 5. Configurar ssh:

	``` 
	ssh-keygen -t dsa 

	ssh-copy-id -i .ssh/id_dsa.pub nacho@maquina-nacho-ubu.cloudapp.net 
	```
	
	![img](https://github.com/nachobit/ETSIIT/blob/master/backup/IV1516/ejercicios/tema6/ssh1.png)

 6. Comprobar conexión con Ansible:

	``` ansible all -u nacho -i ansible_hosts -m ping ```

	![img](https://github.com/nachobit/ETSIIT/blob/master/backup/IV1516/ejercicios/tema6/ansible.png)

 7. Instalar dependencias necesarias en la máquina:

	```
	ansible all -u nacho -m command -a "sudo apt-get install python-setuptools python-pip python-dev build-essential git -y"

	```

8. Clonar el repositorio de git en la máquina de Azure:

	```
	ansible all -u nacho -m git -a "repo=https://github.com/nachobit/DAI_bares.git dest=~/prAnsible version=HEAD"
	```
	```
	ansible all -u nacho -m command -a "sudo pip install -r prAnsible/requirements.txt"
	```

---


###Ejercicio 5. 
####5.1.Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.

 1. Definir el archivo anterior **ansible-hosts**:

	```
	[azure]	maquina-nacho-ubu.cloudapp.net
	```

 2. Crear el **playbook** de Ansible (deploy.yml):

```
- hosts: azure

  become: yes
  become_method: sudo
  remote_user: nacho

  tasks:
  - name: Instalar paquetes
    apt: name=python-setuptools state=present
    apt: name=build-essential state=present
    apt: name=python-dev state=present
    apt: name=git state=present
    
  - name: Instalar pip
    action: apt pkg=python-pip

  - name: Clonar repositorio de git
    git: repo=https://github.com/nachobit/DAI_bares.git  dest=~/DAI2 version=HEAD force=yes
  - name: Permisos de ejecucion al directorio
    file: path=~/DAI state=touch mode="u+rw"

  - name: Instalar requisitos
    shell: cd ~/DAI2 && make install
    
  - name: Abrir puerto 8000
    command: sudo fuser -k 8000/tcp
    
  - name: ejecutar
    shell: cd ~/DAI2 && make runserver
    register: errores
  
  - debug: msg="{{ errores.stdout }}"
  - debug: msg="{{ errores.stderr }}"  
  ```
	
 3. Ejecutar el **playbook**:

	```ansible-playbook -u nacho deploy.yml --ask-become-pass ```
	
	![img](https://github.com/nachobit/ETSIIT/blob/master/backup/IV1516/ejercicios/tema6/ansible.png)

__NOTA__	
Para **apagar** la máquina virtual:
	``` azure vm shutdown maquina-nacho-ubu ```

Para **encender** la máquina virtual:
	``` azure vm start maquina-nacho ubu ```

####Ejercicio 5.2: ¿Ansible o Chef? ¿O cualquier otro que no hemos usado aquí?###

**Chef** utiliza un modelo "master-Agent", y requiere una estación de trabajo para controlar el "master". **Ansible** está escrito en Python y sólo requiere que las bibliotecas de Python estén presentes en los servidores a configurar, que suele ser lo predeterminado en las distros de Linux. Además es más ligero y se implementa de forma más sencilla. Todas las comunicaciones "master-agent" se realizan por comandos SSH estándar.

###Ejercicio 6: Instalar una máquina virtual Debian usando Vagrant y conectar con ella.###

 - Instalar Vagrant en su última versión de su [web](https://releases.hashicorp.com/vagrant/):

	 ```sudo dpkg -i vagrant_1.8.1_x86_64.deb```
	 
 - Instalar VirtualBox:
 
 	```sudo apt-get install virtualbox virtualbox-dkms```
 	
 - Instalar la máquina Debian:
 
 	``` vagrant box add debian http://static.gender-api.com/debian-8-jessie-rc2-x64-slim.box ```
 	
	![img](https://github.com/nachobit/ETSIIT/blob/master/backup/IV1516/ejercicios/tema6/vg0.png) 	
 	
 - Lanzar la máquina en el directorio elegido:
 
 	``` vagrant init debian ```
 	``` vagrant up ```
	
	![img](https://github.com/nachobit/ETSIIT/blob/master/backup/IV1516/ejercicios/tema6/vg1.png)

 - Para conectar por ssh con la máquina creada:
 	```vagrant ssh ```
 	
###Ejercicio 7: Crear un script para provisionar "nginx" o cualquier otro servidor web que pueda ser útil para alguna otra práctica.###

 - Editar el archivo de configuración *Vagrantfile*:
 
	```
 	VAGRANTFILE_API_VERSION = "2"

	Vagrant.configure(2) do |config|
 	config.vm.box = "debian"

 	config.vm.provision "shell",
    inline: "sudo apt-get update && sudo apt-get install -y nginx && sudo 	service nginx start"

	end
	```
 - Ejecutar: ```vagrant provision ```
 
###Ejercicio 8: Configurar tu máquina virtual usando Vagrant con el provisionador Ansible. ###

 - Instalar Vagrant Azure Provider:
 
 	``` vagrant plugin install vagrant-azure ```
 	
	 ![img](https://github.com/nachobit/ETSIIT/blob/master/backup/IV1516/ejercicios/tema6/vazure.png)
 	
 - Instalar *VirtualBox, Vagrant y AzureCLI* en caso de no tenerlos previamente instalados como se ha visto [anteriormente](https://github.com/nachobit/IV-2015-16/blob/master/ejercicios/IgnacioRomeroCabrerizo/ejercicios5.md).
 
 - Realizar el *login* en Azure siguiendo los pasos indicados: ```azure login```
 
 - Crear el **certificado .pem** y obtener el *"id"* de nuestra cuenta en Azure para el siguiente paso:
 
 	![img](https://github.com/nachobit/ETSIIT/blob/master/backup/IV1516/ejercicios/tema6/idazure.png)
 	
 	Para el certificado:
 
 	```
 	openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout ~/Documentos/clave/azure.pem -out ~/Documentos/clave/azure.pem

	openssl x509 -inform pem -in ~/Documentos/clave/azure.pem -outform der -out ~/Documentos/clave/azure.cer
	
 	``` 	 	
 
 - Subir el certificado .cer obtenido en Azure:
 
	![img](https://github.com/nachobit/ETSIIT/blob/master/backup/IV1516/ejercicios/tema6/az1.png)
 
 - Crear el **Vagrantfile** :
 
```Vagrant.configure(2) do |config|  config.vm.box = "azure"  # Create a forwarded port mapping which allows access to a specific port  # within the machine from a port on the host machine. In the example below,  # accessing "localhost:8080" will access port 80 on the guest machine.  config.vm.network "forwarded_port", guest: 8000, host: 8000  # Create a private network, which allows host-only access to the machine  # using a specific IP.  config.vm.network "private_network", ip: "192.168.56.10", virtualbox__intnet: "vboxnet0"  # Create a public network, which generally matched to bridged network.  # Bridged networks make the machine appear as another physical device on  # your network.  config.vm.network "public_network"  config.vm.define "localhost" do |l|    l.vm.hostname = "localhost"  end    # Provider-specific configuration so you can fine-tune various  # backing providers for Vagrant. These expose provider-specific options.  # Example for VirtualBox:  #  config.vm.provider "azure" do |azure|  azure.mgmt_certificate = File.expand_path('~/Documentos/clave/azure.pem')    azure.mgmt_endpoint = 'https://management.core.windows.net'    azure.subscription_id = 'd6bbcf1a-b999-4eab-934d-cd3a395a90cf'    azure.vm_image = 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_3-LTS-amd64-server-20151218-en-us-30GB'    azure.vm_name = 'maquina-nacho-ubu'    azure.vm_password = 'Azure@Nacho16'    azure.vm_location = 'Central US'    azure.ssh_port = '22'    azure.tcp_endpoints = '8000:8000'  end  #config.ssh.username = 'nachobit'  #config.ssh.password = '1234'  # Enable provisioning with a shell script. Additional provisioners such as  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the  # documentation for more information about their specific syntax and use.  config.vm.provision "ansible" do |ansible|    ansible.sudo = true      ansible.playbook = "ansible.yml"      ansible.verbose = "v"      ansible.host_key_checking = false    endend 
       
```
  
 - Crear un "box" con una imagen de azure:
 
 	```vagrant box add azure https://github.com/msopentech/vagrant-azure/raw/master/dummy.box```
 	
 - Crear la máquina con:
 	```vagrant up --provider=azure```
 	
 __NOTA__
 
 En el caso de disponer de la máquina creada bastaría con:
 	
 	``` vagrant provision ```
