# Seguridad de Docker

{% hint style="success" %}
Aprende y practica Hacking en AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprende y practica Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoya a HackTricks</summary>

* Revisa los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparte trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Usa [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) para construir y **automatizar flujos de trabajo** fácilmente, impulsados por las **herramientas comunitarias más avanzadas** del mundo.\
Obtén acceso hoy:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## **Seguridad Básica del Motor de Docker**

El **motor de Docker** emplea los **Namespaces** y **Cgroups** del núcleo de Linux para aislar contenedores, ofreciendo una capa básica de seguridad. Se proporciona protección adicional a través de **Capabilities dropping**, **Seccomp** y **SELinux/AppArmor**, mejorando el aislamiento de los contenedores. Un **plugin de autenticación** puede restringir aún más las acciones del usuario.

![Seguridad de Docker](https://sreeninet.files.wordpress.com/2016/03/dockersec1.png)

### Acceso Seguro al Motor de Docker

El motor de Docker se puede acceder localmente a través de un socket Unix o de forma remota utilizando HTTP. Para el acceso remoto, es esencial emplear HTTPS y **TLS** para garantizar la confidencialidad, integridad y autenticación.

El motor de Docker, por defecto, escucha en el socket Unix en `unix:///var/run/docker.sock`. En sistemas Ubuntu, las opciones de inicio de Docker se definen en `/etc/default/docker`. Para habilitar el acceso remoto a la API y al cliente de Docker, expón el daemon de Docker a través de un socket HTTP añadiendo la siguiente configuración:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H tcp://192.168.56.101:2376"
sudo service docker restart
```
Sin embargo, exponer el daemon de Docker a través de HTTP no se recomienda debido a preocupaciones de seguridad. Es aconsejable asegurar las conexiones utilizando HTTPS. Hay dos enfoques principales para asegurar la conexión:

1. El cliente verifica la identidad del servidor.
2. Tanto el cliente como el servidor se autentican mutuamente la identidad del otro.

Se utilizan certificados para confirmar la identidad de un servidor. Para ejemplos detallados de ambos métodos, consulta [**esta guía**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/).

### Seguridad de las Imágenes de Contenedor

Las imágenes de contenedor pueden almacenarse en repositorios privados o públicos. Docker ofrece varias opciones de almacenamiento para imágenes de contenedor:

* [**Docker Hub**](https://hub.docker.com): Un servicio de registro público de Docker.
* [**Docker Registry**](https://github.com/docker/distribution): Un proyecto de código abierto que permite a los usuarios alojar su propio registro.
* [**Docker Trusted Registry**](https://www.docker.com/docker-trusted-registry): La oferta de registro comercial de Docker, que cuenta con autenticación de usuario basada en roles e integración con servicios de directorio LDAP.

### Escaneo de Imágenes

Los contenedores pueden tener **vulnerabilidades de seguridad** ya sea por la imagen base o por el software instalado sobre la imagen base. Docker está trabajando en un proyecto llamado **Nautilus** que realiza un escaneo de seguridad de los Contenedores y lista las vulnerabilidades. Nautilus funciona comparando cada capa de imagen de Contenedor con un repositorio de vulnerabilidades para identificar agujeros de seguridad.

Para más [**información lee esto**](https://docs.docker.com/engine/scan/).

* **`docker scan`**

El comando **`docker scan`** te permite escanear imágenes de Docker existentes utilizando el nombre o ID de la imagen. Por ejemplo, ejecuta el siguiente comando para escanear la imagen hello-world:
```bash
docker scan hello-world

Testing hello-world...

Organization:      docker-desktop-test
Package manager:   linux
Project name:      docker-image|hello-world
Docker image:      hello-world
Licenses:          enabled

✓ Tested 0 dependencies for known issues, no vulnerable paths found.

Note that we do not currently have vulnerability data for your image.
```
* [**`trivy`**](https://github.com/aquasecurity/trivy)
```bash
trivy -q -f json <container_name>:<tag>
```
* [**`snyk`**](https://docs.snyk.io/snyk-cli/getting-started-with-the-cli)
```bash
snyk container test <image> --json-file-output=<output file> --severity-threshold=high
```
* [**`clair-scanner`**](https://github.com/arminc/clair-scanner)
```bash
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```
### Firma de Imágenes de Docker

La firma de imágenes de Docker garantiza la seguridad e integridad de las imágenes utilizadas en contenedores. Aquí hay una explicación condensada:

* **Docker Content Trust** utiliza el proyecto Notary, basado en The Update Framework (TUF), para gestionar la firma de imágenes. Para más información, consulta [Notary](https://github.com/docker/notary) y [TUF](https://theupdateframework.github.io).
* Para activar la confianza en el contenido de Docker, establece `export DOCKER_CONTENT_TRUST=1`. Esta función está desactivada por defecto en Docker versión 1.10 y posteriores.
* Con esta función habilitada, solo se pueden descargar imágenes firmadas. El primer empuje de imagen requiere establecer frases de paso para las claves raíz y de etiquetado, con Docker también soportando Yubikey para una seguridad mejorada. Más detalles se pueden encontrar [aquí](https://blog.docker.com/2015/11/docker-content-trust-yubikey/).
* Intentar descargar una imagen no firmada con la confianza en el contenido habilitada resulta en un error "No trust data for latest".
* Para los empujes de imagen después del primero, Docker solicita la frase de paso de la clave del repositorio para firmar la imagen.

Para respaldar tus claves privadas, utiliza el comando:
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
Cuando se cambian los hosts de Docker, es necesario mover las claves de root y del repositorio para mantener las operaciones.

***

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Utiliza [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) para construir y **automatizar flujos de trabajo** fácilmente, impulsados por las **herramientas comunitarias más avanzadas** del mundo.\
Obtén acceso hoy:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## Características de Seguridad de Contenedores

<details>

<summary>Resumen de las Características de Seguridad de Contenedores</summary>

**Características Principales de Aislamiento de Procesos**

En entornos contenedorizados, aislar proyectos y sus procesos es fundamental para la seguridad y la gestión de recursos. Aquí hay una explicación simplificada de conceptos clave:

**Namespaces**

* **Propósito**: Asegurar el aislamiento de recursos como procesos, red y sistemas de archivos. Particularmente en Docker, los namespaces mantienen los procesos de un contenedor separados del host y de otros contenedores.
* **Uso de `unshare`**: El comando `unshare` (o la llamada al sistema subyacente) se utiliza para crear nuevos namespaces, proporcionando una capa adicional de aislamiento. Sin embargo, aunque Kubernetes no bloquea esto inherentemente, Docker sí lo hace.
* **Limitación**: Crear nuevos namespaces no permite que un proceso vuelva a los namespaces predeterminados del host. Para penetrar en los namespaces del host, uno normalmente requeriría acceso al directorio `/proc` del host, utilizando `nsenter` para la entrada.

**Grupos de Control (CGroups)**

* **Función**: Principalmente utilizados para asignar recursos entre procesos.
* **Aspecto de Seguridad**: Los CGroups en sí no ofrecen seguridad de aislamiento, excepto por la característica `release_agent`, que, si está mal configurada, podría ser explotada para acceso no autorizado.

**Caída de Capacidades**

* **Importancia**: Es una característica de seguridad crucial para el aislamiento de procesos.
* **Funcionalidad**: Restringe las acciones que un proceso root puede realizar al eliminar ciertas capacidades. Incluso si un proceso se ejecuta con privilegios de root, la falta de las capacidades necesarias impide que ejecute acciones privilegiadas, ya que las llamadas al sistema fallarán debido a permisos insuficientes.

Estas son las **capacidades restantes** después de que el proceso elimina las otras:

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

Está habilitado por defecto en Docker. Ayuda a **limitar aún más las syscalls** que el proceso puede llamar.\
El **perfil de Seccomp predeterminado de Docker** se puede encontrar en [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)

**AppArmor**

Docker tiene una plantilla que puedes activar: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

Esto permitirá reducir capacidades, syscalls, acceso a archivos y carpetas...

</details>

### Namespaces

**Namespaces** son una característica del núcleo de Linux que **particiona los recursos del núcleo** de tal manera que un conjunto de **procesos** **ve** un conjunto de **recursos** mientras que **otro** conjunto de **procesos** ve un **conjunto** diferente de recursos. La característica funciona al tener el mismo namespace para un conjunto de recursos y procesos, pero esos namespaces se refieren a recursos distintos. Los recursos pueden existir en múltiples espacios.

Docker utiliza los siguientes Namespaces del núcleo de Linux para lograr la aislamiento de Contenedores:

* pid namespace
* mount namespace
* network namespace
* ipc namespace
* UTS namespace

Para **más información sobre los namespaces** consulta la siguiente página:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

La característica del núcleo de Linux **cgroups** proporciona la capacidad de **restringir recursos como cpu, memoria, io, ancho de banda de red entre** un conjunto de procesos. Docker permite crear Contenedores utilizando la característica cgroup que permite el control de recursos para el Contenedor específico.\
A continuación se muestra un Contenedor creado con memoria de espacio de usuario limitada a 500m, memoria del núcleo limitada a 50m, participación de cpu a 512, blkioweight a 400. La participación de CPU es una proporción que controla el uso de CPU del Contenedor. Tiene un valor predeterminado de 1024 y un rango entre 0 y 1024. Si tres Contenedores tienen la misma participación de CPU de 1024, cada Contenedor puede utilizar hasta el 33% de la CPU en caso de contención de recursos de CPU. blkio-weight es una proporción que controla el IO del Contenedor. Tiene un valor predeterminado de 500 y un rango entre 10 y 1000.
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
Para obtener el cgroup de un contenedor, puedes hacer:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
Para más información, consulta:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### Capacidades

Las capacidades permiten **un control más fino sobre las capacidades que se pueden permitir** para el usuario root. Docker utiliza la característica de capacidad del núcleo de Linux para **limitar las operaciones que se pueden realizar dentro de un contenedor** independientemente del tipo de usuario.

Cuando se ejecuta un contenedor de Docker, el **proceso elimina capacidades sensibles que el proceso podría usar para escapar de la aislamiento**. Esto intenta asegurar que el proceso no podrá realizar acciones sensibles y escapar:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Seccomp en Docker

Esta es una característica de seguridad que permite a Docker **limitar las syscalls** que se pueden usar dentro del contenedor:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### AppArmor en Docker

**AppArmor** es una mejora del núcleo para confinar **contenedores** a un conjunto **limitado** de **recursos** con **perfiles por programa**.:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### SELinux en Docker

* **Sistema de Etiquetado**: SELinux asigna una etiqueta única a cada proceso y objeto del sistema de archivos.
* **Aplicación de Políticas**: Aplica políticas de seguridad que definen qué acciones puede realizar una etiqueta de proceso sobre otras etiquetas dentro del sistema.
* **Etiquetas de Proceso de Contenedor**: Cuando los motores de contenedor inician procesos de contenedor, generalmente se les asigna una etiqueta SELinux confinada, comúnmente `container_t`.
* **Etiquetado de Archivos dentro de Contenedores**: Los archivos dentro del contenedor suelen etiquetarse como `container_file_t`.
* **Reglas de Política**: La política de SELinux asegura principalmente que los procesos con la etiqueta `container_t` solo pueden interactuar (leer, escribir, ejecutar) con archivos etiquetados como `container_file_t`.

Este mecanismo asegura que incluso si un proceso dentro de un contenedor es comprometido, está confinado a interactuar solo con objetos que tienen las etiquetas correspondientes, limitando significativamente el daño potencial de tales compromisos.

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ y AuthN

En Docker, un plugin de autorización juega un papel crucial en la seguridad al decidir si permitir o bloquear solicitudes al demonio de Docker. Esta decisión se toma examinando dos contextos clave:

* **Contexto de Autenticación**: Esto incluye información completa sobre el usuario, como quiénes son y cómo se han autenticado.
* **Contexto de Comando**: Esto comprende todos los datos pertinentes relacionados con la solicitud que se está realizando.

Estos contextos ayudan a asegurar que solo se procesen solicitudes legítimas de usuarios autenticados, mejorando la seguridad de las operaciones de Docker.

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## DoS desde un contenedor

Si no estás limitando adecuadamente los recursos que un contenedor puede usar, un contenedor comprometido podría causar un DoS al host donde se está ejecutando.

* DoS de CPU
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
* DoS de ancho de banda
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## Banderas de Docker Interesantes

### Banderas --privileged

En la siguiente página puedes aprender **qué implica la bandera `--privileged`**:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

Si estás ejecutando un contenedor donde un atacante logra acceder como un usuario de bajo privilegio. Si tienes un **binario suid mal configurado**, el atacante puede abusar de él y **escalar privilegios dentro** del contenedor. Lo que puede permitirle escapar de él.

Ejecutar el contenedor con la opción **`no-new-privileges`** habilitada **prevendrá este tipo de escalada de privilegios**.
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### Otro
```bash
#You can manually add/drop capabilities with
--cap-add
--cap-drop

# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined

# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined

# You can manually disable selinux in docker with
--security-opt label:disable
```
Para más opciones de **`--security-opt`** consulta: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

## Otras Consideraciones de Seguridad

### Gestión de Secretos: Mejores Prácticas

Es crucial evitar incrustar secretos directamente en las imágenes de Docker o usar variables de entorno, ya que estos métodos exponen tu información sensible a cualquiera con acceso al contenedor a través de comandos como `docker inspect` o `exec`.

**Los volúmenes de Docker** son una alternativa más segura, recomendada para acceder a información sensible. Pueden ser utilizados como un sistema de archivos temporal en memoria, mitigando los riesgos asociados con `docker inspect` y el registro. Sin embargo, los usuarios root y aquellos con acceso `exec` al contenedor aún podrían acceder a los secretos.

**Los secretos de Docker** ofrecen un método aún más seguro para manejar información sensible. Para instancias que requieren secretos durante la fase de construcción de la imagen, **BuildKit** presenta una solución eficiente con soporte para secretos en tiempo de construcción, mejorando la velocidad de construcción y proporcionando características adicionales.

Para aprovechar BuildKit, se puede activar de tres maneras:

1. A través de una variable de entorno: `export DOCKER_BUILDKIT=1`
2. Prefijando comandos: `DOCKER_BUILDKIT=1 docker build .`
3. Habilitándolo por defecto en la configuración de Docker: `{ "features": { "buildkit": true } }`, seguido de un reinicio de Docker.

BuildKit permite el uso de secretos en tiempo de construcción con la opción `--secret`, asegurando que estos secretos no se incluyan en la caché de construcción de la imagen o en la imagen final, utilizando un comando como:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
Para secretos necesarios en un contenedor en ejecución, **Docker Compose y Kubernetes** ofrecen soluciones robustas. Docker Compose utiliza una clave `secrets` en la definición del servicio para especificar archivos secretos, como se muestra en un ejemplo de `docker-compose.yml`:
```yaml
version: "3.7"
services:
my_service:
image: centos:7
entrypoint: "cat /run/secrets/my_secret"
secrets:
- my_secret
secrets:
my_secret:
file: ./my_secret_file.txt
```
Esta configuración permite el uso de secretos al iniciar servicios con Docker Compose.

En entornos de Kubernetes, los secretos son compatibles de forma nativa y se pueden gestionar aún más con herramientas como [Helm-Secrets](https://github.com/futuresimple/helm-secrets). Los controles de acceso basados en roles (RBAC) de Kubernetes mejoran la seguridad de la gestión de secretos, similar a Docker Enterprise.

### gVisor

**gVisor** es un núcleo de aplicación, escrito en Go, que implementa una parte sustancial de la superficie del sistema Linux. Incluye un runtime de [Open Container Initiative (OCI)](https://www.opencontainers.org) llamado `runsc` que proporciona un **límite de aislamiento entre la aplicación y el núcleo del host**. El runtime `runsc` se integra con Docker y Kubernetes, lo que facilita la ejecución de contenedores en sandbox.

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers** es una comunidad de código abierto que trabaja para construir un runtime de contenedores seguro con máquinas virtuales ligeras que se sienten y rinden como contenedores, pero que proporcionan **un aislamiento de carga de trabajo más fuerte utilizando tecnología de virtualización de hardware** como una segunda capa de defensa.

{% embed url="https://katacontainers.io/" %}

### Resumen de Consejos

* **No use la bandera `--privileged` ni monte un** [**socket de Docker dentro del contenedor**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)**.** El socket de Docker permite crear contenedores, por lo que es una forma fácil de tomar el control total del host, por ejemplo, ejecutando otro contenedor con la bandera `--privileged`.
* **No ejecute como root dentro del contenedor. Use un** [**usuario diferente**](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user) **y** [**namespaces de usuario**](https://docs.docker.com/engine/security/userns-remap/)**.** El root en el contenedor es el mismo que en el host a menos que se remapee con namespaces de usuario. Solo está ligeramente restringido por, principalmente, namespaces de Linux, capacidades y cgroups.
* [**Elimine todas las capacidades**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) y habilite solo las que son necesarias** (`--cap-add=...`). Muchas cargas de trabajo no necesitan capacidades y agregarlas aumenta el alcance de un posible ataque.
* [**Use la opción de seguridad “no-new-privileges”**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) para evitar que los procesos obtengan más privilegios, por ejemplo, a través de binarios suid.
* [**Limite los recursos disponibles para el contenedor**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**.** Los límites de recursos pueden proteger la máquina de ataques de denegación de servicio.
* **Ajuste** [**seccomp**](https://docs.docker.com/engine/security/seccomp/)**,** [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(o SELinux)** perfiles para restringir las acciones y syscalls disponibles para el contenedor al mínimo requerido.
* **Use** [**imágenes oficiales de docker**](https://docs.docker.com/docker-hub/official_images/) **y requiera firmas** o construya las suyas propias basadas en ellas. No herede ni use imágenes [con puerta trasera](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/). También almacene claves raíz y frases de paso en un lugar seguro. Docker tiene planes para gestionar claves con UCP.
* **Reconstruya regularmente** sus imágenes para **aplicar parches de seguridad al host y a las imágenes.**
* Gestione sus **secretos sabiamente** para que sea difícil para el atacante acceder a ellos.
* Si **expone el demonio de docker, use HTTPS** con autenticación de cliente y servidor.
* En su Dockerfile, **prefiera COPY en lugar de ADD**. ADD extrae automáticamente archivos comprimidos y puede copiar archivos de URLs. COPY no tiene estas capacidades. Siempre que sea posible, evite usar ADD para no ser susceptible a ataques a través de URLs remotas y archivos Zip.
* Tenga **contenedores separados para cada micro-s**ervicio.
* **No ponga ssh** dentro del contenedor, “docker exec” se puede usar para ssh al contenedor.
* Tenga **imágenes de contenedor más pequeñas**.

## Docker Breakout / Escalación de Privilegios

Si está **dentro de un contenedor de docker** o tiene acceso a un usuario en el **grupo de docker**, podría intentar **escapar y escalar privilegios**:

{% content-ref url="docker-breakout-privilege-escalation/" %}
[docker-breakout-privilege-escalation](docker-breakout-privilege-escalation/)
{% endcontent-ref %}

## Bypass del Plugin de Autenticación de Docker

Si tiene acceso al socket de docker o tiene acceso a un usuario en el **grupo de docker pero sus acciones están siendo limitadas por un plugin de autenticación de docker**, verifique si puede **eludirlo:**

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Endurecimiento de Docker

* La herramienta [**docker-bench-security**](https://github.com/docker/docker-bench-security) es un script que verifica docenas de mejores prácticas comunes en torno al despliegue de contenedores Docker en producción. Las pruebas son todas automatizadas y se basan en el [CIS Docker Benchmark v1.3.1](https://www.cisecurity.org/benchmark/docker/).\
Necesita ejecutar la herramienta desde el host que ejecuta docker o desde un contenedor con suficientes privilegios. Descubra **cómo ejecutarlo en el README:** [**https://github.com/docker/docker-bench-security**](https://github.com/docker/docker-bench-security).

## Referencias

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/_fel1x/status/1151487051986087936](https://twitter.com/_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/)
* [https://en.wikipedia.org/wiki/Linux_namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://www.redhat.com/sysadmin/privileged-flag-container-engines](https://www.redhat.com/sysadmin/privileged-flag-container-engines)
* [https://docs.docker.com/engine/extend/plugins_authorization](https://docs.docker.com/engine/extend/plugins_authorization)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/](https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/)

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) para construir y **automatizar flujos de trabajo** impulsados por las **herramientas comunitarias más avanzadas** del mundo.\
Obtenga acceso hoy:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

{% hint style="success" %}
Aprenda y practique Hacking en AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda y practique Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoye a HackTricks</summary>

* ¡Revise los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únase al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síganos** en **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Comparta trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>
{% endhint %}
