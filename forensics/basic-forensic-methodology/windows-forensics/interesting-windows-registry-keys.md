# Claves del Registro de Windows de Interés

### Claves del Registro de Windows de Interés

{% hint style="success" %}
Aprende y practica Hacking en AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprende y practica Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoya a HackTricks</summary>

* Revisa los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparte trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}

### **Información de la Versión de Windows y del Propietario**
- Encontrarás la versión de Windows, Service Pack, hora de instalación y el nombre del propietario registrado de manera directa en **`Software\Microsoft\Windows NT\CurrentVersion`**.

### **Nombre del Equipo**
- El nombre del host se encuentra en **`System\ControlSet001\Control\ComputerName\ComputerName`**.

### **Configuración de Zona Horaria**
- La zona horaria del sistema se almacena en **`System\ControlSet001\Control\TimeZoneInformation`**.

### **Seguimiento de Tiempo de Acceso**
- Por defecto, el seguimiento del último tiempo de acceso está desactivado (**`NtfsDisableLastAccessUpdate=1`**). Para activarlo, usa:
`fsutil behavior set disablelastaccess 0`

### Versiones de Windows y Service Packs
- La **versión de Windows** indica la edición (por ejemplo, Home, Pro) y su lanzamiento (por ejemplo, Windows 10, Windows 11), mientras que los **Service Packs** son actualizaciones que incluyen correcciones y, a veces, nuevas características.

### Habilitar el Último Tiempo de Acceso
- Habilitar el seguimiento del último tiempo de acceso te permite ver cuándo se abrieron los archivos por última vez, lo cual puede ser crucial para análisis forense o monitoreo del sistema.

### Detalles de Información de Red
- El registro contiene datos extensos sobre configuraciones de red, incluyendo **tipos de redes (inalámbrica, cableada, 3G)** y **categorías de red (Pública, Privada/Hogar, Dominio/Trabajo)**, que son vitales para comprender la configuración de seguridad de la red y los permisos.

### Caché del Lado del Cliente (CSC)
- **CSC** mejora el acceso a archivos sin conexión al almacenar copias de archivos compartidos. Diferentes configuraciones de **CSCFlags** controlan cómo y qué archivos se almacenan en caché, afectando el rendimiento y la experiencia del usuario, especialmente en entornos con conectividad intermitente.

### Programas de Inicio Automático
- Los programas listados en varias claves del registro `Run` y `RunOnce` se inician automáticamente al arrancar, afectando el tiempo de arranque del sistema y pudiendo ser puntos de interés para identificar malware o software no deseado.

### Shellbags
- Las **Shellbags** no solo almacenan preferencias para vistas de carpetas, sino que también proporcionan evidencia forense de acceso a carpetas incluso si la carpeta ya no existe. Son invaluables para investigaciones, revelando la actividad del usuario que no es evidente de otras maneras.

### Información y Forense de Dispositivos USB
- Los detalles almacenados en el registro sobre dispositivos USB pueden ayudar a rastrear qué dispositivos se conectaron a una computadora, vinculando potencialmente un dispositivo a transferencias de archivos sensibles o incidentes de acceso no autorizado.

### Número de Serie del Volumen
- El **Número de Serie del Volumen** puede ser crucial para rastrear la instancia específica de un sistema de archivos, útil en escenarios forenses donde se necesita establecer el origen de un archivo en diferentes dispositivos.

### **Detalles de Apagado**
- La hora y el recuento de apagados (este último solo para XP) se guardan en **`System\ControlSet001\Control\Windows`** y **`System\ControlSet001\Control\Watchdog\Display`**.

### **Configuración de Red**
- Para información detallada de la interfaz de red, consulta **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**.
- Los tiempos de primera y última conexión a la red, incluidas las conexiones VPN, se registran en varias rutas en **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`**.

### **Carpetas Compartidas**
- Las carpetas compartidas y configuraciones se encuentran en **`System\ControlSet001\Services\lanmanserver\Shares`**. Las configuraciones de Caché del Lado del Cliente (CSC) dictan la disponibilidad de archivos sin conexión.

### **Programas que Inician Automáticamente**
- Rutas como **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** y entradas similares en `Software\Microsoft\Windows\CurrentVersion` detallan programas configurados para ejecutarse al inicio.

### **Búsquedas y Rutas Escritas**
- Las búsquedas y rutas escritas en el Explorador se rastrean en el registro bajo **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** para WordwheelQuery y TypedPaths, respectivamente.

### **Documentos Recientes y Archivos de Office**
- Los documentos recientes y archivos de Office accedidos se registran en `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` y rutas específicas de versiones de Office.

### **Elementos Más Recientemente Utilizados (MRU)**
- Las listas MRU, que indican rutas y comandos de archivos recientes, se almacenan en varias subclaves de `ComDlg32` y `Explorer` bajo `NTUSER.DAT`.

### **Seguimiento de Actividad del Usuario**
- La función User Assist registra estadísticas detalladas de uso de aplicaciones, incluido el recuento de ejecuciones y la última vez ejecutada, en **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**.

### **Análisis de Shellbags**
- Las Shellbags, que revelan detalles de acceso a carpetas, se almacenan en `USRCLASS.DAT` y `NTUSER.DAT` bajo `Software\Microsoft\Windows\Shell`. Utiliza **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** para el análisis.

### **Historial de Dispositivos USB**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** y **`HKLM\SYSTEM\ControlSet001\Enum\USB`** contienen detalles completos sobre dispositivos USB conectados, incluyendo fabricante, nombre del producto y marcas de tiempo de conexión.
- El usuario asociado con un dispositivo USB específico se puede identificar buscando en las colmenas de `NTUSER.DAT` el **{GUID}** del dispositivo.
- El último dispositivo montado y su número de serie de volumen se pueden rastrear a través de `System\MountedDevices` y `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt`, respectivamente.

Esta guía condensa las rutas y métodos cruciales para acceder a información detallada del sistema, red y actividad del usuario en sistemas Windows, con el objetivo de claridad y usabilidad.
