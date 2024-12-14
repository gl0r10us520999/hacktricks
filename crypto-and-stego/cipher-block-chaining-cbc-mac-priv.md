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


# CBC

만약 **쿠키**가 **오직** **사용자 이름** (또는 쿠키의 첫 부분이 사용자 이름)이고, 당신이 사용자 이름 "**admin**"으로 가장하고 싶다면, 사용자 이름 **"bdmin"**을 만들고 **첫 바이트**의 쿠키를 **브루트포스**할 수 있습니다.

# CBC-MAC

**암호 블록 체인 메시지 인증 코드** (**CBC-MAC**)는 암호학에서 사용되는 방법입니다. 이 방법은 메시지를 블록 단위로 암호화하며, 각 블록의 암호화는 이전 블록과 연결되어 작동합니다. 이 과정은 **블록의 체인**을 생성하여 원래 메시지의 단일 비트라도 변경하면 암호화된 데이터의 마지막 블록에서 예측할 수 없는 변화를 초래합니다. 이러한 변화를 만들거나 되돌리기 위해서는 암호화 키가 필요하여 보안을 보장합니다.

메시지 m의 CBC-MAC을 계산하기 위해, m을 제로 초기화 벡터로 CBC 모드에서 암호화하고 마지막 블록을 유지합니다. 다음 그림은 비밀 키 k와 블록 암호 E를 사용하여 블록으로 구성된 메시지의 CBC-MAC 계산을 간략하게 나타냅니다![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5):

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Vulnerability

CBC-MAC에서는 일반적으로 **사용되는 IV가 0**입니다.\
이것은 문제입니다. 왜냐하면 2개의 알려진 메시지 (`m1` 및 `m2`)가 독립적으로 2개의 서명 (`s1` 및 `s2`)을 생성하기 때문입니다. 그래서:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

그런 다음 m1과 m2가 연결된 메시지(m3)는 2개의 서명(s31 및 s32)을 생성합니다:

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**이는 암호화 키를 알지 않고도 계산할 수 있습니다.**

당신이 **Administrator**라는 이름을 **8바이트** 블록으로 암호화하고 있다고 상상해 보십시오:

* `Administ`
* `rator\00\00\00`

당신은 **Administ**라는 사용자 이름(m1)을 만들고 서명(s1)을 가져올 수 있습니다.\
그런 다음 `rator\00\00\00 XOR s1`의 결과로 사용자 이름을 만들 수 있습니다. 이것은 `E(m2 XOR s1 XOR 0)`을 생성하여 s32가 됩니다.\
이제 s32를 전체 이름 **Administrator**의 서명으로 사용할 수 있습니다.

### Summary

1. 사용자 이름 **Administ** (m1)의 서명을 가져옵니다. 이는 s1입니다.
2. 사용자 이름 **rator\x00\x00\x00 XOR s1 XOR 0**의 서명을 가져옵니다. 이는 s32입니다.
3. 쿠키를 s32로 설정하면 **Administrator** 사용자에 대한 유효한 쿠키가 됩니다.

# Attack Controlling IV

사용된 IV를 제어할 수 있다면 공격이 매우 쉬울 수 있습니다.\
쿠키가 단순히 암호화된 사용자 이름이라면, 사용자 "**administrator**"로 가장하기 위해 "**Administrator**"라는 사용자를 만들 수 있으며, 그 쿠키를 얻을 수 있습니다.\
이제 IV를 제어할 수 있다면, IV의 첫 번째 바이트를 변경할 수 있습니다. 그래서 **IV\[0] XOR "A" == IV'\[0] XOR "a"**가 되고, 사용자 **Administrator**의 쿠키를 재생성할 수 있습니다. 이 쿠키는 초기 **IV**로 **administrator** 사용자를 **가장하는** 데 유효합니다.

## References

자세한 정보는 [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)에서 확인하세요.


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
