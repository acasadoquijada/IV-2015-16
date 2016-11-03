### Ejercicio 1 : Instalar chef en la máquina virtual que vayamos a usar:

He decidido instalarlo de la manera abreviada tal y como viene en teoría, he introducido en la terminal:
```
curl -L https://www.opscode.com/chef/install.sh | sudo bash
```
Podemos comprobar que se ha instalado con:
```
chef-solo -v
```

![instalación_chef](https://www.dropbox.com/s/z9hk2c2hwgy20ss/img1_1.png?dl=1)

![version_chef](https://www.dropbox.com/s/y8x5scvze519vtr/img1_2.png?dl=1)

### Ejercicio 2: Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.

Para la realización del ejercicio realizaremos los siguientes pasos:

- Creación de los directorios con las recetas.

![instalación_chef](https://www.dropbox.com/s/jqcjd7s5klxfwny/img2_1.png?dl=1)

- Creación de los ficheros **default.rb** dentro de los directorios del anterior paso. Estos ficheros contendrán las recetas para instalar **nginx** y **nano**.

![instalación_chef](https://www.dropbox.com/s/b1mll82omrgb7hk/img2_2.png?dl=1)

![instalación_chef](https://www.dropbox.com/s/3231al2zyluwaiz/img2_3.png?dl=1)

- Creación dentro del directorio **chef** de los archivos **node.json** y **solo.rb**

![instalación_chef](https://www.dropbox.com/s/kxrkfxxme9nek9p/img2_4.png?dl=1)

- Ejecutamos:

![instalación_chef](https://www.dropbox.com/s/yp9pnz82ufi3p6g/img2_5.png?dl=1)

-  La jerarquía resultante del ejercicio es la siguiente:

![instalación_chef](https://www.dropbox.com/s/koohcahccptpccy/img2_6.png?dl=1)

### Ejercicio 3: Escribir en YAML la siguiente estructura de datos en JSON

```
{ uno: "dos",tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }

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

###	Ejercicio 4: Desplegar los fuentes de la aplicación de DAI o cualquier otra aplicación que se encuentre en un servidor git público en la máquina virtual Azure (o una máquina virtual local) usando ansible.

Para realizar el ejercicio realizamos los siguientes pasos:

- Instalo Ansible:

```
$ sudo pip install paramiko PyYAML jinja2 httplib2 ansible

```
- Añado la máquina de Amazon al inventario:

```
$ echo "ec2-52-11-219-71.us-west-2.compute.amazonaws.com" > ansible_hosts

```
- Defino la variable de entorno que indica la localización del fichero:
```
$ export ANSIBLE_HOSTS=ansible_hosts
```

- Configuro ssh para la conexión con Amazon.

![img2](https://www.dropbox.com/s/p8vl61iv8jp4hg9/img2.png?dl=1)

- El archivo **.pem** descargado debo añadirlo a la configuración ssh:

```
ssh-add "ruta-archivo-.pem"

```

- Compruebo la conexión con Ansible:
```
ansible all -u ubuntu -i ansible_hosts -m ping
```

![img3](https://www.dropbox.com/s/pyce0pnm9zqcnll/img3.png?dl=1)

- Instalo librerías en la máquina:
```
ansible all -u ubuntu -m command -a "sudo apt-get install python-setuptools python-dev build-essential git -y"
ansible all -u ubuntu -m command -a "sudo easy_install pip"
```

- Descargo la aplicación:
```
ansible all -u ubuntu -m git -a "repo=https://github.com/lorenmanu/prueba_dai.git  dest=~/pruebaDAI version=HEAD"
```

- Instalo los requisitos:

```
ansible all -u ubuntu -m command -a "sudo pip install -r prueba_dai/requirements.txt"

```

- Ejecuto la aplicación:


```
ansible all -u ubuntu -m command -a "sudo python prueba_dai/manage.py runserver 0.0.0.0:80"

```

- Mi aplicación con los siguientes pasos funcionando:

![img4](img4)


**Nota**: Antes deberíamos haber creado nuestra instancia en Amazon, se puede ver [aquí](https://github.com/lorenmanu/prueba_dai/blob/master/documentacion/ec2.md).

### Ejercicio 5:
### 1.Desplegar la aplicación de DAI con todos los módulos necesarios usando un playbook de Ansible.

My playbook es el siguiente:

```
- hosts: localhost
  remote_user: ubuntu
  sudo: yes
  tasks:
  - name: Actualizar sistema
    apt: update_cache=yes upgrade=dist
  - name: Instalar python-setuptools
    apt: name=python-setuptools state=present
  - name: Instalar build-essential
    apt: name=build-essential state=present
  - name: Instalar pip
    apt: name=python-pip state=present
  - name: Instalar git
    apt: name=git state=present
  - name: Ins Pyp
    apt: pkg=python-pip state=present
  - name: Instalar python-dev
    apt: pkg=python-dev state=present
  - name: Instalar libpq-dev
    apt: pkg=libpq-dev state=present
  - name: Instalar python-psycopg2
    apt: pkg=python-psycopg2 state=present
  - name: Obtener aplicacion de git
    command: git clone https://github.com/lorenmanu/prueba_dai.git
  - name: Permisos de ejecucion
    command: chmod -R +x prueba_dai
  - name: Instalar libreria para pillow
    command: sudo apt-get -y build-dep python-imaging --fix-missing
  - name: Instalar pillow
    command: sudo easy_install Pillow
  - name: Instalar requisitos
    command: sudo pip install -r prueba_dai/requirements.txt
  - name: ejecutar
    command: nohup sudo python prueba_dai/manage.py runserver 0.0.0.0:80

```


### 2.¿Ansible o Chef? ¿O cualquier otro que no hemos usado aquí?.

Yo me decanto por Ansible ya que permite su uso sin la necesidad de estar dentro del servidor.

### Ejercicio 6:Instalar una máquina virtual Debian usando Vagrant y conectar con ella.

Los pasos que debemos seguir son los siguientes:
- Instalar vagrant, como yo uso mac me he descargado el **.dmg**:

- Instalar virtualbox.

- Me bajo la imagen de Debian:

```

vagrant box add debian https://github.com/holms/vagrant-jessie-box/releases/download/Jessie-v0.1/Debian-jessie-amd64-netboot.box

```

![img7](https://www.dropbox.com/s/zf26kmkc75qriq3/img7.png?dl=1)

- Creo el fichero Vagranfile:

```

vagrant init debian

```

![img8](https://www.dropbox.com/s/d0703qbdqwqigth/img8.png?dl=1)

- Inicializo la máquina:
```
vagrant up
```
![img9](https://www.dropbox.com/s/fvuhhp6mtwcliua/img9.png?dl=1)


- Me conecto por ssh con ella:

![img10](https://www.dropbox.com/s/i18irjc4u0t06dy/img10.png?dl=1)


### Ejercicio 7: Crear un script para provisionar `nginx` o cualquier otro servidor web que pueda ser útil para alguna otra práctica

- Modifico el Vagrantfile para nginx:

```

#-*- mode: ruby -*-
#vi: set ft=ruby :

#Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    # All Vagrant configuration is done here. The most common configuration
    # options are documented and commented below. For a complete reference,
    # please see the online documentation at vagrantup.com.

    # Every Vagrant virtual environment requires a box to build off of.
config.vm.box = "debian"
config.vm.provision "shell",
inline: "sudo apt-get update && sudo apt-get install -y nginx && sudo service nginx start"

end
```

- Levanto la máquina , la arranco y compruebo la instalación de nginx:

![img11](https://www.dropbox.com/s/2t6xrzai5m5cm94/img11.png?dl=1)

![img12](https://www.dropbox.com/s/luldywvj6b0cf3l/img12.png?dl=1)

![img13](https://www.dropbox.com/s/qk4p05rulf17pcb/img13.png?dl=1)


### Ejercicio 8 : Configurar tu máquina virtual usando vagrant con el provisionador ansible.
Para la realización del ejercicio voy a usar mi máquina de Amazon. Para ello, seguiré los siguientes pasos:

- Debo instalar el cliente de **Command Line Interface** de Amazon, el cual me permitirá realizar tareas de gestión de mi perfil de usuario. Se puede consultar la documentación oficial, concretamente en el [apartado](http://docs.aws.amazon.com/es_es/cli/latest/userguide/installing.html).

- Una vez instalado en nuestro ordenador, procederemos a configurarlo, para ello debemos poner en la terminal:

```
aws configure

```

Este comando nos pedirá que rellenemos una serie de primitivas, la cual permitirán a **Command Line Interface** conectarse a nuestro perfil de Amazon. No se muestra pantallazo, debido a razones de seguridad.


De la siguiente imagen, pasamos a mostrar como rellenar los campos que nos piden:

- Nos dirigimos a nuestro perfil de Amazon( lo que viene siendo loguearse como en cualquier plataforma). Una vez dentro seleccionamos a nuestro nombre de perfil y le damos al apartado **Security Credentials**.

![img15](https://www.dropbox.com/s/gygkl6oneu1hwrg/img15.png?dl=1)

- Una vez dentro, nos vamos al apartado **Users**:

![img16](https://www.dropbox.com/s/8y7vo3cddw4iluw/img16.png?dl=1)

- Le damos a la pestaña **Create New User**:

![img17](https://www.dropbox.com/s/jc09db3cn307ggn/img17.png?dl=1)

- Una vez dada nos pedirá que rellenemos una seria de campos:

![img18](https://www.dropbox.com/s/tao3m85ray0x8as/img18.png?dl=1)

- Cuando pinchamos en crear, se nos permitirá descargar un archivo **.csv**, deberemos guardarlo ya que contiene los campos necesarios para **aws configure**, ahora lo veremos:

- Si todos los pasos anteriores se ha realizado correctamente debe salirnos ya el usuario, en mi caso yo he creado un usuario denominado **lorenmanu**:

![img19](https://www.dropbox.com/s/r4mjdiqro8jxx7g/img19.png?dl=1)

- Tenemos que darle los permisos necesarios al usuario para que pueda trabajar con nuestro perfil, esto lo haremos en el apartado **Permissions**. Dentro de este apartado se deberá dar a **Attach Policy** para darle permisos. Se nos ofrece una gran variedad de permisos que le podemos dar al usuario, en mi caso he optado por **AmazonEC2FullAccess**, ya que es que me permite trabajar en versión gratuita:

![img20](https://www.dropbox.com/s/bgyoifrn3skbli7/img20.png?dl=1)


- Recuperamos el archivos **.csv**, lo abrimos, y de ahí cogeremos los campos:

```
access_key_id

secret_access_key

```

- Ya podemos volver a usar **aws configure**, y rellenar los campos necesarios. Los dos campos del paso anterior son los importantes, en los otros dos que se nos pedirán rellenar:

- **region**: aquí ponemos la región en la cúal estará nuestra instancia, en mi caso **us-west-2**.
- **text_format**: aquí le di a **enter**, no es relevante de momento.

- Una vez especificado **Command Line Interface**, configuramos la conexión ssh con nuestra instacia, la cual crearemos en el Vagrantfile, para ello deberemos crear un archivo **.pem**. La documentación oficial de Amazon nos explica como realizarlo, concretamente [aquí](http://docs.aws.amazon.com/es_es/cli/latest/userguide/cli-using-param.html). En mi caso me ha valido con seguir la siguiente sintaxis:

```
aws ec2 create-key-pair --key-name my-key-pair

```

![img21](https://www.dropbox.com/s/d4ebg0arvvcxnfc/img21.png?dl=1)

- Creamos un **Security Groups**, el cual usuará para atender peticiones. El apartado de la documentación oficial de Amazon de como crearlo se puede ver [aquí](http://docs.aws.amazon.com/es_es/cli/latest/userguide/cli-ec2-sg.html). Yo he realizado lo siguiente:

```
aws ec2 create-security-group --group-name my-sg --description "My security group"

```

![img22](https://www.dropbox.com/s/fth56hgylgx4kla/img22.png?dl=1)

**Nota**: nos da un error, esto se debe a que lo hemos creado ya.


- Una vez realizado esto, modificamos nuestro **Vangranfile**, tiendo en cuenta:

- Los datos del archivo **.csv** anterior.:

```
aws.access_key_id

aws.secret_access_key

```

- Localización de nuestro sistema operativo:

```
aws.ami

```

- Regíon de nuestra instancia:

```
aws.region



```

- El nombre de nuestro archivo **.pem**:

```

keypair_name

```

- El **security_groups** que se encargará de atender nuestras peticiones:

```
security_groups

```

- El tipo de nuestra instancia, ** importante mirar las que son gratis o no**.

```
instance_type

```
- El nombre del usuario de nuestra máquina:

```
override.ssh.username

```

**Nota**: Usuario por el cual se conectará por ssh, por defecto es **ubuntu** en EC2.


- Por lo que podemos ver nuestro VangrantFile tendrá la siguiente estructura:


```

#-*- mode: ruby -*-
#vi: set ft=ruby :

Vagrant.require_plugin 'vagrant-aws'
Vagrant.require_plugin 'vagrant-omnibus'


Vagrant.configure('2') do |config|
    config.vm.box = "dummy"
    config.vm.box_url = "https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box"
    config.vm.network "public_network"
    config.vm.network "private_network",ip: "192.168.56.10", virtualbox__intnet: "vboxnet0"
    config.vm.network "forwarded_port", guest: 80, host: 80
    config.vm.define "localhost" do |l|
            l.vm.hostname = "localhost"
    end

    config.vm.provider :aws do |aws, override|
        aws.access_key_id = ""
        aws.secret_access_key = ""
        aws.keypair_name = ""
        aws.ami = ""
        aws.region = "us-west-2"
        aws.security_groups = ""
        aws.instance_type = "t2.micro"
        override.ssh.username = "ubuntu"
    end

    config.vm.provision "ansible" do |ansible|
        ansible.sudo = true
        ansible.playbook = "playbook.yml"
        ansible.verbose = "v"
        ansible.host_key_checking = false
  end
end


```

Si se tiene alguna duda, se puede consultar el siguiente [enlacen](https://github.com/mitchellh/vagrant-aws).


- Como podemos ver, para desplegar la aplicación estamos usuando **ansible**, este se encarga de instalar todos los paquetes necesarios, descargar nuestra aplicación de nuestro repositorio y ejecutarla, concretamente mi archivo **.yml** es el siguiente:

```
- hosts: localhost
  sudo: yes
  remote_user: vagrant
  tasks:
  - name: Actualizar sistema
    apt: update_cache=yes upgrade=dist
  - name: Instalar python-setuptools
    apt: name=python-setuptools state=present
  - name: Instalar build-essential
    apt: name=build-essential state=present
  - name: Instalar pip
    apt: name=python-pip state=present
  - name: Instalar git
    apt: name=git state=present
  - name: Ins Pyp
    apt: pkg=python-pip state=present
  - name: Instalar python-dev
    apt: pkg=python-dev state=present
  - name: Instalar libpq-dev
    apt: pkg=libpq-dev state=present
  - name: Instalar python-psycopg2
    apt: pkg=python-psycopg2 state=present
  - name: Obtener aplicacion de git
    git: repo=https://github.com/lorenmanu/prueba_dai.git  dest=prueba_dai clone=yes force=yes
  - name: Permisos de ejecucion
    command: chmod -R +x prueba_dai
  - name: Instalar libreria para pillow
    command: sudo apt-get -y build-dep python-imaging --fix-missing
  - name: Instalar pillow
    command: sudo easy_install Pillow
  - name: Instalar requisitos
    command: sudo pip install -r prueba_dai/requirements.txt
  - name: ejecutar
    command: nohup sudo python prueba_dai/manage.py runserver 0.0.0.0:80


```

- Ya solo nos queda desplegar nuestra aplicación, para ello pondremos en nuestra terminal en el directorio donde está el **Vagrantfile**

```

vagrant up --provider=aws

```

**Nota** en **--provider** estamos indicando el proveedor, en mi caso Amazon, si fuera azure deberíamos poner **azure**.


- Finalmente nuestra aplicación estará desplegada:

![img21](https://www.dropbox.com/s/w6s217d4pld9o0j/img24.png?dl=1)
