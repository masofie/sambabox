# Servidor de Dominio 

## Indice 
- [Servidor de Dominio](#servidor-de-dominio)
  - [Indice](#indice)
  - [Definición](#definición)
  - [1. Configuración Vagrant](#1-configuración-vagrant)
    - [1.1 Vagrantfile](#11-vagrantfile)
  - [2. Scripts Dominios](#2-scripts-dominios)
    - [2.1 Dominio Samba](#21-dominio-samba)
    - [2.2 Unidades Organizativas](#22-unidades-organizativas)
    - [2.3 Grupos](#23-grupos)
    - [2.4 Usuarios (Empelados/Clientes)](#24-usuarios-empeladosclientes)
  - [3. Carperta Compartida Dominio](#3-carperta-compartida-dominio)
    - [3.1 Grupos](#31-grupos)
    - [3.2 Usuarios (Empleados/Clientes)](#32-usuarios-empleadosclientes)


## Definición 

Este equipo tendrá que proporcionar a los usuarios unidades organizativas , grupos , usuarios y un servicio de correo para intercambiar información entre ellos .

## 1. Configuración Vagrant 

### 1.1 Vagrantfile

~~~
Vagrant.configure("2") do |config|
	config.vm.box = "generic/debian12"
	# Definición do router-firewall Server
	config.vm.define "server" do |server|
		server.vm.hostname = "server"
		server.vm.network "private_network", ip: "172.16.5.10", netmask: "255.255.255.0" , gateway: "172.16.5.5"
		
		# Instalacion de Dominio Samba-Tool 
		server.vm.provision "shell",
                        run: "always",
                        path: "dominio_samba.sh"

		# Creacion de Unidades Organizativas 
		server.vm.provision "shell",
                        run: "always",
                        path: "unidades.sh"

		# Creacion de Grupos
		server.vm.provision "shell",
                        run: "always",
                        path: "grupos.sh"

		# Creacion de Empleados  
		server.vm.provision "shell",
                        run: "always",
                        path: "empleados.sh"

		# Creacion de Clientes
		server.vm.provision "shell",
                        run: "always",
                        path: "clientes.sh"

		# Creacion de Servidor de Correos
		server.vm.provision "shell",
                        run: "always",
                        path: "./correo/00_ejecucion_remota.sh"

		# Carpeta Compartida 
		server.vm.synced_folder "./compartida" , "/home/vagrant/compartida"
		server.vm.synced_folder "./correo" , "/home/vagrant/correo"

		# Eliminar default gw da rede NAT creada por defecto
        server.vm.provision "shell",
                        run: "always",
                        inline: "ip route del default"

		# Default Router
		server.vm.provision "shell",
						run: "always",
						inline: "ip route add default via 172.16.5.5"
		server.vm.provider "virtualbox" do |vb|
						vb.name = "server"
						vb.gui = true
						vb.memory = "1024"
						vb.cpus = 1
						vb.linked_clone = true
						vb.customize ["modifyvm", :id, "--groups", "/MasofieAutoDeploy"]
		end
	end
end
~~~


## 2. Scripts Dominios 

### 2.1 Dominio Samba

~~~
#!/bin/bash

# Variables para optimizar Trabajo

DOMINIO="masofiedeploy"
ADMINPASS="abc123."
FORWARDER=8.8.8.8
REALM=$(echo $DOMINIO |tr [[:lower:]] [[:upper:]])
IP=$(hostname -I |cut -d" " -f1)
MAQUINA=$(hostname)

# Atualizacion del sistema 
apt update && apt upgrade

# Instalacion y Configuracion de Zona Horarioa con NTP
apt install ntpsec -y
echo "Configurando ntp"

sed -i "s/^pool/##&/" /etc/ntpsec/ntp.conf
sed -i '/^##pool 3/a\server 0.europe.pool.ntp.org
server 1.europe.pool.ntp.org
server 2.europe.pool.ntp.org
server 3.europe.pool.ntp.org
' /etc/ntpsec/ntp.conf

# Reinicio del Servicio NTP
systemctl restart ntpsec.service
ntpq -P

# Instalacion y Configuracion de Samba-Tool
apt update
apt-get install samba smbclient winbind -y
rm /etc/samba/smb.conf

# Desistalacion de BIND-9
apt remove bind9 -y

# Identificacion de Servidor

echo "search $DOMINIO.local" > /etc/resolv.conf
echo "domain $DOMINIO.local" >> /etc/resolv.conf
echo "nameserver $IP" >> /etc/resolv.conf

# Promocionamos Samba-Tool como Controlador de Dominio 
samba-tool domain provision \
--use-rfc2307 \
--realm=$REALM.LOCAL \
--domain=$REALM \
--server-role="dc" \
--dns-backend=SAMBA_INTERNAL \
--adminpass=$ADMINPASS \
--option="dns forwarder=$FORWARDER"
sed -i "/^\tidmap_ldb/a\\\tallow dns updates = nonsecure" /etc/samba/smb.conf

# Reiniciamos los Servicios de SMBD
smbclient -L localhost -U%
systemctl stop smbd nmbd winbind
systemctl disable smbd nmbd winbind
systemctl unmask samba-ad-dc
systemctl start samba-ad-dc
systemctl enable samba-ad-dc

# Comprobamos los Servicios
host -t SRV _ldap._tcp.$MAQUINA.local
host -t SRV _kerberos._udp.$MAQUINA.local
host -t A $MAQUINA.$DOMINIO.local
~~~

### 2.2 Unidades Organizativas

~~~
#!/bin/bash

# Tomar Nombre de Dominio
DOMINIO=$(grep "^domain" /etc/resolv.conf | cut -d" " -f2 | cut -d"." -f1 )

# Unidades Organizativas de Usuarios
samba-tool ou create "OU=usuarios,DC=$DOMINIO,DC=local" --description='UO usuarios'
samba-tool ou create "OU=empleados,OU=usuarios,DC=$DOMINIO,DC=local" --description="UO empleados usuarios"
samba-tool ou create "OU=clientes,OU=usuarios,DC=$DOMINIO,DC=local" --description="UO clientes usuarios"

# Unidades Organizativas Departamentos
samba-tool ou create "OU=departamento,DC=$DOMINIO,DC=local" --description='UO departamentos'
samba-tool ou create "OU=taller,OU=departamento,DC=$DOMINIO,DC=local" --description="UO taller departamentos"
samba-tool ou create "OU=venta,OU=departamento,DC=$DOMINIO,DC=local" --description="UO venta departamentos"

# Grupos 
samba-tool ou create "OU=grupos,DC=$DOMINIO,DC=local" --description="UO grupos"
~~~

### 2.3 Grupos 

~~~
#!/bin/bash

# Leer Nombre del Dominio
DOMINIO=$(cat /etc/resolv.conf |grep "^domain" |cut -d" " -f2 )

fish_grupo=/home/vagrant/compartida/grupos.cnf

# Creacion de Grupos de Usuarios
for grupo in $(cat $fish_grupo|grep -v "^#" | tr -d '\r')
do
    NOMBRE_GROUP=$(echo $grupo |cut -f1 -d":")
    GIDNUMBER=$(echo $grupo |cut -f2 -d":")
    samba-tool group add $NOMBRE_GROUP --groupou=OU=grupos --gid-number=$GIDNUMBER --nis-domain=$DOMINIO
done
~~~

### 2.4 Usuarios (Empelados/Clientes)

~~~
#!/bin/bash

# Variable de Empleados
ficheros_grupos=/home/vagrant/compartida/empleados.cnf
DOMINIO=$(cat /etc/resolv.conf |grep "^domain" |cut -d" " -f2 )

for emp in $(cat $ficheros_grupos | grep -v "^#" | tr -d '\r'); do
    LOGIN=$(echo $emp | cut -d":" -f1)
    NOMBRE=$(echo $emp | cut -d":" -f2)
    APELLIDOS=$(echo $emp | cut -d":" -f3)
    GRUPO=$(echo $emp | cut -d":" -f4)
    USERID=$(echo $emp | cut -d":" -f5)

    PASSWORD="abc123."

    # Crear el usuario
    samba-tool user create $LOGIN $PASSWORD --given-name=$NOMBRE --surname=$APELLIDOS \
        --userou=OU=empleados,OU=usuarios \
        --uid-number=$USERID \
        --home-drive=Z: --home-directory=\\\\server\\usuarios\\personales\\empleados\\$LOGIN \
        --profile-path=\\\\server\\usuarios\\perfilesWindows\\$LOGIN \
        --script-path=inicio.bat

    # Entregar los Usuarios a su Grupo
    samba-tool group addmembers "usuarios" $LOGIN
    samba-tool group addmembers "$GRUPO-usuarios" $LOGIN
done < /home/vagrant/compartida/empleados.cnf
~~~

~~~
#!/bin/bash

# Variable de Clientes
ficheros_grupos=/home/vagrant/compartida/clientes.cnf
DOMINIO=$(cat /etc/resolv.conf |grep "^domain" |cut -d" " -f2 )

for emp in $(cat $ficheros_grupos | grep -v "^#" | tr -d '\r'); do
    LOGIN=$(echo $emp | cut -d":" -f1)
    NOMBRE=$(echo $emp | cut -d":" -f2)
    APELLIDOS=$(echo $emp | cut -d":" -f3)
    GRUPO=$(echo $emp | cut -d":" -f4)
    USERID=$(echo $emp | cut -d":" -f5)

    PASSWORD="abc123."

    # Crear el usuario
    samba-tool user create $LOGIN $PASSWORD --given-name=$NOMBRE --surname=$APELLIDOS \
        --userou=OU=clientes,OU=usuarios \
        --uid-number=$USERID \
        --home-drive=Z: --home-directory=\\\\server\\usuarios\\personales\\clientes\\$LOGIN \
        --profile-path=\\\\server\\usuarios\\perfilesWindows\\$LOGIN \
        --script-path=inicio.bat

    # Entregar los Usuarios a su Grupo
    samba-tool group addmembers "usuarios" $LOGIN
    samba-tool group addmembers "$GRUPO-usuarios" $LOGIN
done < /home/vagrant/compartida/empleados.cnf
~~~

## 3. Carperta Compartida Dominio

### 3.1 Grupos 

~~~
usuarios:10000
empleados-usuarios:10001
clientes-usuarios:10002
departamentos:10003
taller-departamentos:10004
venta-departamentos:10005
~~~

### 3.2 Usuarios (Empleados/Clientes)

~~~
ana:Ana:Lopez:empleados:10001
pedro:Pedro:Sanchez:empleados:10002
sofia:Sofia:Rodriguez:empleados:10003
david:David:Sanchez:empleados:10004
maria:Maria:Lopez:empleados:10005
lucas:Lucas:Sanchez:empleados:10006
jeny:Jeny:Rodriguez:empleados:10007
amelia:Amelia:Sanchez:empleados:10008
juan:Juan:Felix:empleados:10009
julieta:Julieta:Alcantara:empleados:10010
alejandro:Alejandro:Sanchez:empleados:10011
marco:Marco:Rodriguez:empleados:10012
carlos:Carlos:Sanchez:empleados:10013
valentina:Valentina:Felix:empleados:10014
luis:Luis:Alcantara:empleados:10015
~~~

~~~
elena:Elena:Lopez:clientes:10001
roberto:Roberto:Sanchez:clientes:10002
paula:Paula:Rodriguez:clientes:10003
diego:Diego:Sanchez:clientes:10004
laura:Laura:Lopez:clientes:10005
gonzalo:Gonzalo:Sanchez:clientes:10006
marta:Marta:Rodriguez:clientes:10007
santiago:Santiago:Sanchez:clientes:10008
raquel:Raquel:Felix:clientes:10009
victoria:Victoria:Alcantara:clientes:10010
~~~




