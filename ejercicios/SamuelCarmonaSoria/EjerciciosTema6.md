# Samuel Carmona Soria
# Tema 6

## Ejercicio 1
**Instalar chef en la máquina virtual que vayamos a usar.**

kkPrimero, vamos a conectarnos a nuestra máquina virtual de Azure, primero hacemos ```azure login``` y tras ello, arrancamos la máquina y nos conectamos por ssh, como se ve en la imagen siguiente:
![Acceso SSH a máquina virtual en Azure](http://i.cubeupload.com/6JnC1a.jpg)

Una vez en la máquina, tal como se explica [aquí](http://gettingstartedwithchef.com/first-steps-with-chef.html), una manera fácil de instalar chef es ejecutando la siguiente orden:
```
curl -L https://www.opscode.com/chef/install.sh | sudo bash
```
Éste comando se descargará el script para la instalación de **chef-solo**, que consiste en una versión aislada que permite trabajar en una máquina desde la misma.

![Instalando CHEF](http://i.cubeupload.com/BWafQT.jpg)

Con `chef-solo -v` comprobamos que está instalado, y nos muestra la versión instalada:
![Versión chef-solo](http://i.cubeupload.com/fhJk0O.jpg)


## Ejercicio 2
**Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.**

Antes de todo, creamos los directorios donde irán las recetas para instalar nginx y el editor que más uso, nano:

Desde el directorio /home/samu:

- ```mkdir -p chef/cookbooks/nginx/recipes```
- ```mkdir -p chef/cookbooks/nano/recipes```

Ahora, vamos a configurar los ficheros que contendrán las recetas para instalar "nginx" y "nano".

Para la configuración podemos seguir [este tutorial](http://www.mechanicalfish.net/configure-a-server-with-chef-solo-in-five-minutes/), donde explica como cofigurar un servidor con chef-solo en 5 minutos.

El fichero que contendrá la receta de instalación se llamará "default.rb", que existirá uno en cada uno de los directorios creados anteriormente (chef/cookbooks/nginx/recipes y chef/cookbooks/nano/recipes).

- Archivo receta **default.rb** para nginx:

```
package 'nginx'
directory "/home/samu/Documentos/nginx"
file "/home/samu/Documentos/nginx/LEEME" do
	owner "samu"
	group "samu"
	mode 00544
	action :create
	content "Directorio para nginx"
end
```

- Archivo receta **default.rb** para nano:

```
package 'nano'
directory "/home/samu/Documentos/nano"
file "/home/samu/Documentos/nano/LEEME" do
	owner "samu"
	group "samu"
	mode 00544
	action :create
	content "Directorio para nano"
end
```

El contenido que hemos introducido en ambos de dichos archivos, contiene ordenes que  instalarán el paquete especificado.
Creará un directorio para documentos y dentro de él un fichero que explica de qué se trata.

Antes de seguir hacia adelante, deberemos asegurarnos de que la carpeta "Documentos" y las subcarpetas "nano" y "nginx" están creadas:
```
  mkdir -p Documentos/nano
  mkdir -p Documentos/nginx
```

El siguient paso es crear un fichero llamado "node.json", que irá en el directorio "chef" y va a tener la lista de recetas a ejecutar.

- **node.json**:

```
{
        "run_list":["recipe[nginx]", "recipe[nano]"]
}
```

También vamos a crear el archivo de configuración "solo.rb", que incluirá referencias a los ficheros creados previamente.
Al igual que "node.json", éste también se guardará en el directorio chef.

- **solo.rb**:

```
file_cache_path "/home/samu/chef"
cookbook_path "/home/samu/chef/cookbooks"
json_attribs "/home/samu/chef/node.json"
```

Si queremos ver la estructura del directorio Chef podemos instlar "tree" con:
```
sudo apt-get install tree
```
Tras instalarlo ejecutamos `tree chef` y nos mostrará la estructura de directorios Chef:
![tree chef](http://i.cubeupload.com/bomW6u.jpg)

El último paso es llevar acabo la configuración isntalada, y comprobar que los paquetes  "nano" y "nginx" se instalan con la siguiente orden:
```
sudo chef-solo -c chef/solo.rb
```
Y obtendremos la siguiente salida
![demostración chef](http://i.cubeupload.com/wActlG.jpg)

Podemos ver que como ya indica, nginx y nano ya están instalados antes de la realización del ejercicio. En caso de no estarlo, se instalarían al ejecutar éste comando.


## Ejercicio 3
**Escribir en YAML la siguiente estructura de datos en JSON "{ uno: "dos", tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }".**

La estructura de los datos proporcionados en JSOn, quedarían en YAML tal que así:
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
**Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.**

Para empezar, primero instalamos Ansible:
```
sudo pip install paramiko PyYAML jinja2 httplib2 ansible
```

Añado la máquina la máquina de Azure al "inventario" :
```
echo "iv-ej5-ubuntuserver.cloudapp.net" > ~/ansible_hosts
```

Le indicamos a Ansible donde se encuentra el fichero con la siguiente variable de entorno:
```
export ANSIBLE_HOSTS=~/ansible_hosts
```

Arrancamos la máquina virtual:
```
azure vm start iv-ej5-ubuntuserver
```


Ahora debemos configurar SSH para poder conectar con la máquina:
 ```
ssh-keygen -t dsa
```
 ```
ssh-copy-id -i .ssh/id_dsa.pub samu@iv-ej5-ubuntuserver.cloudapp.net
```

![Configurando SSH](http://i.cubeupload.com/kQhHSi.jpg)

Ahora comprobaremos que tenemos acceso tanto por SSH como desde Ansible:

 ```
ssh samu@iv-ej5-ubuntuserver.cloudapp.net
```

![Conectando por SSH](http://i.cubeupload.com/PKmq15.jpg)
A continuación conectamos con Ansible:
```
ansible all -u samu -m ping
```
![Conectando por Ansible](http://i.cubeupload.com/HD6UOn.jpg)

Podemos ejecutar comandos desde nuestra máquina local, gracias a las características de Ansible:
![Comandos con Ansible](http://i.cubeupload.com/65Vc8A.jpg)


El siguiente proceso será realizar el despliegue de la app.

Primero instalamos los librerías básicos en la máquina:
```
ansible all -u samu -a "sudo apt-get install -y python-setuptools python-dev build-essential git pkg-config libjpeg-dev zlib1g-dev"
ansible all -u samu -m command -a "sudo easy_install pip"
```

Y ahora clonamos el repositorio en la máquina de Azure:

 ```
ansible all -u samu -m git -a "repo=https://github.com/Samuc/Eat-with-Rango.git  dest=~/Eat-with-Rango version=HEAD"
```

![Descargando repo en máquina](http://i.cubeupload.com/aCJv4q.jpg)



Instalamos lo necesario para ejecutar la aplicación:
```
ansible all -u samu -m command -a "pip install -r Eat-with-Rango/requirements.txt"
```

En local: azure vm endpoint create iv-ej5-ubuntuserver 80 80

Desactivamos el servidor web nginx en la máquina azure para que no ocupe el puerto 80, en caso de que lo esté ocupando y en caso de que esté activo:
```
 ansible all -u samu -m command -a "sudo update-rc.d nginx disable;"
```

He creado dos formas de ejecutar la aplicación, la primera, la más fácil, directamente desde ansible.
Primero nos moveremos al directorio de la apliacción, y luego ejecutaremos la app:
```
 ansible all -m shell -a "cd ~/Eat-with-Rango && sudo python manage.py runserver 0.0.0.0:80"
```


La segunda opción es un script que he creado para que ésto lo haga automáticamente, moverse con el comando "cd" al directorio de la aplicación y que ejecute el manager.py para que funcione correctamente.

El script.sh creado es el siguiente:
```
cd ~/Eat-with-Rango/
sudo python manage.py runserver 0.0.0.0:80
```

Ahora, desde la máquina local podemos ejecutar la siguiente línea, y pondrá en funcionamiento nuestra apliación:
```
ansible all -u samu -m command -a "sh ~/Eat-with-Rango/script.sh"
```

Es necesario moverse al directorio de la apliación, porque si lo ejecuto desde el directorio home/<mi usuario>, con "python ~/Eat-with-Rango-Bar-Tapas/manage.py runserver 0.0.0.0:80", luego no funciona bien debido a la incorrecta indexación de los ficheros estáticos, al estar ejecutándose desde otra carpeta, y no desde la raíz de la aplicación.


Ahora si entramos a [http://iv-ej5-ubuntuserver.cloudapp.net](http://iv-ej5-ubuntuserver.cloudapp.net) entraremos a la aplicación de Bares y Tapas con Rango.


## Ejercicio 5.1
**Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.**

Compruebo que el nombre de la máquina de azure está en el fichero "ansible_hosts", si no, se añade como se hizo en el ejercicio 4:
![ansible_hosts](http://i.cubeupload.com/1muxcz.jpg)

Vamos a crear el playbook de Ansible, "despliegueRangoBares.yml" , en el que añadiremos la instalación de cada paquete necesario para la aplicación, así como los comandos necesarios para instalar los paquetes listados en requirements.txt, y ejecutar la aplicación:
```
---
- hosts: RangoAzure
  sudo: yes
  remote_user: samu
  tasks:
  - name: Actualizar cache apt
    apt: update_cache=yes
  - name: Instalar python-setuptools
    apt: name=python-setuptools state=present
  - name: Instalar python-dev
    apt: name=python-dev state=present
  - name: Instalar build-essential
    apt: name=build-essential state=present
  - name: Instalar git
    apt: name=git state=present
  - name: Instalar pkg-config
    apt: name=pkg-config state=present
  - name: Instalar libtiff4-dev
    apt: name=libtiff4-dev state=present
  - name: Instalar libjpeg8-dev
    apt: name=libjpeg8-dev state=present
  - name: Instalar zlib1g-dev
    apt: name=zlib1g-dev state=present
  - name: Instalar PIP
    shell: sudo easy_install pip
  - name: Instalar Pillow
    shell: sudo -H pip install Pillow --upgrade
  - name: Clonando repositorio desde git
    git: repo=https://github.com/Samuc/Eat-with-Rango.git  dest=~/Eat-with-Rango clone=yes force=yes
  - name: Dar permisos a apliacación
    shell: sudo chmod +x ~/Eat-with-Rango
  - name: Instalar requisitos para la app
    shell: sudo pip install -r ~/Eat-with-Rango/requirements.txt
  - name: Ejecutar aplicacion
    shell: cd ~/Eat-with-Rango && sudo python manage.py runserver 0.0.0.0:80

```

*Nota:* el sudo de "python manage.py runserver 0.0.0.0:80" lo ponemos para tener permiso en ejecutar la aplicación por el puerto 80.


Colocamos el fichero de despliegue en ~/ de la máquina local, para que pille bien el ansible_hosts previamente configurado, y seguidamente ya podemos ejecutar el playbook con el siguiente comando:

 `ansible-playbook -u samu despliegueRangoBares.yml`

Y vemos que todo se ejecuta correctamente en las siguientes capturas:

![Despliegue playbook de Ansible 1 ](http://i.cubeupload.com/65veU1.jpg)
![Despliegue playbook de Ansible 2 ](http://i.cubeupload.com/zUnOhR.jpg)


Ahora, mientras no hagamos ctrl+c en el terminal, se estará ejecutando la aplicación en la dirección de nuestra máquina azure, que en mi caso es: http://iv-ej5-ubuntuserver.cloudapp.net/

No hace falta poner el puerto al final, ya que lo hemos configurado para que utilice el de por defecto, el puerto 80:
![Apliacción desplegada con  ansible-playbook](http://i.cubeupload.com/P6Eek4.jpg)


## Ejercicio 5.2
**¿Ansible o Chef? ¿O cualquier otro que no hemos usado aquí? **
Chef requiere configurarse desde dentro del servidor, creación de directorios, ficheros, etc
Ansible se puede configurar desde fuera del servidor.

También, los playbooks son más fáciles de de configurar en Ansible que las recetas de Chef.

Como ventaja, Chef es más ligero, pero es una diferencia no perceptible por mis pruebas realizadas.


## Ejercicio 6
**Instalar una máquina virtual Debian usando Vagrant y conectar con ella.**

Primero debemos instalar Vagrant.
Existe la versión 1.4.3 de los repositorios de Ubuntu, que se instalaría con:
 `sudo apt-get install vagrant`

Pero para los ejercicios para Azure, necesitaremos una versión posterior a las 1.6.

Podemos descargar la última versión, 1.8.1 actualmente, de [aquí](https://releases.hashicorp.com/vagrant/) y luego instalar el paquete descargado con:

 `sudo dpkg -i vagrant_1.8.1_x86_64.deb`

En mi caso he tenido que reiniciar el sistema operativo para que me reconozca vagrant:

![Vagrant instalado](http://i.cubeupload.com/KHLvrV.jpg)


A continuación debemos instalar Virtualbox, si no lo tenemos ya, ya que nos será útil para correr la máquina:
 `sudo apt-get install virtualbox virtualbox-dkms`
Si se prefiere se puede usar el hipervisor que venga en gana.

Lo siguiente es descargar una distribución Debian de [aquí](http://www.vagrantbox.es/) y la descargamos. En esa lista se encuentran las distribuciones disponibles para instalar con Vagrant.

Vamos a por la versión slim box, que ocupará menos:
 `vagrant box add debian http://static.gender-api.com/debian-8-jessie-rc2-x64-slim.box`

![Descargando Debian con Vagrant](http://i.cubeupload.com/okMMA6.jpg)

Cuando termine la descarga, creamos un directorio donde queramos quevaya la máquina, dentro de éste directorio ejecutamos:
 `vagrant init debian`

Éste comando  el fichero "Vagrantfile" que permite trabajar y llevar a cabo cualquier configuración adicional.

![Fichero Vagrantfile](http://i.cubeupload.com/3JMlVc.jpg)

Ahora ya podemos iniciar la máquina:
 `vagrant up`

Cuando termine conectamos por SSH:
 `vagrant ssh`

![Inicio y conección ssh - Vagrant](http://i.cubeupload.com/Vazeij.jpg)



## Ejercicio 7
**Crear un script para provisionar "nginx" o cualquier otro servidor web que pueda ser útil para alguna otra práctica.**

Podemos guiarnos a través de [ésta documentación](http://full-stack-development.readthedocs.org/en/latest/complete_web_dev/vagrant.html) para realizar el aprovisionamiento.

Creamos el fichero Vagrantfile cogiendo las partes que necesitamos del contenido del vagrant de la página comentada:

```
#-*- mode: ruby -*-
#vi: set ft=ruby :

#Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "debian"

  config.vm.provision "shell",
	inline: "sudo apt-get update && sudo apt-get install -y nginx && sudo service nginx start"

end
```

Ahora ya podemos ejecutar:
 `vagrant provision`
Y así, se recargará el nuevo Vagrantfile.

Arranco la máquina y compruebo si se ha instalado nginx:
![nginx instalado](http://i.cubeupload.com/R0ud2E.jpg)

Como podemos observar, se ha instalado correctamente y ya está ejecutándose.


## Ejercicio 8
**Configurar tu máquina virtual usando Vagrant con el provisionador Ansible.**

Vamos a configurar el Vagrantfile para crear una máquina de Azure donde desplegar mi aplicación.

Antes de todo, si no tenemos instalado VirtualBox, AzureCLI y Vagrant, es el momento de instalarlo.

El primer paso es instalar "Vagrant Azure Provider" con el siguiente comando:
 `vagrant plugin install vagrant-azure`

![Install Vagrant Azure Provider](http://i.cubeupload.com/VPBqoc.jpg)


En caso de no haber configurado el acceso a nuestra cuenta de Azure, lo hacemos tal como se vió en el Tema 5.

- Nos conectamos a Azure con
```azure login```
![login](http://i.cubeupload.com/qMCJLH.jpg)



A continuación tenemos que editar en el archivo ansible_host, el nombre del host a "localhost" y la dirección IP (será la IP privada de la máquina de azure que se creará con Vagrant),  y lo llamaremos así desde el el playbook aprovisionamiento.yml.:

**ansible_hosts:**

```
[localhost]
192.168.10.175

```

Exportamos la variable de entorno de Ansible para que reconozca el host:

 `export ANSIBLE_HOSTS=~/ansible_hosts`



**aprovisionamiento.yml:**

```
- hosts: localhost
  sudo: yes
  remote_user: samu
  tasks:
  - name: Actualizar cache apt
    apt: update_cache=yes
  - name: Instalar python-setuptools
    apt: name=python-setuptools state=present
  - name: Instalar python-dev
    apt: name=python-dev state=present
  - name: Instalar build-essential
    apt: name=build-essential state=present
  - name: Instalar git
    apt: name=git state=present
  - name: Instalar pkg-config
    apt: name=pkg-config state=present
  - name: Instalar libtiff4-devi
    apt: name=libtiff4-dev state=present
  - name: Instalar libjpeg8-dev
    apt: name=libjpeg8-dev state=present
  - name: Instalar zlib1g-dev
    apt: name=zlib1g-dev state=present
  - name: Instalar PIP
    shell: sudo easy_install pip
  - name: Instalar Pillow
    shell: sudo -H pip install Pillow --upgrade
  - name: Clonando repositorio desde git
    git: repo=https://github.com/Samuc/Eat-with-Rango.git  dest=~/Eat-with-Rango clone=yes force=yes
  - name: Dar permisos a apliacación
    shell: sudo chmod +x ~/Eat-with-Rango
  - name: Instalar requisitos para la app
    shell: sudo pip install -r ~/Eat-with-Rango/requirements.txt
  - name: Ejecutar aplicacion
    shell: cd ~/Eat-with-Rango && sudo python manage.py runserver 0.0.0.0:80
```

En este fichero .yml, le indicamos las ordenes necesarias a ejecutar en la máquina de azure, y los paquetes a instalar necesarios para la app.
Después de ello, también se le indica que se descargue el repo de la app, instale los paquetes necesarios del mismo, y que ejecute la apliacción por el puerto 80.




Ahora, debemos generar los certificados de Azure, con los siguientes comandos:
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout azurevagrant.key -out azurevagrant.key
chmod 600 azurevagrant.key
openssl x509 -inform pem -in azurevagrant.key -outform der -out azurevagrant.cer
```

El archivo azurevagrant.cer generado, deberemos subirlo al apartado "certificados del panel de configuración de Azure, [aquí](https://manage.windowsazure.com/). Uso la versión antigua del panel de Azure porque es más fácil, hay que ir a configuración->Certificados de Administración->Agregar, y ahí subimos nuestro certificado generado.


Ahora debemos generar el fichero .pem, para que vagrant lo pueda leer.
Ejecutamos los siguientes comandos:
```
openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out azurevagrant.pem

cat azurevagrant.key > azurevagrant.pem
```



A continuación crearemos el Vagrantfile crearemos la máquina de Azure y usaremos el archivo **aprovisionamiento.yml** que ya hemos creado,  para realizar el despliegue de nuestra App.

**Vagrantfile:**
```
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
  config.vm.network "private_network",ip: "192.168.10.175", virtualbox__intnet: "eth0"
  config.vm.network "forwarded_port", guest: 80, host: 80
  config.vm.define "localhost" do |l|
          l.vm.hostname = "localhost"
  end

  config.vm.provider :azure do |azure, override|
      azure.mgmt_certificate = File.expand_path('/home/samu/azurevagrant.pem')
      azure.mgmt_endpoint = 'https://management.core.windows.net'
      azure.subscription_id = 'd6c58002-4537-471f-ab27-888d8170aca4'
      azure.vm_image = 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_2-LTS-amd64-server-20150506-en-us-30GB'
      azure.vm_name = 'Rango'
      azure.vm_password = 'Mi_contraseña1'
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
```
En la primera parte de éste fichero, el nombre de la máquina a usar es azure, la IP privada es la misma ip que la que es encuentra en el fichero ansible_hosts (que previamente hemos modificado). Y el nombre de la interfaz de red que estamos usando.

Luego tenemos que indicar donde se encuentra nuestro certicifado .pem, colocar nuestro ID de azure.
Éste ID lo podemos encontrar desde el panel de Azure, [aquí](https://manage.windowsazure.com/).

Tabmbién, indicamos la imagen a usar, en mi caso, un Ubuntu Server 14-02 amd64.

A continuación, en el siguiente bloque rellenamos con los de la máquina, como su nombre, contraseña, localización, puerto de ssh y endpoints.

Y por último, en el último bloque, añadimos el nombre de nuestro fichero de aprivisionamiento.


Una vez terminada toda la configuración procedemos a su uso, ejecutamos el siguiente comando:
```
vagrant box add azure https://github.com/msopentech/vagrant-azure/raw/master/dummy.box
```

En caso de que ya esté creado previamente, y queramos crearlo de nuevo desde 0, añadimos la opción --force:
```
vagrant box add azure https://github.com/msopentech/vagrant-azure/raw/master/dummy.box --force
```

Ahora, ya podemos proceder a levantar la  máquina, con:

 `sudo vagrant up --provider=azure`

Éste comando, creará la máquina virtual en Azure y posteriormente despliega la app.

Si ocurriese algún error en el aprovisionamiento, ahora debemos ejecutar otra orden, porque si intentamos la anterior de nuevo, nos dirá que la máquina ya está creada. Por lo que nos gustaría solo probar la parte del aprovisionamiento. Por lo tanto, ejecutaríamos el siguiente comando:
 `sudo vagrant provision`

![Ejecucion Vagrant up](http://i.imgbox.com/WU5EucG6.jpg)
![Ejecucion Vagrant up](http://i.imgbox.com/IWYVAeju.jpg)
![Ejecucion Vagrant up](http://i.imgbox.com/oFlY3Cm2.jpg)


Por último, entramos a la dirección de la máquina de azure en el navegador web. La dirección podemos verla también desde el panel de Azure.
Entramos en la dirección, y podremos ya usar nuestra aplicación:
![Captura de Eat-with-Rango desplegada con Vagrant en Azure](http://i.imgbox.com/upm1jgEl.jpg)


Si queremos conectarnos a nuestra máquina por ssh, ejecutamos:
`vagrant ssh`
![Conexión SSH con "vagrant ssh"](http://i.imgbox.com/PAjUWhxA.jpg)
