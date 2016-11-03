# Tema 5
### Ejercicio 1:Instalar los paquetes necesarios para usar KVM. Se pueden seguir estas instrucciones. Ya lo hicimos en el primer tema, pero volver a comprobar si nuestro sistema está preparado para ejecutarlo o hay que conformarse con la paravirtualización. 

Comprobar que podemos usarlo:

    kvm-ok
        INFO: /dev/kvm exists

Instalar algunos paquetes:

    sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils virtinst

Añadimos nuestro usuario a los grupos kvm y libvirtd

    adduser jesus kvm
    adduser jesus libvirtd
    source ~/.bashrc

### Ejercicio 2:
#### 1. Crear varias máquinas virtuales con algún sistema operativo libre tal como Linux o BSD. Si se quieren distribuciones que ocupen poco espacio con el objetivo principalmente de hacer pruebas se puede usar CoreOS (que sirve como soporte para Docker), GALPon Minino, hecha en Galicia para el mundo, Damn Small Linux, SliTaz (que cabe en 35 megas) y ttylinux (basado en línea de órdenes solo).

##### SliTaz
Creamos el disco duro:

    $ qemu-img create -f qcow2 slitaz.qcow2 8G

Creamos la máquina virtual:

    $ qemu-system-x86_64 -machine accel=kvm -hda slitaz.qcow2 -cdrom ~/slitaz-4.0.iso -boot d

##### CoreOS
Creamos el disco duro:

    $ qemu-img create -f qcow2 coreos.qcow2 8G

Creamos la máquina virtual:

    $ qemu-system-x86_64 -machine accel=kvm -hda coreos.qcow2 -cdrom ~/coreos_production_iso_image.iso -m 1G -boot d

##### Usando VirtualBox
Con VirtualBox disponemos de una amigable interfaz gráfica que nos orientará a la hora de crear la máquina a medida del sistema operativo que usemos en ella. Para más info: [https://www.virtualbox.org/](https://www.virtualbox.org/)

### Ejercicio 5: Crear una máquina virtual ubuntu e instalar en ella un servidor nginx para poder acceder mediante web.
Podemos realizar la tarea usando una máquina EC2 de Amazon. Para ello podemos usar Vagrant y Ansible para automatizar el proceso, tal cual se hace en el hito 5 del proyecto. 

El Vagrantfile sería tal que así:

    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    Vagrant.configure(2) do |config|
     
      config.vm.box = "dummybox-aws"

      config.vm.hostname = "pruebanginx"
      config.vm.provider :aws do |aws, override|
     
        #AWS Settings
        aws.access_key_id = "access_key"
        aws.secret_access_key = "secret"
        aws.region = "us-west-2"
     
        aws.tags = {
          'Name' => 'Nginx',
          'Team' => 'Nginx',
          'Status' => 'active'
        }
     
        #Override Settings
        override.ssh.username = "ubuntu"
        override.ssh.private_key_path = "tucert.pem"
     
        aws.region_config "us-west-2" do |region|
          region.ami = 'ami-35143705'
          region.instance_type = 't2.micro'
          region.keypair_name = "yourpairkey"
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

playbook.yml

    ---
    - hosts: default
      remote_user: ubuntu
      sudo: true

      tasks:
        - name: apt-get update
          apt: update_cache=yes
        - name: Instalar nginx
          apt: name=nginx state=present
        - name: Start service
          shell: service nginx start

Ejecutamos Vagrant:

    vagrant up --provider=aws

### Ejercicio 7: Instalar una máquina virtual con Linux Mint para el hipervisor que tengas instalado.

Creamos el disco duro:

    qemu-img create -f qcow2 mint.qcow2 6G

Instalar imagen Linux Mint

    qemu-system-x86_64 -machine accel=kvm -hda mint.qcow2 -cdrom ~/linuxmint-17.3-cinnamon-64bit.iso -m 1G -boot d
 


