##Ejercicios 1: Instalar chef en la máquina virtual que vayamos a usar
He instalado chef con el comando:
```
curl -L https://www.opscode.com/chef/install.sh | sudo bash
```
Aquí podemos ver el chef instalado correctamente mediante el comando chef-solo -v:

![chefinstalado](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-03%20122606_zpse0mbwj3e.png)

##Ejercicios 2: Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.
Lo primero que he hecho ha sido crear los directorios donde irán las recetas de chef. Uno para nginx y otro para nano que es mi editor. 
```
mkdir -p chef/cookbooks/nginx/recipes
mkdir -p chef/cookbooks/nano/recipes

```
Lo siguiente será configurar los ficheros de las recetas de nginx y de nano. El nombre del fichero será "default.rb" y habrá uno en cada directorio de cada aplicación.
**chef/cookbooks/nginx/recipes/default.rb:**
```
package 'nginx'
directory "/home/pgazquez/Documentos/nginx"
file "/home/pgazquez/Documentos/nginx/LEEME" do
    owner "pgazquez"
    group "pgazquez"
    mode 00544
    action :create
    content "Directorio para nginx"
end
```

**chef/cookbooks/nano/recipes/default.rb:**
```
package 'nano'
directory "/home/pgazquez/Documentos/nano"
file "/home/pgazquez/Documentos/nano/LEEME" do
    owner "pgazquez"
    group "pgazquez"
    mode 00544
    action :create
    content "Directorio para nano"
end

```
Si no tenemos la carpeta Documentos la creamos. Lo siguiente es definir el archivo node.json que irá en el directorio chef/ y que tendrá la lista de recetas a ejecutar-

**chef/node.json**
```
{
    "run_list":["recipe[nginx]", "recipe[nano]"]
}

```
El último fichero que necesitamos para este ejercicio es el fichero de configuración "solo.rb" que irá también en el directorio chef y que incluirá las referencias pertinentes a los ficheros creados anteriormente y el directorio que los contiene.

**chef/solo.rb*
```
file_cache_path "/home/pgazquez/chef" 
cookbook_path "/home/pgazquez/chef/cookbooks" 
json_attribs "/home/pgazquez/chef/node.json"

```

En esta imagen vemos como queda el árbol del directorio chef con la orden tree:

![treechef](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-03%20130524_zpsdbwbnqqn.png)

Ahora queda instalar los paquetes con la orden **sudo chef-solo -c chef/solo.rb** y comprobar que todo se ha instalado correctamente:

![todocorrecchefsolo](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-03%20130540_zps2zhfjqtu.png)

Y ha instalado todo correctamente como podemos ver a continuación:

![cheftodocorre](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-03%20130646_zpsl57y3vga.png)


##Ejercicios 3: Escribir en YAML la siguiente estructura de datos en JSON
```
{ uno: "dos",
  tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] } 

```

YAML y JSON son estándares de representaciones de datos como XML, que hacen más fácil el intercambio de información entre servicios y aplicaciones. Aquí la estructura en YAML:

```
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
```

##Ejercicios 4: Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.
Primero instalamos Ansible:
```
sudo pip install paramiko PyYAML jinja2 httplib2 ansible
```
Luego creamos el inventario añadiendo la máquina virtual de Azure:
```
echo "ubuntu-pgazquez.cloudapp.net" > ~/ansible_hosts
```
Y definimos la variable de entorno para que Ansible sepa donde está el fichero de hosts:
```
export ANSIBLE_HOSTS=~/ansible_hosts
```
Lo que hacemos ahora es arrancar la máquina azure del tema anterior, previo logeándonos en azure:
```
azure login
azure vm start ubuntu-pgazquez
```
Luego configuramos SSH generando las claves para conectar con la máquina sin necesidad de autenticarnos cada vez:
```
ssh-keygen -t dsa
ssh-copy-id -i .ssh/id_dsa.pub pgazquez@ubuntu-pgazquez.cloudapp.net
```
Como se puede ver efectivamente después de configurar SSH no nos debería de pedir el password:
![sshazuresin](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-03%20142350_zpsy2fonazf.png)

Ahora comprobamos que hay conexión con Ansible:
```
ansible all -u pgazquez -m ping
```
Y comprobamos que efectivamente hay:
![pingAzure](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-03%20142851_zps9zl3jenn.png)

Lo siguiente es instalar las dependencias neccesarias en la máquina para empezar a arrancar nuestra app:
```
ansible all -u pgazquez -a "sudo apt-get install -y python-setuptools python-dev build-essential git"
ansible all -u pgazquez -m command -a "sudo easy_install pip" 
```
Una vez hecho esto descargamos la aplicación desde nuestro repositorio  de GitHub:
```
ansible all -u pgazquez -m git -a "repo=https://github.com/pedrogazquez/appBares.git  dest=~/pruebaDAI version=HEAD"
```
En la siguiente captura vemos que el repositorio se ha copiado correctamente:
![repoenAzure](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-03%20160827_zpsts3m5kmv.png)

A la hora de instalar los requirements de mi app he tenido problemas con Pillow, así que antes de instalar los requirements ejecutamos el siguiente comando para que no nos de problemas y seguidamente los requirements:
```
ansible all -m shell -a 'sudo apt-get install -y libtiff4-dev libjpeg8-dev zlib1g-dev libfreetype6-dev liblcms1-dev libwebp-dev'
ansible all -u pgazquez -m command -a "sudo pip install -r pruebaDAI/requirements.txt"
```
Ya está todo preparado para ejecutar nuestra app así que procedemos a ello, antes tenemos que liberar el puerto 80 para ejecutarla:
```
ansible all -u pgazquez -m command -a "sudo fuser -k 80/tcp"
ansible all -u pgazquez -m command -a "sudo python pruebaDAI/manage.py runserver 0.0.0.0:80"
```
![arrancandoEnAzure](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-04%20191638_zpsgidqkwec.png)

Y vemos que todo funciona correctamente en el navegador:

![appAzureCorriendo](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-04%20190342_zpsejj0nxrs.png)

Por último y como siempre "apagamos" la máquina de azure:
```
azure vm shutdown ubuntu-pgazquez
```

##Ejercicios 5: 1.Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.
Lo primero que hacemos es añadir lo siguiente en el archivo **ansible_hosts**:

![archivoansihosts](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-04%20194256_zps8o5b7ync.png)

Luego definimos el archivo .yml que en mi caso se llama **scriptansible.yml** y contiene lo siguiente:
```
- hosts: azure
  sudo: yes
  remote_user: pgazquez
  tasks:
  - name: Instalar paquetes 
    apt: name=python-setuptools state=present
    apt: name=build-essential state=present
    apt: name=python-dev state=present
    apt: name=git state=present
  - name: Descargar aplicacion de GitHub
    git: repo=https://github.com/pedrogazquez/appBares.git dest=appBares clone=yes force=yes
  - name: Permisos de ejecucion
    command: chmod -R +x appBares
  - name: Instalar requisitos
    command: sudo pip install -r appBares/requirements.txt
  - name: ejecutar
    command: nohup sudo python appBares/manage.py runserver 0.0.0.0:5050

```
Y lo he ejecutado con la orden **ansible-playbook -u pgazquez scriptansible.yml** y este ha sido el resultado:

![contenidoplaybook-ansible](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-04%20230742_zpslrk2dbxe.png)

Como resultado la aplicación se ha ejecutado correctamente como el ejercicio anterior.
##5.2¿Ansible o Chef? ¿O cualquier otro que no hemos usado aquí?.
Para mi Ansible es mejor que Chef por dos razones: una que se puede ejecutar de manera remota desde fuera del servidor y otra porque los playbooks son más fáciles de realizar y configurar que las recetas de chef en las cuales es necesario además una jerarquización de directorios. Como contrapartida chef es más rápido.

##Ejercicios 6: Instalar una máquina virtual Debian usando Vagrant y conectar con ella.
Para realizar este ejercicio lo primero que he hecho ha sido instalar vagrant:
```
sudo apt-get install vagrant
```
Descargamos el archivo [vagrant_1.8.1_x86_64.deb](https://releases.hashicorp.com/vagrant/1.8.1/) para una versión 5.0.x de VirtualBox y lo instalamos:
```
sudo dpkg -i vagrant_1.8.1_x86_64.deb
```
Lo siguiente es descargar la imagen Debian como se ha explicado:
```
vagrant box add debian https://github.com/holms/vagrant-jessie-box/releases/download/Jessie-v0.1/Debian-jessie-amd64-netboot.box
```

![vagrantbox](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-04%20234853_zpsxlqy3ra5.png)

Creamos el fichero Vagrantfile:
```
vagrant init debian
```
Y arrancamos la máquina:
```
vagrant up
```

![vagrantup](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-04%20235142_zpshy94hyui.png)

Y para terminar conectamos con ella por SSH:
```
vagrant ssh
```

![vagrantssh](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-04%20235209_zpsld4tyu1r.png)

##Ejercicios 7: Crear un script para provisionar `nginx` o cualquier otro servidor web que pueda ser útil para alguna otra práctica
He añadido en el fichero **Vagrantfile** lo siguiente:
```
config.vm.provision "shell",
	inline: "sudo apt-get update && sudo apt-get install -y nginx && sudo service nginx start"
```
Arrancamos la máquina con vagrant up y posteriormente ejecutamos la provisión con **vagrant provision**:

![vagrantprovision](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-05%20101153_zpswvaodxir.png)

Conectamos por ssh como antes (**vagrant ssh**) y luego comprobamos que nginx se ha instalado y está funcionando:
```
sudo service nginx status
```

![nginxvagrant](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-05%20101250_zps1hepxrat.png)

##Ejercicios 8: Configurar tu máquina virtual usando vagrant con el provisionador ansible

Lo primero que hacemos es instalar el provisionador de azure para Vagrant:
```
vagrant plugin install vagrant-azure
```
![pluginazure](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-07%20220453_zpsggqexnuv.png)
Después de esto nos logueamos en azure como anteriormente (azure login) y hacemos **azure account doownload**  esto nos proporciona un enlace al que deberemos acceder para descargar nuestros credenciales y una vez descargados hacemos:
```
azure account import Azure-2-5-2016-credentials.publishsettings
```
Lo siguiente que hacemos es generar los certificados que se subirán a Azure para perminirnos realizar las conexiones con nuestra máquina:
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout azurevagrant.key -out azurevagrant.key
chmod 600 azurevagrant.key
openssl x509 -inform pem -in azurevagrant.key -outform der -out azurevagrant.cer
```
Ahora subimos el fichero **azurevagrant.cer** a Azure en el apartado Management Certificates:
![managcert](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-05%20164823_zpskywjh3fa.png)

Para autenticar Azure desde Vagranfile creamos un archivo .pem y lo concatemos el .key:
```
openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out azurevagrant.pem
cat azurevagrant.key > azurevagrant.pem
```
Luego creamos el fichero Vagranfile:
```
VAGRANTFILE_API_VERSION = '2'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = 'azure'
  config.vm.network "public_network"
  config.vm.network "private_network",ip: "192.168.56.10", virtualbox__intnet: "vboxnet0"
  config.vm.network "forwarded_port", guest: 80, host: 80
  config.vm.define "localhost" do |l|
    l.vm.hostname = "localhost"
  end

  config.vm.provider :azure do |azure|
    azure.mgmt_certificate = File.expand_path('/home/pgazquez/Escritorio/appdefinitiva/azurevagrant.pem')
    azure.mgmt_endpoint = 'https://management.core.windows.net'
    azure.subscription_id = '686945bb-88d5-4919-bd0d-d2e0aeb76c29'
    azure.vm_image = 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_3-LTS-amd64-server-20151218-en-us-30GB'
    azure.vm_name = 'baresquesada'
    azure.vm_user = 'vagrant'
    azure.vm_password = 'Pedro#1992'
    azure.vm_location = 'Central US' 
    azure.ssh_port = '22'
    azure.tcp_endpoints = '80:80'
  end

  config.vm.provision "ansible" do |ansible|
    ansible.sudo = true
    ansible.playbook = "ansiblebares.yml"
    ansible.verbose = "v"
    ansible.host_key_checking = false 
  end
end
```
En este Vagranfile en el primer bloque indicamos la imagen de la caja, el id de subscripción de azure, el usario el password, el puerto SSH, etc. En el segundo bloque se definen las características que tendrá nuestra máquina y por último en el tercero indicar que provisione la máquina mediante ansible y el nombre del fichero yml (ansiblebares.yml en mi caso).
Ahora definimos la variable de entorno ANSIBLE_HOSTS:
```
export ANSIBLE_HOSTS=~/Escritorio/VagrantIV/ansible_hosts
```
Ahora añadimos lo siguiente en el archivo ansible_host:
```
[localhost]
192.168.56.10
```
Y definimos el archivo ansiblebares.yml:
```
- hosts: azure
  sudo: yes
  remote_user: vagrant
  tasks:
  - name: Instalar paquetes 
    apt: name=python-setuptools state=present
    apt: name=build-essential state=present
    apt: name=python-dev state=present
    apt: name=git state=present
  - name: Descargar aplicacion de GitHub
    git: repo=https://github.com/pedrogazquez/appBares.git dest=appBares clone=yes force=yes
  - name: Permisos de ejecucion
    command: chmod -R +x appBares
  - name: Instalar requisitos
    command: sudo pip install -r appBares/requirements.txt
  - name: ejecutar
    command: nohup sudo python appBares/manage.py runserver 0.0.0.0:80
```

Este instala los paquetes necesarios para ejecutar nuestra app después de descargar el repo de esta.
Ahora descargamos la box de azure con el siguiente comando:
```
vagrant box add azure https://github.com/msopentech/vagrant-azure/raw/master/dummy.box

```
![dummybox](http://i1042.photobucket.com/albums/b422/Pedro_Gazquez_Navarrete/Captura%20de%20pantalla%20de%202016-02-05%20165928_zps2kqtfjqy.png)

Y ahora podemos proceder a ejecutar provider para que cree la app con el siguiente comando:
```
vagrant up --provider=azure
vagrant provision (si la máquina ya está creada)
```
Y una vez ejecutado se puede acceder a [nuestra app de azure](http://baresquesada.cloudapp.net/rango/)
