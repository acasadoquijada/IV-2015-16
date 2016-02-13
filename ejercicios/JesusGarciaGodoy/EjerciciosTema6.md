# Ejercicios Tema 6

## Ejercicio 1
**Instalar chef en la máquina virtual que vayamos a usar.**

Primero, vamos a conectarnos a nuestra máquina virtual de Azure, con ```azure login```, arrancamos la máquina y nos conectamos por ssh con:

```
jesmorc@jesmorc-PClaptop ~/Workinout $ sudo vagrant ssh
[sudo] password for jesmorc: 
==> localhost: Attempting to read state for iv-jesmorc-ubuntuserver in iv-jesmorc-ubuntuserver-service-euemq
==> localhost: VM Status: ReadyRole
==> localhost: Attempting to read state for iv-jesmorc-ubuntuserver in iv-jesmorc-ubuntuserver-service-euemq
==> localhost: VM Status: ReadyRole
    localhost: Looking for local port 22
    localhost: Found port mapping 22 --> 22
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.16.0-37-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Sat Feb 13 17:54:21 UTC 2016

  System load:  0.48              Processes:           232
  Usage of /:   4.3% of 28.80GB   Users logged in:     0
  Memory usage: 9%                IP address for eth0: 100.77.224.46
  Swap usage:   0%

  Graph this data and manage this system at:
    https://landscape.canonical.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud


*** System restart required ***
Last login: Sat Feb 13 17:54:22 2016 from 217.216.81.186.dyn.user.ono.com
vagrant@iv-jesmorc-ubuntuserver:~$ 
```


Dentro de la máquina, instalamos *Chef* con:
```
curl -L https://www.opscode.com/chef/install.sh | sudo bash
```

Esto descargará **chef-solo**, una versión aislada para trabajar en una máquina desde la misma.

Para comprobar la versión instalada:
```
vagrant@iv-jesmorc-ubuntuserver:~$ chef-solo -v
Chef: 12.7.2
```

## Ejercicio 2
**Crear una receta para instalar nginx, tu editor favorito y algún directorio y fichero que uses de forma habitual.**

Creamos los directorios donde irán las recetas para instalar nginx y el editor nano:

Desde el *home* :

```
vagrant@iv-jesmorc-ubuntuserver:~$ mkdir -p chef/cookbooks/nginx/recipes
vagrant@iv-jesmorc-ubuntuserver:~$ mkdir -p chef/cookbooks/nano/recipes

```

Ahora, vamos a configurar los ficheros que contendrán las recetas para instalar "nginx" y "nano".

[Tutorial Server chef-solo](http://www.mechanicalfish.net/configure-a-server-with-chef-solo-in-five-minutes/).

El fichero que contendrá la receta de instalación se llamará *default.rb*, que existirá uno en cada uno de los directorios creados antes.
Esto contiene ordenes que  instalarán el paquete especificado, creará un directorio para documentos y dentro de él un fichero que explica de qué se trata.

- Receta **default.rb** para nginx:

```
package 'nginx'
directory "/home/jesmorc/Documentos/nginx"
file "/home/jesmorc/Documentos/nginx/LEEME" do
	owner "jesmorc"
	group "jesmorc"
	mode 00544
	action :create
	content "Directorio para nginx"
end
```

- Receta **default.rb** para nano:

```
package 'nano'
directory "/home/jesmorc/Documentos/nano"
file "/home/jesmorc/Documentos/nano/LEEME" do
	owner "jesmorc"
	group "jesmorc"
	mode 00544
	action :create
	content "Directorio para nano"
end
```


Creamos directorios necesarios:
```
  mkdir -p Documentos/nano
  mkdir -p Documentos/nginx
```

El siguiente paso es crear un fichero llamado **node.json**, que irá en el directorio *chef* y va a tener la lista de recetas a ejecutar. Contiene:

```
{
        "run_list":["recipe[nginx]", "recipe[nano]"]
}
```

Creamos el archivo de configuración **solo.rb**, también en el directorio *chef*, que incluye referencias a los ficheros previos.

```
file_cache_path "/home/jesmorc/chef"
cookbook_path "/home/jesmorc/chef/cookbooks"
json_attribs "/home/jesmorc/chef/node.json"
```

Comprobamos que los paquetes  "nano" y "nginx" se instalan con la siguiente orden:
```
sudo chef-solo -c chef/solo.rb
```

## Ejercicio 3
**Escribir en YAML la siguiente estructura de datos en JSON "{ uno: "dos", tres: [ 4, 5, "Seis", { siete: 8, nueve: [ 10, 11 ] } ] }".**


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


