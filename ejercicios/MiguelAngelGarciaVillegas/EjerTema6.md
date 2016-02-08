# Asignatura: IV Grupo de Practicas Miercoles 8:30h a 10:30h
## Miguel Ángel García Villegas

# Ejercicios Tema 6

## Hay varios tutoriales que te permiten, con relativa rapidez, comenzar a trabajar con Chef-solo en un servidor; este te proporciona una serie de ficheros que puedes usar y este otro es más directo, dando una serie de órdenes. En todo caso, se trata básicamente tener acceso a un servidor o máquina virtual, instalar una serie de aplicaciones en él y ejecutarlas sobre un fichero de configuración

## Ejercicio 1.
### Instalar chef en la máquina virtual que vayamos a usar.

Una forma más rápida de instalar Chef es descargarlo directamente desde la página web.

Así que ejecutamos en el terminal: ***curl -L https://www.opscode.com/chef/install.sh | sudo bash***

Para comprobar que se ha instalado y la versión que tenemos instalada ejecutamos: ***chef-solo -v***

![Chef en ubuntu](https://www.dropbox.com/s/wlkfx0wotff4sqn/01.1.png?dl=1)

## Ejercicio 2.
### Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.

Vamos a crear los directorios en nuestro caso para ***nginx*** y ***nano***

Ejecutamos en el terminal ***mkdir -p chef/cookbooks/nginx/recipes***
y ***mkdir -p chef/cookbooks/nano/recipes***

Ahora en dichos directorios vamos a meter el archivo ***default.rb*** indicando en nuestro caso, para nginx.

default.rb para nginx:

    package 'nginx'
    directory "/home/magvugr/Documentos/nginx"
    file "/home/magvugr/Documentos/nginx/LEEME" do
        owner "magvugr"
        group "magvugr"
        mode 00544
        action :create
        content "Directorio para nginx"
    end

Ahora creamos el archivo ***node.json***

    {
        "run_list":["recipe[nginx]", "recipe[nano]"]
    }

Y finalmente el archivo ***solo.rb***

    file_cache_path "/home/magvugr/chef"
    cookbook_path "/home/magvugr/chef/cookbooks"
    json_attribs "/home/magvugr/chef/node.json"


Quedando nuestros archivos de la siguiente forma:

![archivos](https://www.dropbox.com/s/hgd4e1qqcyiyeeh/02.1.png?dl=1)

Ahora ejecutamos en el terminal ***sudo chef-solo -c chef/solo.rb*** para la instalación de los paquetes.

![chef-solo -c chef/solo.rb](https://www.dropbox.com/s/iogmk4anvw2ufwm/02.2.png?dl=1)
![chef-solo -c chef/solo.rb](https://www.dropbox.com/s/fw2150x6l9b3vh3/02.3.png?dl=1)

Una vez acabado, lo que hacemos es comprobar de nuevo, con el comando ***tree*** los archivos y directorios.

![archivos](https://www.dropbox.com/s/akrl4wud08ipdyl/02.4.png?dl=1)

Comprobamos que el estado de nginx y vemos que está corriendo.
![nginx is running](https://www.dropbox.com/s/qnykas8d1tptlj9/02.6.png?dl=1)

Podemos consultar los resultados

![resultados](https://www.dropbox.com/s/9ubufr9krqs82fk/02.5.png?dl=1)

Hemos añadido una receta más para geany,

Ejecutamos en el terminal ***mkdir -p chef/cookbooks/geany/recipes***

Ahora en dichos directorios vamos a meter el archivo ***default.rb*** indicando en nuestro caso, para geany.

default.rb para geany:

    package 'geany'
    directory "/home/magvugr/Documentos/geany"
    file "/home/magvugr/Documentos/geany/LEEME" do
        owner "magvugr"
        group "magvugr"
        mode 00544
        action :create
        content "Ejer2: Directorio para Geany"
    end

Ahora añadimos al archivo ***node.json***

    {
        "run_list":["recipe[nginx]", "recipe[nano]", "recipe[geany]"]
    }

Y finalmente ejecutamos:


Ahora ejecutamos en el terminal ***sudo chef-solo -c chef/solo.rb*** para la instalación de los paquetes.

![chef-solo -c chef/solo.rb](https://www.dropbox.com/s/tealpisj6mzuypw/ejer2.7.png?dl=1)
![chef-solo -c chef/solo.rb](https://www.dropbox.com/s/gjwpttag5ykwa1m/2.8.png?dl=1)

Una vez acabado, lo que hacemos es comprobar de nuevo, con el comando ***tree*** los archivos y directorios.

![archivos](https://www.dropbox.com/s/qy5xlbgslxo92br/2.9.png?dl=1)

Podemos consultar los resultados

![resultados](https://www.dropbox.com/s/6ax8kuni2yfl7pp/2.10.png?dl=1)


## Ejercicio 3.
### Escribir en YAML la siguiente estructura de datos en JSON, { uno: "dos",  tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }

Formato JSON
***{ "uno": "dos", "tres": [ 4 , 5 ,"Seis" ,{ "siete":8, "nueve":[10,11]}]}***


![Json to Yaml](https://www.dropbox.com/s/goyaicw50kc8t4s/Ejer3.png?dl=1)


## Ejercicio 4.
### Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.

Mi archivo Vagrantfile
#Instalar plugin azure

    Vagrant.configure('2') do |config|
        config.vm.box = 'azure'
        config.vm.network "forwarded_port", guest: 80, host: 80
        config.vm.provider :azure do |azure, override|
            # Mandatory Settings
            azure.mgmt_certificate = File.expand_path('azure.pem')
            azure.mgmt_endpoint    = 'https://management.core.windows.net'
            azure.subscription_id = 'xxx'
            azure.vm_name     = 'insertalogo'
            azure.vm_image    = 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_2-LTS-amd64-server-20150506-en-us-30GB'
            azure.vm_size     = 'Small'
            config.vm.box_url = 'https://github.com/msopentech/vagrant-azure/raw/master/dummy.box'
            azure.cloud_service_name = 'insertalogo'

    	# Optional Settings
            azure.vm_user = 'ma' # defaults to 'vagrant' if not provided
            azure.vm_password = 'xxx'
            azure.vm_location = 'North Europe' # e.g., West US
            azure.ssh_port             = '22'
            azure.tcp_endpoints = '80:80'
          end
          config.ssh.username = 'ma'
          config.ssh.password = 'xxx'
          config.vm.synced_folder ".", "/vagrant",disabled:true #Evitar subir la copia local

    # Instalacion ansible sudo pip install ansible
    #Ansible
      config.vm.provision "ansible" do |ansible|
        ansible.sudo = true
        ansible.raw_arguments = ["-vvvv"] #Depurando errores
        ansible.force_remote_user = false
        ansible.playbook = "ansible/webservice.yml"
        #ansible.playbook = ".vagrant/provisioners/ansible/inventory/webservice.yml"
      end
    end

Mi archivo Ansible

    ---
    - hosts: default
      user: ma
      sudo: yes

      tasks:
        #Instalacion requisitos
      - name: Actualizar
        apt: update_cache=yes

      - name: Instalar buildessential
        apt: name=build-essential state=present

      - name: Install git
        apt: name=git state=present

      - name: Borrar si Actualiza
        shell: rm -rf InsertaLogo

      - name: Clonar InsertaLogo
        git: repo=https://github.com/magvugr/InsertaLogo.git dest=InsertaLogo clone=yes force=yes

      - name: InstalarPip
        apt: name=python-pip state=present

      - name: InstalarPip Dev
        apt: name=python-dev state=present

      - name: InstalarPip libpq-dev
        apt: name=libpq-dev state=present

      - name: Install
        shell: cd InsertaLogo/ && make installa


Ejecutamos en el terminal ***vagrant up --provider=azure***
y cuando se crea la máquina automáticamente vagrant, usa el fichero de ansible en mi caso llamado ***webservice.yml*** para instalar todo lo necesario en nuestra máquina virtual instalada en azure.

## Ejercicio 5.1
### Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.

Para mi práctica hito 5, he utilizado el siguiente playbook.

    ---
    - hosts: default
      user: magvugr
      sudo: yes

      tasks:
        #Instalacion requisitos
      - name: Actualizar
        apt: update_cache=yes

      - name: Instalar buildessential
        apt: name=build-essential state=present

      - name: Install git
        apt: name=git state=present

      - name: Borrar si Actualiza
        shell: rm -rf InsertaLogo

      - name: Clonar InsertaLogo
        git: repo=https://github.com/magvugr/InsertaLogo.git dest=InsertaLogo clone=yes force=yes

      - name: InstalarPip
        apt: name=python-pip state=present

      - name: InstalarPip Dev
        apt: name=python-dev state=present

      - name: Install
        shell: cd InsertaLogo/ && make installa


Una vez creado dicho archivo ejecutamos en el terminal:
***ansible-playbook -i hosts playbook.yml --sudo --verbose***


## Ejercicio 5.2
### ¿Ansible o Chef? ¿O cualquier otro que no hemos usado aquí?.

Yo decido ansible, me ha parecido muy fácil e intuitivo para utilizarlo de forma cómoda y fácil.

Organiza muy bien las tareas mientras que chef, lo he visto muy interesante pero menos intuitivo debido a su jerarquía de directorios.

Realmente depende de para qué lo vayas utilizar te interesará más una cosa u otra, pero dado que los dos son para un mismo fin, me ha parecido más interesante ansible.

## Ejercicio 6.
### Instalar una máquina virtual Debian usando Vagrant y conectar con ella.

Debido a diferentes pruebas, al instalar Vagrant por el terminal ejecutando el comando ***sudo apt-get install vagrant***, esto instala la versión de Vagrant 1.4.3, lo que ocurre es que se requiere para nuestra práctica una versión posterior a la 1.6. Debemos de descargar la versión que queremos e instalarla en mi caso:

Descargo [vagrant_1.8.1_x86_64.deb](https://releases.hashicorp.com/vagrant/1.8.1/vagrant_1.8.1_x86_64.deb) y lo instalo ejecutando:
***sudo dpkg -i vagrant_1.8.1_x86_64.deb***

![instalando vagrant 1.8.1](https://www.dropbox.com/s/rvwgqwkdz8p7ul7/ejer6.png?dl=1)

Ahora elegimos una distribución [Debian](http://www.vagrantbox.es/) en mi caso: ***Debian Jessie 8.0 RC2 64-bit***

Ahora instalamos la versión de debian, ejecutando en el terminal:
``vagrant box add debian http://static.gender-api.com/debian-8-jessie-rc2-x64-slim.box``

![instalando debian vagrant](https://www.dropbox.com/s/g7ms810hvmtn99t/ejer6.2.png?dl=1)

Una vez que termine vamos a ejecutar ***vagrant init debian*** ésto creará en el repositorio un archivo ***Vagrantfile***

![Vagrantfile](https://www.dropbox.com/s/yjztqcn8fgwaczx/ejer6.3.png?dl=1)

Ahora ejecutamos ***vagrant up***

![Vagrantfile](https://www.dropbox.com/s/9rtmy8eujsm1zbg/ejer6.7.png?dl=1)

Vamos a nuestro VirtualBox, y comprobamos que tenemos nuestra máquina virtual debian que acabamos de crear con vagrant.

Vamos a conectarnos con ssh, ejecutando en el terminal ***vagrant ssh*** introducimos vagrant como usuario y contraseña y entramos.

![Vagrantfile](https://www.dropbox.com/s/hy0tfdyz7mjoyi0/eje6.9.png?dl=1)

También he comprobado que si apago la máquina desde VirtualBox, y comprobamos que automáticamente nos desconecta.

![Vagrantfile](https://www.dropbox.com/s/sw9sz2dyt8kvhi8/Ejer6.10.png?dl=1)


## Ejercicio 7.
### Crear un script para provisionar `nginx` o cualquier otro servidor web que pueda ser útil para alguna otra práctica

He creado un script ningx.sh con el siguiente contenido


    #!/bin/bash
    sudo apt-get update && apt-get install -y nginx
    sudo service nginx restart && sudo service nginx status

Vamos a crear el archivo vagrant con el siguiente contenido


    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    Vagrant.configure(2) do |config|
      config.vm.box = "debian"

      config.vm.provision "shell",
      	path: "nginx.sh"
    end

Después de tener los archivos ejecutamos en el terminal, ***vagrant up*** para que nos cree la máquina con el servicio
indicado en el script.

![mv creada](https://www.dropbox.com/s/fc439frzq5wyvny/Ejer7.1.png?dl=1)

## Ejercicio 8.
### Configurar tu máquina virtual usando vagrant con el provisionador ansible

Para mi práctica hito 5, utilizo vagrant, azure y ansible. Éste es mi fichero vagrant donde ejecutando dicho vagrant, crea nuestra máquina en azure mediante ansible.


    #Instalar plugin azure

    Vagrant.configure('2') do |config|
        config.vm.box = 'azure'

        config.vm.provider :azure do |azure, override|
            # Mandatory Settings
            azure.mgmt_certificate = File.expand_path('azure.pem')
            azure.mgmt_endpoint    = 'https://management.core.windows.net'
            azure.subscription_id = 'xxx'
            azure.vm_name     = 'insertalogo'
            azure.vm_image    = 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_2-LTS-amd64-server-20150506-en-us-30GB'
            azure.vm_size     = 'Small'
            config.vm.box_url = 'https://github.com/msopentech/vagrant-azure/raw/master/dummy.box'

    	# Optional Settings
            azure.vm_user = 'xxx' # defaults to 'vagrant' if not provided
            azure.vm_password = 'xxx'
            azure.vm_location = 'North Europe' # e.g., West US
            azure.ssh_port             = '22'
          end
          config.ssh.username = 'xxx'
          config.ssh.password = 'xxx'
          config.vm.synced_folder ".", "/vagrant",disabled:true #Evitar subir la copia local

    # Instalacion ansible sudo pip install ansible
    #Ansible
      config.vm.provision "ansible" do |ansible|
        ansible.sudo = true
        #ansible.raw_arguments = ["-vvvv"] #Depurando errores
        ansible.force_remote_user = false
        ansible.playbook = "ansible/webservice.yml"
        #ansible.playbook = ".vagrant/provisioners/ansible/inventory/webservice.yml"
      end
    end



Resultado
![Maquina en Azure](https://www.dropbox.com/s/rwqz7h0pcosj7zm/Iniciando%20.png?dl=1)

Y también he creado un archivo de fabric:

      from fabric.api import task, run, local, hosts, cd, env

      def informacion_sistema():
          run('uname -a')

      ########   Docker y descargar imagen Inserta Logo
      def getdocker():
      	run('sudo apt-get update')
      	run('sudo apt-get install -y docker.io')
      	run('sudo docker pull magvugr/insertalogo')

      ########   Ejecutamos Docker
      def rundocker():
      	run('sudo docker run -i -t magvugr/insertalogo')

      ########   Clonamos repositorio
      def getapp():
      	run('sudo apt-get update')
      	run('sudo apt-get install -y git')
      	run('sudo git clone https://github.com/magvugr/InsertaLogo.git')

      ########   Install del make
      def install():
      	run('cd InsertaLogo && make install')

      ########   Test
      def test():
      	run('cd InsertaLogo && make test')

      ########   Ejecucion app
      def run_app():
      	run('cd InsertaLogo && sudo python manage.py runserver 0.0.0.0:80')

      def run_app_2():
          run('cd InsertaLogo && sudo gunicorn -b 0.0.0.0:80 InsertaLogo.wsgi --log-file - &')

      ########   Borrar
      def remove():
          run('sudo rm -r InsertaLogo')
