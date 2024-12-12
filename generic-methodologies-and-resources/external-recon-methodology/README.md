# External Recon Methodology

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If you are interested in **hacking career** and hack the unhackable - **we are hiring!** (_fluent polish written and spoken required_).

{% embed url="https://www.stmcyber.com/careers" %}

## Assets discoveries

> तो आपको बताया गया था कि किसी कंपनी से संबंधित सब कुछ दायरे के भीतर है, और आप यह पता लगाना चाहते हैं कि इस कंपनी के पास वास्तव में क्या है।

इस चरण का लक्ष्य मुख्य कंपनी द्वारा स्वामित्व वाली सभी **कंपनियों** और फिर इन कंपनियों के सभी **संपत्तियों** को प्राप्त करना है। ऐसा करने के लिए, हम:

1. मुख्य कंपनी के अधिग्रहणों को खोजेंगे, इससे हमें दायरे के भीतर की कंपनियाँ मिलेंगी।
2. प्रत्येक कंपनी का ASN (यदि कोई हो) खोजेंगे, इससे हमें प्रत्येक कंपनी द्वारा स्वामित्व वाले IP रेंज मिलेंगे।
3. पहले वाले से संबंधित अन्य प्रविष्टियों (संस्थान के नाम, डोमेन...) की खोज के लिए रिवर्स Whois लुकअप का उपयोग करेंगे (यह पुनरावृत्त रूप से किया जा सकता है)।
4. अन्य संपत्तियों की खोज के लिए शोडान `org` और `ssl` फ़िल्टर जैसी अन्य तकनीकों का उपयोग करेंगे (यह `ssl` ट्रिक पुनरावृत्त रूप से की जा सकती है)।

### **Acquisitions**

सबसे पहले, हमें यह जानने की आवश्यकता है कि **मुख्य कंपनी द्वारा कौन सी अन्य कंपनियाँ स्वामित्व में हैं**।\
एक विकल्प है [https://www.crunchbase.com/](https://www.crunchbase.com) पर जाना, **मुख्य कंपनी** के लिए **खोजें**, और "**अधिग्रहण**" पर **क्लिक करें**। वहाँ आप मुख्य कंपनी द्वारा अधिग्रहित अन्य कंपनियाँ देखेंगे।\
दूसरा विकल्प है मुख्य कंपनी के **विकिपीडिया** पृष्ठ पर जाना और **अधिग्रहण** के लिए खोज करना।

> ठीक है, इस बिंदु पर आपको दायरे के भीतर सभी कंपनियों के बारे में पता होना चाहिए। चलिए उनके संपत्तियों को खोजने का तरीका समझते हैं।

### **ASNs**

एक स्वायत्त प्रणाली संख्या (**ASN**) एक **विशिष्ट संख्या** है जो **इंटरनेट असाइन नंबर प्राधिकरण (IANA)** द्वारा एक **स्वायत्त प्रणाली** (AS) को असाइन की जाती है।\
एक **AS** में **IP पते** के **ब्लॉक** होते हैं जिनकी बाहरी नेटवर्क तक पहुँचने के लिए स्पष्ट रूप से परिभाषित नीति होती है और इसे एक ही संगठन द्वारा प्रशासित किया जाता है लेकिन यह कई ऑपरेटरों से मिलकर बन सकता है।

यह जानना दिलचस्प है कि क्या **कंपनी ने किसी ASN को असाइन किया है** ताकि इसके **IP रेंज** को खोजा जा सके। यह **दायरे** के भीतर सभी **होस्ट्स** के खिलाफ **कमजोरी परीक्षण** करना और इन IPs के भीतर **डोमेन** की खोज करना दिलचस्प होगा।\
आप **कंपनी के नाम**, **IP** या **डोमेन** द्वारा [**https://bgp.he.net/**](https://bgp.he.net)** पर खोज सकते हैं।**\
**कंपनी के क्षेत्र के आधार पर ये लिंक अधिक डेटा इकट्ठा करने के लिए उपयोगी हो सकते हैं:** [**AFRINIC**](https://www.afrinic.net) **(अफ्रीका),** [**Arin**](https://www.arin.net/about/welcome/region/)**(उत्तरी अमेरिका),** [**APNIC**](https://www.apnic.net) **(एशिया),** [**LACNIC**](https://www.lacnic.net) **(लैटिन अमेरिका),** [**RIPE NCC**](https://www.ripe.net) **(यूरोप)। वैसे, शायद सभी** उपयोगी जानकारी **(IP रेंज और Whois)** पहले लिंक में पहले से ही दिखाई देती है।
```bash
#You can try "automate" this with amass, but it's not very recommended
amass intel -org tesla
amass intel -asn 8911,50313,394161
```
Also, [**BBOT**](https://github.com/blacklanternsecurity/bbot)**'s** उपडोमेन गणना स्वचालित रूप से स्कैन के अंत में ASNs को एकत्रित और संक्षेपित करती है।
```bash
bbot -t tesla.com -f subdomain-enum
...
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS394161 | 8.244.131.0/24      | 5            | TESLA          | Tesla Motors, Inc.         | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS16509  | 54.148.0.0/15       | 4            | AMAZON-02      | Amazon.com, Inc.           | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS394161 | 8.45.124.0/24       | 3            | TESLA          | Tesla Motors, Inc.         | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS3356   | 8.32.0.0/12         | 1            | LEVEL3         | Level 3 Parent, LLC        | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS3356   | 8.0.0.0/9           | 1            | LEVEL3         | Level 3 Parent, LLC        | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+

```
You can find the IP ranges of an organisation also using [http://asnlookup.com/](http://asnlookup.com) (यह मुफ्त API है)।\
You can find the IP and ASN of a domain using [http://ipv4info.com/](http://ipv4info.com).

### **कमजोरियों की तलाश**

इस बिंदु पर हम **स्कोप के अंदर सभी संपत्तियों** को जानते हैं, इसलिए यदि आपको अनुमति है तो आप सभी होस्ट पर कुछ **कमजोरी स्कैनर** (Nessus, OpenVAS) लॉन्च कर सकते हैं।\
इसके अलावा, आप कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) **या** shodan **जैसी सेवाओं का उपयोग करके** खुले पोर्ट **खोज सकते हैं और जो कुछ भी आप पाते हैं उसके आधार पर आपको** इस पुस्तक में देखना चाहिए कि कैसे कई संभावित सेवाओं का पेंटेस्ट करें।\
**इसके अलावा, यह उल्लेख करना भी फायदेमंद हो सकता है कि आप कुछ** डिफ़ॉल्ट उपयोगकर्ता नाम **और** पासवर्ड **सूचियाँ तैयार कर सकते हैं और** [https://github.com/x90skysn3k/brutespray](https://github.com/x90skysn3k/brutespray) के साथ सेवाओं को** ब्रूटफोर्स **करने की कोशिश कर सकते हैं।**

## डोमेन

> हम स्कोप के अंदर सभी कंपनियों और उनकी संपत्तियों को जानते हैं, अब स्कोप के अंदर डोमेन खोजने का समय है।

_कृपया ध्यान दें कि निम्नलिखित प्रस्तावित तकनीकों में आप उपडोमेन भी खोज सकते हैं और उस जानकारी को कम नहीं आंकना चाहिए।_

सबसे पहले, आपको प्रत्येक कंपनी के **मुख्य डोमेन**(s) की तलाश करनी चाहिए। उदाहरण के लिए, _Tesla Inc._ के लिए _tesla.com_ होगा।

### **रिवर्स DNS**

जैसा कि आपने डोमेन के सभी IP रेंज खोज लिए हैं, आप उन **IPs पर अधिक डोमेन खोजने के लिए** **रिवर्स DNS लुकअप** करने की कोशिश कर सकते हैं। पीड़ित के कुछ DNS सर्वर या कुछ प्रसिद्ध DNS सर्वर (1.1.1.1, 8.8.8.8) का उपयोग करने की कोशिश करें।
```bash
dnsrecon -r <DNS Range> -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d facebook.com -r 157.240.221.35/24 #Using facebooks dns
dnsrecon -r 157.240.221.35/24 -n 1.1.1.1 #Using cloudflares dns
dnsrecon -r 157.240.221.35/24 -n 8.8.8.8 #Using google dns
```
For this to work, the administrator has to enable manually the PTR.\
आप इस जानकारी के लिए एक ऑनलाइन टूल का भी उपयोग कर सकते हैं: [http://ptrarchive.com/](http://ptrarchive.com)

### **Reverse Whois (loop)**

Inside a **whois** you can find a lot of interesting **information** like **organisation name**, **address**, **emails**, phone numbers... But which is even more interesting is that you can find **more assets related to the company** if you perform **reverse whois lookups by any of those fields** (for example other whois registries where the same email appears).\
आप ऑनलाइन टूल का उपयोग कर सकते हैं जैसे:

* [https://viewdns.info/reversewhois/](https://viewdns.info/reversewhois/) - **Free**
* [https://domaineye.com/reverse-whois](https://domaineye.com/reverse-whois) - **Free**
* [https://www.reversewhois.io/](https://www.reversewhois.io) - **Free**
* [https://www.whoxy.com/](https://www.whoxy.com) - **Free** web, not free API.
* [http://reversewhois.domaintools.com/](http://reversewhois.domaintools.com) - Not free
* [https://drs.whoisxmlapi.com/reverse-whois-search](https://drs.whoisxmlapi.com/reverse-whois-search) - Not Free (only **100 free** searches)
* [https://www.domainiq.com/](https://www.domainiq.com) - Not Free

You can automate this task using [**DomLink** ](https://github.com/vysecurity/DomLink)(requires a whoxy API key).\
आप कुछ स्वचालित रिवर्स Whois खोज भी कर सकते हैं [amass](https://github.com/OWASP/Amass) के साथ: `amass intel -d tesla.com -whois`

**Note that you can use this technique to discover more domain names every time you find a new domain.**

### **Trackers**

If find the **same ID of the same tracker** in 2 different pages you can suppose that **both pages** are **managed by the same team**.\
उदाहरण के लिए, यदि आप कई पृष्ठों पर वही **Google Analytics ID** या वही **Adsense ID** देखते हैं।

There are some pages and tools that let you search by these trackers and more:

* [**Udon**](https://github.com/dhn/udon)
* [**BuiltWith**](https://builtwith.com)
* [**Sitesleuth**](https://www.sitesleuth.io)
* [**Publicwww**](https://publicwww.com)
* [**SpyOnWeb**](http://spyonweb.com)

### **Favicon**

Did you know that we can find related domains and sub domains to our target by looking for the same favicon icon hash? This is exactly what [favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py) tool made by [@m4ll0k2](https://twitter.com/m4ll0k2) does. Here’s how to use it:
```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```
![favihash - समान favicon आइकन हैश वाले डोमेन खोजें](https://www.infosecmatter.com/wp-content/uploads/2020/07/favihash.jpg)

साधारण शब्दों में, favihash हमें उन डोमेन को खोजने की अनुमति देगा जिनका favicon आइकन हैश हमारे लक्ष्य के समान है।

इसके अलावा, आप favicon हैश का उपयोग करके तकनीकों की भी खोज कर सकते हैं जैसा कि [**इस ब्लॉग पोस्ट**](https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139) में समझाया गया है। इसका मतलब है कि यदि आप जानते हैं कि **किसी कमजोर वेब तकनीक के संस्करण का favicon का हैश** क्या है, तो आप शोडन में खोज सकते हैं और **अधिक कमजोर स्थानों** को खोज सकते हैं:
```bash
shodan search org:"Target" http.favicon.hash:116323821 --fields ip_str,port --separator " " | awk '{print $1":"$2}'
```
यहाँ बताया गया है कि आप एक वेब का **फेविकॉन हैश** कैसे **गणना** कर सकते हैं:
```python
import mmh3
import requests
import codecs

def fav_hash(url):
response = requests.get(url)
favicon = codecs.encode(response.content,"base64")
fhash = mmh3.hash(favicon)
print(f"{url} : {fhash}")
return fhash
```
### **Copyright / Uniq string**

वेब पृष्ठों के अंदर **स्ट्रिंग्स की खोज करें जो एक ही संगठन में विभिन्न वेब्स के बीच साझा की जा सकती हैं**। **कॉपीराइट स्ट्रिंग** एक अच्छा उदाहरण हो सकता है। फिर उस स्ट्रिंग की **गूगल**, अन्य **ब्राउज़रों** या यहां तक कि **शोडन** में खोज करें: `shodan search http.html:"Copyright string"`

### **CRT Time**

यह सामान्य है कि एक क्रॉन जॉब हो जैसे
```bash
# /etc/crontab
37 13 */10 * * certbot renew --post-hook "systemctl reload nginx"
```
to renew the all the domain certificates on the server. This means that even if the CA used for this doesn't set the time it was generated in the Validity time, it's possible to **find domains belonging to the same company in the certificate transparency logs**.\
Check out this [**writeup for more information**](https://swarm.ptsecurity.com/discovering-domains-via-a-time-correlation-attack/).

### Mail DMARC information

You can use a web such as [https://dmarc.live/info/google.com](https://dmarc.live/info/google.com) or a tool such as [https://github.com/Tedixx/dmarc-subdomains](https://github.com/Tedixx/dmarc-subdomains) to find **domains and subdomain sharing the same dmarc information**.

### **Passive Takeover**

Apparently is common for people to assign subdomains to IPs that belongs to cloud providers and at some point **lose that IP address but forget about removing the DNS record**. Therefore, just **spawning a VM** in a cloud (like Digital Ocean) you will be actually **taking over some subdomains(s)**.

[**This post**](https://kmsec.uk/blog/passive-takeover/) explains a store about it and propose a script that **spawns a VM in DigitalOcean**, **gets** the **IPv4** of the new machine, and **searches in Virustotal for subdomain records** pointing to it.

### **Other ways**

**Note that you can use this technique to discover more domain names every time you find a new domain.**

**Shodan**

As you already know the name of the organisation owning the IP space. You can search by that data in shodan using: `org:"Tesla, Inc."` Check the found hosts for new unexpected domains in the TLS certificate.

You could access the **TLS certificate** of the main web page, obtain the **Organisation name** and then search for that name inside the **TLS certificates** of all the web pages known by **shodan** with the filter : `ssl:"Tesla Motors"` or use a tool like [**sslsearch**](https://github.com/HarshVaragiya/sslsearch).

**Assetfinder**

[**Assetfinder** ](https://github.com/tomnomnom/assetfinder)is a tool that look for **domains related** with a main domain and **subdomains** of them, pretty amazing.

### **Looking for vulnerabilities**

Check for some [domain takeover](../../pentesting-web/domain-subdomain-takeover.md#domain-takeover). Maybe some company is **using some a domain** but they **lost the ownership**. Just register it (if cheap enough) and let know the company.

If you find any **domain with an IP different** from the ones you already found in the assets discovery, you should perform a **basic vulnerability scan** (using Nessus or OpenVAS) and some [**port scan**](../pentesting-network/#discovering-hosts-from-the-outside) with **nmap/masscan/shodan**. Depending on which services are running you can find in **this book some tricks to "attack" them**.\
&#xNAN;_&#x4E;ote that sometimes the domain is hosted inside an IP that is not controlled by the client, so it's not in the scope, be careful._

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**Bug bounty tip**: **sign up** for **Intigriti**, a premium **bug bounty platform created by hackers, for hackers**! Join us at [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) today, and start earning bounties up to **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

## Subdomains

> We know all the companies inside the scope, all the assets of each company and all the domains related to the companies.

It's time to find all the possible subdomains of each found domain.

{% hint style="success" %}
Note that some of the tools and techniques to find domains can also help to find subdomains!
{% endhint %}

### **DNS**

Let's try to get **subdomains** from the **DNS** records. We should also try for **Zone Transfer** (If vulnerable, you should report it).
```bash
dnsrecon -a -d tesla.com
```
### **OSINT**

बहुत सारे सबडोमेन प्राप्त करने का सबसे तेज़ तरीका बाहरी स्रोतों में खोज करना है। सबसे अधिक उपयोग किए जाने वाले **tools** निम्नलिखित हैं (बेहतर परिणामों के लिए API कुंजी कॉन्फ़िगर करें):

* [**BBOT**](https://github.com/blacklanternsecurity/bbot)
```bash
# subdomains
bbot -t tesla.com -f subdomain-enum

# subdomains (passive only)
bbot -t tesla.com -f subdomain-enum -rf passive

# subdomains + port scan + web screenshots
bbot -t tesla.com -f subdomain-enum -m naabu gowitness -n my_scan -o .
```
* [**Amass**](https://github.com/OWASP/Amass)
```bash
amass enum [-active] [-ip] -d tesla.com
amass enum -d tesla.com | grep tesla.com # To just list subdomains
```
* [**subfinder**](https://github.com/projectdiscovery/subfinder)
```bash
# Subfinder, use -silent to only have subdomains in the output
./subfinder-linux-amd64 -d tesla.com [-silent]
```
* [**findomain**](https://github.com/Edu4rdSHL/findomain/)
```bash
# findomain, use -silent to only have subdomains in the output
./findomain-linux -t tesla.com [--quiet]
```
* [**OneForAll**](https://github.com/shmilylty/OneForAll/tree/master/docs/en-us)
```bash
python3 oneforall.py --target tesla.com [--dns False] [--req False] [--brute False] run
```
* [**assetfinder**](https://github.com/tomnomnom/assetfinder)
```bash
assetfinder --subs-only <domain>
```
* [**Sudomy**](https://github.com/Screetsec/Sudomy)
```bash
# It requires that you create a sudomy.api file with API keys
sudomy -d tesla.com
```
* [**vita**](https://github.com/junnlikestea/vita)
```
vita -d tesla.com
```
* [**theHarvester**](https://github.com/laramies/theHarvester)
```bash
theHarvester -d tesla.com -b "anubis, baidu, bing, binaryedge, bingapi, bufferoverun, censys, certspotter, crtsh, dnsdumpster, duckduckgo, fullhunt, github-code, google, hackertarget, hunter, intelx, linkedin, linkedin_links, n45ht, omnisint, otx, pentesttools, projectdiscovery, qwant, rapiddns, rocketreach, securityTrails, spyse, sublist3r, threatcrowd, threatminer, trello, twitter, urlscan, virustotal, yahoo, zoomeye"
```
There are **अन्य दिलचस्प उपकरण/एपीआई** जो सीधे उपडोमेन खोजने में विशेषज्ञ नहीं होने के बावजूद उपडोमेन खोजने में उपयोगी हो सकते हैं, जैसे:

* [**Crobat**](https://github.com/cgboal/sonarsearch)**:** उपडोमेन प्राप्त करने के लिए एपीआई [https://sonar.omnisint.io](https://sonar.omnisint.io) का उपयोग करता है
```bash
# Get list of subdomains in output from the API
## This is the API the crobat tool will use
curl https://sonar.omnisint.io/subdomains/tesla.com | jq -r ".[]"
```
* [**JLDC मुफ्त API**](https://jldc.me/anubis/subdomains/google.com)
```bash
curl https://jldc.me/anubis/subdomains/tesla.com | jq -r ".[]"
```
* [**RapidDNS**](https://rapiddns.io) मुफ्त API
```bash
# Get Domains from rapiddns free API
rapiddns(){
curl -s "https://rapiddns.io/subdomain/$1?full=1" \
| grep -oE "[\.a-zA-Z0-9-]+\.$1" \
| sort -u
}
rapiddns tesla.com
```
* [**https://crt.sh/**](https://crt.sh)
```bash
# Get Domains from crt free API
crt(){
curl -s "https://crt.sh/?q=%25.$1" \
| grep -oE "[\.a-zA-Z0-9-]+\.$1" \
| sort -u
}
crt tesla.com
```
* [**gau**](https://github.com/lc/gau)**:** किसी भी दिए गए डोमेन के लिए AlienVault के Open Threat Exchange, Wayback Machine, और Common Crawl से ज्ञात URLs लाता है।
```bash
# Get subdomains from GAUs found URLs
gau --subs tesla.com | cut -d "/" -f 3 | sort -u
```
* [**SubDomainizer**](https://github.com/nsonaniya2010/SubDomainizer) **&** [**subscraper**](https://github.com/Cillian-Collins/subscraper): ये वेब को स्क्रैप करते हैं, JS फ़ाइलों की तलाश करते हैं और वहां से उपडोमेन निकालते हैं।
```bash
# Get only subdomains from SubDomainizer
python3 SubDomainizer.py -u https://tesla.com | grep tesla.com

# Get only subdomains from subscraper, this already perform recursion over the found results
python subscraper.py -u tesla.com | grep tesla.com | cut -d " " -f
```
* [**Shodan**](https://www.shodan.io/)
```bash
# Get info about the domain
shodan domain <domain>
# Get other pages with links to subdomains
shodan search "http.html:help.domain.com"
```
* [**Censys उपडोमेन खोजक**](https://github.com/christophetd/censys-subdomain-finder)
```bash
export CENSYS_API_ID=...
export CENSYS_API_SECRET=...
python3 censys-subdomain-finder.py tesla.com
```
* [**DomainTrail.py**](https://github.com/gatete/DomainTrail)
```bash
python3 DomainTrail.py -d example.com
```
* [**securitytrails.com**](https://securitytrails.com/) में उपडोमेन और IP इतिहास खोजने के लिए एक मुफ्त API है
* [**chaos.projectdiscovery.io**](https://chaos.projectdiscovery.io/#/)

यह प्रोजेक्ट **बग-बाउंटी कार्यक्रमों से संबंधित सभी उपडोमेन मुफ्त में** प्रदान करता है। आप इस डेटा को [chaospy](https://github.com/dr-0x0x/chaospy) का उपयोग करके भी एक्सेस कर सकते हैं या इस प्रोजेक्ट द्वारा उपयोग किए गए दायरे को भी एक्सेस कर सकते हैं [https://github.com/projectdiscovery/chaos-public-program-list](https://github.com/projectdiscovery/chaos-public-program-list)

आप यहाँ इन उपकरणों की **तुलना** पा सकते हैं: [https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off](https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off)

### **DNS ब्रूट फोर्स**

आइए संभावित उपडोमेन नामों का उपयोग करके DNS सर्वरों को ब्रूट-फोर्स करके नए **उपडोमेन** खोजने की कोशिश करें।

इस क्रिया के लिए आपको कुछ **सामान्य उपडोमेन शब्दसूचियाँ जैसे** चाहिए:

* [https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056](https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056)
* [https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt](https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt)
* [https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip](https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip)
* [https://github.com/pentester-io/commonspeak](https://github.com/pentester-io/commonspeak)
* [https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)

और अच्छे DNS रिसोल्वर्स के IPs भी। विश्वसनीय DNS रिसोल्वर्स की सूची बनाने के लिए आप [https://public-dns.info/nameservers-all.txt](https://public-dns.info/nameservers-all.txt) से रिसोल्वर्स डाउनलोड कर सकते हैं और उन्हें फ़िल्टर करने के लिए [**dnsvalidator**](https://github.com/vortexau/dnsvalidator) का उपयोग कर सकते हैं। या आप उपयोग कर सकते हैं: [https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt](https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt)

DNS ब्रूट-फोर्स के लिए सबसे अनुशंसित उपकरण हैं:

* [**massdns**](https://github.com/blechschmidt/massdns): यह पहला उपकरण था जिसने प्रभावी DNS ब्रूट-फोर्स किया। यह बहुत तेज है हालांकि यह गलत सकारात्मक के प्रति संवेदनशील है।
```bash
sed 's/$/.domain.com/' subdomains.txt > bf-subdomains.txt
./massdns -r resolvers.txt -w /tmp/results.txt bf-subdomains.txt
grep -E "tesla.com. [0-9]+ IN A .+" /tmp/results.txt
```
* [**gobuster**](https://github.com/OJ/gobuster): मुझे लगता है कि यह केवल 1 रिसोल्वर का उपयोग करता है
```
gobuster dns -d mysite.com -t 50 -w subdomains.txt
```
* [**shuffledns**](https://github.com/projectdiscovery/shuffledns) एक `massdns` के चारों ओर एक wrapper है, जो गो में लिखा गया है, जो आपको सक्रिय ब्रूटफोर्स का उपयोग करके मान्य उपडोमेन की गणना करने की अनुमति देता है, साथ ही वाइल्डकार्ड हैंडलिंग और आसान इनपुट-आउटपुट समर्थन के साथ उपडोमेन को हल करता है।
```
shuffledns -d example.com -list example-subdomains.txt -r resolvers.txt
```
* [**puredns**](https://github.com/d3mondev/puredns): यह भी `massdns` का उपयोग करता है।
```
puredns bruteforce all.txt domain.com
```
* [**aiodnsbrute**](https://github.com/blark/aiodnsbrute) असिंक्रोनसली डोमेन नामों को ब्रूट फोर्स करने के लिए asyncio का उपयोग करता है।
```
aiodnsbrute -r resolvers -w wordlist.txt -vv -t 1024 domain.com
```
### Second DNS Brute-Force Round

खुले स्रोतों और ब्रूट-फोर्सिंग का उपयोग करके उपडोमेन खोजने के बाद, आप पाए गए उपडोमेन के परिवर्तनों को उत्पन्न कर सकते हैं ताकि और भी अधिक खोजने की कोशिश की जा सके। इस उद्देश्य के लिए कई उपकरण उपयोगी हैं:

* [**dnsgen**](https://github.com/ProjectAnte/dnsgen)**:** डोमेन और उपडोमेन दिए जाने पर permutations उत्पन्न करता है।
```bash
cat subdomains.txt | dnsgen -
```
* [**goaltdns**](https://github.com/subfinder/goaltdns): डोमेन और सबडोमेन दिए जाने पर उत्परिवर्तन उत्पन्न करें।
* आप goaltdns उत्परिवर्तन **शब्दसूची** [**यहां**](https://github.com/subfinder/goaltdns/blob/master/words.txt) प्राप्त कर सकते हैं।
```bash
goaltdns -l subdomains.txt -w /tmp/words-permutations.txt -o /tmp/final-words-s3.txt
```
* [**gotator**](https://github.com/Josue87/gotator)**:** दिए गए डोमेन और उपडोमेन के लिए संयोजन उत्पन्न करें। यदि संयोजन फ़ाइल निर्दिष्ट नहीं की गई है, तो gotator अपनी स्वयं की फ़ाइल का उपयोग करेगा।
```
gotator -sub subdomains.txt -silent [-perm /tmp/words-permutations.txt]
```
* [**altdns**](https://github.com/infosec-au/altdns): उपडोमेन संयोजनों को उत्पन्न करने के अलावा, यह उन्हें हल करने की भी कोशिश कर सकता है (लेकिन पहले टिप्पणी किए गए उपकरणों का उपयोग करना बेहतर है)।
* आप altdns संयोजनों की **शब्दसूची** [**यहाँ**](https://github.com/infosec-au/altdns/blob/master/words.txt) प्राप्त कर सकते हैं।
```
altdns -i subdomains.txt -w /tmp/words-permutations.txt -o /tmp/asd3
```
* [**dmut**](https://github.com/bp0lr/dmut): उपडोमेन के संयोजन, उत्परिवर्तन और परिवर्तन करने के लिए एक और उपकरण। यह उपकरण परिणाम को ब्रूट फोर्स करेगा (यह dns वाइल्ड कार्ड का समर्थन नहीं करता)।
* आप dmut संयोजन शब्द सूची [**यहाँ**](https://raw.githubusercontent.com/bp0lr/dmut/main/words.txt) प्राप्त कर सकते हैं।
```bash
cat subdomains.txt | dmut -d /tmp/words-permutations.txt -w 100 \
--dns-errorLimit 10 --use-pb --verbose -s /tmp/resolvers-trusted.txt
```
* [**alterx**](https://github.com/projectdiscovery/alterx)**:** एक डोमेन के आधार पर यह **संकेतित पैटर्न के आधार पर नए संभावित उपडोमेन नाम उत्पन्न करता है** ताकि अधिक उपडोमेन खोजने की कोशिश की जा सके।

#### स्मार्ट संयोजन उत्पन्न करना

* [**regulator**](https://github.com/cramppet/regulator): अधिक जानकारी के लिए इस [**पोस्ट**](https://cramppet.github.io/regulator/index.html) को पढ़ें लेकिन यह मूल रूप से **खोजे गए उपडोमेन** के **मुख्य भागों** को प्राप्त करेगा और अधिक उपडोमेन खोजने के लिए उन्हें मिलाएगा।
```bash
python3 main.py adobe.com adobe adobe.rules
make_brute_list.sh adobe.rules adobe.brute
puredns resolve adobe.brute --write adobe.valid
```
* [**subzuf**](https://github.com/elceef/subzuf)**:** _subzuf_ एक सबडोमेन ब्रूट-फोर्स फज़्ज़र है जो एक अत्यंत सरल लेकिन प्रभावी DNS प्रतिक्रिया-निर्देशित एल्गोरिदम के साथ जुड़ा हुआ है। यह एक प्रदान किए गए इनपुट डेटा सेट का उपयोग करता है, जैसे कि एक अनुकूलित शब्द सूची या ऐतिहासिक DNS/TLS रिकॉर्ड, ताकि अधिक संबंधित डोमेन नामों को सटीक रूप से संश्लेषित किया जा सके और DNS स्कैन के दौरान एकत्रित जानकारी के आधार पर उन्हें एक लूप में और भी आगे बढ़ाया जा सके।
```
echo www | subzuf facebook.com
```
### **सबडोमेन खोज कार्यप्रवाह**

इस ब्लॉग पोस्ट को देखें जो मैंने **Trickest कार्यप्रवाह** का उपयोग करके एक डोमेन से **सबडोमेन खोज को स्वचालित करने** के बारे में लिखा है ताकि मुझे अपने कंप्यूटर में कई उपकरणों को मैन्युअल रूप से लॉन्च करने की आवश्यकता न हो:

{% embed url="https://trickest.com/blog/full-subdomain-discovery-using-workflow/" %}

{% embed url="https://trickest.com/blog/full-subdomain-brute-force-discovery-using-workflow/" %}

### **VHosts / वर्चुअल होस्ट**

यदि आपने एक IP पता पाया है जिसमें **एक या एक से अधिक वेब पृष्ठ** सबडोमेन से संबंधित हैं, तो आप **उस IP में वेब के साथ अन्य सबडोमेन खोजने** की कोशिश कर सकते हैं **OSINT स्रोतों** में IP में डोमेन देखकर या **उस IP में VHost डोमेन नामों को ब्रूट-फोर्स करके**।

#### OSINT

आप कुछ **VHosts को IP में खोज सकते हैं** [**HostHunter**](https://github.com/SpiderLabs/HostHunter) **या अन्य APIs का उपयोग करके**।

**ब्रूट फोर्स**

यदि आपको संदेह है कि कुछ सबडोमेन एक वेब सर्वर में छिपा हो सकता है, तो आप इसे ब्रूट फोर्स करने की कोशिश कर सकते हैं:
```bash
ffuf -c -w /path/to/wordlist -u http://victim.com -H "Host: FUZZ.victim.com"

gobuster vhost -u https://mysite.com -t 50 -w subdomains.txt

wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --hc 400,404,403 -H "Host: FUZZ.example.com" -u http://example.com -t 100

#From https://github.com/allyshka/vhostbrute
vhostbrute.py --url="example.com" --remoteip="10.1.1.15" --base="www.example.com" --vhosts="vhosts_full.list"

#https://github.com/codingo/VHostScan
VHostScan -t example.com
```
{% hint style="info" %}
इस तकनीक के साथ आप आंतरिक/छिपे हुए एंडपॉइंट्स तक भी पहुँच सकते हैं।
{% endhint %}

### **CORS Brute Force**

कभी-कभी आप ऐसी पृष्ठों को पाएंगे जो केवल _**Access-Control-Allow-Origin**_ हेडर को लौटाती हैं जब _**Origin**_ हेडर में एक मान्य डोमेन/सबडोमेन सेट किया गया हो। इन परिदृश्यों में, आप इस व्यवहार का दुरुपयोग करके **नए** **सबडोमेन** **खोज** सकते हैं।
```bash
ffuf -w subdomains-top1million-5000.txt -u http://10.10.10.208 -H 'Origin: http://FUZZ.crossfit.htb' -mr "Access-Control-Allow-Origin" -ignore-body
```
### **Buckets Brute Force**

जब **subdomains** की खोज कर रहे हों, तो देखें कि क्या यह किसी प्रकार के **bucket** की ओर **pointing** कर रहा है, और इस मामले में [**permissions की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/)**.**\
इसके अलावा, चूंकि इस बिंदु पर आप दायरे के भीतर सभी डोमेन को जानेंगे, कोशिश करें [**संभावित bucket नामों को brute force करने और permissions की जांच करने के लिए**](../../network-services-pentesting/pentesting-web/buckets/).

### **Monitorization**

आप **Certificate Transparency** Logs की निगरानी करके देख सकते हैं कि किसी डोमेन के **नए subdomains** बनाए गए हैं, [**sublert** ](https://github.com/yassineaboukir/sublert/blob/master/sublert.py) ऐसा करता है।

### **Looking for vulnerabilities**

संभावित [**subdomain takeovers**](../../pentesting-web/domain-subdomain-takeover.md#subdomain-takeover) के लिए जांचें।\
यदि **subdomain** किसी **S3 bucket** की ओर **pointing** कर रहा है, तो [**permissions की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/).

यदि आप किसी **subdomain को एक IP के साथ पाते हैं जो पहले से आपके द्वारा खोजे गए IPs से अलग है**, तो आपको एक **बुनियादी vulnerability scan** (Nessus या OpenVAS का उपयोग करके) और कुछ [**port scan**](../pentesting-network/#discovering-hosts-from-the-outside) **nmap/masscan/shodan** के साथ करना चाहिए। चल रहे सेवाओं के आधार पर, आप **इस पुस्तक में कुछ तरकीबें "हमला" करने के लिए** पा सकते हैं।\
&#xNAN;_&#x4E;ote करें कि कभी-कभी subdomain एक IP के भीतर होस्ट किया जाता है जो ग्राहक द्वारा नियंत्रित नहीं होता है, इसलिए यह दायरे में नहीं है, सावधान रहें।_

## IPs

प्रारंभिक चरणों में, आपने **कुछ IP रेंज, डोमेन और subdomains** पाए हो सकते हैं।\
अब **उन रेंज से सभी IPs को इकट्ठा करने** और **डोमेन/subdomains (DNS queries)** के लिए समय है।

निम्नलिखित **free apis** की सेवाओं का उपयोग करके, आप **डोमेन और subdomains द्वारा उपयोग किए गए पिछले IPs** भी पा सकते हैं। ये IPs अभी भी ग्राहक के स्वामित्व में हो सकते हैं (और आपको [**CloudFlare bypasses**](../../network-services-pentesting/pentesting-web/uncovering-cloudflare.md) खोजने की अनुमति दे सकते हैं)

* [**https://securitytrails.com/**](https://securitytrails.com/)

आप [**hakip2host**](https://github.com/hakluke/hakip2host) टूल का उपयोग करके एक विशिष्ट IP पते की ओर इशारा करने वाले डोमेन की भी जांच कर सकते हैं।

### **Looking for vulnerabilities**

**CDNs से संबंधित सभी IPs का port scan करें** (क्योंकि आप वहां कुछ दिलचस्प नहीं पाएंगे)। खोजे गए चल रहे सेवाओं में आप **vulnerabilities** पा सकते हैं।

**होस्ट स्कैन करने के लिए एक** [**गाइड**](../pentesting-network/) **खोजें।**

## Web servers hunting

> हमने सभी कंपनियों और उनके संपत्तियों को खोज लिया है और हम दायरे के भीतर IP रेंज, डोमेन और subdomains को जानते हैं। अब वेब सर्वरों की खोज करने का समय है।

पिछले चरणों में, आपने शायद पहले से ही खोजे गए IPs और डोमेन का कुछ **recon** किया है, इसलिए आप **संभावित सभी वेब सर्वरों** को पहले से ही पा चुके होंगे। हालाँकि, यदि आपने नहीं किया है, तो हम अब दायरे के भीतर वेब सर्वरों की खोज के लिए कुछ **तेज़ तरकीबें** देखेंगे।

कृपया ध्यान दें कि यह **वेब ऐप्स की खोज के लिए उन्मुख** होगा, इसलिए आपको **vulnerability** और **port scanning** भी करनी चाहिए (**यदि दायरे द्वारा अनुमति दी गई हो**).

**वेब** सर्वरों से संबंधित **खुले ports** की खोज करने के लिए एक **तेज़ विधि** [**masscan** का उपयोग करके यहाँ पाई जा सकती है](../pentesting-network/#http-port-discovery).\
वेब सर्वरों की खोज के लिए एक और उपयोगकर्ता-अनुकूल उपकरण [**httprobe**](https://github.com/tomnomnom/httprobe)**,** [**fprobe**](https://github.com/theblackturtle/fprobe) और [**httpx**](https://github.com/projectdiscovery/httpx) है। आप बस डोमेन की एक सूची पास करते हैं और यह पोर्ट 80 (http) और 443 (https) से कनेक्ट करने की कोशिश करेगा। इसके अतिरिक्त, आप अन्य पोर्ट की कोशिश करने के लिए संकेत दे सकते हैं:
```bash
cat /tmp/domains.txt | httprobe #Test all domains inside the file for port 80 and 443
cat /tmp/domains.txt | httprobe -p http:8080 -p https:8443 #Check port 80, 443 and 8080 and 8443
```
### **Screenshots**

अब जब आपने **सभी वेब सर्वर** खोज लिए हैं जो दायरे में हैं (कंपनी के **IPs** और सभी **डोमेन** और **सबडोमेन** के बीच) तो शायद आप **शुरुआत कहाँ से करें** यह नहीं जानते। तो, इसे सरल बनाते हैं और बस सभी का स्क्रीनशॉट लेना शुरू करते हैं। बस **मुख्य पृष्ठ** पर **नज़र डालकर** आप **अजीब** एंडपॉइंट्स पा सकते हैं जो अधिक **संवेदनशील** हो सकते हैं।

प्रस्तावित विचार को लागू करने के लिए आप [**EyeWitness**](https://github.com/FortyNorthSecurity/EyeWitness), [**HttpScreenshot**](https://github.com/breenmachine/httpscreenshot), [**Aquatone**](https://github.com/michenriksen/aquatone), [**Shutter**](https://shutter-project.org/downloads/third-party-packages/), [**Gowitness**](https://github.com/sensepost/gowitness) या [**webscreenshot**](https://github.com/maaaaz/webscreenshot)** का उपयोग कर सकते हैं।**

इसके अलावा, आप फिर [**eyeballer**](https://github.com/BishopFox/eyeballer) का उपयोग कर सकते हैं ताकि सभी **स्क्रीनशॉट्स** पर चलाकर आपको बता सके कि **क्या संभावित रूप से कमजोरियों को शामिल कर सकता है**, और क्या नहीं।

## सार्वजनिक क्लाउड संपत्तियाँ

किसी कंपनी की संभावित क्लाउड संपत्तियों को खोजने के लिए आपको **उस कंपनी की पहचान करने वाले कीवर्ड की एक सूची से शुरू करना चाहिए**। उदाहरण के लिए, एक क्रिप्टो कंपनी के लिए आप शब्दों का उपयोग कर सकते हैं: `"crypto", "wallet", "dao", "<domain_name>", <"subdomain_names">`।

आपको **बकेट्स में उपयोग किए जाने वाले सामान्य शब्दों** की वर्डलिस्ट भी चाहिए:

* [https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt](https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt)
* [https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt](https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt)
* [https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt](https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt)

फिर, उन शब्दों के साथ आपको **परम्यूटेशन** उत्पन्न करनी चाहिए (अधिक जानकारी के लिए [**दूसरे दौर के DNS ब्रूट-फोर्स**](./#second-dns-bruteforce-round) की जांच करें)।

परिणामी वर्डलिस्ट के साथ आप [**cloud\_enum**](https://github.com/initstring/cloud_enum)**,** [**CloudScraper**](https://github.com/jordanpotti/CloudScraper)**,** [**cloudlist**](https://github.com/projectdiscovery/cloudlist) **या** [**S3Scanner**](https://github.com/sa7mon/S3Scanner)** का उपयोग कर सकते हैं।**

याद रखें कि जब आप क्लाउड संपत्तियों की खोज कर रहे हों तो आपको **AWS में बकेट्स से अधिक की तलाश करनी चाहिए**।

### **कमजोरियों की तलाश**

यदि आप **खुले बकेट्स या क्लाउड फ़ंक्शंस** खोजते हैं तो आपको **उन तक पहुँच बनानी चाहिए** और देखना चाहिए कि वे आपको क्या प्रदान करते हैं और क्या आप उनका दुरुपयोग कर सकते हैं।

## ईमेल

दायरे में **डोमेन** और **सबडोमेन** के साथ आपके पास **ईमेल खोजने के लिए आवश्यक सभी चीजें** हैं। ये हैं **APIs** और **उपकरण** जो मुझे किसी कंपनी के ईमेल खोजने में सबसे अच्छे लगे हैं:

* [**theHarvester**](https://github.com/laramies/theHarvester) - APIs के साथ
* [**https://hunter.io/**](https://hunter.io/) का API (फ्री संस्करण)
* [**https://app.snov.io/**](https://app.snov.io/) का API (फ्री संस्करण)
* [**https://minelead.io/**](https://minelead.io/) का API (फ्री संस्करण)

### **कमजोरियों की तलाश**

ईमेल बाद में **वेब लॉगिन और ऑथ सेवाओं** (जैसे SSH) के लिए **ब्रूट-फोर्स** करने में सहायक होंगे। इसके अलावा, ये **फिशिंग** के लिए आवश्यक हैं। इसके अलावा, ये APIs आपको ईमेल के पीछे के व्यक्ति के बारे में और भी अधिक **जानकारी** देंगी, जो फिशिंग अभियान के लिए उपयोगी है।

## क्रेडेंशियल लीक

**डोमेन,** **सबडोमेन**, और **ईमेल** के साथ आप उन ईमेल से संबंधित अतीत में लीक हुए क्रेडेंशियल्स की तलाश शुरू कर सकते हैं:

* [https://leak-lookup.com](https://leak-lookup.com/account/login)
* [https://www.dehashed.com/](https://www.dehashed.com/)

### **कमजोरियों की तलाश**

यदि आप **मान्य लीक** हुए क्रेडेंशियल्स पाते हैं, तो यह एक बहुत आसान जीत है।

## सीक्रेट्स लीक

क्रेडेंशियल लीक उन कंपनियों के हैक से संबंधित हैं जहाँ **संवेदनशील जानकारी लीक और बेची गई**। हालाँकि, कंपनियाँ **अन्य लीक** से प्रभावित हो सकती हैं जिनकी जानकारी उन डेटाबेस में नहीं है:

### गिटहब लीक

क्रेडेंशियल्स और APIs **कंपनी** या उस गिटहब कंपनी में काम करने वाले **उपयोगकर्ताओं** के **सार्वजनिक रिपॉजिटरी** में लीक हो सकते हैं।\
आप **Leakos** नामक **उपकरण** का उपयोग करके किसी **संगठन** और उसके **डेवलपर्स** के सभी **सार्वजनिक रिपॉजिटरी** को **डाउनलोड** कर सकते हैं और उन पर स्वचालित रूप से [**gitleaks**](https://github.com/zricethezav/gitleaks) चला सकते हैं।

**Leakos** का उपयोग सभी **पाठ** प्रदान किए गए **URLs** पर **gitleaks** चलाने के लिए भी किया जा सकता है क्योंकि कभी-कभी **वेब पृष्ठों में भी रहस्य होते हैं**।

#### गिटहब डॉर्क्स

आप जिस संगठन पर हमला कर रहे हैं, उसमें संभावित **गिटहब डॉर्क्स** के लिए इस **पृष्ठ** की भी जांच करें:

{% content-ref url="github-leaked-secrets.md" %}
[github-leaked-secrets.md](github-leaked-secrets.md)
{% endcontent-ref %}

### पेस्ट लीक

कभी-कभी हमलावर या बस कर्मचारी **कंपनी की सामग्री को एक पेस्ट साइट पर प्रकाशित करेंगे**। इसमें **संवेदनशील जानकारी** हो सकती है या नहीं, लेकिन इसे खोजना बहुत दिलचस्प है।\
आप **Pastos** नामक उपकरण का उपयोग करके एक साथ 80 से अधिक पेस्ट साइटों में खोज कर सकते हैं।

### गूगल डॉर्क्स

पुराने लेकिन सुनहरे गूगल डॉर्क्स हमेशा **वहां नहीं होनी चाहिए ऐसी उजागर जानकारी** खोजने के लिए उपयोगी होते हैं। एकमात्र समस्या यह है कि [**google-hacking-database**](https://www.exploit-db.com/google-hacking-database) में कई **हजारों** संभावित क्वेरी हैं जिन्हें आप मैन्युअल रूप से नहीं चला सकते। इसलिए, आप अपने पसंदीदा 10 को ले सकते हैं या आप **Gorks** जैसे **उपकरण** का उपयोग कर सकते हैं **उन्हें सभी चलाने के लिए**।

_ध्यान दें कि जो उपकरण नियमित Google ब्राउज़र का उपयोग करके सभी डेटाबेस को चलाने की उम्मीद करते हैं, वे कभी समाप्त नहीं होंगे क्योंकि Google आपको बहुत जल्दी ब्लॉक कर देगा।_

### **कमजोरियों की तलाश**

यदि आप **मान्य लीक** हुए क्रेडेंशियल्स या API टोकन पाते हैं, तो यह एक बहुत आसान जीत है।

## सार्वजनिक कोड कमजोरियाँ

यदि आपने पाया कि कंपनी का **ओपन-सोर्स कोड** है, तो आप इसे **विश्लेषण** कर सकते हैं और इसमें **कमजोरियों** की तलाश कर सकते हैं।

**भाषा के आधार पर** आपके पास विभिन्न **उपकरण** हो सकते हैं:

{% content-ref url="../../network-services-pentesting/pentesting-web/code-review-tools.md" %}
[code-review-tools.md](../../network-services-pentesting/pentesting-web/code-review-tools.md)
{% endcontent-ref %}

सार्वजनिक रिपॉजिटरी को **स्कैन** करने के लिए भी मुफ्त सेवाएँ हैं, जैसे:

* [**Snyk**](https://app.snyk.io/)

## [**पेंटेस्टिंग वेब मेथोडोलॉजी**](../../network-services-pentesting/pentesting-web/)

**कमजोरियों** की **अधिकांशता** जो बग हंटर्स द्वारा पाई जाती है, **वेब एप्लिकेशनों** के अंदर होती है, इसलिए इस बिंदु पर मैं एक **वेब एप्लिकेशन परीक्षण पद्धति** के बारे में बात करना चाहता हूँ, और आप [**यहाँ इस जानकारी को पा सकते हैं**](../../network-services-pentesting/pentesting-web/)।

मैं [**वेब स्वचालित स्कैनर्स ओपन-सोर्स टूल्स**](../../network-services-pentesting/pentesting-web/#automatic-scanners) अनुभाग का विशेष उल्लेख करना चाहता हूँ, क्योंकि, यदि आप उनसे बहुत संवेदनशील कमजोरियों की खोज करने की उम्मीद नहीं करते हैं, तो वे **प्रारंभिक वेब जानकारी प्राप्त करने के लिए कार्यप्रवाहों में लागू करने के लिए सहायक होते हैं।**

## पुनरावलोकन

> बधाई हो! इस बिंदु पर आपने पहले ही **सभी बुनियादी गणना** कर ली है। हाँ, यह बुनियादी है क्योंकि और भी बहुत अधिक गणना की जा सकती है (बाद में और तरकीबें देखेंगे)।

तो आपने पहले ही:

1. दायरे में सभी **कंपनियों** को खोज लिया
2. कंपनियों से संबंधित सभी **संपत्तियों** को खोज लिया (और यदि दायरे में हो तो कुछ कमजोरियों का स्कैन किया)
3. कंपनियों से संबंधित सभी **डोमेन** को खोज लिया
4. डोमेन के सभी **सबडोमेन** को खोज लिया (क्या कोई सबडोमेन टेकओवर?)
5. दायरे में सभी **IPs** (CDNs से और **नहीं**) को खोज लिया।
6. सभी **वेब सर्वर** को खोज लिया और उनका **स्क्रीनशॉट** लिया (क्या कुछ अजीब है जो गहराई से देखने लायक है?)
7. कंपनी से संबंधित सभी **संभावित सार्वजनिक क्लाउड संपत्तियों** को खोज लिया।
8. **ईमेल**, **क्रेडेंशियल लीक**, और **सीक्रेट लीक** जो आपको **बहुत आसानी से एक बड़ा लाभ** दे सकते हैं।
9. आपने जो भी वेब खोजी है, उसका **पेंटेस्टिंग** किया।

## **पूर्ण पुनः खोज स्वचालित उपकरण**

कुछ उपकरण हैं जो दिए गए दायरे के खिलाफ प्रस्तावित कार्यों का एक भाग करेंगे।

* [**https://github.com/yogeshojha/rengine**](https://github.com/yogeshojha/rengine)
* [**https://github.com/j3ssie/Osmedeus**](https://github.com/j3ssie/Osmedeus)
* [**https://github.com/six2dez/reconftw**](https://github.com/six2dez/reconftw)
* [**https://github.com/hackerspider1/EchoPwn**](https://github.com/hackerspider1/EchoPwn) - थोड़ा पुराना और अपडेट नहीं किया गया

## **संदर्भ**

* सभी मुफ्त पाठ्यक्रम [**@Jhaddix**](https://twitter.com/Jhaddix) जैसे [**The Bug Hunter's Methodology v4.0 - Recon Edition**](https://www.youtube.com/watch?v=p4JgIu1mceI)

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अचूक को हैक करना चाहते हैं - **हम भर्ती कर रहे हैं!** (_फ्लूएंट पोलिश लिखित और मौखिक आवश्यक है_)।

{% embed url="https://www.stmcyber.com/careers" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **हमें ट्विटर पर फॉलो करें** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपॉजिटरी में PR सबमिट करें।

</details>
{% endhint %}
