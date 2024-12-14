# macOS Archivos, Carpetas, Binarios y Memoria

{% hint style="success" %}
Aprende y practica Hacking en AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprende y practica Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoya a HackTricks</summary>

* Revisa los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparte trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos de github.

</details>
{% endhint %}

## Estructura de la jerarquía de archivos

* **/Applications**: Las aplicaciones instaladas deberían estar aquí. Todos los usuarios podrán acceder a ellas.
* **/bin**: Binarios de línea de comandos
* **/cores**: Si existe, se utiliza para almacenar volcados de núcleo
* **/dev**: Todo se trata como un archivo, por lo que puedes ver dispositivos de hardware almacenados aquí.
* **/etc**: Archivos de configuración
* **/Library**: Se pueden encontrar muchas subcarpetas y archivos relacionados con preferencias, cachés y registros aquí. Existe una carpeta Library en la raíz y en el directorio de cada usuario.
* **/private**: No documentado, pero muchas de las carpetas mencionadas son enlaces simbólicos al directorio privado.
* **/sbin**: Binarios esenciales del sistema (relacionados con la administración)
* **/System**: Archivo para hacer funcionar OS X. Aquí deberías encontrar principalmente solo archivos específicos de Apple (no de terceros).
* **/tmp**: Los archivos se eliminan después de 3 días (es un enlace simbólico a /private/tmp)
* **/Users**: Directorio personal para usuarios.
* **/usr**: Configuración y binarios del sistema
* **/var**: Archivos de registro
* **/Volumes**: Las unidades montadas aparecerán aquí.
* **/.vol**: Al ejecutar `stat a.txt` obtienes algo como `16777223 7545753 -rw-r--r-- 1 username wheel ...` donde el primer número es el número de identificación del volumen donde existe el archivo y el segundo es el número de inode. Puedes acceder al contenido de este archivo a través de /.vol/ con esa información ejecutando `cat /.vol/16777223/7545753`

### Carpetas de Aplicaciones

* **Las aplicaciones del sistema** se encuentran en `/System/Applications`
* **Las aplicaciones instaladas** generalmente se instalan en `/Applications` o en `~/Applications`
* **Los datos de la aplicación** se pueden encontrar en `/Library/Application Support` para las aplicaciones que se ejecutan como root y `~/Library/Application Support` para aplicaciones que se ejecutan como el usuario.
* Los **demonios** de aplicaciones de terceros que **necesitan ejecutarse como root** generalmente se encuentran en `/Library/PrivilegedHelperTools/`
* Las aplicaciones **sandboxed** están mapeadas en la carpeta `~/Library/Containers`. Cada aplicación tiene una carpeta nombrada de acuerdo con el ID del paquete de la aplicación (`com.apple.Safari`).
* El **núcleo** se encuentra en `/System/Library/Kernels/kernel`
* **Las extensiones del núcleo de Apple** se encuentran en `/System/Library/Extensions`
* **Las extensiones del núcleo de terceros** se almacenan en `/Library/Extensions`

### Archivos con Información Sensible

MacOS almacena información como contraseñas en varios lugares:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Instaladores pkg Vulnerables

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## Extensiones Específicas de OS X

* **`.dmg`**: Los archivos de imagen de disco de Apple son muy frecuentes para instaladores.
* **`.kext`**: Debe seguir una estructura específica y es la versión de OS X de un controlador. (es un paquete)
* **`.plist`**: También conocido como lista de propiedades, almacena información en formato XML o binario.
* Puede ser XML o binario. Los binarios se pueden leer con:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Aplicaciones de Apple que siguen la estructura de directorio (es un paquete).
* **`.dylib`**: Bibliotecas dinámicas (como archivos DLL de Windows)
* **`.pkg`**: Son lo mismo que xar (formato de archivo extensible). El comando de instalador se puede usar para instalar el contenido de estos archivos.
* **`.DS_Store`**: Este archivo está en cada directorio, guarda los atributos y personalizaciones del directorio.
* **`.Spotlight-V100`**: Esta carpeta aparece en el directorio raíz de cada volumen en el sistema.
* **`.metadata_never_index`**: Si este archivo está en la raíz de un volumen, Spotlight no indexará ese volumen.
* **`.noindex`**: Los archivos y carpetas con esta extensión no serán indexados por Spotlight.
* **`.sdef`**: Archivos dentro de paquetes que especifican cómo es posible interactuar con la aplicación desde un AppleScript.

### Paquetes de macOS

Un paquete es un **directorio** que **se ve como un objeto en Finder** (un ejemplo de paquete son los archivos `*.app`).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Caché de Biblioteca Compartida de Dyld (SLC)

En macOS (y iOS) todas las bibliotecas compartidas del sistema, como frameworks y dylibs, están **combinadas en un solo archivo**, llamado **caché compartida de dyld**. Esto mejora el rendimiento, ya que el código se puede cargar más rápido.

Esto se encuentra en macOS en `/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/` y en versiones anteriores podrías encontrar la **caché compartida** en **`/System/Library/dyld/`**.\
En iOS puedes encontrarlas en **`/System/Library/Caches/com.apple.dyld/`**.

Similar a la caché compartida de dyld, el núcleo y las extensiones del núcleo también se compilan en una caché del núcleo, que se carga en el momento del arranque.

Para extraer las bibliotecas del archivo único de la caché compartida de dylib, era posible usar el binario [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip) que puede que no funcione hoy en día, pero también puedes usar [**dyldextractor**](https://github.com/arandomdev/dyldextractor):

{% code overflow="wrap" %}
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

{% hint style="success" %}
Ten en cuenta que incluso si la herramienta `dyld_shared_cache_util` no funciona, puedes pasar el **binario compartido de dyld a Hopper** y Hopper podrá identificar todas las bibliotecas y permitirte **seleccionar cuál** deseas investigar:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1152).png" alt="" width="563"><figcaption></figcaption></figure>

Algunos extractores no funcionarán ya que los dylibs están preenlazados con direcciones codificadas, por lo que podrían estar saltando a direcciones desconocidas.

{% hint style="success" %}
También es posible descargar la Caché de Biblioteca Compartida de otros dispositivos \*OS en macos utilizando un emulador en Xcode. Se descargarán dentro de: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, como: `$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### Mapeo de SLC

**`dyld`** utiliza la llamada al sistema **`shared_region_check_np`** para saber si el SLC ha sido mapeado (lo que devuelve la dirección) y **`shared_region_map_and_slide_np`** para mapear el SLC.

Ten en cuenta que incluso si el SLC se desliza en el primer uso, todos los **procesos** utilizan la **misma copia**, lo que **elimina la protección ASLR** si el atacante pudo ejecutar procesos en el sistema. Esto fue explotado en el pasado y se solucionó con el paginador de región compartida.

Los grupos de ramas son pequeños dylibs de Mach-O que crean pequeños espacios entre los mapeos de imágenes, haciendo imposible interponer las funciones.

### Sobrescribir SLCs

Usando las variables de entorno:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> Esto permitirá cargar una nueva caché de biblioteca compartida.
* **`DYLD_SHARED_CACHE_DIR=avoid`** y reemplazar manualmente las bibliotecas con enlaces simbólicos a la caché compartida con las reales (necesitarás extraerlas).

## Permisos Especiales de Archivos

### Permisos de Carpeta

En una **carpeta**, **leer** permite **listar** su contenido, **escribir** permite **eliminar** y **escribir** archivos en ella, y **ejecutar** permite **traversar** el directorio. Así que, por ejemplo, un usuario con **permiso de lectura sobre un archivo** dentro de un directorio donde **no tiene permiso de ejecución** **no podrá leer** el archivo.

### Modificadores de Bandera

Hay algunas banderas que se pueden establecer en los archivos que harán que el archivo se comporte de manera diferente. Puedes **verificar las banderas** de los archivos dentro de un directorio con `ls -lO /path/directory`

* **`uchg`**: Conocida como la bandera **uchange**, **previene cualquier acción** que cambie o elimine el **archivo**. Para establecerla haz: `chflags uchg file.txt`
* El usuario root podría **eliminar la bandera** y modificar el archivo.
* **`restricted`**: Esta bandera hace que el archivo esté **protegido por SIP** (no puedes agregar esta bandera a un archivo).
* **`Sticky bit`**: Si un directorio tiene el sticky bit, **solo** el **propietario del directorio o root puede renombrar o eliminar** archivos. Típicamente, esto se establece en el directorio /tmp para evitar que los usuarios ordinarios eliminen o muevan archivos de otros usuarios.

Todas las banderas se pueden encontrar en el archivo `sys/stat.h` (encuéntralo usando `mdfind stat.h | grep stat.h`) y son:

* `UF_SETTABLE` 0x0000ffff: Máscara de banderas cambiables por el propietario.
* `UF_NODUMP` 0x00000001: No volcar archivo.
* `UF_IMMUTABLE` 0x00000002: El archivo no puede ser cambiado.
* `UF_APPEND` 0x00000004: Las escrituras en el archivo solo pueden agregar.
* `UF_OPAQUE` 0x00000008: El directorio es opaco respecto a la unión.
* `UF_COMPRESSED` 0x00000020: El archivo está comprimido (algunos sistemas de archivos).
* `UF_TRACKED` 0x00000040: Sin notificaciones para eliminaciones/renombrados para archivos con esto establecido.
* `UF_DATAVAULT` 0x00000080: Se requiere autorización para leer y escribir.
* `UF_HIDDEN` 0x00008000: Indica que este elemento no debe mostrarse en una GUI.
* `SF_SUPPORTED` 0x009f0000: Máscara de banderas soportadas por superusuario.
* `SF_SETTABLE` 0x3fff0000: Máscara de banderas cambiables por superusuario.
* `SF_SYNTHETIC` 0xc0000000: Máscara de banderas sintéticas de solo lectura del sistema.
* `SF_ARCHIVED` 0x00010000: El archivo está archivado.
* `SF_IMMUTABLE` 0x00020000: El archivo no puede ser cambiado.
* `SF_APPEND` 0x00040000: Las escrituras en el archivo solo pueden agregar.
* `SF_RESTRICTED` 0x00080000: Se requiere autorización para escribir.
* `SF_NOUNLINK` 0x00100000: El elemento no puede ser eliminado, renombrado o montado.
* `SF_FIRMLINK` 0x00800000: El archivo es un firmlink.
* `SF_DATALESS` 0x40000000: El archivo es un objeto sin datos.

### **ACLs de Archivos**

Las **ACLs** de archivos contienen **ACE** (Entradas de Control de Acceso) donde se pueden asignar permisos **más granulares** a diferentes usuarios.

Es posible otorgar a un **directorio** estos permisos: `listar`, `buscar`, `agregar_archivo`, `agregar_subdirectorio`, `eliminar_hijo`, `eliminar_hijo`.\
Y a un **archivo**: `leer`, `escribir`, `agregar`, `ejecutar`.

Cuando el archivo contiene ACLs, encontrarás un "+" al listar los permisos como en:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
Puedes **leer los ACLs** del archivo con:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
Puedes encontrar **todos los archivos con ACLs** con (esto es muy lento):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Atributos Extendidos

Los atributos extendidos tienen un nombre y cualquier valor deseado, y se pueden ver usando `ls -@` y manipular usando el comando `xattr`. Algunos atributos extendidos comunes son:

* `com.apple.resourceFork`: Compatibilidad con el recurso fork. También visible como `filename/..namedfork/rsrc`
* `com.apple.quarantine`: MacOS: mecanismo de cuarentena de Gatekeeper (III/6)
* `metadata:*`: MacOS: varios metadatos, como `_backup_excludeItem`, o `kMD*`
* `com.apple.lastuseddate` (#PS): Fecha de último uso del archivo
* `com.apple.FinderInfo`: MacOS: información del Finder (por ejemplo, etiquetas de color)
* `com.apple.TextEncoding`: Especifica la codificación de texto de archivos de texto ASCII
* `com.apple.logd.metadata`: Usado por logd en archivos en `/var/db/diagnostics`
* `com.apple.genstore.*`: Almacenamiento generacional (`/.DocumentRevisions-V100` en la raíz del sistema de archivos)
* `com.apple.rootless`: MacOS: Usado por la Protección de Integridad del Sistema para etiquetar archivos (III/10)
* `com.apple.uuidb.boot-uuid`: marcas de logd de épocas de arranque con UUID único
* `com.apple.decmpfs`: MacOS: compresión de archivos transparente (II/7)
* `com.apple.cprotect`: \*OS: datos de cifrado por archivo (III/11)
* `com.apple.installd.*`: \*OS: metadatos utilizados por installd, por ejemplo, `installType`, `uniqueInstallID`

### Recursos Forks | macOS ADS

Esta es una forma de obtener **Flujos de Datos Alternativos en máquinas MacOS**. Puedes guardar contenido dentro de un atributo extendido llamado **com.apple.ResourceFork** dentro de un archivo guardándolo en **file/..namedfork/rsrc**.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
Puedes **encontrar todos los archivos que contienen este atributo extendido** con:

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

El atributo extendido `com.apple.decmpfs` indica que el archivo está almacenado cifrado, `ls -l` reportará un **tamaño de 0** y los datos comprimidos están dentro de este atributo. Cada vez que se accede al archivo, se descifrará en memoria.

Este atributo se puede ver con `ls -lO` indicado como comprimido porque los archivos comprimidos también están etiquetados con la bandera `UF_COMPRESSED`. Si un archivo comprimido se elimina esta bandera con `chflags nocompressed </path/to/file>`, el sistema no sabrá que el archivo estaba comprimido y, por lo tanto, no podrá descomprimir y acceder a los datos (pensará que está vacío).

La herramienta afscexpand se puede usar para forzar la descompresión de un archivo.

## **Universal binaries &** Mach-o Format

Los binarios de Mac OS generalmente se compilan como **binarios universales**. Un **binario universal** puede **soportar múltiples arquitecturas en el mismo archivo**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS Process Memory

## macOS memory dumping

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Risk Category Files Mac OS

El directorio `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` es donde se almacena información sobre el **riesgo asociado con diferentes extensiones de archivo**. Este directorio categoriza los archivos en varios niveles de riesgo, influyendo en cómo Safari maneja estos archivos al descargarlos. Las categorías son las siguientes:

* **LSRiskCategorySafe**: Los archivos en esta categoría se consideran **completamente seguros**. Safari abrirá automáticamente estos archivos después de que se descarguen.
* **LSRiskCategoryNeutral**: Estos archivos no vienen con advertencias y **no se abren automáticamente** por Safari.
* **LSRiskCategoryUnsafeExecutable**: Los archivos bajo esta categoría **activan una advertencia** indicando que el archivo es una aplicación. Esto sirve como una medida de seguridad para alertar al usuario.
* **LSRiskCategoryMayContainUnsafeExecutable**: Esta categoría es para archivos, como archivos comprimidos, que podrían contener un ejecutable. Safari **activará una advertencia** a menos que pueda verificar que todos los contenidos son seguros o neutrales.

## Log files

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: Contiene información sobre archivos descargados, como la URL desde donde fueron descargados.
* **`/var/log/system.log`**: Registro principal de sistemas OSX. com.apple.syslogd.plist es responsable de la ejecución de syslogging (puedes verificar si está deshabilitado buscando "com.apple.syslogd" en `launchctl list`).
* **`/private/var/log/asl/*.asl`**: Estos son los Registros del Sistema de Apple que pueden contener información interesante.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: Almacena archivos y aplicaciones accedidos recientemente a través de "Finder".
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Almacena elementos para iniciar al arrancar el sistema.
* **`$HOME/Library/Logs/DiskUtility.log`**: Archivo de registro para la aplicación DiskUtility (información sobre unidades, incluyendo USBs).
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Datos sobre puntos de acceso inalámbricos.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Lista de demonios desactivados.

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
