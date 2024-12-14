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


## Misingi ya Mifumo

- **Mikataba ya Kijanja** inafafanuliwa kama programu zinazotekelezwa kwenye blockchain wakati masharti fulani yanapotimizwa, zikifanya utekelezaji wa makubaliano bila wahusika wa kati.
- **Programu za Kijamii (dApps)** zinajengwa juu ya mikataba ya kijanja, zikiwa na muonekano wa kirafiki kwa mtumiaji na nyuma inayoweza kukaguliwa kwa uwazi.
- **Tokeni & Sarafu** zinatofautisha ambapo sarafu hutumikia kama pesa za kidijitali, wakati tokeni zinawakilisha thamani au umiliki katika muktadha maalum.
- **Tokeni za Huduma** zinatoa ufikiaji wa huduma, na **Tokeni za Usalama** zinamaanisha umiliki wa mali.
- **DeFi** inasimama kwa Fedha za Kijamii, ikitoa huduma za kifedha bila mamlaka za kati.
- **DEX** na **DAOs** zinarejelea Jukwaa za Kubadilishana za Kijamii na Mashirika ya Kijamii ya Kiotomatiki, mtawalia.

## Mechanisms ya Makubaliano

Mekaniki za makubaliano zinahakikisha uthibitisho salama na wa makubaliano wa muamala kwenye blockchain:
- **Uthibitisho wa Kazi (PoW)** unategemea nguvu za kompyuta kwa uthibitisho wa muamala.
- **Uthibitisho wa Hisa (PoS)** unahitaji wahakiki kuwa na kiasi fulani cha tokeni, kupunguza matumizi ya nishati ikilinganishwa na PoW.

## Msingi wa Bitcoin

### Muamala

Muamala wa Bitcoin unahusisha kuhamasisha fedha kati ya anwani. Muamala unathibitishwa kupitia saini za kidijitali, kuhakikisha ni mmiliki pekee wa funguo za faragha anayeweza kuanzisha uhamasishaji.

#### Vipengele Muhimu:

- **Muamala wa Saini nyingi** unahitaji saini nyingi ili kuidhinisha muamala.
- Muamala unajumuisha **michango** (chanzo cha fedha), **matokeo** (kikundi), **malipo** (yanayolipwa kwa wachimbaji), na **scripts** (sheria za muamala).

### Mtandao wa Mwanga

Unalenga kuboresha uwezo wa Bitcoin kwa kuruhusu muamala mwingi ndani ya channel, ukitangaza tu hali ya mwisho kwenye blockchain.

## Wasiwasi wa Faragha wa Bitcoin

Mashambulizi ya faragha, kama vile **Kuwamiliki Kawaida kwa Michango** na **Utambuzi wa Anwani ya Mabadiliko ya UTXO**, yanatumia mifumo ya muamala. Mikakati kama **Mixers** na **CoinJoin** inaboresha kutotambulika kwa kuficha viungo vya muamala kati ya watumiaji.

## Kupata Bitcoins kwa Siri

Njia ni pamoja na biashara za pesa taslimu, uchimbaji, na kutumia mixers. **CoinJoin** inachanganya muamala mingi ili kuleta ugumu katika kufuatilia, wakati **PayJoin** inaficha CoinJoins kama muamala wa kawaida kwa ajili ya faragha zaidi.


# Mashambulizi ya Faragha ya Bitcoin

# Muhtasari wa Mashambulizi ya Faragha ya Bitcoin

Katika ulimwengu wa Bitcoin, faragha ya muamala na kutotambulika kwa watumiaji mara nyingi ni masuala ya wasiwasi. Hapa kuna muonekano rahisi wa mbinu kadhaa za kawaida ambazo washambuliaji wanaweza kuathiri faragha ya Bitcoin.

## **Dhana ya Kuwamiliki Kawaida kwa Michango**

Kwa kawaida ni nadra kwa michango kutoka kwa watumiaji tofauti kuunganishwa katika muamala mmoja kutokana na ugumu uliohusika. Hivyo, **anwani mbili za michango katika muamala mmoja mara nyingi zinadhaniwa kuwa za mmiliki mmoja**.

## **Utambuzi wa Anwani ya Mabadiliko ya UTXO**

UTXO, au **Matokeo ya Muamala Yasiyotumika**, lazima itumike kabisa katika muamala. Ikiwa sehemu tu ya hiyo inatumwa kwa anwani nyingine, iliyobaki inaenda kwa anwani mpya ya mabadiliko. Waangalizi wanaweza kudhani anwani hii mpya inamhusu mtumaji, ikihatarisha faragha.

### Mfano
Ili kupunguza hili, huduma za kuchanganya au kutumia anwani nyingi zinaweza kusaidia kuficha umiliki.

## **Ufunuo wa Mitandao ya Kijamii na Mijadala**

Watumiaji wakati mwingine hushiriki anwani zao za Bitcoin mtandaoni, na kufanya **rahisi kuunganisha anwani hiyo na mmiliki wake**.

## **Analizi ya Grafu za Muamala**

Miamala inaweza kuonyeshwa kama grafu, ikifunua uhusiano wa uwezekano kati ya watumiaji kulingana na mtiririko wa fedha.

## **Heuristics ya Michango Isiyo ya Lazima (Heuristics ya Mabadiliko Bora)**

Heuristics hii inategemea kuchambua miamala yenye michango na matokeo mengi ili kudhani ni ipi kati ya matokeo ni mabadiliko yanayorejea kwa mtumaji.

### Mfano
```bash
2 btc --> 4 btc
3 btc     1 btc
```
If adding more inputs makes the change output larger than any single input, it can confuse the heuristic.

## **Forced Address Reuse**

Washambuliaji wanaweza kutuma kiasi kidogo kwa anwani zilizotumika awali, wakitumai mpokeaji atachanganya hizi na ingizo zingine katika miamala ya baadaye, hivyo kuunganisha anwani pamoja.

### Correct Wallet Behavior
Mifuko inapaswa kuepuka kutumia sarafu zilizopokelewa kwenye anwani ambazo tayari zimetumika, ili kuzuia uvujaji huu wa faragha.

## **Other Blockchain Analysis Techniques**

- **Exact Payment Amounts:** Miamala bila mabadiliko yanaweza kuwa kati ya anwani mbili zinazomilikiwa na mtumiaji mmoja.
- **Round Numbers:** Nambari ya duara katika muamala inaashiria kuwa ni malipo, huku pato lisilo la duara likiwa mabadiliko.
- **Wallet Fingerprinting:** Mifuko tofauti ina mifumo ya kipekee ya kuunda miamala, ikiruhusu wachambuzi kutambua programu iliyotumika na labda anwani ya mabadiliko.
- **Amount & Timing Correlations:** Kufichua nyakati za muamala au kiasi kunaweza kufanya miamala iweze kufuatiliwa.

## **Traffic Analysis**

Kwa kufuatilia trafiki ya mtandao, washambuliaji wanaweza kuunganisha miamala au vizuizi na anwani za IP, wakihatarisha faragha ya mtumiaji. Hii ni kweli hasa ikiwa shirika linaendesha nodi nyingi za Bitcoin, ikiongeza uwezo wao wa kufuatilia miamala.

## More
Kwa orodha kamili ya mashambulizi ya faragha na ulinzi, tembelea [Bitcoin Privacy on Bitcoin Wiki](https://en.bitcoin.it/wiki/Privacy).

# Anonymous Bitcoin Transactions

## Ways to Get Bitcoins Anonymously

- **Cash Transactions**: Kupata bitcoin kupitia pesa taslimu.
- **Cash Alternatives**: Kununua kadi za zawadi na kuzibadilisha mtandaoni kwa bitcoin.
- **Mining**: Njia ya faragha zaidi ya kupata bitcoins ni kupitia uchimbaji, hasa inapofanywa peke yake kwa sababu makundi ya uchimbaji yanaweza kujua anwani ya IP ya mchimbaji. [Mining Pools Information](https://en.bitcoin.it/wiki/Pooled_mining)
- **Theft**: Kimsingi, kuiba bitcoin kunaweza kuwa njia nyingine ya kuipata kwa siri, ingawa ni haramu na haitashauriwa.

## Mixing Services

Kwa kutumia huduma ya kuchanganya, mtumiaji anaweza **kutuma bitcoins** na kupokea **bitcoins tofauti kwa kurudi**, ambayo inafanya kufuatilia mmiliki wa awali kuwa ngumu. Hata hivyo, hii inahitaji kuaminiana na huduma hiyo isihifadhi kumbukumbu na kurudisha bitcoins kama ilivyokusudiwa. Chaguzi mbadala za kuchanganya ni pamoja na kasino za Bitcoin.

## CoinJoin

**CoinJoin** inachanganya miamala kadhaa kutoka kwa watumiaji tofauti kuwa moja, ikifanya mchakato kuwa mgumu kwa yeyote anayejaribu kulinganisha ingizo na pato. Licha ya ufanisi wake, miamala yenye saizi za kipekee za ingizo na pato bado inaweza kufuatiliwa.

Mifano ya miamala ambayo inaweza kuwa imetumia CoinJoin ni `402d3e1df685d1fdf82f36b220079c1bf44db227df2d676625ebcbee3f6cb22a` na `85378815f6ee170aa8c26694ee2df42b99cff7fa9357f073c1192fff1f540238`.

Kwa maelezo zaidi, tembelea [CoinJoin](https://coinjoin.io/en). Kwa huduma sawa kwenye Ethereum, angalia [Tornado Cash](https://tornado.cash), ambayo inafanya miamala kuwa ya siri kwa fedha kutoka kwa wachimbaji.

## PayJoin

Tofauti ya CoinJoin, **PayJoin** (au P2EP), inaficha muamala kati ya pande mbili (mfano, mteja na mfanyabiashara) kama muamala wa kawaida, bila sifa ya pato sawa ya CoinJoin. Hii inafanya iwe ngumu sana kugundua na inaweza kubatilisha heuristics ya umiliki wa ingizo la kawaida inayotumiwa na mashirika ya ufuatiliaji wa muamala.
```plaintext
2 btc --> 3 btc
5 btc     4 btc
```
Transactions kama hizo zinaweza kuwa PayJoin, zikiongeza faragha wakati zinabaki zisizoweza kutofautishwa na muamala wa kawaida wa bitcoin.

**Matumizi ya PayJoin yanaweza kuvuruga kwa kiasi kikubwa mbinu za ufuatiliaji wa jadi**, na kuifanya kuwa maendeleo ya ahadi katika juhudi za faragha ya muamala.


# Mbinu Bora za Faragha katika Cryptocurrencies

## **Mbinu za Usawazishaji wa Wallet**

Ili kudumisha faragha na usalama, kusawazisha wallets na blockchain ni muhimu. Mbinu mbili zinajitokeza:

- **Node kamili**: Kwa kupakua blockchain yote, node kamili inahakikisha faragha ya juu. Miamala yote ambayo imewahi kufanywa inahifadhiwa kwa ndani, na kufanya iwe vigumu kwa maadui kubaini ni miamala au anwani zipi ambazo mtumiaji anavutiwa nazo.
- **Filtering ya block upande wa mteja**: Mbinu hii inahusisha kuunda filters kwa kila block katika blockchain, ikiruhusu wallets kubaini miamala muhimu bila kufichua maslahi maalum kwa waangalizi wa mtandao. Wallets nyepesi hupakua filters hizi, zikichukua blocks kamili tu wakati kuna mechi na anwani za mtumiaji.

## **Kutumia Tor kwa Anonymity**

Kwa kuwa Bitcoin inafanya kazi kwenye mtandao wa mtu kwa mtu, kutumia Tor inapendekezwa ili kuficha anwani yako ya IP, ikiongeza faragha unaposhirikiana na mtandao.

## **Kuzuia Utumiaji wa Anwani Tena**

Ili kulinda faragha, ni muhimu kutumia anwani mpya kwa kila muamala. Kutumia tena anwani kunaweza kuhatarisha faragha kwa kuunganisha miamala na kitengo kimoja. Wallets za kisasa zinakataza matumizi ya anwani tena kupitia muundo wao.

## **Mikakati ya Faragha ya Muamala**

- **Miamala mingi**: Kugawanya malipo katika miamala kadhaa kunaweza kuficha kiasi cha muamala, kukatisha mashambulizi ya faragha.
- **Kuepuka mabadiliko**: Kuchagua miamala ambayo haitahitaji matokeo ya mabadiliko kunaongeza faragha kwa kuvuruga mbinu za kugundua mabadiliko.
- **Matokeo mengi ya mabadiliko**: Ikiwa kuepuka mabadiliko si rahisi, kuunda matokeo mengi ya mabadiliko bado kunaweza kuboresha faragha.

# **Monero: Mwanga wa Anonymity**

Monero inakidhi hitaji la anonymity kamili katika miamala ya kidijitali, ikipanga kiwango cha juu cha faragha.

# **Ethereum: Gesi na Miamala**

## **Kuelewa Gesi**

Gesi hupima juhudi za kompyuta zinazohitajika kutekeleza operesheni kwenye Ethereum, ikipimwa kwa **gwei**. Kwa mfano, muamala unaogharimu 2,310,000 gwei (au 0.00231 ETH) unahusisha kikomo cha gesi na ada ya msingi, pamoja na tip ili kuhamasisha wachimbaji. Watumiaji wanaweza kuweka ada ya juu ili kuhakikisha hawalipi zaidi, na ziada inarejeshwa.

## **Kutekeleza Miamala**

Miamala katika Ethereum yanahusisha mtumaji na mpokeaji, ambao wanaweza kuwa anwani za mtumiaji au mkataba smart. Yanahitaji ada na lazima yachimbwe. Taarifa muhimu katika muamala inajumuisha mpokeaji, sahihi ya mtumaji, thamani, data ya hiari, kikomo cha gesi, na ada. Kwa kuzingatia, anwani ya mtumaji inapatikana kutoka kwa sahihi, ikiondoa hitaji lake katika data ya muamala.

Mbinu na mifumo hii ni msingi kwa yeyote anayetaka kushiriki na cryptocurrencies huku akipa kipaumbele faragha na usalama.


## Marejeo

* [https://en.wikipedia.org/wiki/Proof\_of\_stake](https://en.wikipedia.org/wiki/Proof\_of\_stake)
* [https://www.mycryptopedia.com/public-key-private-key-explained/](https://www.mycryptopedia.com/public-key-private-key-explained/)
* [https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions](https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions)
* [https://ethereum.org/en/developers/docs/transactions/](https://ethereum.org/en/developers/docs/transactions/)
* [https://ethereum.org/en/developers/docs/gas/](https://ethereum.org/en/developers/docs/gas/)
* [https://en.bitcoin.it/wiki/Privacy](https://en.bitcoin.it/wiki/Privacy#Forced\_address\_reuse)


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
