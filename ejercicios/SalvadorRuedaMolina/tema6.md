## Ejercicio 1
## Instalar chef en la máquina virtual que vayamos a usar.

Voy a instalar chef en mi máquina virtual de Ubuntu, que voy a usar más adelante para realizar los ejercicios. He instalado chef con la siguiente línea de comandos:  
``curl -L https://www.opscode.com/chef/install.sh | sudo bash
``  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/1_zpsi8gnwqw4.png)  

Para saber la versión de chef que tenemos ejecutamos:  
``chef-solo -v``  
## Ejercicio 2
## Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.  

Lo primero que debemos hacer es crear dos directorios uno para nginx y otro para nano. En dichos directorios se encontraran la receta de cada uno.  
`` mkdir chef/cookbooks/nginx/recipes``  
`` mkdir -p chef/cookbooks/nano/recipes``  
Esta debe ser la estructura para que chef pueda funcionar de forma correcta.
Ahora procedo a crear las recetas cuyo nombre será default.rb y se alojará en los directorios creados anteriormente. Me he basado en el ejemplo del tema. Su contenido para  nginx:  
```
package 'nginx'  
directory "/home/salva/Documentos/nginx"  
file "/home/salva/Documentos/nginx/LEEME" do  
    owner "salva"  
    group "salva"  
    mode 00544  
    action :create   
    content "Directorio para nginx"  
end  

```
Y para nano:
```
package 'nano'  
directory "/home/salva/Documentos/nano"  
file "/home/salva/Documentos/nano/LEEME" do  
    owner "salva"  
    group "salva"  
    mode 00544  
    action :create  
    content "Directorio para nano"  
end  
```  

Ahora deberemos crear dos ficheros en el directorio raíz de chef. El primer archivo lo llamaremos node.json y sirve para indicar las recetas que queremos ejecutar. Su contenido es el siguiente:  
``
{
    "run_list":[ "recipe[nginx]", "recipe[nano]"]
}
``

El otro archivo indica las rutas de los ficheros, se llama solo.rb y tiene las siguientes líneas:  
```
file_cache_path "/home/salva/chef"      
cookbook_path "/home/salva/chef/cookbooks"     
json_attribs "/home/salva/chef/node.json"      
``` 

Aqui podemos ver la estructura de la carpeta chef para que nos quede más claro:   
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/2_1_zpsoc8zhhfj.png)  

Ahora ya solo nos queda ejecutarlo con el siguiente comando:  
``chef-solo -c solo.rb``  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/2_2_zpsd9phnuu7.png)  
##Ejercicio 3
##Escribir en YAML la siguiente estructura de datos en JSON
##{ uno: "dos",
##tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }
```
- uno :"dos"   
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
## Ejercicio 4
## Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.

Voy a desplegar mi aplicación de DAI  en una máquina virtual local. Para ello lo primero por lo que he comenzado es por subir mi aplicación de DAI a GitHub, puede verse en el siguiente [enlace](https://github.com/srmf9/Bares).
Ahora procedemos a instalar todo lo necesario y por supuesto ansible con  el siguiente comando:  
``sudo pip install paramiko PyYAML jinja2 httplib2 ansible``  
Ahora añadimos la dirección ip al fichero ansible_hosts.  
A continuación exportamos la variable de entorno:  
``export ANSIBLE_HOSTS=~/ansible_hosts``  
Lo siguiente será configurar ssh para poder conectarnos.  
```
ssh-keygen -t dsa 
ssh-copy-id -i .ssh/id_dsa.pub salva@192.168.251.128
```  
Hacemos ping para ver si se puede conectar a la máquina virtual remota:  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/4_zpsgvs3i8dz.png)  
Ahora procedo a descargarme mi repositorio a través de ansible:  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/4_2_zpsgnqivj4b.png)  
Instalamos el fichero de dependencias de nuestra aplicación web:  
`` ansible all -u salva -m command -a "sudo pip install -r aplicacion_bares/requirements.txt"
``  
Y ya podemos ejecutar y comprobar como nuestra aplicacón esta desplegada:  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/4_3_zpstdrkpsqw.png)  

##Ejercicio 6
## Instalar una máquina virtual Debian usando Vagrant y conectar con ella.
Lo primero que debemos realizar para poder instalar una máquina virtual Debian usando Vagrant es obviamente instalar Vagrant:  
``sudo apt-get install vagrant
``  
Ahora procedo a instalar la imagen que voy  a utilizar:  
``vagrant box add debian https://github.com/holms/vagrant-jessie-box/releases/download/Jessie-v0.1/Debian-jessie-amd64-netboot.box
``  
Creamos el fichero de Vagrant:  
``vagrant init debian``  
Y levantamos la máquina:  
``vagrant up ``  
Para poder conectar con la máquina usare lo más sencillo que es ssh:  
``vagrant ssh``
##Ejercicio 8
##Configurar tu máquina virtual usando vagrant con el provisionador ansible
Tras actualizar vagrant gracias al enlace que paso por telegram Alejandro Casado pude instalar los plugin de la siguiente manera:  
``sudo vagrant plugin install vagrant-azure``  
Ahora deberemos bajarnos nuestro certificado del enlace que nos facilita el siguiente comando:   
``azure account download``  
Importamos la credencial:    
``azure account import 2-6-2016-credentials.publishsettings ``  
Generamos la clave privada:  
``openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout azurevagrant.key -out azurevagrant.key``  
Damos permiso a la clave:  
``chmod 600 ~/.ssh/azurevagrant.key``  
Y genero el .cer que será el certificado que debemos subir a azure:   
``openssl x509 -inform pem -in azurevagrant.key -outform der -out azurevagrant.cer``

imagen    

Ahora generamos el archivo .pem al que hay que añadirle la llave privada:  
``openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out azurevagrant.pem``  
``cat azurevagrant.key > azurevagrant.pem``  
Este archivo será llamado desde Vagrantfile:  
```
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = 'azure' #tipo de donde proviene el SO de la caja
  config.vm.network "public_network"
  config.vm.network "private_network",ip: "192.168.56.10", virtualbox__intnet: "vboxnet0" #Si en lugar de acceder por localhost, queremos hacerlo por IP
  config.vm.network "forwarded_port", guest: 80, host: 80 #redireccionamiento de puertos 
  config.vm.define "localhost" do |l|
          l.vm.hostname = "localhost"
   end
#Creación de la máquina de azure
  config.config.vm.network para que sirvevm.provider :azure do |azure, override|
      azure.mgmt_certificate = File.expand_path('~/.ssh/azurevagrant.pem') #ruta donde se encuentra el certificado
      azure.mgmt_endpoint = 'https://management.core.windows.net'
      azure.subscription_id = '327715f7-dcef-44ef-bf93-d88889e67cb3' #id del certificado que encontramos en la web
      azure.vm_image = 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_3-LTS-amd64-server-20151218-en-us-30GB' #imagen de la máquina virtual
      azure.vm_name = 'ubuntu-iv-eje8' #nombre de la máquina
      azure.vm_password = 'Clave#salva#1'#nombre de usuario y contraseña	
      azure.vm_location = 'Central US'  #localización
      azure.ssh_port = '22' #puerto por el que escucha ssh
      azure.tcp_endpoints = '80:80'
  end
  

  #ansible
  config.vm.provision "ansible" do |ansible|
        ansible.sudo = true #permiso de super usuario
        ansible.playbook = "provision.yml" #archivo con todos los comandos que debe ejecutarse
        ansible.verbose = "v"
        ansible.host_key_checking = false
  end
end
```
Y ahora  deberemos crear el archivo provision.yml que es el encargado de instalar todo lo necesario en nuestra máquina virtual para que se pueda levantar la aplicación. Su contenido es:  
```
- hosts: localhost
  sudo: yes
  remote_user: vagrant
  tasks:
  - name: Actualizando sistema
    apt: update_cache=yes upgrade=dist    
  - name: Instalando paquetes necesarios para la máquina virtual
    apt: name=python-setuptools state=present
    apt: name=build-essential state=present
    apt: name=python-dev state=present
    apt: name=python-pip state=present
    apt: name=git state=present
  - name: Instalando el paquete python-pip
    action: apt pkg=python-pip
  - name: Instalando postgresql
    command: sudo easy_install pip
    command: sudo apt-get install -y python-dev libpq-dev python-psycopg2  
  - name: Descargando la aplicación
    git: repo=https://github.com/srmf9/Proyecto-IV.git  dest=aplicacion-desplegada clone=yes force=yes
  - name: Dando permisos al directorio donde se descarga la aplicación
    command: chmod -R +x aplicacion-desplegada
  - name: Instalando los requisitos de la aplicación
    command: pip install -r aplicacion-desplegada/requirements.txt
  - name: Ejecutando aplicación
    command: nohup python aplicacion-desplegada/manage.py runserver 0.0.0.0:80
```
Con esto  ya tendremos todo lo necesario para desplegar la aplicación. Ahora ejecutamos:  
``sudo vagrant up --provider=azure``  
Este comando creara la máquina virtual y ejecutara el fichero de ansible. Si ya tenemos la máquina creada nos bastara con ejecutar:  
`` sudo vagrant provison``  
Aqui podemos como se ha creado la máquina virtual en azure que cree en el Vangrantfile y la aplicación funcionando:  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/8_1_zpsgcz7zomy.png)  
![imagen](http://i1028.photobucket.com/albums/y349/Salva_Rueda/8_2_zpstth3shfa.png)  
Para poder borrar todo lo que vagrant ha creado se hara con la siguiente línea:   
``vagrant box remove <ubuntu-iv-eje8>``