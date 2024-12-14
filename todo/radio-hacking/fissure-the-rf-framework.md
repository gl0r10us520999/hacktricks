# FISSURE - The RF Framework

**Częstotliwościowo Niezależne Zrozumienie i Inżynieria Wsteczna Sygnałów Oparte na SDR**

FISSURE to otwartoźródłowa platforma RF i inżynierii wstecznej zaprojektowana dla wszystkich poziomów umiejętności, z interfejsami do wykrywania i klasyfikacji sygnałów, odkrywania protokołów, wykonywania ataków, manipulacji IQ, analizy podatności, automatyzacji oraz AI/ML. Platforma została stworzona, aby promować szybkie integrowanie modułów oprogramowania, radii, protokołów, danych sygnałowych, skryptów, grafów przepływu, materiałów referencyjnych i narzędzi firm trzecich. FISSURE to narzędzie umożliwiające przepływ pracy, które utrzymuje oprogramowanie w jednym miejscu i pozwala zespołom na łatwe przystosowanie się, dzieląc się tą samą sprawdzoną konfiguracją bazową dla konkretnych dystrybucji Linuxa.

Platforma i narzędzia zawarte w FISSURE są zaprojektowane do wykrywania obecności energii RF, rozumienia charakterystyki sygnału, zbierania i analizowania próbek, opracowywania technik nadawania i/lub wstrzykiwania oraz tworzenia niestandardowych ładunków lub wiadomości. FISSURE zawiera rosnącą bibliotekę informacji o protokołach i sygnałach, aby wspierać identyfikację, tworzenie pakietów i fuzzing. Istnieją możliwości archiwizacji online, aby pobierać pliki sygnałowe i tworzyć listy odtwarzania do symulacji ruchu i testowania systemów.

Przyjazna baza kodu Python i interfejs użytkownika pozwala początkującym szybko nauczyć się popularnych narzędzi i technik związanych z RF i inżynierią wsteczną. Nauczyciele w dziedzinie cyberbezpieczeństwa i inżynierii mogą skorzystać z wbudowanego materiału lub wykorzystać platformę do demonstrowania własnych aplikacji w rzeczywistych warunkach. Programiści i badacze mogą używać FISSURE do codziennych zadań lub do prezentowania swoich nowatorskich rozwiązań szerszej publiczności. W miarę jak świadomość i wykorzystanie FISSURE rośnie w społeczności, tak samo wzrośnie zakres jego możliwości i różnorodność technologii, które obejmuje.

**Dodatkowe Informacje**

* [Strona AIS](https://www.ainfosec.com/technologies/fissure/)
* [Prezentacje GRCon22](https://events.gnuradio.org/event/18/contributions/246/attachments/84/164/FISSURE\_Poore\_GRCon22.pdf)
* [Artykuł GRCon22](https://events.gnuradio.org/event/18/contributions/246/attachments/84/167/FISSURE\_Paper\_Poore\_GRCon22.pdf)
* [Wideo GRCon22](https://www.youtube.com/watch?v=1f2umEKhJvE)
* [Transkrypcja Hack Chat](https://hackaday.io/event/187076-rf-hacking-hack-chat/log/212136-hack-chat-transcript-part-1)

## Rozpoczęcie

**Obsługiwane**

W FISSURE znajdują się trzy gałęzie, aby ułatwić nawigację po plikach i zredukować redundancję kodu. Gałąź Python2\_maint-3.7 zawiera bazę kodu opartą na Python2, PyQt4 i GNU Radio 3.7; gałąź Python3\_maint-3.8 jest oparta na Python3, PyQt5 i GNU Radio 3.8; a gałąź Python3\_maint-3.10 jest oparta na Python3, PyQt5 i GNU Radio 3.10.

|   System Operacyjny   |   Gałąź FISSURE   |
| :------------------: | :----------------: |
|  Ubuntu 18.04 (x64)  | Python2\_maint-3.7 |
| Ubuntu 18.04.5 (x64) | Python2\_maint-3.7 |
| Ubuntu 18.04.6 (x64) | Python2\_maint-3.7 |
| Ubuntu 20.04.1 (x64) | Python3\_maint-3.8 |
| Ubuntu 20.04.4 (x64) | Python3\_maint-3.8 |
|  KDE neon 5.25 (x64) | Python3\_maint-3.8 |

**W Trakcie (beta)**

Te systemy operacyjne są nadal w statusie beta. Są w fazie rozwoju i wiadomo, że brakuje kilku funkcji. Elementy w instalatorze mogą kolidować z istniejącymi programami lub nie instalować się, dopóki status nie zostanie usunięty.

|     System Operacyjny     |    Gałąź FISSURE   |
| :----------------------: | :-----------------: |
| DragonOS Focal (x86\_64) |  Python3\_maint-3.8 |
|    Ubuntu 22.04 (x64)    | Python3\_maint-3.10 |

Uwaga: Niektóre narzędzia oprogramowania nie działają na każdym systemie operacyjnym. Odwołaj się do [Oprogramowanie i Konflikty](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Help/Markdown/SoftwareAndConflicts.md)

**Instalacja**
```
git clone https://github.com/ainfosec/FISSURE.git
cd FISSURE
git checkout <Python2_maint-3.7> or <Python3_maint-3.8> or <Python3_maint-3.10>
git submodule update --init
./install
```
To zainstaluje zależności oprogramowania PyQt wymagane do uruchomienia interfejsów instalacyjnych, jeśli nie zostaną znalezione.

Następnie wybierz opcję, która najlepiej odpowiada twojemu systemowi operacyjnemu (powinna być wykryta automatycznie, jeśli twój system operacyjny odpowiada opcji).

|                                          Python2\_maint-3.7                                          |                                          Python3\_maint-3.8                                          |                                          Python3\_maint-3.10                                         |
| :--------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------: |
| ![install1b](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1b.png) | ![install1a](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1a.png) | ![install1c](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1c.png) |

Zaleca się zainstalowanie FISSURE na czystym systemie operacyjnym, aby uniknąć istniejących konfliktów. Wybierz wszystkie zalecane pola wyboru (przycisk domyślny), aby uniknąć błędów podczas korzystania z różnych narzędzi w FISSURE. W trakcie instalacji pojawi się wiele komunikatów, głównie pytających o podwyższone uprawnienia i nazwy użytkowników. Jeśli element zawiera sekcję "Weryfikacja" na końcu, instalator uruchomi polecenie, które następuje, i podświetli element pola wyboru na zielono lub czerwono w zależności od tego, czy polecenie wygeneruje jakiekolwiek błędy. Zaznaczone elementy bez sekcji "Weryfikacja" pozostaną czarne po zakończeniu instalacji.

![install2](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install2.png)

**Użycie**

Otwórz terminal i wpisz:
```
fissure
```
Refer to the FISSURE Help menu for more details on usage.

## Details

**Components**

* Dashboard
* Central Hub (HIPRFISR)
* Target Signal Identification (TSI)
* Protocol Discovery (PD)
* Flow Graph & Script Executor (FGE)

![components](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/components.png)

**Capabilities**

| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/detector.png)_**Detektor sygnału**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/iq.png)_**Manipulacja IQ**_      | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/library.png)_**Wyszukiwanie sygnału**_          | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/pd.png)_**Rozpoznawanie wzorców**_ |
| --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/attack.png)_**Ataki**_           | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/fuzzing.png)_**Fuzzing**_         | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/archive.png)_**Playlisty sygnałów**_       | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/gallery.png)_**Galeria obrazów**_  |
| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/packet.png)_**Tworzenie pakietów**_   | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/scapy.png)_**Integracja Scapy**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/crc\_calculator.png)_**Kalkulator CRC**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/log.png)_**Rejestrowanie**_            |

**Hardware**

The following is a list of "supported" hardware with varying levels of integration:

* USRP: X3xx, B2xx, B20xmini, USRP2, N2xx
* HackRF
* RTL2832U
* 802.11 Adapters
* LimeSDR
* bladeRF, bladeRF 2.0 micro
* Open Sniffer
* PlutoSDR

## Lessons

FISSURE comes with several helpful guides to become familiar with different technologies and techniques. Many include steps for using various tools that are integrated into FISSURE.

* [Lekcja1: OpenBTS](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson1\_OpenBTS.md)
* [Lekcja2: Lua Dissectors](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson2\_LuaDissectors.md)
* [Lekcja3: Sound eXchange](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson3\_Sound\_eXchange.md)
* [Lekcja4: ESP Boards](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson4\_ESP\_Boards.md)
* [Lekcja5: Śledzenie radiosond](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson5\_Radiosonde\_Tracking.md)
* [Lekcja6: RFID](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson6\_RFID.md)
* [Lekcja7: Typy danych](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson7\_Data\_Types.md)
* [Lekcja8: Niestandardowe bloki GNU Radio](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson8\_Custom\_GNU\_Radio\_Blocks.md)
* [Lekcja9: TPMS](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson9\_TPMS.md)
* [Lekcja10: Egzaminy na radioamatora](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson10\_Ham\_Radio\_Exams.md)
* [Lekcja11: Narzędzia Wi-Fi](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson11\_WiFi\_Tools.md)

## Roadmap

* [ ] Add more hardware types, RF protocols, signal parameters, analysis tools
* [ ] Support more operating systems
* [ ] Develop class material around FISSURE (RF Attacks, Wi-Fi, GNU Radio, PyQt, etc.)
* [ ] Create a signal conditioner, feature extractor, and signal classifier with selectable AI/ML techniques
* [ ] Implement recursive demodulation mechanisms for producing a bitstream from unknown signals
* [ ] Transition the main FISSURE components to a generic sensor node deployment scheme

## Contributing

Suggestions for improving FISSURE are strongly encouraged. Leave a comment in the [Discussions](https://github.com/ainfosec/FISSURE/discussions) page or in the Discord Server if you have any thoughts regarding the following:

* New feature suggestions and design changes
* Software tools with installation steps
* New lessons or additional material for existing lessons
* RF protocols of interest
* More hardware and SDR types for integration
* IQ analysis scripts in Python
* Installation corrections and improvements

Contributions to improve FISSURE are crucial to expediting its development. Any contributions you make are greatly appreciated. If you wish to contribute through code development, please fork the repo and create a pull request:

1. Fork the project
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a pull request

Creating [Issues](https://github.com/ainfosec/FISSURE/issues) to bring attention to bugs is also welcomed.

## Collaborating

Contact Assured Information Security, Inc. (AIS) Business Development to propose and formalize any FISSURE collaboration opportunities–whether that is through dedicating time towards integrating your software, having the talented people at AIS develop solutions for your technical challenges, or integrating FISSURE into other platforms/applications.

## License

GPL-3.0

For license details, see LICENSE file.

## Contact

Join the Discord Server: [https://discord.gg/JZDs5sgxcG](https://discord.gg/JZDs5sgxcG)

Follow on Twitter: [@FissureRF](https://twitter.com/fissurerf), [@AinfoSec](https://twitter.com/ainfosec)

Chris Poore - Assured Information Security, Inc. - poorec@ainfosec.com

Business Development - Assured Information Security, Inc. - bd@ainfosec.com

## Credits

We acknowledge and are grateful to these developers:

[Credits](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/CREDITS.md)

## Acknowledgments

Special thanks to Dr. Samuel Mantravadi and Joseph Reith for their contributions to this project.
