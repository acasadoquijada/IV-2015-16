# Tema 6
### Ejercicio 1:Instalar chef en la máquina virtual que vayamos a usar.

Para instalarlo:

    curl -L https://www.opscode.com/chef/install.sh | sudo bash

Para comprobar si fue bien:

    chef-solo -v

### Ejercicio 2: Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.

Creamos los directorios:

    mkdir -p chef/cookbooks/nginx/recipes
    mkdir -p chef/cookbooks/geany/recipes

Los ficheros default.rb:

    chef/cookbooks/nginx/recipes/default.rb:
        package 'nginx'

    chef/cookbooks/geany/recipes/default.rb:
        package 'geany'
        directory '/home/jesus/Documents/geany'
        file "/home/jesus/Documents/geany/LEEME" do
            owner "jesus"
            group "jesus"
            mode 00644
            action :create
            content "Directorio Geany"
        end

Crear el fichero node.json:

    {
        "run_list":["recipe[nginx]", "recipe[geany]"]
    }

Fichero de configuración solo.rb:

    file_cache_path "/home/jesus/chef"
    cookbook_path "/home/jesus/chef/cookbooks"
    json_attribs "/home/jesus/chef/node.json"

Finalmente:

    sudo chef-solo -c chef/solo.rb 

### Ejercicio 3:Escribir en YAML la siguiente estructura de datos en JSON

JSON
    
    { 
        uno: "dos",
        tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] 
    }

YAML

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

### Ejercicio 4 y 5
####Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible. 
####Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.

Vagrantfile:

    # -*- mode: ruby -*-
    # vi: set ft=ruby :
     
    # Vagrant on AWS 

     
    Vagrant.configure(2) do |config|
     
      config.vm.box = "dummybox-aws"

      config.vm.hostname = "try2learn"
      config.vm.provider :aws do |aws, override|
     
        #AWS Settings
        aws.access_key_id = "accesskey"
        aws.secret_access_key = "secret"
        aws.region = "us-west-2"
     
        aws.tags = {
          'Name' => 'Try-2-Learn',
          'Team' => 'Try-2-Learn',
          'Status' => 'active'
        }
     
        #Override Settings
        override.ssh.username = "ubuntu"
        override.ssh.private_key_path = "try-2-learn.pem"
     
        aws.region_config "us-west-2" do |region|
          region.ami = 'ami-35143705'
          region.instance_type = 't2.micro'
          region.keypair_name = "try-2-learn"
          region.security_groups = "launch-wizard-3"
        end

        #Provisionamiento
        config.vm.provision :ansible do |ansible|  
            ansible.playbook = "playbook.yml"
            ansible.limit = 'all'
            ansible.verbose = "vv"
        end 
      end
    end


Playbook:

    ---
    - hosts: default
      remote_user: ubuntu
      sudo: true

      tasks:
        - name: Add apt-key for docker
          command: apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

        - name: Add apt-repository
          shell: echo 'deb https://apt.dockerproject.org/repo ubuntu-trusty main' >> /etc/apt/sources.list.d/docker.list

        - name: apt-get update
          apt: update_cache=yes
       
        - name: apt policy for docker
          command: apt-cache policy docker-engine 

        - name: apt-get update
          apt: update_cache=yes

        - name: Instalar curl
          apt: name=curl state=present
        - name: Instalar build-essential
          apt: name=build-essential state=present
        - name: Instalar git
          apt: name=git state=present
        - name: Instalar nodejs
          apt: name=nodejs state=present
        - name: Instalar npm
          apt: name=npm state=present
        - name: Instalar docker
          apt: name=docker-engine state=present

        - name: Clone Try-2-Learn
          git: repo=https://github.com/jesusgn90/Try-2-Learn.git  dest=/home/ubuntu/Try-2-Learn

    #    - name: npm install
    #      shell: cd /home/ubuntu/Try-2-Learn && npm install

    #    - name: Runs docker
    #      shell: nohup docker -d 

    #    - name: Runs app
    #      shell: nohup nodejs /home/ubuntu/Try-2-Learn/bin/www 

Orden:

    vagrant up --provider=aws

##### ¿Ansible o Chef? ¿O cualquier otro que no hemos usado aquí?.

La verdad Ansible me ha encantado, es super cómodo de usar, existe completa documentación y gran comunidad y su forma de uso mediante playbooks es clara y concisa.

