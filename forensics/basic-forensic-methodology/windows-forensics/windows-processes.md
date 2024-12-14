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


## smss.exe

**Session Manager**.\
La sesión 0 inicia **csrss.exe** y **wininit.exe** (**servicios** **del** **SO**) mientras que la sesión 1 inicia **csrss.exe** y **winlogon.exe** (**sesión** **de** **usuario**). Sin embargo, deberías ver **solo un proceso** de ese **binario** sin hijos en el árbol de procesos.

Además, sesiones distintas de 0 y 1 pueden significar que están ocurriendo sesiones RDP.


## csrss.exe

**Proceso del Subsistema de Ejecución Cliente/Servidor**.\
Gestiona **procesos** y **hilos**, hace que la **API** **de** **Windows** esté disponible para otros procesos y también **asigna letras de unidad**, crea **archivos temporales** y maneja el **proceso** de **apagado**.

Hay uno **ejecutándose en la Sesión 0 y otro en la Sesión 1** (así que **2 procesos** en el árbol de procesos). Se crea otro **por cada nueva Sesión**.


## winlogon.exe

**Proceso de Inicio de Sesión de Windows**.\
Es responsable de los **inicios**/**cierres** de **sesión** de usuario. Lanza **logonui.exe** para pedir el nombre de usuario y la contraseña y luego llama a **lsass.exe** para verificarlos.

Luego lanza **userinit.exe** que está especificado en **`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`** con la clave **Userinit**.

Además, el registro anterior debería tener **explorer.exe** en la clave **Shell** o podría ser abusado como un **método de persistencia de malware**.


## wininit.exe

**Proceso de Inicialización de Windows**. \
Lanza **services.exe**, **lsass.exe** y **lsm.exe** en la Sesión 0. Solo debería haber 1 proceso.


## userinit.exe

**Aplicación de Inicio de Sesión de Userinit**.\
Carga el **ntduser.dat en HKCU** e inicializa el **entorno** **del** **usuario** y ejecuta **scripts** de **inicio de sesión** y **GPO**.

Lanza **explorer.exe**.


## lsm.exe

**Administrador de Sesiones Locales**.\
Trabaja con smss.exe para manipular sesiones de usuario: Inicio/cierre de sesión, inicio de shell, bloqueo/desbloqueo del escritorio, etc.

Después de W7, lsm.exe se transformó en un servicio (lsm.dll).

Solo debería haber 1 proceso en W7 y de ellos un servicio ejecutando la DLL.


## services.exe

**Administrador de Control de Servicios**.\
**Carga** **servicios** configurados como **auto-inicio** y **controladores**.

Es el proceso padre de **svchost.exe**, **dllhost.exe**, **taskhost.exe**, **spoolsv.exe** y muchos más.

Los servicios están definidos en `HKLM\SYSTEM\CurrentControlSet\Services` y este proceso mantiene una base de datos en memoria de información de servicios que puede ser consultada por sc.exe.

Nota cómo **algunos** **servicios** van a estar ejecutándose en un **proceso propio** y otros van a estar **compartiendo un proceso svchost.exe**.

Solo debería haber 1 proceso.


## lsass.exe

**Subsistema de Autoridad de Seguridad Local**.\
Es responsable de la **autenticación** del usuario y de crear los **tokens** de **seguridad**. Utiliza paquetes de autenticación ubicados en `HKLM\System\CurrentControlSet\Control\Lsa`.

Escribe en el **registro** **de** **eventos** **de** **seguridad** y solo debería haber 1 proceso.

Ten en cuenta que este proceso es altamente atacado para volcar contraseñas.


## svchost.exe

**Proceso Genérico de Host de Servicios**.\
Aloja múltiples servicios DLL en un solo proceso compartido.

Por lo general, encontrarás que **svchost.exe** se lanza con la bandera `-k`. Esto lanzará una consulta al registro **HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost** donde habrá una clave con el argumento mencionado en -k que contendrá los servicios a lanzar en el mismo proceso.

Por ejemplo: `-k UnistackSvcGroup` lanzará: `PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

Si la **bandera `-s`** también se usa con un argumento, entonces se le pide a svchost que **solo lance el servicio especificado** en este argumento.

Habrá varios procesos de `svchost.exe`. Si alguno de ellos **no está usando la bandera `-k`**, entonces eso es muy sospechoso. Si encuentras que **services.exe no es el padre**, eso también es muy sospechoso.


## taskhost.exe

Este proceso actúa como un host para procesos que se ejecutan desde DLLs. También carga los servicios que se están ejecutando desde DLLs.

En W8 esto se llama taskhostex.exe y en W10 taskhostw.exe.


## explorer.exe

Este es el proceso responsable del **escritorio del usuario** y de lanzar archivos a través de extensiones de archivo.

**Solo 1** proceso debería ser generado **por cada usuario conectado.**

Esto se ejecuta desde **userinit.exe** que debería ser terminado, así que **no debería aparecer un padre** para este proceso.


# Capturando Procesos Maliciosos

* ¿Se está ejecutando desde la ruta esperada? (Ningún binario de Windows se ejecuta desde una ubicación temporal)
* ¿Está comunicándose con IPs extrañas?
* Verifica las firmas digitales (los artefactos de Microsoft deberían estar firmados)
* ¿Está escrito correctamente?
* ¿Se está ejecutando bajo el SID esperado?
* ¿Es el proceso padre el esperado (si lo hay)?
* ¿Son los procesos hijos los esperados? (¿sin cmd.exe, wscript.exe, powershell.exe..?)
