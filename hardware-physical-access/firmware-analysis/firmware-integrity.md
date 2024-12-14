## Integritet firmvera

**Prilagođeni firmware i/ili kompajlirani binarni fajlovi mogu biti otpremljeni kako bi se iskoristile greške u integritetu ili verifikaciji potpisa**. Sledeći koraci se mogu pratiti za kompajlaciju backdoor bind shell-a:

1. Firmware se može ekstrahovati koristeći firmware-mod-kit (FMK).
2. Treba identifikovati arhitekturu firmware-a i endianness.
3. Može se izgraditi cross compiler koristeći Buildroot ili druge odgovarajuće metode za okruženje.
4. Backdoor se može izgraditi koristeći cross compiler.
5. Backdoor se može kopirati u /usr/bin direktorijum ekstrahovanog firmware-a.
6. Odgovarajući QEMU binarni fajl može se kopirati u rootfs ekstrahovanog firmware-a.
7. Backdoor se može emulirati koristeći chroot i QEMU.
8. Backdoor se može pristupiti putem netcat-a.
9. QEMU binarni fajl treba biti uklonjen iz rootfs ekstrahovanog firmware-a.
10. Izmenjeni firmware može biti ponovo upakovan koristeći FMK.
11. Backdoored firmware može biti testiran emulacijom sa alatom za analizu firmware-a (FAT) i povezivanjem na IP i port ciljanog backdoor-a koristeći netcat.

Ako je već dobijen root shell putem dinamičke analize, manipulacije bootloader-om ili testiranja hardverske sigurnosti, prekompajlirani zlonamerni binarni fajlovi kao što su implanti ili reverse shell-ovi mogu biti izvršeni. Automatizovani alati za payload/implant kao što je Metasploit framework i 'msfvenom' mogu se iskoristiti sledećim koracima:

1. Treba identifikovati arhitekturu firmware-a i endianness.
2. Msfvenom se može koristiti za specificiranje ciljanog payload-a, IP adrese napadača, broja slušnog porta, tipa fajla, arhitekture, platforme i izlaznog fajla.
3. Payload se može preneti na kompromitovani uređaj i osigurati da ima dozvole za izvršavanje.
4. Metasploit se može pripremiti za obradu dolaznih zahteva pokretanjem msfconsole-a i konfigurisanjem postavki prema payload-u.
5. Meterpreter reverse shell može biti izvršen na kompromitovanom uređaju.
