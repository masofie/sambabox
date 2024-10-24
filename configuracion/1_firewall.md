# Servidor de Internet (Firewall)

## Indice 
- [Servidor de Internet (Firewall)](#servidor-de-internet-firewall)
	- [Indice](#indice)
	- [Definición](#definición)
	- [1. Firewall](#1-firewall)
		- [1.1 Vagrant](#11-vagrant)
	- [2. Ficheros Necesarios](#2-ficheros-necesarios)
	- [2.1 Instalación de Iptables](#21-instalación-de-iptables)
	- [2.2 Configuración del Enrutamiento](#22-configuración-del-enrutamiento)
	- [2.3 Configuración del Inventario](#23-configuración-del-inventario)

## Definición

Este es el equipo que tendrá la capacidad de proporcionarle a las otras máquinas que estan en la misma red le de servicio a internet . O sea va hacer como un puente para acceder a internet .

## 1. Firewall 

### 1.1 Vagrant 

~~~
# Variables
interfaz_bridge="eno1"
ip_pub="192.168.18.212"
mascara="255.255.255.0"

Vagrant.configure("2") do |config|
	config.vm.box = "generic/debian12"
	# Definición do router-firewall FW
	config.vm.define "fw" do |fw|
		fw.vm.hostname = "fw"
		fw.vm.network "public_network", bridge: interfaz_bridge , ip: ip_pub , netmask: mascara
		fw.vm.network "private_network", ip: "172.16.5.5", netmask: "255.255.255.0"
		#configurar enrutado
		fw.vm.provision "shell",
                        run: "always",
                        path: "enrutamiento.sh"

		# Configurar Iptables
		fw.vm.provision "shell",
			run: "always",
			path: "iptables.sh"
		# Eliminar default gw da rede NAT creada por defecto
                fw.vm.provision "shell",
                        run: "always",
                        inline: "ip route del default"

		# Default Router
		fw.vm.provision "shell",
			run: "always",
			inline: "ip route add default via 192.168.18.1"
		fw.vm.provider "virtualbox" do |vb|
			vb.name = "fw"
			vb.gui = true
			vb.memory = "1024"
			vb.cpus = 1
			vb.linked_clone = true
			vb.customize ["modifyvm", :id, "--groups", "/MasofieAutoDeploy"]
		end
	end
end
~~~

## 2. Ficheros Necesarios 

## 2.1 Instalación de Iptables 

~~~
apt install iptables -y
DEBIAN_FRONTEND=noninteractive apt install iptables-persistent -y
iptables -F
iptables -X
iptables -Z
iptables -t nat -F
#ESTABLECENSE POLITICAS POR DEFECTO
#iptables -P INPUT ACCEPT
#iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -t nat -P PREROUTING ACCEPT
iptables -t nat -P POSTROUTING ACCEPT

#PERMITIR SALIDA AL EXTERIOR DE LAS LAN
iptables -t nat -A POSTROUTING -s 192.168.56.0/24 -o eth1 -j MASQUERADE

##MODIFICAMOS LA IP Y PUERTO CON EL DNAT
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 4022 -j DNAT --to 192.168.56.253:22

#MODIFICAMOS LA IP Y PUERTO CON EL DNAT
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 4080 -j DNAT --to 192.168.56.253:80

iptables-save > /etc/iptables/rules.v4
~~~

## 2.2 Configuración del Enrutamiento

~~~
#!/bin/bash
sed -i "s/^#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/" /etc/sysctl.conf
sysctl -p
~~~


## 2.3 Configuración del Inventario

~~~
[default]
serverweb ansible_connection=local
~~~
