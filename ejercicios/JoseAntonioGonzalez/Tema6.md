#Ejercicio 1
#### Instalar chef en la máquina virtual que vayamos a usar
Para instalar Cheff, nos iremos a [su página de getting started](http://gettingstartedwithchef.com/first-steps-with-chef.html), y seguiremos la órden que se indica:

	 curl -L https://www.opscode.com/chef/install.sh | sudo bash
    
De esta forma se descargará un script que acto seguido, ejecutará. Dicho script es la instalación propia de cheff solo.

En el momento de realización de éste ejercicio, parece que la descarga del scriptinstala una versión ya obsoleta de cheff, donde nos advierte que se actualizará solo y sin dejar advertencia.

![](https://www.dropbox.com/s/nupnawtw829z4ju/Ejercicio1.png?dl=1)

#Ejercicio 2
#### Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.
PRimero, crearemos los directorios (dentro de la máquina virtual):
	
    mkdir -p chef/cookbooks/nginx/recipes
	mkdir -p chef/cookbooks/nano/recipes
 
Después de crear los directorios, debemos navegar hacia ellos, e incluir en cada uno, el fichero **default.rb ** correspondiente:

	cd chef/cookbooks/
- Para nginx 
```
cd nginx/recipes
vim default.rb
``` 
Y escribimos el siguiente contenido:
```
package 'nginx'
directory "/home/joseantonio/Documentos/nginx"
file "/home/joseantonio/Documentos/nginx/LEEME" do
    owner "joseantonio"
    group "joseantonio"
    mode 00544
    action :create
    content "Directorio para nginx"
end
```
- Para nano:
```
cd nano/recipes
vim default.rb
```
Y escribimos el siguiente contenido:
```
package 'nano'
directory "/home/joseantonio/Documentos/nano"
file "/home/joseantonio/Documentos/nano/LEEME" do
    owner "joseantonio"
    group "joseantonio"
    mode 00544
    action :create
    content "Directorio para nano"
end
```
Esto instalará los paquetes correspondientes, crea una carpeta para ubicar los archivos, y un documento LEEME donde se explica qué hace el paquete.

Con las recetas creadas, debemos ahora crear un archivo json que será el que cheff utilizará para ejecutar las recetas. Dicho archivo tiene que ir en el directorio **chef**. El archivo se llamará **node.json**, y tendrá el siguiente contenido:

```
{
    "run_list":["recipe[nginx]", "recipe[nano]"]
}
```

Por último, debemos crear el fichero de configuración (que llamaremos **solo.rb**), para utilizar los ficheros anteriormente creados. Lo ubicaremos en el mismo sitio que **node.json**, es decir, dentro de la carpeta **chef**, y con el siguiente contenido:
```
file_cache_path "/home/joseantonio/chef" 
cookbook_path "/home/joseantonio/chef/cookbooks" 
json_attribs "/home/joseantonio/chef/node.json" 
```

Ya está todo listo. Sólo queda lanzar la instalación mediante la orden:

	sudo chef-solo -c chef/solo.rb
    
Y si todo va bien, debemos ver en pantalla algo como lo indicado en las imagenes

![](https://www.dropbox.com/s/p4aj0rp12j0polz/Ejercicio2-1.png?dl=1)
![](https://www.dropbox.com/s/gy3z6swo828k5dp/Ejercicio2-2.png?dl=1)

#Ejercicio 3
#### Escribir en YAML la siguiente estructura de datos en JSON
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
#Ejercicio 6
#### Instalar una máquina virtual Debian usando Vagrant y conectar con ella.
En la práctica hemos usado una imagen de Ubuntu (ejercicio 8), que en éste caso se hace igual, pero cambiando el nombre de imagen a instalar.

Como no sabemos qué imágenes hay disponibles de Debian, ni sus versiones, ni tan siquiera su nombre, lo primero que debemos hacer es ver que imágenes podemos utilizar. Para eso introducimos la siguiente orden:

	azure vm image list "Central Us" debian

Con esto, vemos las imagenes, y nos quedaremos por ejemplo con la de nombre 6a83c2d016534a7a917bcd21b6e1c0c9__Debian-8-amd64-20160107.0  

Ahora, toca usar vagrant. Si no lo hemos instalado aún, se puede descargar [desde aquí](https://www.vagrantup.com/downloads.html). Hecho eso, lo instalamos (al usar ubuntu la instalación es fácil, simplemente con un doble click se abre el centro de software donde se te pide permiso para instalar),

Hecho eso, instalaremos el plugin de azure:

	vagrant plugin install vagrant-azure

Hacemos lo siguiente para crear el fichero Vagrantfile:

	vagrant init

Ahora toca hacer el Vagrantfile. Atendiendo principalmente al nombre de máquina, nombre de imagen, clave de suscripción, url de la box, etc.. Se explican más detalladamente en el [README](https://github.com/JA-Gonz/SMS_Estadisticas) del repositorio del proyecto. Es muy importante también la parte de creación de certificados.

El Vagrantfile debe tener el siguiente aspecto:

```

Vagrant.configure(2) do |config|
  	config.vm.box = "azure"
	config.vm.network "public_network"

  	#end
  	config.vm.provider :azure do |azure, override|
        # Mandatory Settings
		azure.mgmt_certificate = File.expand_path("azure.pem")
		azure.mgmt_endpoint    = "https://management.core.windows.net"
		azure.subscription_id = "3252f376-df66-4dae-b865-76048fcb3c63"
		azure.vm_name     = "ejercicioIVssss"
		azure.vm_image    = "6a83c2d016534a7a917bcd21b6e1c0c9__Debian-8-amd64-20160107.0  "
		azure.vm_size     = "Small"
		config.vm.box_url = "https://github.com/msopentech/vagrant-azure/raw/master/dummy.box"
	
		azure.vm_user = "ja" # defaults to 'vagrant' if not provided
		azure.vm_password = "12345678!Ab"
		azure.vm_location = "Central US" # e.g., West US
		azure.ssh_port = "22"
	end 
end
```

Hecho esto, lanzamos la creación con la orden:

	vagrant up --provider=azure

![](https://www.dropbox.com/s/lwrfuh3tpa1vz9m/ejercicio6%20tema6.png?dl=1)
    
    

#Ejercicio 8
#### Configurar tu máquina virtual usando vagrant con el provisionador
ansible
En el proyecto se ha realizado una creación de máquina virtual (en Azure) mediante Vagrant, y se ha aprovisionado con Ansible. Podemos ver el resultado del proyecto [aquí](https://github.com/JA-Gonz/SMS_Estadisticas).

En profundidad, en este ejercicio se pide principalmente configurar un [Vagrantfile](https://github.com/JA-Gonz/SMS_Estadisticas/blob/master/Vagrantfile) y un fichero de [configuración de Ansible](https://github.com/JA-Gonz/SMS_Estadisticas/blob/master/ansible/configuracion_ansible.yml). Siguiendo los enlaces puede verse en detale como están hechos.