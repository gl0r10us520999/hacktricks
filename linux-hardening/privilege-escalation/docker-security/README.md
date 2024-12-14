# Docker Security

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

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## **Podstawowe zabezpieczenia silnika Docker**

**Silnik Docker** wykorzystuje **Namespaces** i **Cgroups** jądra Linux do izolacji kontenerów, oferując podstawową warstwę zabezpieczeń. Dodatkową ochronę zapewniają **Capabilities dropping**, **Seccomp** oraz **SELinux/AppArmor**, co zwiększa izolację kontenerów. **Plugin autoryzacji** może dodatkowo ograniczyć działania użytkowników.

![Docker Security](https://sreeninet.files.wordpress.com/2016/03/dockersec1.png)

### Bezpieczny dostęp do silnika Docker

Silnik Docker można uzyskać lokalnie za pomocą gniazda Unix lub zdalnie przy użyciu HTTP. W przypadku dostępu zdalnego istotne jest stosowanie HTTPS i **TLS**, aby zapewnić poufność, integralność i uwierzytelnienie.

Silnik Docker, domyślnie, nasłuchuje na gnieździe Unix pod adresem `unix:///var/run/docker.sock`. W systemach Ubuntu opcje uruchamiania Dockera są definiowane w `/etc/default/docker`. Aby umożliwić zdalny dostęp do API Dockera i klienta, należy udostępnić demon Dockera przez gniazdo HTTP, dodając następujące ustawienia:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H tcp://192.168.56.101:2376"
sudo service docker restart
```
Jednakże, udostępnianie demona Docker przez HTTP nie jest zalecane z powodu obaw dotyczących bezpieczeństwa. Zaleca się zabezpieczenie połączeń za pomocą HTTPS. Istnieją dwa główne podejścia do zabezpieczenia połączenia:

1. Klient weryfikuje tożsamość serwera.
2. Zarówno klient, jak i serwer wzajemnie uwierzytelniają swoją tożsamość.

Certyfikaty są wykorzystywane do potwierdzenia tożsamości serwera. Aby uzyskać szczegółowe przykłady obu metod, zapoznaj się z [**tym przewodnikiem**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/).

### Bezpieczeństwo obrazów kontenerów

Obrazy kontenerów mogą być przechowywane w prywatnych lub publicznych repozytoriach. Docker oferuje kilka opcji przechowywania obrazów kontenerów:

* [**Docker Hub**](https://hub.docker.com): Publiczna usługa rejestru od Docker.
* [**Docker Registry**](https://github.com/docker/distribution): Projekt open-source, który pozwala użytkownikom hostować własny rejestr.
* [**Docker Trusted Registry**](https://www.docker.com/docker-trusted-registry): Komercyjna oferta rejestru Docker, z uwierzytelnianiem użytkowników opartym na rolach i integracją z usługami katalogowymi LDAP.

### Skanowanie obrazów

Kontenery mogą mieć **luki w zabezpieczeniach** zarówno z powodu obrazu bazowego, jak i z powodu oprogramowania zainstalowanego na obrazie bazowym. Docker pracuje nad projektem o nazwie **Nautilus**, który przeprowadza skanowanie bezpieczeństwa kontenerów i wymienia luki. Nautilus działa, porównując każdą warstwę obrazu kontenera z repozytorium luk, aby zidentyfikować luki w zabezpieczeniach.

Aby uzyskać więcej [**informacji, przeczytaj to**](https://docs.docker.com/engine/scan/).

* **`docker scan`**

Polecenie **`docker scan`** pozwala na skanowanie istniejących obrazów Docker za pomocą nazwy lub identyfikatora obrazu. Na przykład, uruchom następujące polecenie, aby przeskanować obraz hello-world:
```bash
docker scan hello-world

Testing hello-world...

Organization:      docker-desktop-test
Package manager:   linux
Project name:      docker-image|hello-world
Docker image:      hello-world
Licenses:          enabled

✓ Tested 0 dependencies for known issues, no vulnerable paths found.

Note that we do not currently have vulnerability data for your image.
```
* [**`trivy`**](https://github.com/aquasecurity/trivy)
```bash
trivy -q -f json <container_name>:<tag>
```
* [**`snyk`**](https://docs.snyk.io/snyk-cli/getting-started-with-the-cli)
```bash
snyk container test <image> --json-file-output=<output file> --severity-threshold=high
```
* [**`clair-scanner`**](https://github.com/arminc/clair-scanner)
```bash
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```
### Podpisywanie obrazów Docker

Podpisywanie obrazów Docker zapewnia bezpieczeństwo i integralność obrazów używanych w kontenerach. Oto skrócone wyjaśnienie:

* **Docker Content Trust** wykorzystuje projekt Notary, oparty na The Update Framework (TUF), do zarządzania podpisywaniem obrazów. Więcej informacji można znaleźć w [Notary](https://github.com/docker/notary) i [TUF](https://theupdateframework.github.io).
* Aby aktywować zaufanie do treści Docker, ustaw `export DOCKER_CONTENT_TRUST=1`. Ta funkcja jest domyślnie wyłączona w wersji Docker 1.10 i nowszych.
* Po włączeniu tej funkcji można pobierać tylko podpisane obrazy. Początkowe przesyłanie obrazu wymaga ustawienia haseł dla kluczy głównych i tagujących, a Docker obsługuje również Yubikey dla zwiększonego bezpieczeństwa. Więcej szczegółów można znaleźć [tutaj](https://blog.docker.com/2015/11/docker-content-trust-yubikey/).
* Próba pobrania niepodpisanego obrazu z włączonym zaufaniem do treści skutkuje błędem "No trust data for latest".
* Przy przesyłaniu obrazów po pierwszym, Docker prosi o hasło klucza repozytorium, aby podpisać obraz.

Aby wykonać kopię zapasową swoich prywatnych kluczy, użyj polecenia:
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
When switching Docker hosts, it's necessary to move the root and repository keys to maintain operations.

***

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## Containers Security Features

<details>

<summary>Podsumowanie funkcji zabezpieczeń kontenerów</summary>

**Główne funkcje izolacji procesów**

W środowiskach kontenerowych izolacja projektów i ich procesów jest kluczowa dla bezpieczeństwa i zarządzania zasobami. Oto uproszczone wyjaśnienie kluczowych pojęć:

**Przestrzenie nazw**

* **Cel**: Zapewnienie izolacji zasobów, takich jak procesy, sieć i systemy plików. Szczególnie w Dockerze, przestrzenie nazw utrzymują procesy kontenera oddzielone od hosta i innych kontenerów.
* **Użycie `unshare`**: Komenda `unshare` (lub podstawowe wywołanie syscall) jest wykorzystywana do tworzenia nowych przestrzeni nazw, co zapewnia dodatkową warstwę izolacji. Jednak podczas gdy Kubernetes nie blokuje tego z natury, Docker to robi.
* **Ograniczenie**: Tworzenie nowych przestrzeni nazw nie pozwala procesowi na powrót do domyślnych przestrzeni nazw hosta. Aby przeniknąć do przestrzeni nazw hosta, zazwyczaj wymagany jest dostęp do katalogu `/proc` hosta, używając `nsenter` do wejścia.

**Grupy kontrolne (CGroups)**

* **Funkcja**: Głównie używane do alokacji zasobów między procesami.
* **Aspekt bezpieczeństwa**: CGroups same w sobie nie oferują bezpieczeństwa izolacji, z wyjątkiem funkcji `release_agent`, która, jeśli jest źle skonfigurowana, może być potencjalnie wykorzystana do nieautoryzowanego dostępu.

**Ograniczenie możliwości**

* **Znaczenie**: To kluczowa funkcja zabezpieczeń dla izolacji procesów.
* **Funkcjonalność**: Ogranicza działania, które proces root może wykonać, poprzez usunięcie niektórych możliwości. Nawet jeśli proces działa z uprawnieniami roota, brak niezbędnych możliwości uniemożliwia mu wykonywanie uprzywilejowanych działań, ponieważ wywołania syscall zakończą się niepowodzeniem z powodu niewystarczających uprawnień.

To są **pozostałe możliwości** po usunięciu innych przez proces:

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

Jest włączony domyślnie w Dockerze. Pomaga **jeszcze bardziej ograniczyć syscalls**, które proces może wywołać.\
**Domyślny profil Seccomp Docker** można znaleźć pod adresem [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)

**AppArmor**

Docker ma szablon, który możesz aktywować: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

To pozwoli na ograniczenie możliwości, syscalls, dostępu do plików i folderów...

</details>

### Namespaces

**Namespaces** to funkcja jądra Linux, która **dzieli zasoby jądra** w taki sposób, że jeden zestaw **procesów** **widzi** jeden zestaw **zasobów**, podczas gdy **inny** zestaw **procesów** widzi **inny** zestaw zasobów. Funkcja działa poprzez posiadanie tego samego namespace dla zestawu zasobów i procesów, ale te namespace odnoszą się do odrębnych zasobów. Zasoby mogą istnieć w wielu przestrzeniach.

Docker wykorzystuje następujące Namespaces jądra Linux do osiągnięcia izolacji kontenerów:

* pid namespace
* mount namespace
* network namespace
* ipc namespace
* UTS namespace

Aby uzyskać **więcej informacji o namespaces**, sprawdź następującą stronę:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

Funkcja jądra Linux **cgroups** zapewnia możliwość **ograniczenia zasobów, takich jak cpu, pamięć, io, przepustowość sieci wśród** zestawu procesów. Docker pozwala na tworzenie kontenerów z wykorzystaniem funkcji cgroup, co umożliwia kontrolę zasobów dla konkretnego kontenera.\
Poniżej znajduje się kontener utworzony z ograniczoną pamięcią przestrzeni użytkownika do 500m, pamięcią jądra ograniczoną do 50m, udziałem CPU do 512, blkiowe ciężar do 400. Udział CPU to wskaźnik, który kontroluje wykorzystanie CPU przez kontener. Ma domyślną wartość 1024 i zakres od 0 do 1024. Jeśli trzy kontenery mają ten sam udział CPU wynoszący 1024, każdy kontener może zająć do 33% CPU w przypadku kontestacji zasobów CPU. blkio-weight to wskaźnik, który kontroluje IO kontenera. Ma domyślną wartość 500 i zakres od 10 do 1000.
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
Aby uzyskać cgroup kontenera, możesz zrobić:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
For more information check:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### Capabilities

Capabilities allow **finer control for the capabilities that can be allowed** for root user. Docker uses the Linux kernel capability feature to **limit the operations that can be done inside a Container** irrespective of the type of user.

When a docker container is run, the **process drops sensitive capabilities that the process could use to escape from the isolation**. This try to assure that the process won't be able to perform sensitive actions and escape:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Seccomp in Docker

This is a security feature that allows Docker to **limit the syscalls** that can be used inside the container:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### AppArmor in Docker

**AppArmor** is a kernel enhancement to confine **containers** to a **limited** set of **resources** with **per-program profiles**.:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### SELinux in Docker

* **System etykietowania**: SELinux przypisuje unikalną etykietę do każdego procesu i obiektu systemu plików.
* **Egzekwowanie polityki**: Egzekwuje polityki bezpieczeństwa, które definiują, jakie działania etykieta procesu może wykonywać na innych etykietach w systemie.
* **Etykiety procesów kontenerów**: Gdy silniki kontenerów inicjują procesy kontenerów, zazwyczaj przypisywana jest im ograniczona etykieta SELinux, zwykle `container_t`.
* **Etykietowanie plików w kontenerach**: Pliki w kontenerze są zazwyczaj etykietowane jako `container_file_t`.
* **Reguły polityki**: Polityka SELinux przede wszystkim zapewnia, że procesy z etykietą `container_t` mogą wchodzić w interakcje (czytać, pisać, wykonywać) tylko z plikami oznaczonymi jako `container_file_t`.

Ten mechanizm zapewnia, że nawet jeśli proces w kontenerze zostanie skompromitowany, jest ograniczony do interakcji tylko z obiektami, które mają odpowiadające etykiety, znacznie ograniczając potencjalne szkody wynikające z takich kompromisów.

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ & AuthN

In Docker, an authorization plugin plays a crucial role in security by deciding whether to allow or block requests to the Docker daemon. This decision is made by examining two key contexts:

* **Kontekst uwierzytelniania**: To obejmuje szczegółowe informacje o użytkowniku, takie jak kim jest i jak się uwierzytelnił.
* **Kontekst polecenia**: To obejmuje wszystkie istotne dane związane z wysyłanym żądaniem.

These contexts help ensure that only legitimate requests from authenticated users are processed, enhancing the security of Docker operations.

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## DoS from a container

If you are not properly limiting the resources a container can use, a compromised container could DoS the host where it's running.

* CPU DoS
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
* Atak DoS na pasmo
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## Interesujące flagi Dockera

### Flaga --privileged

Na następnej stronie możesz się dowiedzieć **co oznacza flaga `--privileged`**:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

Jeśli uruchamiasz kontener, w którym atakujący zdołał uzyskać dostęp jako użytkownik o niskich uprawnieniach. Jeśli masz **źle skonfigurowany binarny plik suid**, atakujący może go nadużyć i **eskalować uprawnienia wewnątrz** kontenera. Co może pozwolić mu na ucieczkę z niego.

Uruchomienie kontenera z włączoną opcją **`no-new-privileges`** **zapobiegnie tego rodzaju eskalacji uprawnień**.
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### Inne
```bash
#You can manually add/drop capabilities with
--cap-add
--cap-drop

# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined

# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined

# You can manually disable selinux in docker with
--security-opt label:disable
```
For more **`--security-opt`** options check: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

## Inne rozważania dotyczące bezpieczeństwa

### Zarządzanie sekretami: najlepsze praktyki

Konieczne jest unikanie osadzania sekretów bezpośrednio w obrazach Docker lub używania zmiennych środowiskowych, ponieważ te metody narażają Twoje wrażliwe informacje na dostęp dla każdego, kto ma dostęp do kontenera za pomocą poleceń takich jak `docker inspect` lub `exec`.

**Wolumeny Docker** są bezpieczniejszą alternatywą, zalecaną do uzyskiwania dostępu do wrażliwych informacji. Mogą być wykorzystywane jako tymczasowy system plików w pamięci, łagodząc ryzyko związane z `docker inspect` i logowaniem. Jednak użytkownicy root i ci, którzy mają dostęp do `exec` w kontenerze, mogą nadal uzyskać dostęp do sekretów.

**Sekrety Docker** oferują jeszcze bezpieczniejszą metodę obsługi wrażliwych informacji. W przypadku instancji wymagających sekretów podczas fazy budowy obrazu, **BuildKit** przedstawia efektywne rozwiązanie z obsługą sekretów w czasie budowy, zwiększając prędkość budowy i oferując dodatkowe funkcje.

Aby skorzystać z BuildKit, można go aktywować na trzy sposoby:

1. Poprzez zmienną środowiskową: `export DOCKER_BUILDKIT=1`
2. Poprzez prefiksowanie poleceń: `DOCKER_BUILDKIT=1 docker build .`
3. Poprzez włączenie go domyślnie w konfiguracji Docker: `{ "features": { "buildkit": true } }`, a następnie ponowne uruchomienie Dockera.

BuildKit pozwala na użycie sekretów w czasie budowy z opcją `--secret`, zapewniając, że te sekrety nie są uwzględniane w pamięci podręcznej budowy obrazu ani w finalnym obrazie, używając polecenia takiego jak:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
Dla sekretów potrzebnych w działającym kontenerze, **Docker Compose i Kubernetes** oferują solidne rozwiązania. Docker Compose wykorzystuje klucz `secrets` w definicji usługi do określenia plików sekretów, jak pokazano w przykładzie `docker-compose.yml`:
```yaml
version: "3.7"
services:
my_service:
image: centos:7
entrypoint: "cat /run/secrets/my_secret"
secrets:
- my_secret
secrets:
my_secret:
file: ./my_secret_file.txt
```
Ta konfiguracja umożliwia użycie sekretów podczas uruchamiania usług za pomocą Docker Compose.

W środowiskach Kubernetes sekrety są natywnie obsługiwane i mogą być zarządzane za pomocą narzędzi takich jak [Helm-Secrets](https://github.com/futuresimple/helm-secrets). Kontrola dostępu oparta na rolach (RBAC) w Kubernetes zwiększa bezpieczeństwo zarządzania sekretami, podobnie jak w Docker Enterprise.

### gVisor

**gVisor** to jądro aplikacji, napisane w Go, które implementuje znaczną część powierzchni systemu Linux. Zawiera runtime [Open Container Initiative (OCI)](https://www.opencontainers.org) o nazwie `runsc`, który zapewnia **granice izolacji między aplikacją a jądrem hosta**. Runtime `runsc` integruje się z Dockerem i Kubernetes, co ułatwia uruchamianie kontenerów w piaskownicy.

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers** to społeczność open source, która pracuje nad budowaniem bezpiecznego runtime kontenerów z lekkimi maszynami wirtualnymi, które działają i zachowują się jak kontenery, ale zapewniają **silniejszą izolację obciążenia roboczego przy użyciu technologii wirtualizacji sprzętowej** jako drugiej warstwy obrony.

{% embed url="https://katacontainers.io/" %}

### Podsumowanie wskazówek

* **Nie używaj flagi `--privileged` ani nie montuj** [**gniazda Docker wewnątrz kontenera**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)**.** Gniazdo Docker umożliwia uruchamianie kontenerów, więc jest to łatwy sposób na przejęcie pełnej kontroli nad hostem, na przykład uruchamiając inny kontener z flagą `--privileged`.
* **Nie uruchamiaj jako root wewnątrz kontenera. Użyj** [**innego użytkownika**](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user) **i** [**przestrzeni nazw użytkowników**](https://docs.docker.com/engine/security/userns-remap/)**.** Root w kontenerze jest taki sam jak na hoście, chyba że jest przemapowany za pomocą przestrzeni nazw użytkowników. Jest on tylko lekko ograniczony przez, głównie, przestrzenie nazw Linuxa, możliwości i cgroups.
* [**Zrzuć wszystkie możliwości**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) i włącz tylko te, które są wymagane** (`--cap-add=...`). Wiele obciążeń roboczych nie potrzebuje żadnych możliwości, a ich dodanie zwiększa zakres potencjalnego ataku.
* [**Użyj opcji zabezpieczeń „no-new-privileges”**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) aby zapobiec uzyskiwaniu przez procesy większych uprawnień, na przykład przez binaria suid.
* [**Ogranicz zasoby dostępne dla kontenera**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**.** Limity zasobów mogą chronić maszynę przed atakami typu denial of service.
* **Dostosuj** [**seccomp**](https://docs.docker.com/engine/security/seccomp/)**,** [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(lub SELinux)** profile, aby ograniczyć działania i wywołania syscalls dostępne dla kontenera do minimum.
* **Używaj** [**oficjalnych obrazów docker**](https://docs.docker.com/docker-hub/official_images/) **i wymagaj podpisów** lub buduj własne na ich podstawie. Nie dziedzicz ani nie używaj [obrazów z backdoorem](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/). Przechowuj również klucze root, hasła w bezpiecznym miejscu. Docker planuje zarządzać kluczami za pomocą UCP.
* **Regularnie** **przebudowuj** swoje obrazy, aby **zastosować poprawki zabezpieczeń do hosta i obrazów.**
* Zarządzaj swoimi **sekretami mądrze**, aby utrudnić atakującemu dostęp do nich.
* Jeśli **udostępniasz demona docker, użyj HTTPS** z uwierzytelnianiem klienta i serwera.
* W swoim Dockerfile, **preferuj COPY zamiast ADD**. ADD automatycznie wyodrębnia pliki skompresowane i może kopiować pliki z adresów URL. COPY nie ma tych możliwości. Kiedy to możliwe, unikaj używania ADD, aby nie być podatnym na ataki przez zdalne adresy URL i pliki Zip.
* Miej **osobne kontenery dla każdego mikroserwisu**
* **Nie umieszczaj ssh** wewnątrz kontenera, „docker exec” może być używane do ssh do kontenera.
* Miej **mniejsze** obrazy kontenerów

## Docker Breakout / Eskalacja uprawnień

Jeśli jesteś **wewnątrz kontenera docker** lub masz dostęp do użytkownika w **grupie docker**, możesz spróbować **uciec i eskalować uprawnienia**:

{% content-ref url="docker-breakout-privilege-escalation/" %}
[docker-breakout-privilege-escalation](docker-breakout-privilege-escalation/)
{% endcontent-ref %}

## Ominięcie wtyczki uwierzytelniania Docker

Jeśli masz dostęp do gniazda docker lub masz dostęp do użytkownika w **grupie docker, ale twoje działania są ograniczane przez wtyczkę uwierzytelniania docker**, sprawdź, czy możesz **to obejść:**

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Wzmacnianie Dockera

* Narzędzie [**docker-bench-security**](https://github.com/docker/docker-bench-security) to skrypt, który sprawdza dziesiątki powszechnych najlepszych praktyk dotyczących wdrażania kontenerów Docker w produkcji. Testy są w pełni zautomatyzowane i oparte na [CIS Docker Benchmark v1.3.1](https://www.cisecurity.org/benchmark/docker/).\
Musisz uruchomić narzędzie z hosta uruchamiającego dockera lub z kontenera z wystarczającymi uprawnieniami. Dowiedz się **jak to uruchomić w README:** [**https://github.com/docker/docker-bench-security**](https://github.com/docker/docker-bench-security).

## Odnośniki

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/_fel1x/status/1151487051986087936](https://twitter.com/_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/)
* [https://en.wikipedia.org/wiki/Linux_namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://www.redhat.com/sysadmin/privileged-flag-container-engines](https://www.redhat.com/sysadmin/privileged-flag-container-engines)
* [https://docs.docker.com/engine/extend/plugins_authorization](https://docs.docker.com/engine/extend/plugins_authorization)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/](https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/)

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Użyj [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security), aby łatwo budować i **automatyzować przepływy pracy** zasilane przez **najbardziej zaawansowane** narzędzia społeczności.\
Uzyskaj dostęp już dziś:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegram**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Podziel się sztuczkami hackingowymi, przesyłając PR do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}
