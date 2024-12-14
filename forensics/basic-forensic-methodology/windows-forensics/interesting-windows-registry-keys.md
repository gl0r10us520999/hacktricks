# Ciekawe klucze rejestru systemu Windows

### Ciekawe klucze rejestru systemu Windows

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się sztuczkami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}


### **Informacje o wersji systemu Windows i właścicielu**
- Znajdziesz informacje o wersji systemu Windows, pakiecie Service Pack, czasie instalacji oraz nazwie zarejestrowanego właściciela w **`Software\Microsoft\Windows NT\CurrentVersion`**.

### **Nazwa komputera**
- Nazwa hosta znajduje się w **`System\ControlSet001\Control\ComputerName\ComputerName`**.

### **Ustawienia strefy czasowej**
- Strefa czasowa systemu jest przechowywana w **`System\ControlSet001\Control\TimeZoneInformation`**.

### **Śledzenie czasu dostępu**
- Domyślnie śledzenie ostatniego czasu dostępu jest wyłączone (**`NtfsDisableLastAccessUpdate=1`**). Aby je włączyć, użyj:
`fsutil behavior set disablelastaccess 0`

### Wersje systemu Windows i pakiety Service Pack
- **Wersja systemu Windows** wskazuje edycję (np. Home, Pro) oraz jej wydanie (np. Windows 10, Windows 11), podczas gdy **pakiety Service Pack** to aktualizacje, które zawierają poprawki i czasami nowe funkcje.

### Włączanie śledzenia ostatniego czasu dostępu
- Włączenie śledzenia ostatniego czasu dostępu pozwala zobaczyć, kiedy pliki były ostatnio otwierane, co może być kluczowe dla analizy kryminalistycznej lub monitorowania systemu.

### Szczegóły informacji o sieci
- Rejestr zawiera obszerne dane na temat konfiguracji sieci, w tym **typy sieci (bezprzewodowe, kablowe, 3G)** oraz **kategorie sieci (Publiczna, Prywatna/Domowa, Domenowa/Praca)**, które są istotne dla zrozumienia ustawień bezpieczeństwa sieci i uprawnień.

### Klient Side Caching (CSC)
- **CSC** poprawia dostęp offline do plików, przechowując kopie udostępnionych plików. Różne ustawienia **CSCFlags** kontrolują, jak i jakie pliki są buforowane, co wpływa na wydajność i doświadczenia użytkownika, szczególnie w środowiskach z przerywaną łącznością.

### Programy uruchamiające się automatycznie
- Programy wymienione w różnych kluczach rejestru `Run` i `RunOnce` są automatycznie uruchamiane przy starcie, co wpływa na czas uruchamiania systemu i może być punktami zainteresowania przy identyfikacji złośliwego oprogramowania lub niechcianego oprogramowania.

### Shellbags
- **Shellbags** nie tylko przechowują preferencje dotyczące widoków folderów, ale także dostarczają dowodów kryminalistycznych dotyczących dostępu do folderów, nawet jeśli folder już nie istnieje. Są nieocenione w dochodzeniach, ujawniając aktywność użytkownika, która nie jest oczywista w inny sposób.

### Informacje o USB i kryminalistyka
- Szczegóły przechowywane w rejestrze dotyczące urządzeń USB mogą pomóc w ustaleniu, które urządzenia były podłączone do komputera, potencjalnie łącząc urządzenie z transferami wrażliwych plików lub incydentami nieautoryzowanego dostępu.

### Numer seryjny woluminu
- **Numer seryjny woluminu** może być kluczowy do śledzenia konkretnej instancji systemu plików, co jest przydatne w scenariuszach kryminalistycznych, gdzie należy ustalić pochodzenie pliku na różnych urządzeniach.

### **Szczegóły zamknięcia**
- Czas zamknięcia i liczba zamknięć (to drugie tylko dla XP) są przechowywane w **`System\ControlSet001\Control\Windows`** oraz **`System\ControlSet001\Control\Watchdog\Display`**.

### **Konfiguracja sieci**
- Aby uzyskać szczegółowe informacje o interfejsie sieciowym, odwołaj się do **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**.
- Czas pierwszego i ostatniego połączenia sieciowego, w tym połączeń VPN, jest rejestrowany w różnych ścieżkach w **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`**.

### **Foldery udostępnione**
- Foldery udostępnione i ustawienia znajdują się w **`System\ControlSet001\Services\lanmanserver\Shares`**. Ustawienia Client Side Caching (CSC) określają dostępność plików offline.

### **Programy, które uruchamiają się automatycznie**
- Ścieżki takie jak **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** i podobne wpisy w `Software\Microsoft\Windows\CurrentVersion` szczegółowo opisują programy ustawione do uruchamiania przy starcie.

### **Wyszukiwania i wpisane ścieżki**
- Wyszukiwania w Eksploratorze i wpisane ścieżki są śledzone w rejestrze pod **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** dla WordwheelQuery i TypedPaths, odpowiednio.

### **Ostatnie dokumenty i pliki Office**
- Ostatnie dokumenty i pliki Office, do których uzyskano dostęp, są notowane w `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` oraz w określonych ścieżkach wersji Office.

### **Najczęściej używane (MRU) elementy**
- Listy MRU, wskazujące na ostatnie ścieżki plików i polecenia, są przechowywane w różnych podkluczach `ComDlg32` i `Explorer` w `NTUSER.DAT`.

### **Śledzenie aktywności użytkownika**
- Funkcja User Assist rejestruje szczegółowe statystyki użycia aplikacji, w tym liczbę uruchomień i czas ostatniego uruchomienia, w **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**.

### **Analiza Shellbags**
- Shellbags, ujawniające szczegóły dostępu do folderów, są przechowywane w `USRCLASS.DAT` i `NTUSER.DAT` w `Software\Microsoft\Windows\Shell`. Użyj **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** do analizy.

### **Historia urządzeń USB**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** i **`HKLM\SYSTEM\ControlSet001\Enum\USB`** zawierają bogate szczegóły dotyczące podłączonych urządzeń USB, w tym producenta, nazwy produktu i znaczniki czasowe połączenia.
- Użytkownika powiązanego z konkretnym urządzeniem USB można zidentyfikować, przeszukując zbiory `NTUSER.DAT` w poszukiwaniu **{GUID}** urządzenia.
- Ostatnio zamontowane urządzenie i jego numer seryjny woluminu można śledzić przez `System\MountedDevices` oraz `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt`, odpowiednio.

Ten przewodnik podsumowuje kluczowe ścieżki i metody uzyskiwania szczegółowych informacji o systemie, sieci i aktywności użytkownika w systemach Windows, dążąc do jasności i użyteczności.



{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się sztuczkami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}
