# Abusing Docker Socket for Privilege Escalation

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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

Hay algunas ocasiones en las que solo tienes **acceso al socket de docker** y quieres usarlo para **escalar privilegios**. Algunas acciones pueden ser muy sospechosas y es posible que desees evitarlas, así que aquí puedes encontrar diferentes flags que pueden ser útiles para escalar privilegios:

### Via mount

Puedes **montar** diferentes partes del **sistema de archivos** en un contenedor que se ejecuta como root y **acceder** a ellas.\
También podrías **abusar de un mount para escalar privilegios** dentro del contenedor.

* **`-v /:/host`** -> Monta el sistema de archivos del host en el contenedor para que puedas **leer el sistema de archivos del host.**
* Si quieres **sentirte como si estuvieras en el host** pero estando en el contenedor, podrías desactivar otros mecanismos de defensa usando flags como:
* `--privileged`
* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `-security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* \*\*`--device=/dev/sda1 --cap-add=SYS_ADMIN --security-opt apparmor=unconfined` \*\* -> Esto es similar al método anterior, pero aquí estamos **montando el disco del dispositivo**. Luego, dentro del contenedor ejecuta `mount /dev/sda1 /mnt` y puedes **acceder** al **sistema de archivos del host** en `/mnt`
* Ejecuta `fdisk -l` en el host para encontrar el dispositivo `</dev/sda1>` para montar
* **`-v /tmp:/host`** -> Si por alguna razón solo puedes **montar algún directorio** del host y tienes acceso dentro del host. Móntalo y crea un **`/bin/bash`** con **suid** en el directorio montado para que puedas **ejecutarlo desde el host y escalar a root**.

{% hint style="info" %}
Ten en cuenta que tal vez no puedas montar la carpeta `/tmp`, pero puedes montar una **carpeta diferente escribible**. Puedes encontrar directorios escribibles usando: `find / -writable -type d 2>/dev/null`

**Ten en cuenta que no todos los directorios en una máquina linux soportarán el bit suid!** Para verificar qué directorios soportan el bit suid, ejecuta `mount | grep -v "nosuid"` Por ejemplo, generalmente `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` y `/var/lib/lxcfs` no soportan el bit suid.

Ten en cuenta también que si puedes **montar `/etc`** o cualquier otra carpeta **que contenga archivos de configuración**, puedes cambiarlos desde el contenedor de docker como root para **abusar de ellos en el host** y escalar privilegios (quizás modificando `/etc/shadow`)
{% endhint %}

### Escaping from the container

* **`--privileged`** -> Con este flag [eliminamos toda la aislamiento del contenedor](docker-privileged.md#what-affects). Consulta técnicas para [escapar de contenedores privilegiados como root](docker-breakout-privilege-escalation/#automatic-enumeration-and-escape).
* **`--cap-add=<CAPABILITY/ALL> [--security-opt apparmor=unconfined] [--security-opt seccomp=unconfined] [-security-opt label:disable]`** -> Para [escalar abusando de capacidades](../linux-capabilities.md), **concede esa capacidad al contenedor** y desactiva otros métodos de protección que pueden evitar que el exploit funcione.

### Curl

En esta página hemos discutido formas de escalar privilegios usando flags de docker, puedes encontrar **formas de abusar de estos métodos usando el comando curl** en la página:

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
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
