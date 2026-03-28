# Definición 
Aunque siempre existe la posibilidad de trabajar a nivel de particiones en linux, esto es muy poco flexible. Por eso optamos por la tecnología **LVM**.
**Logical Volume Manager (LVM)** es una tecnología que se utiliza en los sistemas Linux que **facilita la gestión de los volúmenes** de almacenamiento. Al contrario que en el caso de las particiones, este nos permite hacer crecer las volúmenes lógicos, por lo que es más versátil

Como hemos comentado tiene **muchas ventajas**, sobre todo en cuanto a la flexibilidad. Este organiza el almacenamiento empleando tres componentes: Physical Volumes, Volume Groups y Logical Volumes.

Durante la instalación de **Rocky Linux**, el asistente de instalación configuró automáticamente **LVM sobre el disco principal**, por lo que ya estamos utilizando esta tecnología desde el inicio del sistema.

---
# Características
Uno de los conceptos más importantes de LVM es el **volume group** que es donde acaban todos los volúmenes físicos y de donde **obtendremos toda la información** que necesitamos **usando logical volumes**, de esta manera no nos interesa de donde se obtienen los datos ya que **pueden venir de varios volúmenes físicos**.
### Physical Volumes (PV)
Los **Physical Volumes** representan los **discos físicos o particiones reales** que se incorporan al sistema LVM.
Estos pueden ser:
- discos completos (`/dev/sdb`)
- particiones (`/dev/sdb1`)
- dispositivos RAID (`/dev/md0`)
Si nos fijamos, siempre va a corresponderse a un dispositivo físico real, y estos se encuentran el el directorio `/dev` como comentábamos en [Sistemas de archivos linux](RAID.md##sistema-de-archivos-linux)

En nuestro caso, utilizaremos el dispositivo RAID que hemos creado previamente.
### Volume Groups (VG)
Un **Volume Group** agrupa varios **discos físicos** en un único conjunto de almacenamiento.

Podemos imaginarlo como un **pool de almacenamiento** del que posteriormente podremos extraer espacio para crear volúmenes lógicos.

Una de las principales ventajas de este sistema es que **los datos pueden almacenarse en varios discos físicos sin que el sistema tenga que preocuparse de dónde se encuentran exactamente**.
### Logical Volumes (LV)
Los **Logical Volumes** son unidades de almacenamiento lógico creadas dentro de un **Volume Group (VG)** en LVM. Estos volúmenes funcionan de manera similar a una partición tradicional, pero con la ventaja de que no dependen directamente de un único disco físico.  Al estar construidos sobre uno o varios **Physical Volumes (PV)** agrupados en un **Volume Group**, los datos pueden distribuirse entre múltiples discos físicos. Esto permite una mayor flexibilidad en la gestión del almacenamiento.  
  
Una de las principales ventajas de los **Logical Volumes** es su capacidad de **crecimiento**. Si se añaden nuevos **Physical Volumes** al **Volume Group**, el espacio disponible puede ampliarse y posteriormente asignarse a los **Logical Volumes**, aumentando así su capacidad sin necesidad de modificar directamente las particiones físicas.

---
Después de ver estos conceptos nos podemos plantear la idea de mantener un **Volume Group** entero que abarque todos los **discos físicos** del **servidor** para conseguir la mayor **flexibilidad de almacenamiento** posible.
Sin embargo, en muchos **entornos de producción** es habitual utilizar diferentes **Volume Groups** según las necesidades de **almacenamiento** y **rendimiento**.
De esta manera se minimizan los posibles problemas agrupando **servicios** con **necesidades similares**. Por ejemplo, se evita que en caso de que se **dañe un disco físico** se vean afectados **varios servicios**, o que al **llenarse un Volume Group** en gran medida este no interfiera con el funcionamiento del resto de los **servicios del sistema**.

---
# Uso en VBox
Una vez creados los correspondientes [RAID's](RAID.md) podremos pasar a montar un **volume group** sobre los mismos, de manera que podremos manejar de manera más flexible todo lo correspondiente a los RAID's.

Antes de empezar a montar nuestro LVM encima de los RAID's, veamos una curiosidad. Como dijimos con anterioridad, el sistema linux usa LVM en su instalación, esto lo podemos apreciar en nuestro servidor Rocky con el comando `lsblk`:
```bash
[hachmiss ~]$ lsblk
NAME              MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda                 8:0    0   20G  0 disk  
├─sda1              8:1    0    1G  0 part  /boot
└─sda2              8:2    0   19G  0 part  
  ├─rlm_vbox-root 253:0    0   17G  0 lvm   /
  └─rlm_vbox-swap 253:1    0    2G  0 lvm   [SWAP]
sdb                 8:16   0   20G  0 disk  
└─md0               9:0    0   20G  0 raid1 
sdc                 8:32   0   20G  0 disk  
└─md0               9:0    0   20G  0 raid1 
sr0                11:0    1 1024M  0 rom  
```
Donde podemos apreciar dos volúmenes lógicos `rlm_vbox-root` y `rlm_vbox-swap` de los que ya hablamos en [lsblk](RAID.md#lsblk), y podemos ver en la columna `TYPE` que efectivamente son LVM.

## Creación de la jerarquía de volúmenes
Vamos a añadir el dispositivo RAID a LVM, por lo que primero tendremos que **inicializarlo como un Physical Volume (PV)**.  
De esta manera podrá ser utilizado posteriormente para crear un **Volume Group (VG)** sobre el que gestionaremos el almacenamiento mediante **Logical Volumes (LV)**.

> [!TIP]
> Podemos aprovechar el autocompletado del bash de Rocky, será muy útil en esta sección. Por ejemplo queremos ver las opciones sobre los **PV** entonces simplemente hacemos `pv + <TAB>` y nos saldrán las listas de opciones.
### Creación del PV
Usando el tip anterior, veamos todas las opciones disponibles para los volúmenes físicos:

```bash
[hachmiss ~]$ pv
pvchange   pvcreate   pvmove     pvresize   pvscan     
pvck       pvdisplay  pvremove   pvs 
```

Ahora como lo que nos interesa es inicializar el RAID anteriormente creado, nos será de interés el comando `pvcreate`. Simplemente le pasaremos como parámetro el dispositivo físico que nos interese para usarlo como **Physical Volume**, en este caso nos **interesa el RAID** que encontramos en `/dev/md0` 

```bash
sudo pvcreate /dev/md0
```

Y podemos visualizarlo con `pvdisplay`

```bash
[hachmiss ~]$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               rlm_vbox
  PV Size               <19,00 GiB / not usable 3,00 MiB
  Allocatable           yes (but full)
  PE Size               4,00 MiB
  Total PE              4863
  Free PE               0
  Allocated PE          4863
  PV UUID               fuJeWH-PqlG-0flz-GmBo-Hz9f-2y3S-J2Po5P
   
  "/dev/md0" is a new physical volume of "19,98 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/md0        <------- nuestro PV
  VG Name               
  PV Size               19,98 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               mEbgB6-dcSK-D7XD-9EWs-2aQp-dk1F-s1QW9o
```
### Creación del VG
Igual que en la sección anterior, veamos **todas las opciones** disponibles para los grupos de volúmenes:

```bash
[hachmiss ~]$ vg
vgcfgbackup      vgconvert        vgextend         vgmerge          vgrename
vgcfgrestore     vgcreate         vgimport         vgmknodes        vgs
vgchange         vgdisplay        vgimportclone    vgreduce         vgscan
vgck             vgexport         vgimportdevices  vgremove         vgsplit
```

Creamos el **Volume Group** con el siguiente comando:

```bash
sudo vgcreate raid1 /dev/md0
```

Donde tenemos los siguientes parámetros:
- `raid1`: nombre que queremos asignarle al **VG**
- `/dev/md0`: **PV** que queremos que formen parte el **VG**

Y podemos confirmar que se ha creado de manera correcta con `vgdisplay`

```bash
[hachmiss ~]$ sudo vgdisplay
  --- Volume group ---
  VG Name               raid1      <------- nuestro VG
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               19,98 GiB
  PE Size               4,00 MiB
  Total PE              5115
  Alloc PE / Size       0 / 0   
  Free  PE / Size       5115 / 19,98 GiB
  VG UUID               FxvtP1-CKQr-XnU0-Qute-9STX-Qdg9-IXRtrX
   
  --- Volume group ---
  VG Name               rlm_vbox
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <19,00 GiB
  PE Size               4,00 MiB
  Total PE              4863
  Alloc PE / Size       4863 / <19,00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               5z8rp7-ccOe-s652-Qoyt-nuM7-IChh-6g24p1
```

### Creación del LV
Igual que venimos haciendo, veamos las diferentes opciones para los **LV**:

```bash
[hachmiss ~]$ lv
lvchange        lvextend        lvmdiskscan     lvmsadc         lvrename
lvconvert       lvm             lvmdump         lvmsar          lvresize
lvcreate        lvmconfig       lvm_import_vdo  lvreduce        lvs
lvdisplay       lvmdevices      lvmpolld        lvremove        lvscan
```

Y ejecutaremos el siguiente comando para crear un **LV**:

```bash
sudo lvcreate -L 4G -n nvar raid1
```

Con este comando conseguimos crear un volumen lógico donde le indicamos con los parámetros lo siguiente:
- `4G`: indicamos el tamaño del volumen lógico es de 4GB
- `nvar`: el nombre del volumen lógico
- `raid1`: **VG** sobre el que queremos montar el volumen lógico

Y podemos ver que efectivamente todo se creó de manera correcta con el comando `lvdisplay`:

```bash
[hachmiss ~]$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/raid1/nvar    <-------- LV creado por nosotros
  LV Name                nvar
  VG Name                raid1
  LV UUID                O9KTC3-0Ada-VTxp-5Str-xkZa-j3R4-OXkUhA
  LV Write Access        read/write
  LV Creation host, time ****, 2026-03-15 20:40:18 +0100
  LV Status              available
  # open                 0
  LV Size                4,00 GiB
  Current LE             1024
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/rlm_vbox/swap
  LV Name                swap
  VG Name                rlm_vbox
  LV UUID                fgHQh8-aXzh-VMRk-xhOe-PtjE-BLaE-pkugEJ
  LV Write Access        read/write
  LV Creation host, time vbox, 2026-03-14 19:15:35 +0100
  LV Status              available
  # open                 2
  LV Size                2,00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/rlm_vbox/root
  LV Name                root
  VG Name                rlm_vbox
  LV UUID                L7vlrB-TfA4-PFF2-U5DW-CMQQ-1P4Q-BAV7Pn
  LV Write Access        read/write
  LV Creation host, time vbox, 2026-03-14 19:15:35 +0100
  LV Status              available
  # open                 1
  LV Size                <17,00 GiB
  Current LE             4351
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

Y también si volvemos a observar el **Volume Group** veremos que ya están en uso 4 gigas de las 20 iniciales:

```bash
  --- Volume group ---
  VG Name               raid1
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               19,98 GiB
  PE Size               4,00 MiB
  Total PE              5115
  Alloc PE / Size       1024 / 4,00 GiB       <------- nuestro LV
  Free  PE / Size       4091 / 15,98 GiB
  VG UUID               FxvtP1-CKQr-XnU0-Qute-9STX-Qdg9-IXRtrX
```

Y de esta manera ya tendríamos disponible el **LV** para su posterior uso. Veremos como podemos usarlo de manera correcta en el siguiente apartado.

---
## Montaje del Volumen
Una vez creado el volumen nos interesaría realizar un montaje en dicho volumen (véase [MOUNT](MOUNT.md) para el montaje del volumen lógico). 

Una vez realizado el montaje, mediante el siguiente comando copiaremos todo lo que estaba en el directorio `var` a nuestro nuevo volumen lógico.

> [!WARNING]
> Copiar directamente el contenido del directorio **`/var`** mientras el sistema está en funcionamiento puede dejar la nueva copia **inconsistente**, ya que este directorio es utilizado constantemente por numerosos servicios del sistema (logs, gestores de paquetes, sockets, procesos temporales, etc.).
>
> Para evitar este problema es recomendable entrar en **modo de mantenimiento**, deteniendo la mayoría de servicios activos antes de realizar la copia. Esto puede hacerse con el siguiente comando:
>
> `sudo systemctl isolate runlevel1.target`
>
> En sistemas Linux existen distintos **niveles de ejecución (*runlevels*)** que determinan qué servicios están activos en cada momento. Tradicionalmente se definen los siguientes:
>
> - **0** → apagado del sistema  
> - **1** → modo mantenimiento (usuario único, servicios mínimos)  
> - **2–5** → distintos niveles multiusuario según la configuración del sistema  
> - **6** → reinicio del sistema

```bash
sudo cp -a /var/* /mnt/nvar/
```

> [!NOTE]  
> A lo largo de esta sección hemos utilizado el nombre **`nvar`** para el **Logical Volume (LV)**, haciendo referencia a *new var*, ya que posteriormente se utilizará para almacenar el contenido del directorio **`/var`**.  
>  
> No obstante, esta elección es únicamente un ejemplo: el mismo procedimiento podría aplicarse a cualquier otro directorio del sistema que se desee separar en un volumen independiente por motivos de organización, rendimiento o administración del almacenamiento.
### Configuración final del montaje  
Una vez copiado el contenido de `/var` al nuevo volumen lógico, el siguiente paso consiste en configurar su montaje automático mediante el fichero `/etc/fstab`, ya que de por si tendríamos que montarlo de nuevo nosotros a mano.  
  
Primero revisamos el contenido actual del fichero:  
  
```bash  
sudo nano /etc/fstab
```

> [!NOTE]
> El **orden de las entradas en `/etc/fstab`** puede ser relevante para la correcta inicialización del sistema.  
> Durante el arranque, el sistema monta primero los sistemas de archivos esenciales, como la **raíz (`/`)**, seguidos de otros puntos de montaje definidos en el fichero.
>
> El orden físico de las líneas en `fstab` no siempre determina estrictamente el orden de montaje. Sin embargo, es recomendable mantener una **organización lógica**, colocando primero los sistemas de archivos fundamentales (por ejemplo `/`, `/boot` o `swap`) y posteriormente los puntos de montaje adicionales como `/var`, `/home` o `/data`.
>
> Mantener esta estructura facilita la lectura del fichero y reduce la probabilidad de errores durante la administración del sistema.

Dentro del fichero `/etc/fstab` habrá que añadir una nueva línea correspondiente al sistema de archivos que queremos montar, en este caso sería:

```bash
/dev/mapper/raid1-nvar /mnt/nvar                  ext4    defaults        0 0
```

Una vez guardados los cambios, podemos comprobar que la configuración es correcta ejecutando el siguiente comando:

```bash
sudo mount -a
```

Este comando intenta montar todos los sistemas de archivos definidos en `/etc/fstab`.  
Si no aparece ningún error, significa que la configuración es válida y que el sistema podrá montar correctamente los dispositivos durante el arranque.

Una vez realizados los cambios en `/etc/fstab`, es recomendable recargar la configuración de `systemd` para que el sistema tenga en cuenta las modificaciones:  
  
```bash  
sudo systemctl daemon-reload
```

---

> [!TIP]  
> Para comprobar que el RAID funciona correctamente, podemos realizar una pequeña escritura sobre el dispositivo RAID:
> 
> `echo 1 > /dev/md0`
> 
> Esto escribe un dato directamente sobre el dispositivo RAID y permite verificar que el volumen responde correctamente.

---

> [!WARNING]  
> No se debe escribir directamente sobre los **discos físicos que forman parte del RAID** (por ejemplo `/dev/sdb` o `/dev/sdc`).  
> Hacerlo podría **corromper el arreglo RAID**, ya que se modificaría el contenido de uno de los discos sin pasar por el controlador RAID.

---
Una vez completada esta sección ya se deberá saber realizar RAIDs así como montar un manejador LVM para los discos. Podremos pasar sin problemas a la sección de [FIREWALL_&_SSHD](FIREWALL_&_SSHD.md).