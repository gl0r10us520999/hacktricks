# Técnicas Anti-forenses

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

## Tiempos

Un atacante puede estar interesado en **cambiar las marcas de tiempo de los archivos** para evitar ser detectado.\
Es posible encontrar las marcas de tiempo dentro del MFT en los atributos `$STANDARD_INFORMATION` \_\_ y \_\_ `$FILE_NAME`.

Ambos atributos tienen 4 marcas de tiempo: **Modificación**, **acceso**, **creación** y **modificación del registro MFT** (MACE o MACB).

**El explorador de Windows** y otras herramientas muestran la información de **`$STANDARD_INFORMATION`**.

### TimeStomp - Herramienta anti-forense

Esta herramienta **modifica** la información de la marca de tiempo dentro de **`$STANDARD_INFORMATION`** **pero** **no** la información dentro de **`$FILE_NAME`**. Por lo tanto, es posible **identificar** **actividad** **sospechosa**.

### Usnjrnl

El **USN Journal** (Journal de Número de Secuencia de Actualización) es una característica del NTFS (sistema de archivos de Windows NT) que rastrea los cambios en el volumen. La herramienta [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) permite examinar estos cambios.

![](<../../.gitbook/assets/image (801).png>)

La imagen anterior es la **salida** mostrada por la **herramienta** donde se puede observar que se **realizaron algunos cambios** en el archivo.

### $LogFile

**Todos los cambios de metadatos en un sistema de archivos se registran** en un proceso conocido como [escritura anticipada](https://en.wikipedia.org/wiki/Write-ahead\_logging). Los metadatos registrados se mantienen en un archivo llamado `**$LogFile**`, ubicado en el directorio raíz de un sistema de archivos NTFS. Herramientas como [LogFileParser](https://github.com/jschicht/LogFileParser) se pueden usar para analizar este archivo e identificar cambios.

![](<../../.gitbook/assets/image (137).png>)

Nuevamente, en la salida de la herramienta es posible ver que **se realizaron algunos cambios**.

Usando la misma herramienta es posible identificar **a qué hora se modificaron las marcas de tiempo**:

![](<../../.gitbook/assets/image (1089).png>)

* CTIME: Hora de creación del archivo
* ATIME: Hora de modificación del archivo
* MTIME: Modificación del registro MFT del archivo
* RTIME: Hora de acceso del archivo

### Comparación de `$STANDARD_INFORMATION` y `$FILE_NAME`

Otra forma de identificar archivos modificados sospechosos sería comparar el tiempo en ambos atributos buscando **desajustes**.

### Nanosegundos

Las marcas de tiempo de **NTFS** tienen una **precisión** de **100 nanosegundos**. Por lo tanto, encontrar archivos con marcas de tiempo como 2010-10-10 10:10:**00.000:0000 es muy sospechoso**.

### SetMace - Herramienta anti-forense

Esta herramienta puede modificar ambos atributos `$STARNDAR_INFORMATION` y `$FILE_NAME`. Sin embargo, desde Windows Vista, es necesario que un sistema operativo en vivo modifique esta información.

## Ocultamiento de Datos

NFTS utiliza un clúster y el tamaño mínimo de información. Eso significa que si un archivo ocupa un clúster y medio, la **mitad restante nunca se utilizará** hasta que se elimine el archivo. Entonces, es posible **ocultar datos en este espacio de relleno**.

Existen herramientas como slacker que permiten ocultar datos en este espacio "oculto". Sin embargo, un análisis del `$logfile` y `$usnjrnl` puede mostrar que se agregaron algunos datos:

![](<../../.gitbook/assets/image (1060).png>)

Entonces, es posible recuperar el espacio de relleno usando herramientas como FTK Imager. Ten en cuenta que este tipo de herramienta puede guardar el contenido ofuscado o incluso cifrado.

## UsbKill

Esta es una herramienta que **apagará la computadora si se detecta algún cambio en los puertos USB**.\
Una forma de descubrir esto sería inspeccionar los procesos en ejecución y **revisar cada script de python en ejecución**.

## Distribuciones de Linux en Vivo

Estas distribuciones se **ejecutan dentro de la memoria RAM**. La única forma de detectarlas es **en caso de que el sistema de archivos NTFS esté montado con permisos de escritura**. Si está montado solo con permisos de lectura, no será posible detectar la intrusión.

## Eliminación Segura

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

## Configuración de Windows

Es posible deshabilitar varios métodos de registro de Windows para hacer que la investigación forense sea mucho más difícil.

### Deshabilitar Marcas de Tiempo - UserAssist

Esta es una clave de registro que mantiene las fechas y horas en que cada ejecutable fue ejecutado por el usuario.

Deshabilitar UserAssist requiere dos pasos:

1. Establecer dos claves de registro, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` y `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`, ambas a cero para señalar que queremos deshabilitar UserAssist.
2. Limpiar tus subárboles de registro que se parecen a `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>`.

### Deshabilitar Marcas de Tiempo - Prefetch

Esto guardará información sobre las aplicaciones ejecutadas con el objetivo de mejorar el rendimiento del sistema Windows. Sin embargo, esto también puede ser útil para prácticas forenses.

* Ejecutar `regedit`
* Seleccionar la ruta del archivo `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Hacer clic derecho en `EnablePrefetcher` y `EnableSuperfetch`
* Seleccionar Modificar en cada uno de estos para cambiar el valor de 1 (o 3) a 0
* Reiniciar

### Deshabilitar Marcas de Tiempo - Última Hora de Acceso

Cada vez que se abre una carpeta desde un volumen NTFS en un servidor Windows NT, el sistema toma el tiempo para **actualizar un campo de marca de tiempo en cada carpeta listada**, llamado la última hora de acceso. En un volumen NTFS muy utilizado, esto puede afectar el rendimiento.

1. Abre el Editor del Registro (Regedit.exe).
2. Navega a `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`.
3. Busca `NtfsDisableLastAccessUpdate`. Si no existe, agrega este DWORD y establece su valor en 1, lo que deshabilitará el proceso.
4. Cierra el Editor del Registro y reinicia el servidor.

### Eliminar Historial de USB

Todas las **Entradas de Dispositivos USB** se almacenan en el Registro de Windows bajo la clave de registro **USBSTOR** que contiene subclaves que se crean cada vez que conectas un dispositivo USB a tu PC o Laptop. Puedes encontrar esta clave aquí `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Eliminando esto** eliminarás el historial de USB.\
También puedes usar la herramienta [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html) para asegurarte de que los has eliminado (y para eliminarlos).

Otro archivo que guarda información sobre los USB es el archivo `setupapi.dev.log` dentro de `C:\Windows\INF`. Este también debe ser eliminado.

### Deshabilitar Copias de Sombra

**Lista** las copias de sombra con `vssadmin list shadowstorage`\
**Elimínalas** ejecutando `vssadmin delete shadow`

También puedes eliminarlas a través de la GUI siguiendo los pasos propuestos en [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html)

Para deshabilitar las copias de sombra [pasos desde aquí](https://support.waters.com/KB\_Inf/Other/WKB15560\_How\_to\_disable\_Volume\_Shadow\_Copy\_Service\_VSS\_in\_Windows):

1. Abre el programa de Servicios escribiendo "services" en el cuadro de búsqueda de texto después de hacer clic en el botón de inicio de Windows.
2. En la lista, encuentra "Copia de Sombra de Volumen", selecciónalo y luego accede a Propiedades haciendo clic derecho.
3. Elige Deshabilitado en el menú desplegable "Tipo de inicio" y luego confirma el cambio haciendo clic en Aplicar y Aceptar.

También es posible modificar la configuración de qué archivos se van a copiar en la copia de sombra en el registro `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot`

### Sobrescribir archivos eliminados

* Puedes usar una **herramienta de Windows**: `cipher /w:C` Esto indicará a cipher que elimine cualquier dato del espacio de disco no utilizado disponible dentro de la unidad C.
* También puedes usar herramientas como [**Eraser**](https://eraser.heidi.ie)

### Eliminar registros de eventos de Windows

* Windows + R --> eventvwr.msc --> Expandir "Registros de Windows" --> Hacer clic derecho en cada categoría y seleccionar "Borrar Registro"
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

### Deshabilitar registros de eventos de Windows

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* Dentro de la sección de servicios deshabilitar el servicio "Registro de Eventos de Windows"
* `WEvtUtil.exec clear-log` o `WEvtUtil.exe cl`

### Deshabilitar $UsnJrnl

* `fsutil usn deletejournal /d c:`

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
