# Equipos Clientes 

## Indice 
- [Equipos Clientes](#equipos-clientes)
  - [Indice](#indice)
  - [1. Windows 10](#1-windows-10)
    - [1.1 Configuración Básica](#11-configuración-básica)
    - [1.2 Unir Equipo al Dominio](#12-unir-equipo-al-dominio)
    - [1.3 Comprobaciones](#13-comprobaciones)
  - [2. Debian-10](#2-debian-10)
    - [2.1 Configuración Básica](#21-configuración-básica)
    - [2.2 Instalación de Paquetes Necesarios](#22-instalación-de-paquetes-necesarios)
    - [2.3 Comprobaciones](#23-comprobaciones)


## 1. Windows 10

### 1.1 Configuración Básica

En el equipo de **Windows 10** visualizamos la ip de nuestra máquina para ver si tomo la configuración correcta 


![Mostrar ip Antes de la Configuración](./img/clientes_w10/mostrar_ip_red_antes.png)


Añadimos la ip del servidor de dominio en los servidores de resolución de ***dns*** para poder añadir el equipo al dominio 

![IP de Resolución de Dominio](./img/clientes_w10/2_ip_del_servidor_samba.png)


Después cambaimos el nombre al equipo para así poder indentificarlo mejor a la hora de buscarlo en el servidor . Te pedirá que reinicies el equipo , lo reinicias y esperamos a que reinice para que tome el nuevo nombre 

![Cambio de Nombre al Equipo Windows](./img/clientes_w10/3_cambio_de_nombre.png)

### 1.2 Unir Equipo al Dominio

Ahora iniciamos con la unión del dominio de la siguiente manera . Lo unimos con el usuario **"Administrator"** que es el biene por defecto en los dominios para iniciar sesión 

![Uniendo el equipo al dominio -1](./img/clientes_w10/4_unir_al_dominio.png)


***"POSIBLE ERROR : A LA HORA DE UNIR EL EQUIPO AL DOMINIO PUEDE QUE NO LO ENCUENTRE Y TE DE FALLOS ENTONCES PARA SOLUCIONAR ESTE PROBLEMA SOLO ACTULIZAMOS EL SERVIDOR CON UN UPDATE Y YA VEREMOS QUE NOS FUNCIONA"***

![Uniendo el equipo al dominio - UPDATE](./img/clientes_w10/5_unir_al_dominio_UPDATE.png)

Si el dominio se unió correctamente nos muestra el siguiente mensaje en el entorno de windows , se ve de la siguiente manera 

![Uniendo el equipo al dominio - MENSAJE](./img/clientes_w10/6_unir_al_dominio_MENSAJE.png)


### 1.3 Comprobaciones 

Ahora comprobamos en el servidor que podmeos ver nuestro equipo de **windows 10** añadido correctamente 

~~~
samba-tool computer list
~~~

![Uniendo el equipo al dominio - COMPUTER](./img/clientes_w10/7_w10_servidor_computer.png)

En el equipo de **Windows 10** iniciamos sesión con el usuario ***administrator*** de la siguiente manera 

![Uniendo el equipo al dominio-INICIO DE SESIÓN ADMINISTRATOR](./img/clientes_w10/8_unir_al_dominio_ADMINISTRATOR.png)

Iniciamos con un usuario que este en el dominio y desde el terminal podemos ver que esta unido correctamente 

~~~
ipconfig /all
~~~


![Uniendo el equipo al dominio-INICIO DE SESIÓN](./img/clientes_w10/8_w10_inicio_de_sesion.png)


## 2. Debian-10 

### 2.1 Configuración Básica

Antes de unir el equipo al dominio comprobamos el nombre del equipo para poder comprobar desde el servidor el nombre del cliente y poder identificarlos 

~~~
hostname
~~~


![Cliente Linux - Nombre del Equipo](./img/cliente_linux/1_cliente_linux_nombre.png)


Añadimos el servidor de **DNS** en el fichero ***/etc/resolv.conf*** para buscar la resolución de nombre del dominio 

~~~
nano /etc/resolv.conf
~~~


![Cliente Linux - Servidores de Búsqueda DNS](./img/cliente_linux/2_cliente_linux_fichero_dns.png)


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
