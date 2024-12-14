# macOS MDM

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

**Para aprender sobre MDMs de macOS, consulta:**

* [https://www.youtube.com/watch?v=ku8jZe-MHUU](https://www.youtube.com/watch?v=ku8jZe-MHUU)
* [https://duo.com/labs/research/mdm-me-maybe](https://duo.com/labs/research/mdm-me-maybe)

## Basics

### **Descripción general de MDM (Mobile Device Management)**

[Mobile Device Management](https://en.wikipedia.org/wiki/Mobile\_device\_management) (MDM) se utiliza para supervisar varios dispositivos de usuario final como teléfonos inteligentes, laptops y tabletas. Particularmente para las plataformas de Apple (iOS, macOS, tvOS), implica un conjunto de características, API y prácticas especializadas. El funcionamiento de MDM depende de un servidor MDM compatible, que puede ser comercial o de código abierto, y debe soportar el [Protocolo MDM](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Los puntos clave incluyen:

* Control centralizado sobre los dispositivos.
* Dependencia de un servidor MDM que cumpla con el protocolo MDM.
* Capacidad del servidor MDM para enviar varios comandos a los dispositivos, por ejemplo, borrado de datos de forma remota o instalación de configuraciones.

### **Conceptos básicos de DEP (Device Enrollment Program)**

El [Device Enrollment Program](https://www.apple.com/business/site/docs/DEP\_Guide.pdf) (DEP) ofrecido por Apple simplifica la integración de Mobile Device Management (MDM) al facilitar la configuración sin contacto para dispositivos iOS, macOS y tvOS. DEP automatiza el proceso de inscripción, permitiendo que los dispositivos estén operativos desde el primer momento, con mínima intervención del usuario o del administrador. Los aspectos esenciales incluyen:

* Permite que los dispositivos se registren de forma autónoma con un servidor MDM predefinido al activarse por primera vez.
* Principalmente beneficioso para dispositivos nuevos, pero también aplicable a dispositivos que están siendo reconfigurados.
* Facilita una configuración sencilla, haciendo que los dispositivos estén listos para su uso organizacional rápidamente.

### **Consideración de seguridad**

Es crucial notar que la facilidad de inscripción proporcionada por DEP, aunque beneficiosa, también puede presentar riesgos de seguridad. Si las medidas de protección no se aplican adecuadamente para la inscripción MDM, los atacantes podrían explotar este proceso simplificado para registrar su dispositivo en el servidor MDM de la organización, haciéndose pasar por un dispositivo corporativo.

{% hint style="danger" %}
**Alerta de seguridad**: La inscripción simplificada de DEP podría permitir potencialmente el registro no autorizado de dispositivos en el servidor MDM de la organización si no se implementan las salvaguardias adecuadas.
{% endhint %}

### Conceptos básicos ¿Qué es SCEP (Simple Certificate Enrollment Protocol)?

* Un protocolo relativamente antiguo, creado antes de que TLS y HTTPS fueran comunes.
* Proporciona a los clientes una forma estandarizada de enviar una **Solicitud de Firma de Certificado** (CSR) con el propósito de obtener un certificado. El cliente pedirá al servidor que le proporcione un certificado firmado.

### ¿Qué son los Perfiles de Configuración (también conocidos como mobileconfigs)?

* La forma oficial de Apple de **configurar/implementar la configuración del sistema.**
* Formato de archivo que puede contener múltiples cargas útiles.
* Basado en listas de propiedades (el tipo XML).
* “pueden ser firmados y cifrados para validar su origen, asegurar su integridad y proteger su contenido.” Conceptos básicos — Página 70, Guía de Seguridad de iOS, enero de 2018.

## Protocolos

### MDM

* Combinación de APNs (**servidores de Apple**) + API RESTful (**servidores de proveedores de MDM**)
* **La comunicación** ocurre entre un **dispositivo** y un servidor asociado con un **producto de gestión de dispositivos**
* **Los comandos** se entregan desde el MDM al dispositivo en **diccionarios codificados en plist**
* A través de **HTTPS**. Los servidores MDM pueden ser (y generalmente son) fijados.
* Apple otorga al proveedor de MDM un **certificado APNs** para autenticación

### DEP

* **3 APIs**: 1 para revendedores, 1 para proveedores de MDM, 1 para identidad de dispositivo (no documentada):
* La llamada [API de "servicio en la nube" de DEP](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Esta es utilizada por los servidores MDM para asociar perfiles DEP con dispositivos específicos.
* La [API de DEP utilizada por los Revendedores Autorizados de Apple](https://applecareconnect.apple.com/api-docs/depuat/html/WSImpManual.html) para inscribir dispositivos, verificar el estado de inscripción y verificar el estado de transacción.
* La API privada de DEP no documentada. Esta es utilizada por los Dispositivos Apple para solicitar su perfil DEP. En macOS, el binario `cloudconfigurationd` es responsable de comunicarse a través de esta API.
* Más moderna y basada en **JSON** (vs. plist)
* Apple otorga un **token OAuth** al proveedor de MDM

**API de "servicio en la nube" de DEP**

* RESTful
* sincroniza registros de dispositivos de Apple al servidor MDM
* sincroniza “perfiles DEP” a Apple desde el servidor MDM (entregados por Apple al dispositivo más tarde)
* Un “perfil” de DEP contiene:
* URL del servidor del proveedor de MDM
* Certificados adicionales de confianza para la URL del servidor (fijación opcional)
* Configuraciones adicionales (por ejemplo, qué pantallas omitir en el Asistente de Configuración)

## Número de serie

Los dispositivos de Apple fabricados después de 2010 generalmente tienen números de serie alfanuméricos de **12 caracteres**, con los **primeros tres dígitos representando la ubicación de fabricación**, los siguientes **dos** indicando el **año** y **semana** de fabricación, los siguientes **tres** dígitos proporcionando un **identificador único**, y los **últimos** **cuatro** dígitos representando el **número de modelo**.

{% content-ref url="macos-serial-number.md" %}
[macos-serial-number.md](macos-serial-number.md)
{% endcontent-ref %}

## Pasos para la inscripción y gestión

1. Creación del registro del dispositivo (Revendedor, Apple): Se crea el registro para el nuevo dispositivo
2. Asignación del registro del dispositivo (Cliente): El dispositivo se asigna a un servidor MDM
3. Sincronización del registro del dispositivo (Proveedor de MDM): MDM sincroniza los registros de dispositivos y envía los perfiles DEP a Apple
4. Registro en DEP (Dispositivo): El dispositivo obtiene su perfil DEP
5. Recuperación del perfil (Dispositivo)
6. Instalación del perfil (Dispositivo) a. incl. cargas útiles de MDM, SCEP y CA raíz
7. Emisión de comandos MDM (Dispositivo)

![](<../../../.gitbook/assets/image (694).png>)

El archivo `/Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/System/Library/PrivateFrameworks/ConfigurationProfiles.framework/ConfigurationProfiles.tbd` exporta funciones que pueden considerarse **"pasos" de alto nivel** del proceso de inscripción.

### Paso 4: Registro en DEP - Obtención del Registro de Activación

Esta parte del proceso ocurre cuando un **usuario inicia un Mac por primera vez** (o después de un borrado completo)

![](<../../../.gitbook/assets/image (1044).png>)

o al ejecutar `sudo profiles show -type enrollment`

* Determinar **si el dispositivo está habilitado para DEP**
* El Registro de Activación es el nombre interno para el **"perfil" DEP**
* Comienza tan pronto como el dispositivo está conectado a Internet
* Impulsado por **`CPFetchActivationRecord`**
* Implementado por **`cloudconfigurationd`** a través de XPC. El **"Asistente de Configuración"** (cuando el dispositivo se inicia por primera vez) o el comando **`profiles`** **contactarán a este daemon** para recuperar el registro de activación.
* LaunchDaemon (siempre se ejecuta como root)

Sigue unos pasos para obtener el Registro de Activación realizado por **`MCTeslaConfigurationFetcher`**. Este proceso utiliza un cifrado llamado **Absinthe**

1. Recuperar **certificado**
1. GET [https://iprofiles.apple.com/resource/certificate.cer](https://iprofiles.apple.com/resource/certificate.cer)
2. **Inicializar** estado desde el certificado (**`NACInit`**)
1. Utiliza varios datos específicos del dispositivo (es decir, **Número de serie a través de `IOKit`**)
3. Recuperar **clave de sesión**
1. POST [https://iprofiles.apple.com/session](https://iprofiles.apple.com/session)
4. Establecer la sesión (**`NACKeyEstablishment`**)
5. Hacer la solicitud
1. POST a [https://iprofiles.apple.com/macProfile](https://iprofiles.apple.com/macProfile) enviando los datos `{ "action": "RequestProfileConfiguration", "sn": "" }`
2. La carga útil JSON está cifrada usando Absinthe (**`NACSign`**)
3. Todas las solicitudes a través de HTTPs, se utilizan certificados raíz incorporados

![](<../../../.gitbook/assets/image (566) (1).png>)

La respuesta es un diccionario JSON con algunos datos importantes como:

* **url**: URL del host del proveedor de MDM para el perfil de activación
* **anchor-certs**: Array de certificados DER utilizados como anclas de confianza

### **Paso 5: Recuperación del perfil**

![](<../../../.gitbook/assets/image (444).png>)

* Solicitud enviada a **la URL proporcionada en el perfil DEP**.
* **Certificados de anclaje** se utilizan para **evaluar la confianza** si se proporcionan.
* Recordatorio: la propiedad **anchor\_certs** del perfil DEP
* **La solicitud es un simple .plist** con identificación del dispositivo
* Ejemplos: **UDID, versión de OS**.
* Firmado por CMS, codificado en DER
* Firmado usando el **certificado de identidad del dispositivo (de APNS)**
* **La cadena de certificados** incluye un **Apple iPhone Device CA** expirado

![](<../../../.gitbook/assets/image (567) (1) (2) (2) (2) (2) (2) (2) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (2).png>)

### Paso 6: Instalación del perfil

* Una vez recuperado, **el perfil se almacena en el sistema**
* Este paso comienza automáticamente (si está en **asistente de configuración**)
* Impulsado por **`CPInstallActivationProfile`**
* Implementado por mdmclient a través de XPC
* LaunchDaemon (como root) o LaunchAgent (como usuario), dependiendo del contexto
* Los perfiles de configuración tienen múltiples cargas útiles para instalar
* El marco tiene una arquitectura basada en plugins para instalar perfiles
* Cada tipo de carga útil está asociado con un plugin
* Puede ser XPC (en el marco) o Cocoa clásica (en ManagedClient.app)
* Ejemplo:
* Las cargas útiles de certificados utilizan CertificateService.xpc

Típicamente, el **perfil de activación** proporcionado por un proveedor de MDM incluirá **las siguientes cargas útiles**:

* `com.apple.mdm`: para **inscribir** el dispositivo en MDM
* `com.apple.security.scep`: para proporcionar de forma segura un **certificado de cliente** al dispositivo.
* `com.apple.security.pem`: para **instalar certificados CA de confianza** en el llavero del sistema del dispositivo.
* La instalación de la carga útil de MDM es equivalente a **MDM check-in en la documentación**
* La carga útil **contiene propiedades clave**:
*
* URL de Check-In de MDM (**`CheckInURL`**)
* URL de sondeo de comandos de MDM (**`ServerURL`**) + tema de APNs para activarlo
* Para instalar la carga útil de MDM, se envía una solicitud a **`CheckInURL`**
* Implementado en **`mdmclient`**
* La carga útil de MDM puede depender de otras cargas útiles
* Permite **solicitudes que se fijen a certificados específicos**:
* Propiedad: **`CheckInURLPinningCertificateUUIDs`**
* Propiedad: **`ServerURLPinningCertificateUUIDs`**
* Entregado a través de la carga útil PEM
* Permite que el dispositivo se le atribuya un certificado de identidad:
* Propiedad: IdentityCertificateUUID
* Entregado a través de la carga útil SCEP

### **Paso 7: Escuchando comandos MDM**

* Después de que se complete el registro MDM, el proveedor puede **emitir notificaciones push usando APNs**
* Al recibirlas, son manejadas por **`mdmclient`**
* Para sondear comandos MDM, se envía una solicitud a ServerURL
* Utiliza la carga útil de MDM previamente instalada:
* **`ServerURLPinningCertificateUUIDs`** para fijar la solicitud
* **`IdentityCertificateUUID`** para el certificado de cliente TLS

## Ataques

### Inscripción de Dispositivos en Otras Organizaciones

Como se comentó anteriormente, para intentar inscribir un dispositivo en una organización **solo se necesita un Número de Serie que pertenezca a esa Organización**. Una vez que el dispositivo está inscrito, varias organizaciones instalarán datos sensibles en el nuevo dispositivo: certificados, aplicaciones, contraseñas de WiFi, configuraciones de VPN [y así sucesivamente](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf).\
Por lo tanto, esto podría ser un punto de entrada peligroso para los atacantes si el proceso de inscripción no está correctamente protegido:

{% content-ref url="enrolling-devices-in-other-organisations.md" %}
[enrolling-devices-in-other-organisations.md](enrolling-devices-in-other-organisations.md)
{% endcontent-ref %}

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
