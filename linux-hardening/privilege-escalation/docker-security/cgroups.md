# CGroups

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Basic Information

**Los Grupos de Control de Linux**, o **cgroups**, son una característica del núcleo de Linux que permite la asignación, limitación y priorización de recursos del sistema como CPU, memoria y E/S de disco entre grupos de procesos. Ofrecen un mecanismo para **gestionar y aislar el uso de recursos** de colecciones de procesos, beneficioso para propósitos como la limitación de recursos, el aislamiento de cargas de trabajo y la priorización de recursos entre diferentes grupos de procesos.

Hay **dos versiones de cgroups**: versión 1 y versión 2. Ambas pueden ser utilizadas de manera concurrente en un sistema. La principal distinción es que **la versión 2 de cgroups** introduce una **estructura jerárquica, similar a un árbol**, que permite una distribución de recursos más matizada y detallada entre grupos de procesos. Además, la versión 2 trae varias mejoras, incluyendo:

Además de la nueva organización jerárquica, la versión 2 de cgroups también introdujo **cambios y mejoras adicionales**, como soporte para **nuevos controladores de recursos**, mejor soporte para aplicaciones heredadas y un rendimiento mejorado.

En general, **la versión 2 de cgroups ofrece más características y mejor rendimiento** que la versión 1, pero esta última aún puede ser utilizada en ciertos escenarios donde la compatibilidad con sistemas más antiguos es una preocupación.

Puedes listar los cgroups v1 y v2 para cualquier proceso mirando su archivo de cgroup en /proc/\<pid>. Puedes comenzar mirando los cgroups de tu shell con este comando:
```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```
La estructura de salida es la siguiente:

* **Números 2–12**: cgroups v1, con cada línea representando un cgroup diferente. Los controladores para estos se especifican adyacentes al número.
* **Número 1**: También cgroups v1, pero únicamente para fines de gestión (establecido por, por ejemplo, systemd), y carece de un controlador.
* **Número 0**: Representa cgroups v2. No se listan controladores, y esta línea es exclusiva en sistemas que solo ejecutan cgroups v2.
* Los **nombres son jerárquicos**, asemejándose a rutas de archivos, indicando la estructura y relación entre diferentes cgroups.
* **Nombres como /user.slice o /system.slice** especifican la categorización de cgroups, con user.slice típicamente para sesiones de inicio gestionadas por systemd y system.slice para servicios del sistema.

### Visualizando cgroups

El sistema de archivos se utiliza típicamente para acceder a **cgroups**, desviándose de la interfaz de llamada del sistema Unix tradicionalmente utilizada para interacciones con el kernel. Para investigar la configuración del cgroup de un shell, se debe examinar el archivo **/proc/self/cgroup**, que revela el cgroup del shell. Luego, al navegar al directorio **/sys/fs/cgroup** (o **`/sys/fs/cgroup/unified`**) y localizar un directorio que comparta el nombre del cgroup, se pueden observar varios ajustes e información sobre el uso de recursos pertinente al cgroup.

![Cgroup Filesystem](<../../../.gitbook/assets/image (1128).png>)

Los archivos de interfaz clave para cgroups están prefijados con **cgroup**. El archivo **cgroup.procs**, que se puede ver con comandos estándar como cat, lista los procesos dentro del cgroup. Otro archivo, **cgroup.threads**, incluye información sobre los hilos.

![Cgroup Procs](<../../../.gitbook/assets/image (281).png>)

Los cgroups que gestionan shells típicamente abarcan dos controladores que regulan el uso de memoria y el conteo de procesos. Para interactuar con un controlador, se deben consultar los archivos que llevan el prefijo del controlador. Por ejemplo, **pids.current** se referiría para determinar el conteo de hilos en el cgroup.

![Cgroup Memory](<../../../.gitbook/assets/image (677).png>)

La indicación de **max** en un valor sugiere la ausencia de un límite específico para el cgroup. Sin embargo, debido a la naturaleza jerárquica de los cgroups, los límites pueden ser impuestos por un cgroup en un nivel inferior en la jerarquía de directorios.

### Manipulando y Creando cgroups

Los procesos se asignan a cgroups mediante **escribir su ID de Proceso (PID) en el archivo `cgroup.procs`**. Esto requiere privilegios de root. Por ejemplo, para agregar un proceso:
```bash
echo [pid] > cgroup.procs
```
De manera similar, **modificar los atributos de cgroup, como establecer un límite de PID**, se realiza escribiendo el valor deseado en el archivo correspondiente. Para establecer un máximo de 3,000 PIDs para un cgroup:
```bash
echo 3000 > pids.max
```
**Crear nuevos cgroups** implica hacer un nuevo subdirectorio dentro de la jerarquía de cgroup, lo que hace que el kernel genere automáticamente los archivos de interfaz necesarios. Aunque los cgroups sin procesos activos pueden ser eliminados con `rmdir`, ten en cuenta ciertas restricciones:

* **Los procesos solo pueden ser colocados en cgroups hoja** (es decir, los más anidados en una jerarquía).
* **Un cgroup no puede poseer un controlador ausente en su padre**.
* **Los controladores para cgroups hijos deben ser declarados explícitamente** en el archivo `cgroup.subtree_control`. Por ejemplo, para habilitar los controladores de CPU y PID en un cgroup hijo:
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
El **cgroup raíz** es una excepción a estas reglas, permitiendo la colocación directa de procesos. Esto se puede utilizar para eliminar procesos de la gestión de systemd.

**Monitorear el uso de CPU** dentro de un cgroup es posible a través del archivo `cpu.stat`, que muestra el tiempo total de CPU consumido, útil para rastrear el uso a través de los subprocesos de un servicio:

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>Estadísticas de uso de CPU como se muestra en el archivo cpu.stat</p></figcaption></figure>

## Referencias

* **Libro: Cómo funciona Linux, 3ra edición: Lo que cada superusuario debería saber por Brian Ward**

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
