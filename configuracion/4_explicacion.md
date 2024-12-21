# Explicación de Ejecución de Vagranfile y Dominio


*************************************************
	- MÉTODO DE EJECUCIÓN DE VAGRANT : 

    1 - RECOMENDACIÓN : CREA UNA CAREPTA LLAMADA VAGARNT Y AHI AÑADES LA CARPETA DE SAMBABOX
	2 - LUEGO POSICIONATE EN LA CARPETA DESDE EL TERMINAL (CMD)
	3 - CUANDO ESTES EN EL CMD EJECUTA EL COMANDO "VAGRANT UP" (PARA CREAR LAS MÁQUINAS)
	
	LA MÁQUINA WINDOWS ES MÁS PESADA PUEDE TARDAR UN POCO EN CREARSE , ESPERA UNOS MINUTOS 
*************************************************

*************************************************
	- URIR EQUIPO AL DOMINIO :

	1 - PARA UNIRTE AL DOMINIO LA CONTRASEÑA ES "abc123."
    LA CONTRASEÑA ES LA MISMA PARA TODOS LOS EQUIPOS Y USUARIOS QUE ESTAN EN EL DOMINIO
*************************************************


*************************************************
	- LINEAS DE MODIFICACIÓN :

    RECUERDA QUE LAS MÁQUINAS VAN HA TENER INTERNET CON LA RED QUE ESTAS UTILIZANDO EN EL MOMENTO 
	MODIFICA EL SCRIPT EN LA PARTE DE LA RED PUBLICA Y AÑADE AHÍ TU RED , POR SI TIENES ALGÚN PROBLEMA CON ESO . 

	OSEA EN ESTAS PARTES EXACTAMENTE :

	ip_pub="**********"
	fw.vm.network "private_network", ip: "172.16.5.5", netmask: "255.255.255.0" , gateway: "**********"
	inline: "ip route add default via **********"
*************************************************

