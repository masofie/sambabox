# 🔧 SambaBox: Gestión de Dominio, Usuarios y Servicios
<br>

**``SambaBox``** es una herramienta educativa que utiliza Vagrant y Samba-Tool para desplegar y administrar un **dominio samba4** de forma **automática** en un entorno virtualizado. Incluye un *``firewall``* , un *``servidor de dominio``* , y *``clientes Windows``* y Ubuntu para realizar pruebas reales.

> Ideal para aprender y practicar administración de redes y sistemas 🧠💻
<br>
<br>


## 📦 Máquinas Virtuales Incluidas
<br>


| Máquina           | Función                                                                 |
|----------------   |-------------------------------------------------------------------------|
| 🔥 Firewall       | Controla el acceso a Internet y enruta la red interna                   |
| 🖥️ Servidor       | Aloja el dominio Samba4 y ejecuta los scripts de configuración          |
| 🐧 Cliente Linux  | Cliente Ubuntu para probar autenticación en el dominio                  |
| 🪟 Cliente Win    | Cliente Windows (requiere imagen ISO) para unirse al dominio            |

<br>
<br>

## ⚙️ Requisitos
<br>

Antes de comenzar, asegúrate de tener instalado:

- 🧰 [Vagrant](https://www.vagrantup.com/downloads)
- 📦 [VirtualBox](https://www.virtualbox.org/)
- 🪟 Windows (con Vagrant añadido al PATH)
- 💽 ISO de Windows (para el cliente Windows, si se incluye)

<br>
<br>

## 🚀 Instrucciones de uso
<br>

1. **Clona el repositorio**  
   ```bash
   git clone https://github.com/tuusuario/SambaBox.git
   cd SambaBox

<br>
<br>


# 🗂️ Estructura del Repositorio
<br>



- [Contenido](./configuracion/README.md)
