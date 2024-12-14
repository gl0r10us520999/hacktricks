# FISSURE - The RF Framework

**फ्रीक्वेंसी इंडिपेंडेंट SDR-आधारित सिग्नल समझ और रिवर्स इंजीनियरिंग**

FISSURE एक ओपन-सोर्स RF और रिवर्स इंजीनियरिंग फ्रेमवर्क है जिसे सभी कौशल स्तरों के लिए डिज़ाइन किया गया है जिसमें सिग्नल डिटेक्शन और वर्गीकरण, प्रोटोकॉल खोज, हमले का निष्पादन, IQ हेरफेर, कमजोरियों का विश्लेषण, स्वचालन, और AI/ML के लिए हुक शामिल हैं। यह फ्रेमवर्क सॉफ़्टवेयर मॉड्यूल, रेडियो, प्रोटोकॉल, सिग्नल डेटा, स्क्रिप्ट, फ्लो ग्राफ़, संदर्भ सामग्री, और तृतीय-पक्ष उपकरणों के त्वरित एकीकरण को बढ़ावा देने के लिए बनाया गया था। FISSURE एक वर्कफ़्लो सक्षम करने वाला है जो सॉफ़्टवेयर को एक स्थान पर रखता है और टीमों को एक ही सिद्ध बुनियादी कॉन्फ़िगरेशन साझा करते हुए तेजी से गति प्राप्त करने की अनुमति देता है।

FISSURE के साथ शामिल फ्रेमवर्क और उपकरण RF ऊर्जा की उपस्थिति का पता लगाने, सिग्नल की विशेषताओं को समझने, नमूने एकत्र करने और विश्लेषण करने, ट्रांसमिट और/या इंजेक्शन तकनीकों को विकसित करने, और कस्टम पेलोड या संदेश बनाने के लिए डिज़ाइन किए गए हैं। FISSURE में पहचान, पैकेट क्राफ्टिंग, और फज़िंग में सहायता के लिए प्रोटोकॉल और सिग्नल जानकारी का एक बढ़ता हुआ पुस्तकालय है। ऑनलाइन आर्काइव क्षमताएँ सिग्नल फ़ाइलें डाउनलोड करने और ट्रैफ़िक का अनुकरण करने और सिस्टम का परीक्षण करने के लिए प्लेलिस्ट बनाने के लिए मौजूद हैं।

मित्रवत Python कोडबेस और उपयोगकर्ता इंटरफ़ेस शुरुआती लोगों को RF और रिवर्स इंजीनियरिंग से संबंधित लोकप्रिय उपकरणों और तकनीकों के बारे में जल्दी से सीखने की अनुमति देता है। साइबर सुरक्षा और इंजीनियरिंग में शिक्षकों को अंतर्निहित सामग्री का लाभ उठाने या अपने वास्तविक दुनिया के अनुप्रयोगों को प्रदर्शित करने के लिए फ्रेमवर्क का उपयोग करने का अवसर मिलता है। डेवलपर्स और शोधकर्ता अपने दैनिक कार्यों के लिए या अपने अत्याधुनिक समाधानों को व्यापक दर्शकों के सामने लाने के लिए FISSURE का उपयोग कर सकते हैं। जैसे-जैसे समुदाय में FISSURE के प्रति जागरूकता और उपयोग बढ़ता है, इसकी क्षमताओं का विस्तार और इसके द्वारा शामिल तकनीक की चौड़ाई भी बढ़ेगी।

**अतिरिक्त जानकारी**

* [AIS पृष्ठ](https://www.ainfosec.com/technologies/fissure/)
* [GRCon22 स्लाइड](https://events.gnuradio.org/event/18/contributions/246/attachments/84/164/FISSURE\_Poore\_GRCon22.pdf)
* [GRCon22 पेपर](https://events.gnuradio.org/event/18/contributions/246/attachments/84/167/FISSURE\_Paper\_Poore\_GRCon22.pdf)
* [GRCon22 वीडियो](https://www.youtube.com/watch?v=1f2umEKhJvE)
* [हैक चैट ट्रांसक्रिप्ट](https://hackaday.io/event/187076-rf-hacking-hack-chat/log/212136-hack-chat-transcript-part-1)

## शुरू करना

**समर्थित**

FISSURE में फ़ाइल नेविगेशन को आसान बनाने और कोड की पुनरावृत्ति को कम करने के लिए तीन शाखाएँ हैं। Python2\_maint-3.7 शाखा में Python2, PyQt4, और GNU Radio 3.7 के चारों ओर निर्मित कोडबेस है; Python3\_maint-3.8 शाखा Python3, PyQt5, और GNU Radio 3.8 के चारों ओर निर्मित है; और Python3\_maint-3.10 शाखा Python3, PyQt5, और GNU Radio 3.10 के चारों ओर निर्मित है।

|   ऑपरेटिंग सिस्टम   |   FISSURE शाखा   |
| :------------------: | :----------------: |
|  Ubuntu 18.04 (x64)  | Python2\_maint-3.7 |
| Ubuntu 18.04.5 (x64) | Python2\_maint-3.7 |
| Ubuntu 18.04.6 (x64) | Python2\_maint-3.7 |
| Ubuntu 20.04.1 (x64) | Python3\_maint-3.8 |
| Ubuntu 20.04.4 (x64) | Python3\_maint-3.8 |
|  KDE neon 5.25 (x64) | Python3\_maint-3.8 |

**प्रगति में (बीटा)**

ये ऑपरेटिंग सिस्टम अभी भी बीटा स्थिति में हैं। ये विकासाधीन हैं और कई सुविधाएँ ज्ञात रूप से गायब हैं। इंस्टॉलर में आइटम मौजूदा कार्यक्रमों के साथ संघर्ष कर सकते हैं या स्थिति हटाए जाने तक स्थापित करने में विफल हो सकते हैं।

|     ऑपरेटिंग सिस्टम     |    FISSURE शाखा   |
| :----------------------: | :-----------------: |
| DragonOS Focal (x86\_64) |  Python3\_maint-3.8 |
|    Ubuntu 22.04 (x64)    | Python3\_maint-3.10 |

नोट: कुछ सॉफ़्टवेयर उपकरण हर OS के लिए काम नहीं करते हैं। [सॉफ़्टवेयर और संघर्ष](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Help/Markdown/SoftwareAndConflicts.md) देखें।

**स्थापना**
```
git clone https://github.com/ainfosec/FISSURE.git
cd FISSURE
git checkout <Python2_maint-3.7> or <Python3_maint-3.8> or <Python3_maint-3.10>
git submodule update --init
./install
```
यह PyQt सॉफ़्टवेयर निर्भरताएँ स्थापित करेगा जो स्थापना GUI को लॉन्च करने के लिए आवश्यक हैं यदि वे नहीं मिलती हैं।

अगला, उस विकल्प का चयन करें जो आपके ऑपरेटिंग सिस्टम से सबसे अच्छा मेल खाता है (यदि आपका OS एक विकल्प से मेल खाता है तो इसे स्वचालित रूप से पहचान लिया जाएगा)।

|                                          Python2\_maint-3.7                                          |                                          Python3\_maint-3.8                                          |                                          Python3\_maint-3.10                                         |
| :--------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------: |
| ![install1b](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1b.png) | ![install1a](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1a.png) | ![install1c](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1c.png) |

यह अनुशंसा की जाती है कि FISSURE को एक साफ ऑपरेटिंग सिस्टम पर स्थापित किया जाए ताकि मौजूदा संघर्षों से बचा जा सके। विभिन्न उपकरणों के संचालन के दौरान त्रुटियों से बचने के लिए सभी अनुशंसित चेकबॉक्स (डिफ़ॉल्ट बटन) का चयन करें। स्थापना के दौरान कई संकेत होंगे, ज्यादातर उच्च अनुमति और उपयोगकर्ता नाम के लिए पूछते हुए। यदि किसी आइटम के अंत में "सत्यापित करें" अनुभाग है, तो इंस्टॉलर उस आदेश को चलाएगा जो इसके बाद आता है और आदेश द्वारा उत्पन्न किसी भी त्रुटि के आधार पर चेकबॉक्स आइटम को हरा या लाल हाइलाइट करेगा। "सत्यापित करें" अनुभाग के बिना चेक किए गए आइटम स्थापना के बाद काले रहेंगे।

![install2](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install2.png)

**उपयोग**

एक टर्मिनल खोलें और दर्ज करें:
```
fissure
```
Refer to the FISSURE Help menu for more details on usage.

## Details

**Components**

* डैशबोर्ड
* केंद्रीय हब (HIPRFISR)
* लक्ष्य सिग्नल पहचान (TSI)
* प्रोटोकॉल खोज (PD)
* फ्लो ग्राफ़ और स्क्रिप्ट निष्पादक (FGE)

![components](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/components.png)

**Capabilities**

| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/detector.png)_**सिग्नल डिटेक्टर**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/iq.png)_**IQ मैनिपुलेशन**_      | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/library.png)_**सिग्नल लुकअप**_          | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/pd.png)_**पैटर्न पहचान**_ |
| --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/attack.png)_**हमले**_           | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/fuzzing.png)_**फज़िंग**_         | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/archive.png)_**सिग्नल प्लेलिस्ट**_       | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/gallery.png)_**इमेज गैलरी**_  |
| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/packet.png)_**पैकेट क्राफ्टिंग**_   | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/scapy.png)_**स्कैपी इंटीग्रेशन**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/crc\_calculator.png)_**CRC कैलकुलेटर**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/log.png)_**लॉगिंग**_            |

**Hardware**

The following is a list of "supported" hardware with varying levels of integration:

* USRP: X3xx, B2xx, B20xmini, USRP2, N2xx
* HackRF
* RTL2832U
* 802.11 एडाप्टर्स
* LimeSDR
* bladeRF, bladeRF 2.0 माइक्रो
* ओपन स्निफर
* PlutoSDR

## Lessons

FISSURE comes with several helpful guides to become familiar with different technologies and techniques. Many include steps for using various tools that are integrated into FISSURE.

* [पाठ1: OpenBTS](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson1\_OpenBTS.md)
* [पाठ2: Lua डिसेक्टर](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson2\_LuaDissectors.md)
* [पाठ3: साउंड एक्सचेंज](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson3\_Sound\_eXchange.md)
* [पाठ4: ESP बोर्ड](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson4\_ESP\_Boards.md)
* [पाठ5: रेडियोसोंड ट्रैकिंग](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson5\_Radiosonde\_Tracking.md)
* [पाठ6: RFID](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson6\_RFID.md)
* [पाठ7: डेटा प्रकार](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson7\_Data\_Types.md)
* [पाठ8: कस्टम GNU रेडियो ब्लॉक्स](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson8\_Custom\_GNU\_Radio\_Blocks.md)
* [पाठ9: TPMS](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson9\_TPMS.md)
* [पाठ10: हैम रेडियो परीक्षा](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson10\_Ham\_Radio\_Exams.md)
* [पाठ11: वाई-फाई उपकरण](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson11\_WiFi\_Tools.md)

## Roadmap

* [ ] अधिक हार्डवेयर प्रकार, RF प्रोटोकॉल, सिग्नल पैरामीटर, विश्लेषण उपकरण जोड़ें
* [ ] अधिक ऑपरेटिंग सिस्टम का समर्थन करें
* [ ] FISSURE के चारों ओर कक्षा सामग्री विकसित करें (RF हमले, वाई-फाई, GNU रेडियो, PyQt, आदि)
* [ ] एक सिग्नल कंडीशनर, फीचर एक्सट्रैक्टर, और सिग्नल क्लासिफायर बनाएं जिसमें चयन योग्य AI/ML तकनीकें हों
* [ ] अज्ञात सिग्नलों से बिटस्ट्रीम उत्पन्न करने के लिए पुनरावृत्त डिमोड्यूलेशन तंत्र लागू करें
* [ ] मुख्य FISSURE घटकों को एक सामान्य सेंसर नोड तैनाती योजना में स्थानांतरित करें

## Contributing

Suggestions for improving FISSURE are strongly encouraged. Leave a comment in the [Discussions](https://github.com/ainfosec/FISSURE/discussions) page or in the Discord Server if you have any thoughts regarding the following:

* नई फीचर सुझाव और डिज़ाइन परिवर्तन
* सॉफ़्टवेयर उपकरणों के साथ स्थापना चरण
* नए पाठ या मौजूदा पाठों के लिए अतिरिक्त सामग्री
* रुचि के RF प्रोटोकॉल
* एकीकरण के लिए अधिक हार्डवेयर और SDR प्रकार
* Python में IQ विश्लेषण स्क्रिप्ट
* स्थापना सुधार और सुधार

Contributions to improve FISSURE are crucial to expediting its development. Any contributions you make are greatly appreciated. If you wish to contribute through code development, please fork the repo and create a pull request:

1. प्रोजेक्ट को फोर्क करें
2. अपनी फीचर शाखा बनाएं (`git checkout -b feature/AmazingFeature`)
3. अपने परिवर्तनों को कमिट करें (`git commit -m 'Add some AmazingFeature'`)
4. शाखा में पुश करें (`git push origin feature/AmazingFeature`)
5. एक पुल अनुरोध खोलें

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
