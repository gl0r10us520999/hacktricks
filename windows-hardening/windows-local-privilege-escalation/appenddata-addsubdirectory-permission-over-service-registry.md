**Originalni post je** [**https://itm4n.github.io/windows-registry-rpceptmapper-eop/**](https://itm4n.github.io/windows-registry-rpceptmapper-eop/)

## Sažetak

Dva ključa u registru su pronađena kao zapisiva od strane trenutnog korisnika:

- **`HKLM\SYSTEM\CurrentControlSet\Services\Dnscache`**
- **`HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`**

Preporučeno je da se provere dozvole servisa **RpcEptMapper** koristeći **regedit GUI**, posebno karticu **Efikasne dozvole** u prozoru **Napredne bezbednosne postavke**. Ovaj pristup omogućava procenu dodeljenih dozvola određenim korisnicima ili grupama bez potrebe da se ispituje svaki unos kontrole pristupa (ACE) pojedinačno.

Snimak ekrana je prikazao dozvole dodeljene korisniku sa niskim privilegijama, među kojima je bila istaknuta dozvola **Kreiraj podključ**. Ova dozvola, takođe poznata kao **AppendData/AddSubdirectory**, odgovara nalazima skripte.

Primećena je nemogućnost direktne izmene određenih vrednosti, ali mogućnost kreiranja novih podključeva. Primer koji je istaknut bio je pokušaj izmene vrednosti **ImagePath**, što je rezultiralo porukom o odbijenom pristupu.

Uprkos ovim ograničenjima, identifikovana je potencijalna mogućnost eskalacije privilegija kroz mogućnost korišćenja podključa **Performance** unutar registracione strukture servisa **RpcEptMapper**, podključa koji nije prisutan po defaultu. Ovo bi omogućilo registraciju DLL-a i praćenje performansi.

Konsultovana je dokumentacija o podključe **Performance** i njegovoj upotrebi za praćenje performansi, što je dovelo do razvoja dokaza o konceptu DLL-a. Ovaj DLL, koji demonstrira implementaciju funkcija **OpenPerfData**, **CollectPerfData** i **ClosePerfData**, testiran je putem **rundll32**, potvrđujući njegovu operativnu uspešnost.

Cilj je bio primorati **RPC Endpoint Mapper servis** da učita kreirani Performance DLL. Posmatranja su pokazala da izvršavanje WMI klasa upita vezanih za podatke o performansama putem PowerShell-a rezultira kreiranjem log fajla, omogućavajući izvršavanje proizvoljnog koda pod kontekstom **LOCAL SYSTEM**, čime se dodeljuju povišene privilegije.

Istaknuta je postojanost i potencijalne posledice ove ranjivosti, naglašavajući njenu relevantnost za strategije post-eksploatacije, lateralno kretanje i izbegavanje antivirusnih/EDR sistema.

Iako je ranjivost prvobitno otkrivena nenamerno kroz skriptu, naglašeno je da je njena eksploatacija ograničena na zastarele verzije Windows-a (npr. **Windows 7 / Server 2008 R2**) i zahteva lokalni pristup.
