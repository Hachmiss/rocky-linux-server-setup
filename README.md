# Instalación de Rocky Linux y configuración de VM

## SO Rocky
Para realizar la **virtualización del servidor** y configurar diferentes utilidades comunes en entornos de servidores, utilizaremos el sistema operativo **Rocky Linux**.

En concreto, si se quiere seguir la configuración **exactamente igual que en este proyecto**, se ha utilizado:
`Rocky-9.7-x86_64-minimal.iso`

**Tip:**  
No siempre es recomendable usar la última versión disponible, ya que algunas pueden presentar problemas o cambios recientes. Conviene informarse antes de empezar.

Página oficial:
[https://rockylinux.org](https://rockylinux.org)

---

Una vez descargado el sistema operativo, ya tenemos la **imagen ISO lista**.
El siguiente paso será descargar una herramienta de **virtualización** para crear la máquina virtual.
En este proyecto se utilizará:
`VirtualBox`

Aunque realmente **se podría usar cualquier otro sistema de virtualización**.

---

# Primeros pasos con VirtualBox
El objetivo inicial es **levantar el sistema operativo dentro de una máquina virtual** y empezar a configurarlo como si fuera un servidor.

Durante este proyecto veremos cómo:
- Iniciar y configurar correctamente el sistema operativo en una **VM**
- Conectarnos al servidor mediante **SSH**
- Configurar **RAIDs**
- Configurar **LVM**
- Gestionar el **firewall del servidor**
- Configurar **SSHD**
- Automatizar configuraciones usando **Ansible**
    

---

# Flujo general del proyecto
El orden aproximado de trabajo será:

1. Crear la máquina virtual  
2. Instalar Rocky Linux  
3. Configurar acceso SSH  
4. Configurar almacenamiento (RAID + LVM)  
5. Configurar seguridad del servidor  
6. Automatizar configuraciones con Ansible

---
Para seguir el proyecto en orden se recomienda continuar con [VM](./docs/VM.md).