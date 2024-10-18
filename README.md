# sambabox


**SambaBox** es una herramienta que integra la entornos virtualizados utilizando **Vagrant** y **Samba-Tool**  para gestionar los dominios . 
Este proyecto permitirá gestionar dominios , redes , usuarios y servicios en **Samba-Tool** utilizando entornos virtualizados de **Vagrant** , automatizando las tareas de instalación y configuración del sistema , lo que mejorará el rendimiento y reducirá los errores .
En nuestro espacio tendremos 4 máquinas , que son las siguientes : 

<br>

- **Máquina Firewall :** proporcionará la red a los equipos del dominio .
- **Máquina Server :** es la que alojará el dominio , con todos los scripts necesarios .
- **Máquina Clientes :** con Windows e Ubuntu para probar el funcionamiento del dominio con 
- los usuarios finales .

Con todo esto buscamos crear un entorno donde los equipos estén conectados al dominio de 
forma automatizadas , garantizando una administración centralizada y eficiente .

## Especificaciones Técnicas

Para desarrollar este proyecto se utilizará **Vagrant** junto con un proveedor de virtualización que será **VirtualBox** . Se configurará el entorno donde se gestionarán las máquinas virtuales 
necesarias para gestionar las pruebas . Y el sistema tiene que estar preparado para soportar la 
configuración de  redes y almacenamiento compartido .
Vagrant se instalará en **Windows** en este caso , asegurando la correcta configuración de la **PATH** y disponibilidad de los boxes necesarios para cada entorno . Esto facilitará las recopilación del entorno en las diferentes máquinas y sistemas operativos .
Y ademas se instalá **Samba-Tool**  mediante scripts para automatizar la creación y gestión del 
dominio dentro de la red , permitido administrar usuarios y permisos . 


- [Contenido](./configuracion/README.md)
