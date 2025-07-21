# Explicación de Ejecución de Vagrantfile y Dominio

Sigue los siguientes pasos para ejecutar e iniciar el Vagrantfile y el dominio de `samba-tool`, ponle atención a las recomendaciones, te servirán de ayuda:

~~~~
- 🚀 MÉTODO DE EJECUCIÓN DE VAGRANT:

1 - RECOMENDACIÓN: CREA UNA CARPETA LLAMADA `vagrant` Y AHÍ AÑADES LA CARPETA DE `SambaBox`  
2 - LUEGO POSICIÓNATE EN LA CARPETA DESDE EL TERMINAL (CMD)  
3 - CUANDO ESTÉS EN EL CMD, EJECUTA EL COMANDO `vagrant up` (PARA CREAR LAS MÁQUINAS)

⚠️ LA MÁQUINA WINDOWS ES MÁS PESADA, PUEDE TARDAR UN POCO EN CREARSE. ESPERA UNOS MINUTOS.

- 🔐 UNIR EQUIPO AL DOMINIO:

1 - PARA UNIRTE AL DOMINIO, LA CONTRASEÑA ES `"abc123."`  
LA CONTRASEÑA ES LA MISMA PARA TODOS LOS EQUIPOS Y USUARIOS QUE ESTÁN EN EL DOMINIO.

- ⚙️ LÍNEAS DE MODIFICACIÓN:

RECUERDA QUE LAS MÁQUINAS VAN A TENER INTERNET CON LA RED QUE ESTÉS UTILIZANDO EN ESE MOMENTO.  
MODIFICA EL SCRIPT EN LA PARTE DE LA RED PÚBLICA Y AÑADE AHÍ TU RED, POR SI TIENES ALGÚN PROBLEMA CON ESO.

OSEA, EN ESTAS PARTES EXACTAMENTE:

```bash
ip_pub="**********"
fw.vm.network "private_network", ip: "172.16.5.5", netmask: "255.255.255.0", gateway: "**********"
inline: "ip route add default via **********"
~~~~
