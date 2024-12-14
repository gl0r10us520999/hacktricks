## Integralność oprogramowania układowego

**Niestandardowe oprogramowanie układowe i/lub skompilowane binaria mogą być przesyłane w celu wykorzystania luk w integralności lub weryfikacji podpisu**. Można wykonać następujące kroki w celu skompilowania backdoora z bind shell:

1. Oprogramowanie układowe można wyodrębnić za pomocą firmware-mod-kit (FMK).
2. Należy zidentyfikować architekturę oprogramowania układowego i endianness.
3. Można zbudować kompilator krzyżowy za pomocą Buildroot lub innych odpowiednich metod dla środowiska.
4. Backdoor można zbudować za pomocą kompilatora krzyżowego.
5. Backdoor można skopiować do katalogu /usr/bin wyodrębnionego oprogramowania układowego.
6. Odpowiedni binarny plik QEMU można skopiować do rootfs wyodrębnionego oprogramowania układowego.
7. Backdoor można emulować za pomocą chroot i QEMU.
8. Backdoor można uzyskać za pomocą netcat.
9. Binarne pliki QEMU należy usunąć z rootfs wyodrębnionego oprogramowania układowego.
10. Zmodyfikowane oprogramowanie układowe można spakować ponownie za pomocą FMK.
11. Oprogramowanie układowe z backdoorem można przetestować, emulując je za pomocą zestawu narzędzi do analizy oprogramowania układowego (FAT) i łącząc się z docelowym adresem IP i portem backdoora za pomocą netcat.

Jeśli już uzyskano dostęp do powłoki root poprzez analizę dynamiczną, manipulację bootloaderem lub testowanie bezpieczeństwa sprzętu, można uruchomić prekompilowane złośliwe binaria, takie jak implanty lub reverse shelle. Zautomatyzowane narzędzia do payloadów/implantów, takie jak framework Metasploit i 'msfvenom', można wykorzystać, wykonując następujące kroki:

1. Należy zidentyfikować architekturę oprogramowania układowego i endianness.
2. Msfvenom można użyć do określenia docelowego payloadu, adresu IP atakującego, numeru portu nasłuchującego, typu pliku, architektury, platformy i pliku wyjściowego.
3. Payload można przesłać do skompromitowanego urządzenia i upewnić się, że ma uprawnienia do wykonania.
4. Metasploit można przygotować do obsługi przychodzących żądań, uruchamiając msfconsole i konfigurując ustawienia zgodnie z payloadem.
5. Reverse shell meterpreter można uruchomić na skompromitowanym urządzeniu.
