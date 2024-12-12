# Robando la Divulgación de Información Sensible de una Web

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

Si en algún momento encuentras una **página web que te presenta información sensible basada en tu sesión**: Tal vez esté reflejando cookies, o imprimiendo detalles de tarjetas de crédito o cualquier otra información sensible, puedes intentar robarla.\
Aquí te presento las principales formas en que puedes intentar lograrlo:

* [**CORS bypass**](../pentesting-web/cors-bypass.md): Si puedes eludir los encabezados CORS, podrás robar la información realizando una solicitud Ajax para una página maliciosa.
* [**XSS**](../pentesting-web/xss-cross-site-scripting/): Si encuentras una vulnerabilidad XSS en la página, es posible que puedas abusar de ella para robar la información.
* [**Danging Markup**](../pentesting-web/dangling-markup-html-scriptless-injection/): Si no puedes inyectar etiquetas XSS, aún puedes robar la información utilizando otras etiquetas HTML regulares.
* [**Clickjaking**](../pentesting-web/clickjacking.md): Si no hay protección contra este ataque, es posible que puedas engañar al usuario para que te envíe los datos sensibles (un ejemplo [aquí](https://medium.com/bugbountywriteup/apache-example-servlet-leads-to-61a2720cac20)).

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
