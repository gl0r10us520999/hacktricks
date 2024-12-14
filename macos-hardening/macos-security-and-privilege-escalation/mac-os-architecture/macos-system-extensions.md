# macOS System Extensions

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

## System Extensions / Endpoint Security Framework

A diferencia de las extensiones del kernel, **las extensiones del sistema se ejecutan en el espacio de usuario** en lugar del espacio del kernel, lo que reduce el riesgo de un fallo del sistema debido a un mal funcionamiento de la extensión.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Hay tres tipos de extensiones del sistema: **Extensiones DriverKit**, **Extensiones de Red** y **Extensiones de Seguridad de Endpoint**.

### **Extensiones DriverKit**

DriverKit es un reemplazo para las extensiones del kernel que **proporcionan soporte de hardware**. Permite que los controladores de dispositivos (como USB, Serial, NIC y controladores HID) se ejecuten en el espacio de usuario en lugar del espacio del kernel. El marco DriverKit incluye **versiones de espacio de usuario de ciertas clases de I/O Kit**, y el kernel reenvía eventos normales de I/O Kit al espacio de usuario, ofreciendo un entorno más seguro para que estos controladores se ejecuten.

### **Extensiones de Red**

Las extensiones de red proporcionan la capacidad de personalizar comportamientos de red. Hay varios tipos de extensiones de red:

* **Proxy de Aplicación**: Se utiliza para crear un cliente VPN que implementa un protocolo VPN personalizado orientado a flujos. Esto significa que maneja el tráfico de red basado en conexiones (o flujos) en lugar de paquetes individuales.
* **Túnel de Paquetes**: Se utiliza para crear un cliente VPN que implementa un protocolo VPN personalizado orientado a paquetes. Esto significa que maneja el tráfico de red basado en paquetes individuales.
* **Filtrar Datos**: Se utiliza para filtrar "flujos" de red. Puede monitorear o modificar datos de red a nivel de flujo.
* **Filtrar Paquete**: Se utiliza para filtrar paquetes individuales de red. Puede monitorear o modificar datos de red a nivel de paquete.
* **Proxy DNS**: Se utiliza para crear un proveedor DNS personalizado. Puede usarse para monitorear o modificar solicitudes y respuestas DNS.

## Endpoint Security Framework

La Seguridad de Endpoint es un marco proporcionado por Apple en macOS que ofrece un conjunto de APIs para la seguridad del sistema. Está destinado a ser utilizado por **proveedores de seguridad y desarrolladores para construir productos que puedan monitorear y controlar la actividad del sistema** para identificar y protegerse contra actividades maliciosas.

Este marco proporciona una **colección de APIs para monitorear y controlar la actividad del sistema**, como ejecuciones de procesos, eventos del sistema de archivos, eventos de red y del kernel.

El núcleo de este marco se implementa en el kernel, como una Extensión del Kernel (KEXT) ubicada en **`/System/Library/Extensions/EndpointSecurity.kext`**. Este KEXT está compuesto por varios componentes clave:

* **EndpointSecurityDriver**: Actúa como el "punto de entrada" para la extensión del kernel. Es el principal punto de interacción entre el sistema operativo y el marco de Seguridad de Endpoint.
* **EndpointSecurityEventManager**: Este componente es responsable de implementar ganchos del kernel. Los ganchos del kernel permiten que el marco monitoree eventos del sistema interceptando llamadas al sistema.
* **EndpointSecurityClientManager**: Este gestiona la comunicación con los clientes del espacio de usuario, manteniendo un registro de qué clientes están conectados y necesitan recibir notificaciones de eventos.
* **EndpointSecurityMessageManager**: Este envía mensajes y notificaciones de eventos a los clientes del espacio de usuario.

Los eventos que el marco de Seguridad de Endpoint puede monitorear se clasifican en:

* Eventos de archivos
* Eventos de procesos
* Eventos de sockets
* Eventos del kernel (como cargar/descargar una extensión del kernel o abrir un dispositivo de I/O Kit)

### Arquitectura del Marco de Seguridad de Endpoint

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**La comunicación en el espacio de usuario** con el marco de Seguridad de Endpoint ocurre a través de la clase IOUserClient. Se utilizan dos subclases diferentes, dependiendo del tipo de llamador:

* **EndpointSecurityDriverClient**: Esto requiere el derecho `com.apple.private.endpoint-security.manager`, que solo posee el proceso del sistema `endpointsecurityd`.
* **EndpointSecurityExternalClient**: Esto requiere el derecho `com.apple.developer.endpoint-security.client`. Esto se utilizaría típicamente por software de seguridad de terceros que necesita interactuar con el marco de Seguridad de Endpoint.

Las Extensiones de Seguridad de Endpoint:**`libEndpointSecurity.dylib`** es la biblioteca C que utilizan las extensiones del sistema para comunicarse con el kernel. Esta biblioteca utiliza el I/O Kit (`IOKit`) para comunicarse con el KEXT de Seguridad de Endpoint.

**`endpointsecurityd`** es un daemon del sistema clave involucrado en la gestión y lanzamiento de extensiones del sistema de seguridad de endpoint, particularmente durante el proceso de arranque temprano. **Solo las extensiones del sistema** marcadas con **`NSEndpointSecurityEarlyBoot`** en su archivo `Info.plist` reciben este tratamiento de arranque temprano.

Otro daemon del sistema, **`sysextd`**, **valida las extensiones del sistema** y las mueve a las ubicaciones adecuadas del sistema. Luego pide al daemon relevante que cargue la extensión. El **`SystemExtensions.framework`** es responsable de activar y desactivar las extensiones del sistema.

## Bypassing ESF

ESF es utilizado por herramientas de seguridad que intentarán detectar a un red teamer, por lo que cualquier información sobre cómo evitar esto suena interesante.

### CVE-2021-30965

El problema es que la aplicación de seguridad necesita tener **permisos de Acceso Completo al Disco**. Así que si un atacante pudiera eliminar eso, podría evitar que el software se ejecute:
```bash
tccutil reset All
```
Para **más información** sobre este bypass y otros relacionados, consulta la charla [#OBTS v5.0: "El talón de Aquiles de EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

Al final, esto se solucionó otorgando el nuevo permiso **`kTCCServiceEndpointSecurityClient`** a la aplicación de seguridad gestionada por **`tccd`**, de modo que `tccutil` no borrará sus permisos, impidiendo que deje de funcionar.

## Referencias

* [**OBTS v3.0: "Seguridad y falta de seguridad en Endpoint" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

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
