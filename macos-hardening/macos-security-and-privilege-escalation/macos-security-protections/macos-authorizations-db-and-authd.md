# macOS Authorizations DB & Authd



{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## **Athorizarions DB**

Hifadhidata iliyoko katika `/var/db/auth.db` ni hifadhidata inayotumika kuhifadhi ruhusa za kufanya operesheni nyeti. Operesheni hizi zinafanywa kabisa katika **nafasi ya mtumiaji** na kwa kawaida hutumiwa na **huduma za XPC** ambazo zinahitaji kuangalia **kama mteja anayepiga simu ameidhinishwa** kufanya kitendo fulani kwa kuangalia hifadhidata hii.

Kwanza, hifadhidata hii inaundwa kutoka kwa maudhui ya `/System/Library/Security/authorization.plist`. Kisha, huduma zingine zinaweza kuongeza au kubadilisha hifadhidata hii ili kuongeza ruhusa nyingine.

Sheria zinaifadhiwa katika jedwali la `rules` ndani ya hifadhidata na zina vitu vifuatavyo:

* **id**: Kitambulisho cha kipekee kwa kila sheria, kinachoongezeka kiotomatiki na kutumikia kama funguo kuu.
* **name**: Jina la kipekee la sheria linalotumika kutambua na kurejelea ndani ya mfumo wa idhini.
* **type**: Inaeleza aina ya sheria, iliyozuiliwa kwa thamani 1 au 2 ili kufafanua mantiki yake ya idhini.
* **class**: Inagawanya sheria katika darasa maalum, kuhakikisha ni nambari chanya.
* "allow" kwa ruhusu, "deny" kwa kataa, "user" ikiwa mali ya kundi inaonyesha kundi ambalo uanachama wake unaruhusu ufikiaji, "rule" inaonyesha katika orodha sheria inayopaswa kutimizwa, "evaluate-mechanisms" ikifuatwa na orodha ya `mechanisms` ambazo ni ama za ndani au jina la kifurushi ndani ya `/System/Library/CoreServices/SecurityAgentPlugins/` au /Library/Security//SecurityAgentPlugins
* **group**: Inaonyesha kundi la mtumiaji linalohusishwa na sheria kwa ajili ya idhini ya msingi ya kundi.
* **kofn**: Inawakilisha parameter ya "k-of-n", ikiamua ni subrules ngapi zinapaswa kutimizwa kutoka kwa jumla.
* **timeout**: Inaeleza muda katika sekunde kabla ya idhini iliyotolewa na sheria kuisha.
* **flags**: Inashikilia bendera mbalimbali zinazobadilisha tabia na sifa za sheria.
* **tries**: Inapunguza idadi ya majaribio ya idhini yanayoruhusiwa ili kuongeza usalama.
* **version**: Inafuatilia toleo la sheria kwa ajili ya udhibiti wa toleo na masasisho.
* **created**: Inarekodi muda wa kuunda sheria kwa ajili ya madhumuni ya ukaguzi.
* **modified**: Inahifadhi muda wa mabadiliko ya mwisho yaliyofanywa kwa sheria.
* **hash**: Inashikilia thamani ya hash ya sheria ili kuhakikisha uaminifu wake na kugundua udanganyifu.
* **identifier**: Inatoa kitambulisho cha kipekee cha mfuatano, kama UUID, kwa marejeleo ya nje kwa sheria.
* **requirement**: Inashikilia data iliyosimbwa ikielezea mahitaji maalum ya idhini ya sheria na mitambo.
* **comment**: Inatoa maelezo yanayoweza kusomeka na binadamu au maoni kuhusu sheria kwa ajili ya nyaraka na uwazi.

### Example
```bash
# List by name and comments
sudo sqlite3 /var/db/auth.db "select name, comment from rules"

# Get rules for com.apple.tcc.util.admin
security authorizationdb read com.apple.tcc.util.admin
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>class</key>
<string>rule</string>
<key>comment</key>
<string>For modification of TCC settings.</string>
<key>created</key>
<real>701369782.01043606</real>
<key>modified</key>
<real>701369782.01043606</real>
<key>rule</key>
<array>
<string>authenticate-admin-nonshared</string>
</array>
<key>version</key>
<integer>0</integer>
</dict>
</plist>
```
Zaidi ya hayo katika [https://www.dssw.co.uk/reference/authorization-rights/authenticate-admin-nonshared/](https://www.dssw.co.uk/reference/authorization-rights/authenticate-admin-nonshared/) inawezekana kuona maana ya `authenticate-admin-nonshared`:
```json
{
'allow-root' : 'false',
'authenticate-user' : 'true',
'class' : 'user',
'comment' : 'Authenticate as an administrator.',
'group' : 'admin',
'session-owner' : 'false',
'shared' : 'false',
'timeout' : '30',
'tries' : '10000',
'version' : '1'
}
```
## Authd

Ni deamon ambayo itapokea maombi ya kuidhinisha wateja kufanya vitendo nyeti. Inafanya kazi kama huduma ya XPC iliyoainishwa ndani ya folda ya `XPCServices/` na hutumia kuandika kumbukumbu zake katika `/var/log/authd.log`.

Zaidi ya hayo, kwa kutumia zana ya usalama inawezekana kujaribu APIs nyingi za `Security.framework`. Kwa mfano, `AuthorizationExecuteWithPrivileges` inayoendesha: `security execute-with-privileges /bin/ls`

Hiyo itafork na exec `/usr/libexec/security_authtrampoline /bin/ls` kama root, ambayo itauliza ruhusa katika dirisha la kuingia ili kutekeleza ls kama root:

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
