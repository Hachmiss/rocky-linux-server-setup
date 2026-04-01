# Discos y particiones

Cuando un sistema dispone de varios dispositivos de almacenamiento, estos se identifican siguiendo una **convención de nombres estándar** en sistemas basados en Linux. 

Los discos se enumeran de forma secuencial mediante letras, comenzando normalmente por **sda**, seguido de **sdb**, **sdc**, **sdd**, y así sucesivamente. Cada uno de estos identificadores representa un **dispositivo físico de almacenamiento** conectado al sistema.

Dentro de cada disco pueden crearse varias **particiones**, que son divisiones lógicas del propio disco destinadas a organizar el almacenamiento de datos o instalar distintos sistemas de archivos. Estas particiones se identifican añadiendo un **número al nombre del disco correspondiente**. Por ejemplo, en el disco **sda** las particiones se nombran como **sda1**, **sda2**, **sda3**, etc., mientras que en el disco **sdb** aparecerían como **sdb1**, **sdb2**, **sdb3**, y así sucesivamente.

Este sistema de nomenclatura permite **identificar de manera clara tanto el disco físico como la partición concreta** dentro de él, facilitando la gestión del almacenamiento, la instalación de sistemas operativos y la administración de volúmenes en el sistema.

> [!TIP]
> Puede encontrar más información en [partition-naming-scheme](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_storage_devices/disk-partitions_managing-storage-devices#partition-naming-scheme_disk-partitions)

---
## Sistemas de ficheros y montaje de discos  
Para montar discos en sistemas Linux primero es necesario **crear un sistema de archivos** sobre el dispositivo que queremos utilizar.  Para ello se utiliza el comando `mkfs`(**make file system**).  

Podemos ver los diferentes tipos de sistemas de archivos disponibles utilizando:  
  
```bash  
mkfs. + <TAB>
```

Esto mostrará las distintas opciones disponibles en el sistema.
### Sistemas de archivos más utilizados
En esta práctica nos interesan principalmente: `ext4`, `xfs`
Ambos están orientados a **sistemas transaccionales**, lo que significa que utilizan **journaling**.  
Esto permite registrar operaciones antes de realizarlas, lo que ayuda a **evitar corrupción de datos** y facilita la recuperación del sistema en caso de fallos.
### Creación del sistema de archivos
Una vez creado el dispositivo lógico (en este caso mediante [RAID](RAID.md) y [LVM](LVM.md)), podemos crear el sistema de archivos usando `mkfs`. En este caso haremos uso de `ext4` y montaremos el sistema de archivos sobre el volumen lógico `nvar` creado en [LVM](LVM.md###creación-del-lv).

```bash
sudo mkfs.ext4 /dev/mapper/raid1-nvar
```
O también podemos hacer equivalentemente:
```bash
sudo mkfs.ext4 /dev/raid1/nvar
```
### Montaje del sistema de archivos
Una vez creado el sistema de archivos, podemos montarlo utilizando el comando `mount`. 
Para comprobar los sistemas de archivos actualmente montados podemos ejecutar:

```bash
mount
```

### Montaje temporal
Para realizar un montaje temporal podemos utilizar el directorio `/mnt`.

Pero para evitar malas prácticas vamos a montar el sistema de archivos sobre un directorio temporal que crearemos en `/mnt` para evitar montarlo directamente en `/mnt`. 

Asi que creamos el directorio `/mnt/nvar` 

```bash
sudo mkdir /mnt/nvar
```

y ejecutamos el siguiente comando para el montaje.

```bash
mount -t ext4 /dev/raid1/nvar /mnt/nvar
```

La opción `-t` indica el **tipo de sistema de archivos**, aunque en algunos casos puede ser redundante si el sistema es capaz de detectarlo automáticamente.

---
Una vez montado el sistema de archivos y si se esta siguiendo el proyecto volvemos al apartado de [Montaje del volumen](LVM.md#montaje-del-volumen) de LVM.

---
### Desmontar un sistema de archivos
Para desmontar un disco utilizamos el comando:`umount`

```bash
umount /mnt/nvar
```

Esto liberará el sistema de archivos y permitirá desmontarlo de forma segura.