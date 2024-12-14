# DDexec / EverythingExec

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

## Contexto

En Linux, para ejecutar un programa, debe existir como un archivo, debe ser accesible de alguna manera a través de la jerarquía del sistema de archivos (así es como funciona `execve()`). Este archivo puede residir en el disco o en la RAM (tmpfs, memfd), pero necesitas una ruta de archivo. Esto ha facilitado mucho el control de lo que se ejecuta en un sistema Linux, hace fácil detectar amenazas y herramientas del atacante o prevenir que intenten ejecutar algo de su parte (_e. g._ no permitir que los usuarios no privilegiados coloquen archivos ejecutables en ningún lugar).

Pero esta técnica está aquí para cambiar todo esto. Si no puedes iniciar el proceso que deseas... **entonces secuestras uno que ya existe**.

Esta técnica te permite **eludir técnicas de protección comunes como solo lectura, noexec, listas blancas de nombres de archivos, listas blancas de hash...**

## Dependencias

El script final depende de las siguientes herramientas para funcionar, deben ser accesibles en el sistema que estás atacando (por defecto, las encontrarás en todas partes):
```
dd
bash | zsh | ash (busybox)
head
tail
cut
grep
od
readlink
wc
tr
base64
```
## La técnica

Si puedes modificar arbitrariamente la memoria de un proceso, entonces puedes tomar el control de él. Esto se puede usar para secuestrar un proceso ya existente y reemplazarlo con otro programa. Podemos lograr esto ya sea utilizando la llamada al sistema `ptrace()` (que requiere que tengas la capacidad de ejecutar llamadas al sistema o que tengas gdb disponible en el sistema) o, más interesante, escribiendo en `/proc/$pid/mem`.

El archivo `/proc/$pid/mem` es un mapeo uno a uno de todo el espacio de direcciones de un proceso (_p. ej._ desde `0x0000000000000000` hasta `0x7ffffffffffff000` en x86-64). Esto significa que leer o escribir en este archivo en un desplazamiento `x` es lo mismo que leer o modificar el contenido en la dirección virtual `x`.

Ahora, tenemos cuatro problemas básicos que enfrentar:

* En general, solo root y el propietario del programa del archivo pueden modificarlo.
* ASLR.
* Si intentamos leer o escribir en una dirección no mapeada en el espacio de direcciones del programa, obtendremos un error de I/O.

Estos problemas tienen soluciones que, aunque no son perfectas, son buenas:

* La mayoría de los intérpretes de shell permiten la creación de descriptores de archivo que luego serán heredados por los procesos hijos. Podemos crear un fd apuntando al archivo `mem` de la shell con permisos de escritura... así que los procesos hijos que usen ese fd podrán modificar la memoria de la shell.
* ASLR ni siquiera es un problema, podemos consultar el archivo `maps` de la shell o cualquier otro del procfs para obtener información sobre el espacio de direcciones del proceso.
* Así que necesitamos `lseek()` sobre el archivo. Desde la shell esto no se puede hacer a menos que se use el infame `dd`.

### Con más detalle

Los pasos son relativamente fáciles y no requieren ningún tipo de experiencia para entenderlos:

* Analiza el binario que queremos ejecutar y el cargador para averiguar qué mapeos necesitan. Luego elabora un "shell"code que realizará, en términos generales, los mismos pasos que el kernel hace en cada llamada a `execve()`:
* Crear dichos mapeos.
* Leer los binarios en ellos.
* Configurar permisos.
* Finalmente, inicializar la pila con los argumentos para el programa y colocar el vector auxiliar (necesario para el cargador).
* Saltar al cargador y dejar que haga el resto (cargar las bibliotecas necesarias para el programa).
* Obtener del archivo `syscall` la dirección a la que el proceso regresará después de la llamada al sistema que está ejecutando.
* Sobrescribir ese lugar, que será ejecutable, con nuestro shellcode (a través de `mem` podemos modificar páginas no escribibles).
* Pasar el programa que queremos ejecutar a la entrada estándar del proceso (será `read()` por dicho "shell"code).
* En este punto, depende del cargador cargar las bibliotecas necesarias para nuestro programa y saltar a él.

**Consulta la herramienta en** [**https://github.com/arget13/DDexec**](https://github.com/arget13/DDexec)

## EverythingExec

Hay varias alternativas a `dd`, una de las cuales, `tail`, es actualmente el programa predeterminado utilizado para `lseek()` a través del archivo `mem` (que era el único propósito de usar `dd`). Dichas alternativas son:
```bash
tail
hexdump
cmp
xxd
```
Estableciendo la variable `SEEKER` puedes cambiar el buscador utilizado, _p. ej._:
```bash
SEEKER=cmp bash ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Si encuentras otro seeker válido que no esté implementado en el script, aún puedes usarlo configurando la variable `SEEKER_ARGS`:
```bash
SEEKER=xxd SEEKER_ARGS='-s $offset' zsh ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Bloquea esto, EDRs.

## Referencias
* [https://github.com/arget13/DDexec](https://github.com/arget13/DDexec)

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
