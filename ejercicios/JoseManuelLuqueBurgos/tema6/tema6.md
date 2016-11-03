# Ejercicios Tema 6

## Ejercicio 3
### Escribir en YAML la siguiente estructura de datos en JSON "{ uno: "dos", tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }".

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
### Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.

Vamos a partir de la base en la que ya tenemos todo lo necesario configurado en azure para realizar la instalación (máquina virtual creada y lanzada, acceso ssh...)

El primer paso es instalar los paquetes necesarios de ansible, en arch nos instala todas las dependencias con el paquete *ansible*:

```
jose@tux2duo ⮀ ~/iv ⮀ ⭠ master ⮀ yaourt -S ansible
resolving dependencies...
looking for conflicting packages...

Packages (11) python2-crypto-2.6.1-3  python2-ecdsa-0.13-2  python2-jinja-2.8-2  python2-markupsafe-0.23-3  python2-packaging-16.0-1  python2-paramiko-1.16.0-1
              python2-pyparsing-2.0.7-1  python2-setuptools-1:19.7-1  python2-six-1.10.0-1  python2-yaml-3.11-3  ansible-2.0.0.2-2

Total Installed Size:  27.19 MiB

:: Proceed with installation? [Y/n] y
(11/11) checking keys in keyring                                                                       [############################################################] 100%
(11/11) checking package integrity                                                                     [############################################################] 100%
(11/11) loading package files                                                                          [############################################################] 100%
(11/11) checking for file conflicts                                                                    [############################################################] 100%
(11/11) checking available disk space                                                                  [############################################################] 100%
:: Processing package changes...
( 1/11) installing python2-yaml                                                                        [############################################################] 100%
==> Note that even though this package uses libyaml library,
==> slower pure python implementation is used by default.
==> See http://pyyaml.org/wiki/PyYAMLDocumentation
( 2/11) installing python2-crypto                                                                      [############################################################] 100%
( 3/11) installing python2-ecdsa                                                                       [############################################################] 100%
( 4/11) installing python2-paramiko                                                                    [############################################################] 100%
( 5/11) installing python2-pyparsing                                                                   [############################################################] 100%
( 6/11) installing python2-packaging                                                                   [############################################################] 100%
( 7/11) installing python2-six                                                                         [############################################################] 100%
( 8/11) installing python2-setuptools                                                                  [############################################################] 100%
( 9/11) installing python2-markupsafe                                                                  [############################################################] 100%
(10/11) installing python2-jinja                                                                       [############################################################] 100%
(11/11) installing ansible                                                                             [############################################################] 100%
Optional dependencies for ansible
    python2-passlib: crypt values for vars_prompt

```

Creamos el archivo **ansible_hosts** con el siguiente contenido:

```
rsmap.cloudapp.net
```

Y establecemos la variable de entorno apuntándolo
```
jose@tux2duo ⮀ ~ ⮀ export ANSIBLE_HOSTS=~/ansible_hosts
```

Probamos que ansible pueda conectarse correctamente:

```
jose@tux2duo ⮀ ~ ⮀ ansible all -u jose -m ping
rsmap.cloudapp.net | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Ahora procedamos a instalar la aplicación web, para ello como si se tratase de un equipo local lanzamos los comandos através de ansible.

- Dependencias básicas

```
jose@tux2duo ⮀ ~ ⮀ ansible all -u jose -a "sudo apt-get install -y python-setuptools python-dev build-essential git"
```

- Clonado del repositorios

```
jose@tux2duo ⮀ ~ ⮀ ansible all -u jose -m git -a "repo=https://github.com/luqueburgosjm/RSMapWeb.git dest=~/myapp version=HEAD"
```

- Instalando con el Makefile de la aplicación

```
jose@tux2duo ⮀ ~ ⮀ ansible all -m shell -a 'cd myapp/RSMApWeb && make install'
```

- Lanzado la aplicación

```
jose@tux2duo ⮀ ~ ⮀ ansible all -m shell -a 'cd myapp/RSMApWeb && make run'
```

Con ésto ya tenemos la app funcionando en Azure.

## Ejercicio 5
### Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.

En primer lugar designamos un tag para nuestra app en el archivo **ansible_hosts**

```
[rsmap]
rsmap.cloudapp.net

```

Ahora creamos un playbook llamado **rsmapdeploy.yml**, el contenido es parecido a el proceso que hemos seguido anteriormente con la salvedad de que nos adaptamos a las reglas definidas por los playbooks de ansible

```
---
- hosts: pruebaAzure
  sudo: yes
  remote_user: romi
  tasks:
  - name: Dependencias basicas
    apt: name=python-setuptools state=present
    apt: name=python-dev state=present
    apt: name=build-essential state=present
    apt: name=git state=present
  - name: Descarga del repositorio
    git: repo=repo=https://github.com/luqueburgosjm/RSMapWeb.git dest=myapp clone=yes force=yes
  - name: Dependecias de la app
    shell: cd myapp/RSMapWeb && make install
  - name: Lanzar app
    shell: cd myapp/RSMapWeb && make run

```


Por último lanzamos el playbook con la siguiente orden:

```
jose@tux2duo ⮀ ~ ⮀ansible-playbook -u jose rsmapdeploy.yml

```

### ¿Ansible o Chef?

La configuración de Ansible es mucho más llevadera y además permite hacerlo de manera *remota*, además la dificultad entre playbooks (ansible) y las recetas (chef) es mucho menor en el primero.

## Ejercicio 6
### Instalar una máquina virtual Debian usando Vagrant y conectar con ella.

Instalación de vagrant:


```
 jose@tux2duo ⮀ ~/RSMapWeb ⮀ ⭠ master ⮀ yaourt -S vagrant
[sudo] password for jose:
resolving dependencies...
looking for conflicting packages...

Packages (2) vagrant-substrate-524.64c5261-1  vagrant-1.8.1-1

Total Download Size:   26.13 MiB
Total Installed Size:  82.46 MiB

:: Proceed with installation? [Y/n] y
:: Retrieving packages ...
 vagrant-substrate-524.64c5261-1-x86_64                                         9.2 MiB  3.95M/s 00:02 [############################################################] 100%
 vagrant-1.8.1-1-x86_64                                                        16.9 MiB  4.50M/s 00:04 [############################################################] 100%
(2/2) checking keys in keyring                                                                         [############################################################] 100%
(2/2) checking package integrity                                                                       [############################################################] 100%
(2/2) loading package files                                                                            [############################################################] 100%
(2/2) checking for file conflicts                                                                      [############################################################] 100%
(2/2) checking available disk space                                                                    [############################################################] 100%
:: Processing package changes...
(1/2) installing vagrant-substrate                                                                     [############################################################] 100%
(2/2) installing vagrant                                                                               [############################################################] 100%

jose@tux2duo ⮀ ~/RSMapWeb ⮀ ⭠ master ⮀ vagrant --version
Vagrant 1.8.1

```

Ahora procedemos a bajarnos una imagen de debian reducida para minimizar el tiempo de descarga mediante vagrant

```
 jose@tux2duo ⮀ ~ ⮀ vagrant box add debian http://static.gender-api.com/debian-8
-jessie-rc2-x64-slim.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'debian' (v0) for provider:
    box: Downloading: http://static.gender-api.com/debian-8-jessie-rc2-x64-slim.
box
==> box: Successfully added box 'debian' (v0) for 'virtualbox'!
vagrant box add debian   8.60s user 4.04s system 15% cpu 1:19.78 total
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'debian' (v0) for provider:
    box: Downloading: http://static.gender-api.com/debian-8-jessie-rc2-x64-slim.
box
==> box: Successfully added box 'debian' (v0) for 'virtualbox'!
vagrant box add debian   8.60s user 4.04s system 15% cpu 1:19.78 total

```

Para iniciarla ejecutamos:

```
jose@tux2duo ⮀ ~ ⮀ vagrant init debian
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

```

Ahora la levantamos

```
jose@tux2duo ⮀ ~ ⮀ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'debian'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: ivfinal_default_1454790557854_89481
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version o
f
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you
 see
    default: shared folder errors, please make sure the guest additions within t
he
    default: virtual machine match the version of VirtualBox you have installed
on
    default: your host and reload your VM.
    default:
    default: Guest Additions Version: 4.2.16
    default: VirtualBox Version: 5.0
==> default: Mounting shared folders...
    default: /vagrant => /home/jose/Desktop/ivfinal
vagrant up  7.06s user 3.40s system 13% cpu 1:18.66 total

```

Y accedemos a ella mediante ssh

```
jose@tux2duo ⮀ ~ ⮀ vagrant ssh
Linux vagrant-wheezy 3.2.0-4-686-pae #1 SMP Debian 3.2.51-1 i686
          _                         
__      _| |__   ___  ___ _____   _
\ \ /\ / / '_ \ / _ \/ _ \_  / | | |
 \ V  V /| | | |  __/  __// /| |_| |
  \_/\_/ |_| |_|\___|\___/___|\__, |
                              |___/

Last login: Wed Oct 23 00:57:09 2013
vagrant@vagrant-wheezy ~ %

```

## Ejercicio 7
### Crear un script para provisionar "nginx" o cualquier otro servidor web que pueda ser útil para alguna otra práctica.


Añadimos lo siguiente al vagrantfile:
```
config.vm.provision "shell",
    inline: "sudo apt-get update && sudo apt-get install -y nginx && sudo service nginx start"
```

Para efecutar los cambios ejecutamos:

```
jose@tux2duo ⮀ ~ ⮀ vagrant provision
```

## Ejercicio 8
### Configurar tu máquina virtual usando Vagrant con el provisionador Ansible.

Aprovecho para realizar el último hito.

Partimos de la base en la que tenemos instalado y configurado un servicio cloud en Azure, el cliente de azure así como el plugin para azure de vagrant.

Definimos el Vagrantfile de la siguiente manera:

```
Vagrant.configure('2') do |config|

  config.vm.define "localhost" do |l|
    l.vm.hostname = "localhost"
    l.vm.box = 'azure'
    l.vm.network "public_network"
    l.vm.network "private_network",ip: "192.168.56.10", virtualbox__intnet: "vboxnet0"
    l.vm.network "forwarded_port", guest: 8000, host: 8000

    l.vm.provider :azure do |azure, override|
 	azure.mgmt_certificate = File.expand_path('/home/jose/claves/azure.pem')
 	azure.mgmt_endpoint = 'https://management.core.windows.net'
 	azure.subscription_id = '5748f1ee-1ca8-4749-b2d3-739d2747f319'
 	azure.vm_image = 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_3-LTS-amd64-server-20151218-en-us-30GB'
 	azure.vm_name = 'rsmap'
  azure.cloud_service_name = 'rsmap'
  azure.vm_user = 'jose'
  azure.vm_password = 'jose'
 	azure.vm_location = 'Central US'
 	azure.ssh_port = '22'
  azure.tcp_endpoints = '8000:8000'

  config.ssh.username = 'jose'

 	end
     l.vm.provision "ansible" do |ansible|
    	ansible.sudo = true
    	ansible.playbook = "rsmapdeploy.yml"
    	ansible.verbose = "vvv"
    	ansible.host_key_checking = false
  	end
 end
end
```

El fichero indicado en Vagrantifle **rsmapdeploy.yml** es el playbook de Ansible, que contiene lo siguiente:

```
---
- hosts: localhost
  sudo: yes
  remote_user: jose
  tasks:
  - name: Update sys
    apt: update_cache=yes upgrade=dist
  - name: Basic dependencies
    action: apt pkg={{ item }} state=installed
    with_items:
      - python-setuptools
      - python-dev
      - python-pip
      - build-essential
      - git
      - make
  - name: Git clone
    git: repo=https://github.com/luqueburgosjm/RSMapWeb.git dest=~/RSMapWeb clone=yes force=yes
  - name: Permissions
    command: chmod -R +x ~/RSMapWeb
  - name: App requirements
    pip: requirements=~/RSMapWeb/requirements.txt
  - name : Stop production server if running
    script: ~/RSMapWeb/scripts/stopifrunning.sh
    ignore_errors: yes
  - name: Run app
    command: chdir=~/RSMapWeb nohup python manage.py runserver 0.0.0.0:8000

```

El archivo host de ansible debe quedar de la siguiente forma
```
[localhost]
192.168.56.10	ansible_connection=local

```

Con la variable de entorno apuntándolo **export ANSIBLE_HOSTS=~/ansible_hosts**

Por último si no tenemos la máquina creada ejecutamos

```
vagrant up --provider=azure
```

En caso de que sólo queramos actualizar el contenido usamos
```
vagrant provision
```
