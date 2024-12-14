# JTAG

{% hint style="success" %}
Aprende y practica Hacking en AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprende y practica Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoya a HackTricks</summary>

* Revisa los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos en** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparte trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>
{% endhint %}

## JTAGenum

[**JTAGenum** ](https://github.com/cyphunk/JTAGenum)es una herramienta que se puede usar con un Raspberry PI o un Arduino para intentar encontrar pines JTAG de un chip desconocido.\
En el **Arduino**, conecta los **pines del 2 al 11 a 10 pines que potencialmente pertenecen a un JTAG**. Carga el programa en el Arduino y intentará hacer un bruteforce a todos los pines para encontrar si alguno pertenece a JTAG y cuál es cada uno.\
En el **Raspberry PI** solo puedes usar **pines del 1 al 6** (6 pines, por lo que irás más lento probando cada pin potencial de JTAG).

### Arduino

En Arduino, después de conectar los cables (pin 2 a 11 a los pines JTAG y GND de Arduino a GND de la placa base), **carga el programa JTAGenum en Arduino** y en el Monitor Serial envía un **`h`** (comando para ayuda) y deberías ver la ayuda:

![](<../../.gitbook/assets/image (939).png>)

![](<../../.gitbook/assets/image (578).png>)

Configura **"Sin final de línea" y 115200baud**.\
Envía el comando s para comenzar a escanear:

![](<../../.gitbook/assets/image (774).png>)

Si estás contactando un JTAG, encontrarás una o varias **líneas que comienzan con FOUND!** indicando los pines de JTAG.

{% hint style="success" %}
Aprende y practica Hacking en AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprende y practica Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoya a HackTricks</summary>

* Revisa los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos en** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparte trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>
{% endhint %}
