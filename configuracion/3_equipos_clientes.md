# 💻 Equipos Clientes 
<br>

**📑 Indice**
- [💻 Equipos Clientes](#-equipos-clientes)
	- [🪟 1. Windows 10](#-1-windows-10)
		- [⚙️ 1.1 Configuración Básica](#️-11-configuración-básica)
		- [🔐 1.2 Unir Equipo al Dominio](#-12-unir-equipo-al-dominio)
		- [✅ 1.3 Comprobaciones](#-13-comprobaciones)
	- [🐧 2. Debian 10](#-2-debian-10)
		- [⚙️ 2.1 Configuración Básica](#️-21-configuración-básica)
		- [2.2 Instalación de Paquetes Necesarios](#22-instalación-de-paquetes-necesarios)
		- [2.3 Comprobaciones](#23-comprobaciones)
	- [3. Archivo Necesario](#3-archivo-necesario)
		- [3.1 Vagrantfile](#31-vagrantfile)

<br>

## 🪟 1. Windows 10
<br>

### ⚙️ 1.1 Configuración Básica
<br>


1 - 📡 Verificamos la ``ip`` del equipo para confirmar que recibió la configuración de red correctamente .

![Mostrar ip Antes de la Configuración](./img/clientes_w10/mostrar_ip_red_antes.png)
<br>
<br>


2 - 🧭 Añadimos la ``ip`` del servidor de dominio en las configuraciones de ``dns`` para que pueda resolver el nombre del dominio.

![IP de Resolución de Dominio](./img/clientes_w10/2_ip_del_servidor_samba.png)
<br>
<br>


3 - 🖥️ Cambiamos el nombre del equipo para facilitar su identificación desde el servidor .💡 Windows solicitará reiniciar para aplicar el nuevo nombre.

![Cambio de Nombre al Equipo Windows](./img/clientes_w10/3_cambio_de_nombre.png)
<br>
<br>



### 🔐 1.2 Unir Equipo al Dominio
<br>

1 - Iniciamos el proceso de unión al dominio con el usuario ``Administrator`` (por defecto en Samba)

![Uniendo el equipo al dominio -1](./img/clientes_w10/4_unir_al_dominio.png)
<br>
<br>



2 - ⚠️ ``POSIBLE ERROR`` : Si no encuentra el dominio, actualiza el servidor con :
 
 ```bash
apt update
 ```

![Uniendo el equipo al dominio - UPDATE](./img/clientes_w10/5_unir_al_dominio_UPDATE.png)

<br>
<br>



3 - Si la unión es correcta, Windows mostrará un mensaje de confirmación. 🎉

![Uniendo el equipo al dominio - MENSAJE](./img/clientes_w10/6_unir_al_dominio_MENSAJE.png)
<br>
<br>



### ✅ 1.3 Comprobaciones
<br>

1 - Desde el servidor, ejecutamos :

 ```bash
samba-tool computer list
 ```

🔎 Aquí debe aparecer el nombre del equipo Windows.


![Uniendo el equipo al dominio - COMPUTER](./img/clientes_w10/7_w10_servidor_computer.png)
<br>
<br>


2 - En Windows, iniciamos sesión como ``Administrator`` del dominio.
 

![Uniendo el equipo al dominio-INICIO DE SESIÓN ADMINISTRATOR](./img/clientes_w10/8_unir_al_dominio_ADMINISTRATOR.png)
<br>
<br>


3 - También podemos comprobar desde terminal :

~~~
ipconfig /all
~~~

![Uniendo el equipo al dominio-INICIO DE SESIÓN](./img/clientes_w10/8_w10_inicio_de_sesion.png)
<br>
<br>



## 🐧 2. Debian 10
<br>

### ⚙️ 2.1 Configuración Básica
<br>

1 - Verificamos el nombre del equipo con :

~~~
hostname
~~~

🏷️ Esto facilita identificarlo desde el servidor.

![Cliente Linux - Nombre del Equipo](./img/cliente_linux/1_cliente_linux_nombre.png)
<br>
<br>



2 - Editamos el archivo ``/etc/resolv.conf`` y añadimos la IP del servidor ``dns`` :

~~~
nano /etc/resolv.conf
~~~

![Cliente Linux - Servidores de Búsqueda DNS](./img/cliente_linux/2_cliente_linux_fichero_dns.png)
<br>
<br>



### 2.2 Instalación de Paquetes Necesarios 

Antes de instalar cualquier paquete tenemos que saber que el comando ***apt*** no funciona , para eso utilizamos el comando ***aptitude*** . Y lo instalamos . 

***"POSIBLE ERROR :*** ***ANTES DE EJECUTAR ESTOS COMANDOS ACTULIZAMOS EL SISTEMA CON UN "APT UPDATE" PARA ACTULIZAR LOS PAQUETES Y QUE FUNCIONEN CORRECTAMENTE"***

~~~
apt install aptitude
~~~

Una vez que instalamos los siguientes paquetes para configurar el Reino de **kerberos** 

~~~
aptitude search krb5-user krb5-config 
~~~

![Cliente Linux - Instalación de Paquetes-1](./img/cliente_linux/3_cliente_linux_comando_aptitude_1.png)

Ahora instalamos el paquete ***smbclient*** , nos va dar la posibilidad de tener acceso al dominio 

~~~
aptitude install smbclient samba-common samba-common-bin
~~~

![Cliente Linux - Instalación de Paquetes-2](./img/cliente_linux/3_cliente_linux_comando_aptitude_1.png)


Editamos el fichero ***/etc/samba/smb.conf*** para especificar el dominio 

~~~
nano /etc/samba/smb.conf
~~~

![Cliente Linux - Fichero smb](./img/cliente_linux/5_cliente_linux_fichero_smb.png)


Creamos un fichero llamado ***secrets.tdb*** que lo va ha buscar en el fichero anterior los datos . Este fichero lo creamos en el directorio ***/var/lib/samba/private/*** con el nombre del usuario y el dominio que vamos ha utilizar 

~~~
nano /var/lib/samba/private/secrets.tdb
~~~

![Cliente Linux - Fichero secrets](./img/cliente_linux/6_cliente_linux_fichero_secrets.png)

Ejecutamos el comando para unir el equipo al dominio , para ver si funciona correctamente 

~~~
net ads join -U administrator
~~~

![Cliente Linux - Equipo Unido al Dominio](./img/cliente_linux/7_cliente_linux_fichero_equipo_unido.png)

### 2.3 Comprobaciones 

Desde los clientes podemos ejecutar este comando para ver si nos resulve el nombre del dominio 

~~~
net ads testjoin
~~~
~~~
net ads info
~~~


![Cliente Linux - Comando net](./img/cliente_linux/8_cliente_linux_comando_net.png)


Desde el servidor comprobamos que se ha unido el equipo al cominio mostrados los equipos 

~~~
samba-tool computer list 
~~~

![Cliente Linux - Comando computer](./img/cliente_linux/9_cliente_linux_comando_computer.png)

## 3. Archivo Necesario
### 3.1 Vagrantfile

~~~
# Variables
interfaz_bridge="eno1"
ip_pub="192.168.1.100"
mascara="255.255.255.0"

Vagrant.configure("2") do |config|
	config.vm.box = "generic/debian12"
	# Definición do router-firewall FW
	config.vm.define "fw" do |fw|
		fw.vm.hostname = "fw"
		fw.vm.network "public_network", bridge: interfaz_bridge , ip: ip_pub , netmask: mascara
		fw.vm.network "private_network", ip: "172.16.5.5", netmask: "255.255.255.0" , gateway: "172.16.5.1"
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
			inline: "ip route add default via 192.168.1.1"
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