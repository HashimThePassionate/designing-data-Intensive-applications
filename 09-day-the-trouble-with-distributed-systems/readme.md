# The Trouble with Distributed Systems

A.A. Milne ki book *The House at Pooh Corner* ka ek bohot pyaara quote hai: **"Accidents bohot ajeeb cheezein hain. Jab tak aapke sath accident ho nahi jata, tab tak aapko lagta hai ke yeh nahi hoga."**

Isi tarah software engineering mein bhi jab tak system crash nahi hota, hum samajhte hain ke sab theek chal raha hai. Chalein ab is text ke har ek point aur concept ko bohot hi aasan aur dilchasp tarike se break down karte hain.

---

### Happy Path Trap aur Mindset ka Badalna

Jab hum software banate hain, to hamara pura dhyan **Happy Path** par hota hai. Happy path ka matlab hai ke "sab kuch achha achha chal raha hai"—user ne sahi data dala, internet bilkul perfect chal raha hai, aur database ne foran respond kiya. Kyunke 90% time cheezein theek chalti hain, isliye developers faults (galatiyon ya kharabiyon) ko ignore kar dete hain kyunke unko handle karna bohot boring aur mushkil lagta hai.

Lekin agar aap ek aisa system banana chahte hain jo kabhi na ruke (**Reliable System**), to aapko apni soch ko bilkul badalna hoga. Aapko har waqt bura sochna padega (**Pessimism**). Agar kisi kharabi ka chance laakhon mein ek (one-in-a-million) bhi hai, to ek bade system mein (jahan har roz karodon requests aati hain) woh laakhon mein ek wali kharabi **har roz** hogi! Jo log bade systems chalate hain, unka ek hi asool hota hai: **"Jo cheez kharab ho sakti hai, woh ek na ek din zaroor kharab hogi."**

Single computer par code likhna aur **Distributed System** (jahan bohot saare computers aapas mein network ke zariye jude hote hain) par code likhna bilkul alag baat hai. Distributed systems mein faults naye aur ajeeb tareeqon se aate hain, jaise network ka slow ho jana ya computer ki ghardiyon (clocks) ka aapas mein aage peeche ho jana.

---

## Faults and Partial Failures

### Single Computer Ka Behavior (Predictable aur Deterministic)

Jab aap ek akele computer par koi program chalate hain, to uska behavior bohot saaf aur seedha hota hai: **ya to program poora chalega, ya bilkul nahi chalega.** * **Deterministic:** Iska matlab hai ke agar computer ka hardware theek hai, to aap ek hi kaam baar baar karenge to har baar bilkul same result aayega.

* **Total Failure:** Agar hardware mein koi masla aa jaye (jaise RAM kharab ho jaye ya koi wire dheeli ho jaye), to poora computer hi baith jata hai. Aapke samne **Blue Screen of Death (BSOD)** aa jayegi ya kernel panic ho jayega. Computer ya to poora zinda hota hai ya poora mar jata hai—bich ki koi state nahi hoti.

Computer ko jaan booch kar aisa design kiya gaya hai. Agar koi andaruni kharabi aaye, to behtar hai ke computer poora crash ho jaye, bajaye iske ke woh aapko galat calculation karke de. Kyunke agar computer chupchaap galat data generate karta raha, to usay pakadna bohot mushkil ho jayega.

CPU aur Memory asal mein physical wires aur transistors se bane hain jahan halki-fulki kharabiyan (silent corruption) hoti rehti hain, lekin computer ka hardware unhein hum se is tarah chupata hai jaise sab kuch mathematically perfect chal raha ho. Hum is ideal duniya ke itne aadi ho chuke hain ke hum in faults ko bhool hi jaate hain.

### Distributed Systems Ka Behavior (The Messy Physical World)

Lekin jab aapka software ek computer ke bajaye bohot saare computers par chal raha ho jo network se jude hain, to kahani bilkul badal jati hai. Ab aap is perfect duniya mein nahi rehte, balki physical duniya ki haqeeqat se aapka saamna hota hai.

Writer ne yahan **Coda Hale** ka ek bohot hi mazahiya aur real-world experience share kiya hai ke unho ne data centers mein kya kya kharabiyan dekhi hain:

> "Main koi ops (operations) ka banda nahi hoon, phir bhi apne chote se career mein main ne ye sab aafatain dekhi hain: Data center ke andar lambe arsay tak network ka toot jana, power supply units (PDU) ka phat jana, network switches ka kharab hona, poore ke poore server racks ki banti gul ho jana, data center ka main internet network fail ho jana, poore data center ki light chale jana... aur to aur, ek baar ek sugar (hypoglycemic) ka mareez driver apni Ford pickup truck lekar data center ke AC (HVAC) system ke andar ghus gaya!"

### Partial Failure Kya Hai?

Distributed system mein sab se bada masla **Partial Failure** (aadhi-adhuri kharabi) ka hota hai.

* **Definition:** Iska matlab hai ke system ka ek hissa (kuch computers ya network) bilkul toot chuka hai ya kharab hai, jabki system ka baki hissa bilkul sahi kaam kar raha hai.
* **Nondeterministic:** Yeh kharabiyan aisi hoti hain jinke baare mein aap pehle se andaza nahi laga sakte. Agar aap network ke zariye 3 computers ko koi kaam bhejenge, to ho sakta hai ek baar kaam ho jaye, aur agli baar achanak fail ho jaye. Aapko kabhi kabhi yeh bhi pata nahi chalta ke jo request aap ne bheji thi, woh kamyab hui bhi hai ya nahi!

Yahi aadhi-adhuri kharabi (nondeterminism) distributed systems ko design karna sab se mushkil kaam banati hai.

### Unreliable Components Se Reliable System Banana

Lekin iska ek dusra rukh bohot powerful hai. Agar hum ek aisa system design kar lein jo in **Partial Failures** ko bardasht kar sake (**Fault Tolerant**), to hum saste aur aam computers (jo kabhi bhi kharab ho sakte hain) ko mila kar ek bohot hi pakka aur mazboot system bana sakte hain.

Iska sab se bada faida **Rolling Upgrade** hota hai. Agar aapne software ko update karna hai, to aapko poora system band karne ki zaroorat nahi hai. Aap ek waqt mein ek computer (node) ko band karenge, usay update karke reboot karenge, aur tab tak baki computers user ka load sambhal rahe honge. Is tarah user ko pata bhi nahi chalega aur aapka poora system bina ruke update ho jayega.

### Sabak (Conclusion)

Distributed system mein kamyab hone ke liye aapko har waqt **shak (suspicion), bura sochne ki aadat (pessimism), aur darr (paranoia)** ki zaroorat hoti hai. Aapko apne testing environment mein jaan booch kar tarah tarah ki kharabiyan (jaise network band karna, servers crash karna) khud paida karni chahiye taake aap dekh sakein ke aap ka system un halaat mein kaise behave karta hai.

---

### Revision Hints (Tezi se yaad karne ke liye)

* **Happy Path Trap:** Sirf sab theek chalne par focus karna aur edge cases ko chor dena.
* **One-in-a-million Rule:** Bade systems mein kam se kam chance wali kharabi bhi har roz hoti hai.
* **Single Computer:** Ya to poora chalta hai ya poora crash hota hai (Deterministic).
* **Distributed System:** Kuch hissa chalta hai, kuch kharab ho jata hai (Partial Failure / Nondeterministic).
* **Fault Tolerance Ka Jadu:** Kamzor aur unstable computers ko jor kar ek nihayat reliable aur hamesha chalne wala system banana.

---

## Unreliable Networks

Purane zamane ke jo bade computers hote the, jinhein **Mainframes** kaha jata tha, unhein reliable (mazboot) banane ka tareeqa alag tha. Un ke andar hi double double components laga diye jate the—jaise agar ek hard disk kharab ho jaye to doosri sambhal le (jisay hum **RAID** kehte hain).

Lekin is book mein hum jin distributed systems par baat kar rahe hain, woh **Shared-nothing systems** hain. Iska matlab hai ke yeh bohot saari alag-alag machines (computers) ka ek jhund hai jo sirf ek network ke zariye aapas mein jude hain. Har machine ka apna dimagh (Memory/RAM) aur apni alag jeb (Disk) hoti hai. Ek machine doosri machine ki RAM ya disk ko direct haath nahi laga sakti; agar koi kaam hai to network par request bhejni padegi. Hatta ke agar hum shared object storage bhi use karein, to us se baat karne ke liye bhi network ki sadak hi istemal karni padti hai.

Hamara internet aur data centers ke andar ka network (Ethernet) **Asynchronous packet networks** hote hain. "Asynchronous" ka matlab hai ke jab ek computer doosre ko data ka ek tukda (packet) bhejta hai, to network is baat ki koi guarantee nahi deta ke woh packet kab pahonchega, ya woh bilkul pahonchega bhi ya nahi!

Agar aap ne ek request bheji aur aap jawab ka intezar kar rahe hain, to yeh **6 buraayein** ho sakti hain:

* **Request lost:** Aap ki request raste mein hi ghum gayi (ho sakta hai kisi ka paon lagne se network cable nikal gayi ho).
* **Queue mein phansna:** Aap ki request line (queue) mein lag gayi hai aur der se pahonchegi (ho sakta hai network par bohot rush ho ya aage wala server overloaded ho).
* **Remote node failed:** Jis server ko request bheji thi, woh crash ho gaya ya uski power chali gayi.
* **Temporary Process Pause:** Server mara nahi hai, temporary ruk gaya hai (jaise Java/Go wagera mein jab bada **Garbage Collection (GC) pause** aata hai, to server thodi der ke liye sunn ho jata hai), lekin baad mein hosh mein aa jayega.
* **Response lost:** Server ne aap ka kaam kar diya, par wapas aate huay network switch ki galat configuration ki wajah se jawab raste mein mar gaya.
* **Response delayed:** Server ne kaam bhi kiya, jawab bhi bheja, par network par load ki wajah se ya aap ke apne computer par load ki wajah se jawab bohot der se pahonch raha hai.

Bhejne wale (sender) ko khud se kabhi pata nahi chalta ke packet deliver hua ya nahi. Wahid tareeqa yeh hai ke samne wala wapas jawab bheje, aur woh jawab bhi to raste mein ghum ya der se aa sakta hai! Is asynchronous network mein aap ko sirf ek baat peta hoti hai: **"Mujhe abhi tak jawab nahi mila."** Iske alawa aap bilkul andha-dhund hain.

Is masle ka aam hal **Timeout** hota hai: yani hum ek time rakh lete hain ke "agar thodi der tak jawab nahi aaya, to main intezar chor dunga aur samajh lunga ke kaam nahi hua." Lekin timeout hone par bhi aap ko sach nahi pata chalta! Ho sakta hai request abhi bhi kahin line mein khari ho aur aap ke umeed chorne ke baad server us par kaam kar de.

### Figure 9-1 Ka Mukammal Breakdown

<div align="center">
  <img src="./images/01.png" width="700"/>
</div>

Is diagram mein teen horizontal lines hain jo **Client** (jo request bhej raha hai), **Network** (jis ke zariye data ja raha hai), aur **Service** (jo database ya server hai) ki timeline ko dikhati hain. Jab Client koi request bhejta hai aur usay wapas koi jawab nahi milta, to Client ke liye yeh teeno surtehaalan (a, b, aur c) bilkul ek jaisi hoti hain—yani sirf teen sawaaliya nishaan (**???**). Client ko pata hi nahi chalta ke buraai kahan hui:

* **(a) The request was lost:** Client ne request bheji, lekin network mein hi kahin ghum gayi (jaise kisi ne wire nikaal di). Service tak baat pahonchi hi nahi. Client baitha intezar kar raha hai (`???`).
* **(b) The remote node is down:** Client ne request bheji, network ne usay sahi-salamat server tak pahonchaya bhi, lekin aage se server hi mara pada hai (**Node unresponsive**). Server crash ho chuka hai ya band hai, isliye koi jawab nahi aaya (`???`).
* **(c) The response was lost:** Yeh sab se khatarnak scene hai! Client ne request bheji, network se hoti hui server tak gayi, server bilkul theek tha, usne request ko process kiya, database mein entry bhi kar di (**OK**), aur wapas jawab (response) bheja. Lekin wapas aate huay network ne us jawab ko gira diya (drop kar diya). Client ke paas phir bhi kuch nahi pahoncha (`???`).

**Sab se bada masla:** Client ke liye yeh teeno halaat bilkul ek jaisi hain. Remote node par kya hua, yeh jaan-na client ke liye namumkin hai.

---

## The Limitations of TCP

Network par jo packets jate hain, unka ek maximum size hota hai (kuch kilobytes). Lekin hamari applications ko baday baday data streams ya requests bhejni hoti hain. Isliye hum **TCP (Transmission Control Protocol)** (ya naye protocols jaise **QUIC**, **SCTP**, ya **uTP**) ka istemal karte hain jo baday data ko chhote packets mein todte hain aur doosri taraf unhein dobara jor kar seedha kar dene ka kaam karte hain.

TCP ko aam tor par "Reliable" protocol kaha jata hai, kyunke iske paas yeh khubiyan hain:

* **Retransmission:** Agar koi packet raste mein ghum jaye, to yeh usay dobara bhejta hai.
* **Reordering:** Agar packets aage peeche pahonchein, to unhein sahi tarteeb mein lata hai.
* **Checksum:** Checksum ke zariye check karta hai ke data raste mein kharab (corrupt) to nahi hua.
* **Congestion/Flow Control (Backpressure):** Yeh andaza lagata hai ke data kitni tezi se bhejna chahiye taake network ya samne wali machine par bohot zyada bojh na pad jaye.

**TCP Kaam Kaise Karta Hai?**
Jab aap code mein data ko socket par "send" karte hain, to woh foran internet par nahi jata. Woh aap ke Operating System (OS) ke ek **buffer** (temporary storage) mein baith jata hai. Jab congestion control algorithm dekhta hai ke network par jagah hai, to woh us buffer se data utha kar packet banata hai aur network interface ko deta hai. Packet mukhtalif routers aur switches se hota hua samne wale computer ke OS ke receive buffer mein jata hai. Wahan se samne wale ka OS ek **Acknowledgment (ACK)** packet wapas bhejta hai ke "haan, mujhe mil gaya." Jab ACK wapas aati hai, tabhi OS application ke level par data delivered ka notification deta hai.

**To phir hamien fikar karne ki kya zaroorat hai agar TCP reliable hai?**
Afsoos, TCP network ki na-kaabili-e-etemaad (unreliability) ko poori tarah khatam nahi kar sakta:

* Agar ACK wapas nahi aayi, to TCP samajhta hai packet ghum gaya aur dobara bhejta hai. Lekin TCP ko yeh nahi pata ke data bhejte huay ghuma tha ya ACK wapas aate huay ghumi thi.
* Agar network ki taar hi tooti hui hai, to TCP usay khud se jor nahi sakta. Ek had tak retry karne ke baad TCP hath khade kar deta hai aur application ko error de deta hai.
* TCP ka data ko double ہونے se bachana (**Deduplication**) sirf ek single connection ke andar kaam karta hai. Agar connection toot gaya aur aap ki application ne naya connection bana kar data dobara bheja, to data **duplicate** ho sakta hai (server par do baar entry ho sakti hai).
* Agar TCP connection error ke sath band ho jaye, to aap ko nahi pata chal sakta ke samne wale computer ne kitna data process kiya tha. Hatta ke agar ACK mil bhi jaye, to iska matlab sirf yeh hai ke samne wale ke OS kernel ko data mila hai; ho sakta hai data application tak pahonchne se pehle hi woh application crash ho gayi ho! Agar aap ko paka yakeen chahiye ke kaam hua hai, to aap ko **Application Level** se haan ka jawab chahiye.

Phir bhi, TCP bohot faidemand hai kyunke yeh baday messages ko handle karne ka aasan rasta deta hai. Ek baar connection ban jaye, to hum header mein message ki length bakaida likh kar ek hi connection par kai requests aur responses bhej sakte hain (jaise HTTP aur RPC kaam karte hain).

---

## Network Faults in Practice

Hum kai dahaiyon (decades) se computer networks bana rahe hain, lagta تھا ke ab tak hum network ko perfect banana seekh gaye honge, par aisa nahi hua. Real-world studies se pata chalta hai ke network ke masle aam hain, hatta ke un data centers mein bhi jahan har cheez ek hi company control kar rahi hoti hai:

* **Data Center Study:** Ek darmiyani size ke data center ki study mein dekha gaya ke wahan har mahine taqreeban **12 network faults** aate hain. Un mein se aadhe faults ek akele computer ko network se kaat dete hain, aur baqi aadhe poore ke poore server rack (servers ka poora dhaanchan) ko hi alag kar dete hain.
* **Redundancy Ki Limitations:** Switches aur load balancers ko double double (redundant) karne se bhi faults utne kam nahi hote jitna hum sochte hain, kyunke redundancy **Human Error** (insani galatiyon, jaise switches ki galat configuration) ko nahi rok sakti, jo ke outages ki bohot badi wajah hai.
* **Janwar aur Insan:** Zameen par bichhi fiber links tootne ka ilzam kabhi **gaayon (cows)** par, kabhi **udbilaw (beavers)** par, aur samundar ke andar **sharks** par aata raha hai (halaanke submarine cables ki shielding ab behtar ho gayi hai). Insan bhi kam nahi hain, aksar galti se galat config kar dete hain, ya taarein chori (**scavenging**) kar lete hain ya jaan booch kar sabotage karte hain.
* **Arbitrary Delays:** Cloud regions ke darmiyan kabhi kabhi round-trip time kuch minutes tak chala jata hai! Ek hi data center ke andar, agar kisi switch ka software upgrade chal raha ho aur network apna rasta dobara badal raha ho (**network topology reconfiguration**), to packet 1 minute se zyada delay ho sakta hai. Isliye distributed system mein hum yeh maan kar chalte hain ke message kitna bhi delay ho sakta hai.
* **Asymmetric & Non-transitive Faults:** Kabhi kabhi aadhi-adhuri kharabi hoti hai. Jaise Machine A, Machine B se baat kar sakti hai, aur Machine B, Machine C se baat kar sakti hai, lekin Machine A aur Machine C aapas mein baat nahi kar saktay (**Non-transitive**). Ya phir aisa ajeeb masla ke network card bahar data bhej raha hai par andar aane wale saare packets drop kar raha hai (One-way link).
* **Long-lasting Repercussions:** Network ka jhatka bhale hi kuch seconds ka ho, par uske asraat system par ghanton tak reh sakte hain.

> ### Network partitions
> 
> 
> Jab network ke kisi fault ki wajah se cluster ka ek hissa baqi hissay se bilkul kat jaye, to isay **Network Partition** ya **Netsplit** kehte hain. Yeh koi alag bala nahi hai, balki aam network faults jaisa hi hota hai. (Yaad rahe, iska database ki sharding/partitioning se koi talluq nahi hai).

Agar aap ne apne software mein network faults ko handle karne ka tareeqa pehle se code nahi kiya aur usay test nahi kiya, to kuch bhi bura ho sakta hai: aap ka poora cluster **Deadlock** ho sakta hai (yani network theek hone ke baad bhi kaam karna band kar dega) ya phir aap ka saara data delete ho sakta hai!

**Hal kya hai?**
Faults ko handle karne ka matlab hamesha yeh nahi hota ke system ko har haal mein chalte rehna chahiye. Agar aap ka network aam tor par theek rehta hai, to masle ke waqt user ko sirf ek saaf saaf **Error Message** dikha dena bhi ek valid tareeqa hai. Lekin aap ko yeh zaroor pata hona chahiye ke aap ka software kharabi par kya harkat karega aur woh khud ko dobara theek kaise karega. Isliye testing ke dauran jaan booch kar network kharab karke check karna (**Fault Injection**) bohot zaroori hai.

---

### Revision Hints (Tezi se yaad karne ke liye)

* **Figure 9-1 (Sawaaliya Nishaan):** Client ko kabhi pata nahi chalta ke request khoyi, server down hai, ya response khoya—teeno scenes same lagte hain.
* **Shared-Nothing:** Computers jo sirf network ke zariye jude hain, koi direct memory/disk share nahi hoti.
* **Asynchronous Network:** Arrival time ya delivery ki koi pakki guarantee nahi hoti.
* **TCP Reliability Limits:** TCP packet retransmit karta hai par connection tootne par yeh nahi bata sakta ke application ne data process kiya ya nahi. Deduplication sirf ek connection tak chalti hai.
* **Real-world Faults:** Networks mein delays arbitrary ho sakte hain, links asymmetric (ek taraf chalne wale) ho sakte hain, aur insani ya janwaron ki wajah se taarein toot sakti hain.
* **Fault Injection:** Jaan booch kar network kharab karke system ki recovery test karna zaroori hai.

---