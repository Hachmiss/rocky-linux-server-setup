# Definición

Un **firewall (cortafuegos)** es un sistema de seguridad que **monitorea y filtra el tráfico** de red, entrante y saliente, basándose en **reglas predefinidas.** Actúa como una barrera entre **redes de confianza** (como una red interna) e **inseguras** (como internet), **bloqueando accesos no autorizados** y amenazas como malware.

---
# Configuración del firewall

Antes de comenzar con la configuración del firewall  habrá que comprobar que el sistema tenga levantado el servicio firewalld
Para comprobar si el firewall esta levantado, usaremos el comando

```bash
sudo firewall-cmd --state
``` 

o de igual forma:

```bash
sudo systemctl status firewalld
```

El último comando lo podemos usar para comprobar cualquier servicio en nuestro sistema, es tan simple como ejecutar el comando especificando de que servicio nos interesa conocer el estado mediante el comando `sudo systemctl status <servicio>`.

Si queremos listar todas las reglas que están actualmente aplicándose ejecutaremos el comando `sudo firewall-cmd --list-all`, y la salida es la veremos a continuación:

```bash
[hachmiss ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

De aquí lo que más nos interesa son los campos de servicios y puertos (`services` y `ports`), ya que indican qué servicios y puertos están permitidos actualmente a través del firewall.
## Añadir servicios
Podemos permitir tráfico en el firewall de dos formas:
**Por puerto**:
```bash  
sudo firewall-cmd --add-port=<puerto>/<protocolo>
```
**Por servicio**:
```bash
sudo firewall-cmd --add-service=<servicio>
```
Esto abrirá el puerto/servicio indicado en el firewall.
## Eliminar servicios
De igual forma, podemos eliminar reglas:
**Por puerto**:
```bash
sudo firewall-cmd --remove-port=<puerto>/<protocolo>
```
**Por servicio**:
```bash
sudo firewall-cmd --remove-service=<servicio>
```
Esto eliminará el puerto/servicio indicado en el firewall.

> [!NOTE]  
> Los **servicios** y los **puertos** representan lo mismo: reglas de acceso en el firewall.  
> La diferencia es que los servicios son configuraciones conocidas (por ejemplo, `http` -> puerto 80), mientras que los puertos son una configuración más específica y manual.

Todos los nombres de servicios configurados a puertos se pueden ver mediante:

```bash
sudo firewall-cmd --get-services
```
## Persistencia del firewall
Todos estos cambios que se realizan en el firewall no son persistentes hasta que confirmemos los mismos, es decir, si reiniciamos la máquina las nuevas reglas se pierden. Veamos como hacerlos persistentes (que se mantengan en disco) :

```bash
sudo firewall-cmd --runtime-to-permanent
```

Hay otra manera y es hacer los cambios directamente permanente desde la línea de comandos a la vez que se aplican las reglas, esto se hace mediante la entrada `--permanent` pero los cambios no se aplican hasta ejecutar una recarga mediante `firewall-cmd --reload`.

---
# Instalación de nuestro primer servicio HTTP y SSH
## Descarga de servidor web
Existen una diversidad de servidores web que podremos descargar, pero los que veremos en esta parte serán solamente **Nginx y Apache**. Veamos como podemos descargar y activar cada uno de ellos.
### Instalación Nginx
Es simplemente un paquete así que lo descargaremos como cualquier otro paquete:
```bash
sudo dnf install nginx
```

Y comprobamos si esta activo mediante el comando

```bash
sudo systemctl status nginx
```

Si el servicio no está activo, podemos iniciarlo manualmente con:

```bash
sudo systemctl start nginx
```

y habilitarlo para que se inicie automáticamente en cada arranque del sistema:

```bash
sudo systemctl enable nginx
```

> [!NOTE]  
 >El servicio `nginx` se ejecuta como un proceso gestionado por `systemd`, por lo que su control (inicio, parada, estado) se realiza mediante `systemctl`.
### Instalación Apache
Es muy similar a Nginx ya que se trata de otro paquete. Se descarga mediante el comando:

```bash
sudo dnf install httpd
```

Para comprobar el estado:

```bash
sudo systemctl status httpd
```

Y se inicializa y habilita mediante:

```bash
sudo systemctl start httpd  
sudo systemctl enable httpd
```

### Comprobación del servicio web
Una vez realizada la instalación del servicio web, podemos intentar acceder a la página desde el host.

Para ello, utilizaremos un navegador web en el host y nos conectaremos a la **dirección IP** o al **nombre de la máquina virtual**.  
Si todo está correctamente configurado, se mostrará la página de bienvenida del servidor web.

En caso contrario, seguiremos una serie de pasos de depuración para identificar y solucionar el problema.

> [!NOTE]
> Aunque hayas seguido todos los pasos correctamente, es posible que la página no se muestre todavía.  
> Esto es intencionado, ya que nos permite aprender a identificar y solucionar los problemas más comunes al empezar a configurar un servidor web.
### Comprobación local del servidor  
Lo primero que haremos para ver que esta fallando es intentar conectarse de manera local usando por ejemplo el comando:  

```bash  
curl http://localhost
```

De esta forma observamos si el error esta en el firewall o en el servidor web.  
En caso de estar bien, el comando `curl` devolvería el contenido de la página de bienvenida del servidor, por lo que el problema podría estar en el firewall ya que de manera local nos podríamos conectar.

> [!TIP]  
> Podemos añadir la opción `-v` a `curl` para obtener información detallada de la conexión:
> 
> `curl -v http://localhost`
### Comprobación del firewall
Comprobamos el firewall mediante el comando`firewall-cmd --list-all` que explicábamos en la sección anterior y comprobamos los puertos abiertos. Nos fijaremos en concreto en el puerto 80 que corresponde al servicio HTTP. En caso de no tener abierto el puerto tendremos que abrirlo, para ello usaremos el comando que presentábamos en la sección de firewall .
### Directorios por defecto
**En Nginx**, la página principal se encuentra por defecto en el directorio `/usr/share/nginx/html/index.html`, desde donde se sirve el contenido web inicial.
**En Apache**, la página principal se encuentra por defecto en el directorio `/var/www/html/index.html`, que actúa como raíz del sitio web servido.

De esta forma ya tendremos configurado por completo el servidor para servicios web y quedando el sistema preparado para la configuración y despliegue de servicios web.

---
## Servicio SSH
Antes de entender el funcionamiento de SSH, es conveniente conocer algunos conceptos básicos de criptografía que permiten garantizar la seguridad de la comunicación.  
  
**SSH (Secure Shell)** es un protocolo que permite el acceso remoto a un sistema de forma segura.    
A diferencia de otros protocolos como **Telnet**, actualmente en desuso, SSH cifra toda la información que se transmite a través de la red, evitando que los datos viajen en texto plano y puedan ser interceptados, siendo más seguro frente a ataques del tipo *Man in the Middle*.  
  
Para establecer una conexión SSH básica podemos utilizar:  
  
```bash  
ssh servidor
```

Tras esto, el sistema nos pedirá:
- usuario (_login_)
- contraseña (_password_)
También es posible especificar directamente el usuario:

```bash
ssh usuario@servidor
```

donde `servidor` puede ser una **dirección IP** o un **nombre de dominio**.

A continuación, veremos cómo SSH consigue establecer una comunicación segura mediante técnicas de cifrado. 

> [!Note]
> Si ya se conocen los conceptos de ***clave simétrica, asimétrica, firma digital y certificado digital***  entonces se recomienda saltar directamente a la sección de [Configuración SSH](FIREWALL_&_SSHD.md#funcionamiento-de-ssh) .
### Criptografía de clave simétrica
En este tipo de criptografía, los participantes comparten una **clave secreta común**.  
El emisor utiliza esta clave para **cifrar la información**, y el receptor emplea la misma clave para **descifrarla**.  

Se dice *simétrica* porque **la misma clave se utiliza tanto para cifrar como para descifrar**, por lo que solo quienes conocen dicha clave pueden acceder al contenido que se transmite.

Algunos algoritmos clásicos de este tipo son **DES** y **Triple DES (TDES)**, aunque hoy en día han sido en gran parte sustituidos por otros más seguros como **AES**.

El principal inconveniente de este sistema es la **gestión de claves**, especialmente en redes grandes:
- Si se utiliza una única clave compartida, perderla o que se filtre compromete toda la comunicación.  
- Si se utilizan múltiples claves (una por cada comunicación), el número de claves crece rápidamente, dificultando su gestión y aumentando el riesgo de pérdida o exposición.
Por este motivo, la criptografía simétrica por sí sola no es suficiente, y suele combinarse con otros métodos más seguros.
### Criptografía de clave asimétrica (RSA)
En la criptografía de clave asimétrica, cada participante dispone de un **par de claves**:
- una **clave pública**, que puede compartirse con cualquier usuario
- una **clave privada**, que debe mantenerse en secreto
Estas claves están matemáticamente relacionadas (aunque en la práctica no son simplemente “números primos”, sino que se generan a partir de ellos mediante algoritmos como **RSA**).

Cuando un usuario desea enviar información de forma segura, cifra el mensaje utilizando la **clave pública del destinatario**.  
De esta manera, solo el propietario de la **clave privada correspondiente** podrá descifrar el mensaje.

Además, la criptografía asimétrica permite garantizar:
- **Confidencialidad** → solo el destinatario puede leer el mensaje  
- **Autenticación** → si un mensaje está firmado con una clave privada, podemos verificar quién lo ha enviado  
- **Integridad** → podemos comprobar que el mensaje no ha sido modificado  
Esto último se consigue mediante el uso de **firmas digitales**, donde el emisor cifra un resumen (*hash*, por ejemplo uno de los más conocidos es el **SHA-256**) del mensaje con su clave privada.

Sin embargo, la criptografía asimétrica por sí sola no garantiza completamente el **no repudio**, ya que para ello es necesario vincular la clave pública con la identidad real del usuario.  Esto se logra mediante el uso de **certificados digitales**, emitidos por autoridades de certificación (CA).

A continuación, veremos el sistema más utilizado para gestionar estos certificados.
### Certificados digitales
Un **certificado digital** es una forma de asegurar que una **clave pública pertenece realmente a una persona o servidor**.  
Para ello interviene una **entidad de confianza** (normalmente estatal), que valida esa información.

Estos certificados se organizan en una especie de “cadena de confianza” (cadena de certificación), donde cada certificado está validado por otro hasta llegar a una entidad reconocida.
#### Visualización de certificados
Podemos ver esta información fácilmente en el navegador.  
Al acceder a una página segura (**HTTPS**), aparece un **candado** en la barra de direcciones.  
Al pulsarlo, se puede consultar quién ha emitido el certificado y comprobar que la conexión es segura.

También es posible ver los certificados instalados en el navegador desde la configuración.
### Configuración SSH
#### Funcionamiento de SSH
Antes de entrar en la configuración y uso de SSH, es importante entender cómo funciona internamente una conexión, ya que a simple vista puede confundir.

SSH combina distintos modos de criptografía para asegurar una comunicación segura entre cliente y servidor.
Cuando un cliente se conecta a un servidor SSH ocurre que:

1. El servidor envía su **clave pública** al cliente
2. El cliente comprueba si ya confía en ese servidor mediante el archivo,`~/.ssh/known_hosts`
Si es la primera vez que se conecta, se solicitará confirmación para confiar en el servidor.
3. Se establece un **canal cifrado** utilizando criptografía
    - Se utiliza criptografía asimétrica para establecer la conexión
    - Después se emplea criptografía simétrica para cifrar el tráfico (es más eficiente)
4. Una vez establecida la conexión segura, se realiza la **autenticación del usuario**

Aquí es donde entran los dos métodos principales:
- **Autenticación por contraseña**  
    El usuario introduce su contraseña, que viaja cifrada dentro del canal seguro.
- **Autenticación mediante claves**  
    El servidor lanza un reto y comprueba que el cliente puede resolverlo usando su clave privada asociada a la clave pública autorizada.
#### Autenticación mediante claves
SSH permite autenticarse sin contraseña utilizando criptografía asimétrica, donde la clave privada permanece en el cliente y la clave pública se copia al servidor para permitir la verificación de la identidad.

Para generar el par de claves:

```bash
ssh-keygen
```

Esto creará por defecto las claves:
- **Clave privada**:`~/.ssh/id_rsa` 
- **Clave pública:**`~/.ssh/id_rsa.pub`  

> [!NOTE]  
> En entornos reales se recomienda proteger la clave privada con contraseña y utilizar herramientas como `ssh-agent`.

Para poder autenticarnos sin contraseña, debemos copiar la clave pública al servidor:

```bash
ssh-copy-id usuario@servidor
```

Entonces se pedirá la contraseña una única vez y la clave pública se guardará en fichero `~/.ssh/authorized_keys`

> [!TIP]  
>Es posible generar una nueva clave SSH especificando su nombre mediante la opción `-f <nombre_clave>`, lo que permite utilizar distintas claves para diferentes servidores.
>
> En este caso, será necesario indicar explícitamente qué clave queremos copiar al servidor utilizando la opción `-i <nombre_clave>.pub` al ejecutar `ssh-copy-id`.
>
> Por convención, las claves SSH se almacenan en el directorio `~/.ssh/`.

A partir de ese momento podremos conectarnos sin contraseña de la misma manera que comentábamos al principio de la sección:

```bash
ssh usuario@servidor
```

Además SSH permite ejecutar comandos directamente sin abrir una sesión interactiva:

```bash
ssh usuario@servidor <comando>
```

Esto es especialmente útil para automatización y administración remota.
#### Configuración del servicio SSH
El archivo principal de configuración del servidor SSH es `/etc/ssh/sshd_config`. Algunos parámetros importantes son:
-  Puerto por defecto:`Port 22` 
-  Deshabilita acceso root: `PermitRootLogin no`
-  Permite login por contraseña: `PasswordAuthentication yes` 

> [!WARNING]
> No debemos confundir el archivo `ssh_config` con `sshd_config`.
> 
> El archivo `ssh_config` define la configuración del **cliente SSH**, mientras que `sshd_config` configura el **demonio SSH del servidor** (`sshd`), es decir, el servicio que acepta conexiones entrantes.
> 
> En este contexto, nos interesa modificar `sshd_config`, ya que estamos configurando el servidor y no el cliente.

Después de modificar el archivo, aplicamos cambios con:

```bash
sudo systemctl reload sshd
```

Y comprobamos el estado:

```bash
sudo systemctl status sshd
```
#### Cambio de puerto SSH
Para modificar el puerto por defecto nos dirigiremos al fichero de modificación del demonio de SSH y modificaremos la línea `#Port 22` descomentándola, y si nos fijamos en el mensaje superior a dicha linea veremos que nos indica que si el sistema utiliza SELinux, debemos permitir el nuevo puerto:

```bash
sudo semanage port -a -t ssh_port_t -p tcp <PUERTO>
```

Después de haber ejecutado el comando anterior (en caso de tener un sistema SELinux como lo es Rocky) reiniciamos el servicio con el comando:

```bash
sudo systemctl restart sshd
```
o si estamos conectados por SSH:
```bash
sudo systemctl reload sshd
```

Después de esto no tenemos que olvidar abrir el nuevo puerto en el firewall del servidor con el protocolo `tcp` como explicabamos en la sección de Firewall-Añadir servicios 
Y a partir de este momento cuando nos intentemos conectar deberemos indicar el nuevo puerto de la siguiente manera:

```bash
ssh -p <PUERTO> usuario@servidor
```

> [!NOTE]
> Esto puede ser interesante para evitar conexiones de equipos que no conozcan el puerto que hemos asignado, pero resulta una capa de seguridad muy pobre debido a que con un escaneo de puertos mediante `nmap` se convierte en solo cuestión de tiempo conocer el nuevo puerto asignado

> [!TIP]  
> Podemos utilizar `nmap --open --top-ports 100 <IP_SERVIDOR>` para realizar un escaneo rápido de los puertos más comunes y comprobar cuáles se encuentran abiertos en el servidor.
> 
> Esto resulta especialmente útil para verificar si servicios como SSH o HTTP están accesibles desde la red.

---
Si todo ha salido bien hasta el momento podemos pasar a la siguiente sección que será [ANSIBLE.bak](ANSIBLE.bak.md) para abordar la automatización.

