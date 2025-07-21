# 🔥 Servidor de Internet (Firewall)
<br>

**📑 Indice**
- [🔥 Servidor de Internet (Firewall)](#-servidor-de-internet-firewall)
	- [📖 Definición](#-definición)
	- [🔧 1. Firewall](#-1-firewall)
		- [🛠️ 1.1 Vagrant](#️-11-vagrant)
	- [📂 2. Ficheros Necesarios](#-2-ficheros-necesarios)
	- [📦 2.1 Instalación de Iptables](#-21-instalación-de-iptables)
	- [🔁 2.2 Configuración del Enrutamiento](#-22-configuración-del-enrutamiento)
	- [📋 2.3 Configuración del Inventario](#-23-configuración-del-inventario)

<br>

## 📖 Definición
<br>

Este equipo cumple el rol de proveedor de acceso a Internet para el resto de máquinas en la red privada. Funciona como un puente ``(gateway)`` que enruta el tráfico entre la red interna y externa , aplicando además reglas de seguridad mediante ``iptables`` .

<br>

## 🔧 1. Firewall
<br>

Aquí se define la configuración general del ``firewall`` , incluyendo el uso de Vagrant para levantar la máquina virtual y conectarla correctamente a las redes.

<br>

### 🛠️ 1.1 Vagrant
<br>

Este bloque de código configura una máquina virtual Debian con dos interfaces de red: una pública y otra privada . Además , automatiza la configuración del enrutamiento y de las reglas ``iptables`` .

<br>
<br>

~~~
# Variables
interfaz_bridge="eno1"
ip_pub="192.168.18.212"
mascara="255.255.255.0"

Vagrant.configure("2") do |config|
  config.vm.box = "generic/debian12"

  config.vm.define "fw" do |fw|
    fw.vm.hostname = "fw"
    
    fw.vm.network "public_network", bridge: interfaz_bridge, ip: ip_pub, netmask: mascara
    fw.vm.network "private_network", ip: "172.16.5.5", netmask: "255.255.255.0"

    fw.vm.provision "shell", run: "always", path: "enrutamiento.sh"
    fw.vm.provision "shell", run: "always", path: "iptables.sh"
    fw.vm.provision "shell", run: "always", inline: "ip route del default"
    fw.vm.provision "shell", run: "always", inline: "ip route add default via 192.168.18.1"

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
<br>
<br>


## 📂 2. Ficheros Necesarios
<br>
En esta sección se explican los archivos adicionales necesarios para que el ``firewall`` funcione correctamente : reglas ``iptables`` , enrutamiento activado y archivo de inventario para ``ansible`` .

<br>
<br>

## 📦 2.1 Instalación de Iptables
<br>

Este script instala ``iptables`` y ``iptables-persistent`` , define políticas por defecto y permite el tráfico desde la red interna hacia el exterior . También configura redirecciones de puertos (DNAT).

<br>
<br>

~~~
apt install iptables -y
DEBIAN_FRONTEND=noninteractive apt install iptables-persistent -y

iptables -F
iptables -X
iptables -Z
iptables -t nat -F

iptables -P FORWARD ACCEPT
iptables -t nat -P PREROUTING ACCEPT
iptables -t nat -P POSTROUTING ACCEPT

iptables -t nat -A POSTROUTING -s 192.168.56.0/24 -o eth1 -j MASQUERADE

iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 4022 -j DNAT --to 192.168.56.253:22
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 4080 -j DNAT --to 192.168.56.253:80

iptables-save > /etc/iptables/rules.v4
~~~
<br>
<br>


## 🔁 2.2 Configuración del Enrutamiento
<br>

Este pequeño script activa el reenvío de paquetes ``(IP forwarding)`` en el sistema, necesario para permitir que otras máquinas accedan a Internet a través del firewall .

<br>
<br>

~~~
#!/bin/bash
sed -i "s/^#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/" /etc/sysctl.conf
sysctl -p
~~~
<br>
<br>


## 📋 2.3 Configuración del Inventario
<br>

Archivo de inventario de Ansible, útil para referenciar esta máquina en tareas de automatización si se desea usar Ansible .

~~~
[default]
serverweb ansible_connection=local
~~~
