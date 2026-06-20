# Consistency and Consensus

An ancient adage warns, “Never go to sea with two chronometers; take one or three.”
—Frederick P. Brooks Jr., The Mythical Man-Month: Essays on Software Engineering (1995)

---

### Introduction: Fault Tolerance Aur Replication Ka Masla

Distributed systems (jahan bohot saare computers mil kar kaam karte hain) mein bohot si cheezein kharab ho sakti hain. Agar hum chahte hain ke hamara system ya service kharabiyon ke bawajood bina ruke kaam karti rahe, toh hamein **fault tolerance** (buraqt kharabi ko jhelne ki salahiyat) ke tareeqe dhoondne honge.

Fault tolerance haasil karne ka sab se behtareen tareeqa **Replication** hai. Replication ka matlab hai ek hi data ki alag-alag copies (replicas) alag-alag computers par rakhna.

Lekin iska ek bohot bara nuksan (trade-off) hota hai: **Inconsistencies (Na-barabari)** ka khatra barh jata hai.

* **Stale Reads (Purana Data):** Sochein aap ke paas do copies hain. Ek copy update ho gayi lekin doosri abhi purani hai. Agar koi user us purani copy ko parhay ga, toh usay purana ya "baasi" data milega (jisay stale results kehte hain).
* **Write Conflicts (Takraat):** Agar do alag-alag computers par aik hi waqt mein alag-alag data likha (write kiya) jaye, toh un dono copies ke darmiyan larai ho jayegi ke kaun sa data sahi hai.

In masail se nipatne ke liye hamare paas do baray falsafay (philosophies) hain:

---

### Eventual consistency

Is falsafay (philosophy) mein, application developer (jo software bana raha hai) ko yeh pehle se pata hota hai ke data alag-alag computers par copy ho raha hai aur is mein deri ho sakti hai.

* **Bacho ki Tarah Samajhein:** Yeh bilkul aisa hai jaise aap ne do doston ko ek kahani sunayi. Ek dost ko kahani foran samajh aa gayi, doosre tak baat thodi der se pohanchi. Kuch dair ke liye dono ke paas alag-alag jankari hogi, lekin **eventually (aakhir kaar)** kuch waqt baad dono ke paas same kahani pohanch jaye gi.
* **Developer Ki Zimmedari:** Is approach mein system khud se sab kuch theek nahi karta. Developer ko khud code likhna parta hai ke agar data mein koi takraat (conflict) ya purana data samne aaye, toh usay kaise hal karna hai.
* **Kahan Use Hota Hai?** Yeh aam tor par un systems mein istemaal hota hai jahan **Multi-Leader Replication** (jahan aik se zyada computers writes accept karte hain) ya **Leaderless Replication** (jahan koi main leader nahi hota) istemaal ho rahi ho.

---

### Strong consistency

Is falsafay mein, application developer ko is baat ki bilkul fikar nahi karni parti ke background mein data kitne computers par copy ho raha hai ya replication kaise chal rahi hai.

* **Bacho ki Tarah Samajhein:** Yeh aisa hai jaise poori duniya mein sirf **aik hi khilona (single node)** ho. Agar koi bhi bacha us khilone se khelega ya usay badlega, toh har ek ko wahi badla hua khilona nazar aayega. Koi purani ya galat copy ka chakkar hi nahi hota. System aisa behave karta hai jaise poora data sirf ek hi computer par para ہو۔
* **Fayda (Advantage):** Yeh application banane wale ke liye bohot asaan hai kyunke usay kisi kisam ke data conflict ya purani report ki fikar nahi karni parti.
* **Nuksan (Disadvantage):** Strong consistency ki ek keemat (performance cost) hoti hai. System ko har computer ko aapas mein sync karne ke liye bohot waqt lagana parta hai, jis se speed kam ho sakti hai. Iske ilawa, agar network mein koi kharabi aa jaye (jo eventual consistency jhel sakti hai), toh strong consistency wala system kaam karna band (outage) kar sakta hai.

---

### Kaun Sa Tareeqa Behtar Hai? (The Decision Factor)

Dono mein se kaun sa tareeqa behtar hai, yeh bilkul aap ki application ki zaroorat par depend karta hai:

* **Offline Data:** Agar aap ki app aiysi hai jahan user internet ke bina (offline) bhi data mein badlao kar sakta hai (jaise local sync engines), toh wahan **Eventual Consistency** lazmi ho jati hai.
* **Tez Aur Reliable Network:** Agar aap ke saare computers baray datacenters mein pare hain jahan internet communication bohot tez aur bharosay mand hai, toh **Strong Consistency** zyada behtar hai kyunke wahan iska performance cost bardasht kiya ja sakta hai.

---

### Three Areas of Focus in This Chapter

Is chapter mein hum **Strong Consistency** ki gehrai mein jayenge aur khaas tor par teen cheezon par focus karenge:

* **Linearizability:** Kyunke "strong consistency" ek aam aur thoda sa mubham (vague) lafz hai, is liye hum is ki ek bilkul saaf aur solid tareef (precise definition) banayenge, jisay hum **Linearizability** kehte hain (yani sab ke liye data ka aik hi seedha timeline hona).
* **IDs Aur Timestamps Generate Karna:** Hum dekhenge ke distributed systems mein uniques IDs aur timestamps (waqt ka hisab) kaise banaye jate hain. Yeh sunne mein shayad consistency se alag lage, lekin yeh aapas mein bohot gehray juray hue hain.
* **Consensus Algorithms:** Hum yeh explore karenge ke distributed systems kharabiyon ke bawajood kaise Linearizability haasil kar sakte hain. Iska jawab **Consensus Algorithms** (sab computers ka aapas mein aik baat par raazi hona) hai.

Is safar mein hamein yeh bhi pata chalega ke distributed systems mein kuch aisi bunyadi hudood (fundamental limits) hain jinse aage jana mumkin nahi hai (jaise aap aik hi waqt mein har cheez perfect nahi kar sakte).

---

### Distributed Systems Kyun Mushkil Hain? (The Challenge)

Is chapter ke topics ko sahi tareeqe se implement karna bohot zyada mushkil samjha jata hai.

* **Happy Path Vs Edge Cases:** Aise systems banana bohot asaan hai jo normal halat (jab koi kharabi na ho) mein bilkul theek chalte hain. Lekin jab achanak koi aisi kharabi ya messages ka agay-piche hona (unlucky combination of faults) samne aata hai jiske baare mein designer ne socha bhi nahi tha, toh poora system tabah ho jata hai.
* **Theory Ki Zaroorat:** In mushkil mawaqe (edge cases) ko samajhne aur unse bachne ke liye bohot saari theoretical research ki gayi hai taake hum aise systems bana sakein jo har tarah ki kharabi ko mardana-war jhel sakein.

> **Note:** Yeh chapter sirf is gehri theory ki satah (surface) ko touch karega. Hum algorithm ki gehraiyon, mathematical models, aur proofs mein nahi jayenge balkay aam faham samajh (informal intuitions) par focus karenge. Agar aap ko future mein serious consensus systems par kaam karna hai, toh aap ko is theory ko mazeed gehrai se parhna hoga.

---

## Linearizability

Agar aap chahte hain ke aap ka replicated database (aik se zyada computers par chalne wala database) chalne aur istemaal karne mein bilkul asaan ho, toh aap ko isay aisa banana hoga jaise yeh sirf **aik single-node database** (yani poori duniya mein sirf aik hi computer) ho.

Iska sab se bara fayda yeh hoga ke software developers ko replication lag (data der se pohnchna), data conflicts (takraat), aur doosri kharabiyon ki fikar nahi karni paregi. Aap ko fault tolerance (kharabi se bachne) ka fayda bhi mil jayega aur system ko chalana bhi mushkil nahi lagega.

Isi soch ko hum **Linearizability** kehte hain. Iske aur bhi bohot saare naam hain jaise:

* **Atomic consistency**
* **Strong consistency**
* **Immediate consistency**
* **External consistency**

Linearizability ki asli definition thodi gehri hai, lekin iska bunyadi maqsad yeh hai: **System ko bahar se dekhne par aisa lagna chahiye ke data ki sirf aik hi copy maujood hai, aur jo bhi kaam (operations) us par ho rahe hain woh atomic (aik hi jhatkay mein mukammal) hain.** Is guarantee ki wajah se application ko background mein chalne wale saare computers ki koi chinta nahi hoti.

Linearizable system mein jaise hi koi client database mein koi nayi cheez likhta (write karta) hai aur usay "OK" ka jawab mil jata hai, toh uske baad jo bhi client data parhay ga, usay wahi bilkul naya aur up-to-date data nazar aana chahiye. Linearizability asal mein **Recency Guarantee** hai (yani yeh pakka karna ke data bilkul naya aur taza ho, na ke kisi purani cache ya lagging computer se aa raha ho).

Is baat ko mazeed achi tarah samajhne ke liye, hum aik aise system ki misaal dekhte hain jo linearizable **nahi** hai.

---

#### Figure 10-1 Ka Breakdown: Non-linearizable Sports Website

Writer ne **Figure 10-1** mein ek match ke live score ki misaal di hai. Sochein do dost hain, **Aaliyah** aur **Bryce**, jo aik hi kamray mein baithay apne phones par match ka score dekh rahe hain.

<div align="center">
  <img src="./images/01.png" width="700"/>
</div>

**Step-by-Step Breakdown:**

1. **Referee Ka Action (Write Operation):** Referee match khatam hote hi database mein final score enter karta hai. Modern SQL format mein yeh query banti hai:
```sql
INSERT INTO final_scores (player1, score1, player2, score2) 
VALUES ('Team A', 1, 'Team B', 0);

```


2. **Leader Aur Followers Ka Role:** Yeh query sab se pehle **Leader** database ke paas jati hai. Leader usay save karke Referee ko **"OK"** ka ishara bhej deta hai. Ab Leader is data ko aage do database computers (**Follower 1** aur **Follower 2**) par copy karne ke liye bhejta hai.
3. **Aaliyah Ka Refresh (Fast Follower):** Aaliyah apne phone par page refresh karti hai. Uska phone query bhejta hai:
```sql
SELECT * FROM final_scores;

```


Aaliyah ki request **Follower 1** ke paas jati hai, jo pehle hi up-to-date ho chuka tha. Aaliyah ko naya score (Team A jeet gayi) nazar aata hai aur woh khushi se Bryce ko batati hai.
4. **Bryce Ka Refresh (Lagging Follower - The Problem):** Bryce jaise hi Aaliyah se sunta hai, woh foran (Aaliyah ke refresh karne ke baad) apna phone refresh karta hai. Uske phone se bhi wahi `SELECT *` query jati hai, lekin network ki susti ki wajah se uski request **Follower 2** ke paas chali jati hai. Follower 2 tak abhi tak Leader se naya data nahi pohancha tha (woh lag kar raha tha).
5. **Nateeja (Result):** Bryce ke phone par show hota hai ke match abhi tak chal raha hai ("Website says they're still playing"). Bryce heran ho jata hai ke Aaliyah keh rahi hai match khatam ho gaya aur mere phone par abhi bhi live chal raha hai.

**Linearizability Ki Violation:**
Agar Aaliyah aur Bryce bilkul ek hi microsecond par refresh karte, toh dono ko alag score dikhna shayed itna ajeeb na hota, kyunke hamein nahi pata ke server ne pehle kiski request process ki. Lekin yahan Bryce ne **Aaliyah ke dekhne ke baad** refresh kiya hai. Is liye Bryce ka haq banta hai ke usay data kam az kam itna naya mile jitna Aaliyah ko mila tha. Purana data milna hi **Linearizability ka tootna (violation)** kehlata hai.

---

### What Makes a System Linearizable?

Linearizability ko mazeed bariki se samajhne ke liye hum kuch aur technical designs dekhte hain. **Figure 10-2** mein teen alag clients (A, B, aur C) aik hi waqt mein database ke aik hi object $x$ ko parh aur likh rahe hain. Distributed systems ki theory mein is $x$ object ko **Register** kehte hain (asal zindagi mein yeh Redis ki koi key, SQL database ki koi row, ya MongoDB ka koi document ho sakta hai).

Is system mein do tarah ke kaam (operations) ho rahe hain:

* $\text{Read}(x) \implies v$ : Client ne $x$ ki value parhni chahi, aur database ne usay $v$ value wapis ki.
* $\text{Write}(x, v) \implies r$ : Client ne $x$ ki value ko badal kar $v$ karne ki request ki, aur database ne badlay mein $r$ (yani OK ya Error) bheja.

---

#### Figure 10-2 Ka Breakdown: Concurrent Reads Aur Writes

<div align="center">
  <img src="./images/02.png" width="700"/>
</div>

**Figure 10-2** mein clients ke point of view se requests ko horizontal bars (lambay ڈبوں) ki shakal mein dikhaya gaya hai.

* **Bar Ka Start Aur End:** Jahan se bar shuru hoti hai, wahan client ne request bheji thi. Jahan bar khatam hoti hai, wahan client ko jawab (response) mila. Network slow ya tez hone ki wajah se client ko exact microsecond nahi pata hota ke database ne kab kaam kiya, bas itna pata hota hai ke kaam is bar ke shuru aur khatam hone ke darmiyan hi kahin hua hai.
* **Shuruati Value:** Shuru mein $x$ ki value $0$ thi. Client C database mein `Write(x, 1)` ki request bhejta hai taake value $1$ ho jaye. Is dauran Client A aur Client B baar baar data read kar rahe hain.

**Possible Responses Ka Breakdown:**

* **Client A Ka Pehla Read:** Yeh read operation Client C ke write shuru karne se **pehle hi khatam** ho gaya tha. Is liye isko har haal mein purani value **0** hi milegi.
* **Client A Ka Aakhri Read:** Yeh read operation Client C ke write mukammal hone ke **baad shuru** hua hai. Is liye agar system linearizable hai, toh isay har haal mein nayi value **1** hi milni chahiye.
* **Overlapping / Concurrent Reads:** Client A aur Client B ke jo reads Client C ke write ke **darmiyan (overlap)** ho rahe hain, unhein **0 ya 1** mein se kuch bhi mil sakta hai. Kyunke hamein nahi pata ke database ne write ka kaam pehle kiya ya read ka. In operations ko hum *concurrent* kehte hain.

Lekin sirf itna hona hi linearizability ke liye kaafi nahi hai. Agar concurrent reads mein kabhi 0 aur kabhi 1 milta rahe, toh bache ko aisa lagega ke data aage piche chal raha hai (yani pehle 1 dikha, phir achanak doosray read mein 0 dikha, phir 1 dikha). Yeh single copy ka ehsas nahi deta.

---

#### Figure 10-3 Ka Breakdown: The Atomicity Constraint

Is maslay ko hal karne ke liye hum aik aur rule lagate hain jo **Figure 10-3** mein dikhaya gaya hai.

<div align="center">
  <img src="./images/03.png" width="700"/>
</div>

* **The Magic Point:** Linearizable system mein hum yeh maante hain ke poore write operation ke dauran aik aisa achanak lamha (point in time) aata hai jahan $x$ ki value jhatkay se $0$ se $1$ ho jati hai.
* **The Arrow Constraint:** Agar kisi aik client ke read ne nayi value **1** dekh li, toh uske baad shuru hone wale **saare ke saare reads** ko har haal mein **1** hi milna chahiye, chahe abhi main write operation poori tarah khatam na bhi hua ho!
* **Figure 10-3 Ke Steps:** Client A ka read operation chal raha hai aur usay value **1** mil jati hai. Client A ke naye data parhne ke foran baad Client B naya read shuru karta hai. Chunke B ka read strictly A ke baad shuru hua hai, is liye diagram mein bani **kaali arrow** ke mutabaq B ko bhi **1** hi milega, chahe Client C ka write abhi tak chal hi kyun na raha ho. (Yeh bilkul Aaliyah aur Bryce jaisa scene hai).

---

#### Figure 10-4 Ka Breakdown: Complex Visual Timeline

Ab hum mazeed complex operations dekhte hain jo **Figure 10-4** mein hain. Yahan aik naya operation use hua hai:

<div align="center">
  <img src="./images/04.png" width="700"/>
</div>


$$\text{CAS}(x, v_{\text{old}}, v_{\text{new}}) \implies r$$


Isay **Compare-and-Set** kehte hain. Iska matlab hai: "Agar $x$ ki maujooda value $v_{\text{old}}$ ke barabar hai, toh usay badal kar $v_{\text{new}}$ kar do aur **OK** bhej do. Agar value badal chuki hai, toh kuch mat karo aur **Error** bhej do."

Figure 10-4 mein har operation ke bar ke andar ek **vertical line (khari lakeer)** lagi hai jo yeh batati hai ke database ne asal mein kis waqt us request ko execute kiya. Linearizability ka sab se bara asool yeh hai ke **yeh vertical lines hamesha waqt mein aage (left to right) jani chahiye, kabhi piche (right to left) nahi mudni chahiye.**

Chalein is mushkil diagram ke saare points ko asaan karke breakdown karte hain:

* **B Ka Pehla Read Aur Concurrency:** Client B ne pehle read bheja, phir Client D ne `Write(x, 0)` bheja, aur Client A ne `Write(x, 1)` bheja. Lekin Client B ke read ko **1** mil raha hai. Yeh bilkul sahi hai! Iska matlab hai network ke delay ki wajah se database ne pehle D ka write chalaya, phir A ka write chalaya, aur aakhir mein B ka read chalaya. Order aage piche ho sakta hai kyunke yeh teeno kaam aik hi waqt mein (concurrently) ho rahe the.
* **OK Se Pehle Data Milna:** Client B ko value **1** mil gayi, jabke Client A ko abhi tak database se "OK" ka response nahi mila tha. Yeh bhi bilkul normal hai, kyunke database ne data write kar diya tha, bas A tak kamyabi ka message network line mein thoda slow aa raha tha.
* **Client C Ke Do Reads:** Client C pehle read karta hai toh usay **1** milta hai. Phir Client B aik operation chalata hai: `CAS(x, 1, 2) => OK`. Chunke value 1 thi, toh B ne usay badal kar 2 kar diya. Is liye jab Client C doosri baar read karta hai, toh usay **2** milta hai.
* **Client D Ka Fail Operation:** Client D aik request bhejta hai: `CAS(x, 0, 3) => Error`. D ko error is liye mila kyunke jab database ne is request ko pakra, us waqt tak value 0 nahi rahi thi balkay badal kar 2 ho chuki تھی۔
* **The Non-Linearizable Error (Grey Bar):** Diagram ke aakhir mein Client B ka aakhri read (jo grey rang ke dabbe mein hai aur upar khatray ka nishan $\triangle$ bana hai) **linearizable nahi hai!** Kyun? Kyunke Client C ne ek operation chalaya tha `CAS(x, 2, 4)` jis se value 4 ho gayi thi. Client A ne us 4 value ko parh bhi liya tha. Ab jab Client A up-to-date value (4) parh chuka hai, toh uske BAAD shuru hone wale B ke read ko purani value **2** nahi mil sakti! Yeh time travelling jaisa gunah hai distributed systems mein, is liye yeh linearizability ki sarii sharton ko tor raha hai.

---

### Linearizability Versus Serializability

Log aksar in dono lafzon mein confuse ho jate hain kyunke dono ka matlab aisa lagta hai ke "cheezon ko aik seedhi line mein tarteeb dena". Lekin distributed systems mein yeh dono bilkul alag alag guarantees hain:

| Khasiyat (Feature) | Serializability | Linearizability |
| --- | --- | --- |
| **Kiske Liye Hai?** | **Transactions** ke liye (Jahan aik transaction mein bohot saari rows ya objects badle jate hain). | **Single Object / Register** ke liye (Aik waqt mein sirf aik key ya row par kaam). |
| **Asal Maqsad?** | Multi-object isolation level hai. Har transaction aise chale jaise poore database par sirf wahi akeli chal rahi ho (bina interference ke). | Recency guarantee hai. Jo bhi kaam ho, real-time mein foran sab ko bilkul naya dikhna chahiye. |
| **Stale Reads (Purana Data)** | **Allowed hai.** Agar transactions ka serial order purana data dikhata hai toh serializability ko aitraz nahi hota. | **Strictly Not Allowed.** Aik dafa naya data aa gaya toh purana data dikhana mana hai. |
| **Multi-object Problems (Write Skew)** | Yeh unhein **rokta hai**, kyunke yeh poori transaction ko secure karta hai. | Yeh unhein **nahi rok sakta**, kyunke yeh sirf ek single object par focus karta hai. |

#### Dono Ko Milana: Strict Serializability

Agar koi database in dono guarantees ko aik sath faraham kare (yani transactions bhi perfect hon aur data bhi bilkul naya ho), toh usay hum **Strict Serializability** ya **Strong One-Copy Serializability (strong-1SR)** kehte hain.

* **Single-Node Databases:** Jo database sirf aik hi computer par chalte hain, un mein aam tor par yeh dono khubiyan pehle se hoti hain.
* **Distributed Databases Ka Masla:** Jo databases bohot saare computers par phailay hote hain, unke liye yeh mushkil hota hai. Misaal ke tor par, **CockroachDB** serializability toh deta hai, lekin har transaction par bohot zyada coordination ke kharche (performance cost) se bachne ke liye strict serializability nahi deta.
* **Perfect Examples:** Dusri taraf, Google ka **Spanner** aur **FoundationDB** strict serializability faraham karte hain, chahe is ke liye unhein jitni bhi heavy coordination karni paray.

---

## Relying on Linearizability

Hum ne yeh toh dekh liya ke **Linearizability** kya hoti hai, lekin sawal yeh paida hota hai ke hamein is ki zaroorat kahan parti hai? Agar koi sports website score dikhane mein do-chaar seconds late bhi ho jaye, toh koi aatish-fashan nahi phat parega. Lekin kuch jagahain aisi hain jahan agar linearizability na ho, toh poora system tabah ho sakta hai. Chalein un mawaqe ko deeply samajhte hain:

---

#### Locking and leader election

Distributed systems mein (jahan bohot saare computers mil kar kaam karte hain) agar single-leader architecture use ho raha ho, toh hamein har haal mein yeh pakka karna parta hai ke poore system mein **sirf aur sirf aik hi leader** ho. Agar do computers khud ko leader samajhne lagein, toh is kharab bimari ko distributed systems mein **Split Brain** kehte hain (yani aik jism ke do dimaagh, jo system ko pagal kar dete hain).

* **Lease (Kiraye ka mada):** Leader chunne ka aik tareeqa yeh hota hai ke aik "Lease" (aik makhsoos waqt ke liye ijazat nama) banaya jata hai. Jo computer sab se pehle is lease ko pakar leta hai, woh leader ban jata hai.
* **Linearizability Ki Shart:** Is lease ko chalane wala system har haal mein linearizable hona chahiye. Aisa hargiz nahi hona chahiye ke do alag computers aik hi microsecond par request bhejein aur system dono ko keh de: "Haan, tum leader ho!"
* **Apache ZooKeeper aur etcd:** Log distributed leases aur leader election ke liye ZooKeeper ya etcd jaise coordination tools use karte hain. Yeh background mein consensus algorithms use kar ke linearizable kaam faraham karte hain.
* *Choti Bareeki:* ZooKeeper mein writes toh linearizable hoti hain, lekin purani versions mein reads stale (purani) ho sakti thi kyunke woh har dafa main leader se data nahi parhta tha. Lekin **etcd (version 3 se)** by default bilkul linearizable reads deta hai.


* **Oracle Real Application Clusters (RAC):** Kuch databases bohot barik level (disk page level) par locking karte hain jahan saare nodes aik hi storage share kar rahe hote hain. Chunke yeh linearizable locks database ki speed par asar daalte hain, is liye RAC ke liye aik alag se bohot tez internal network network interconnect banaya jata hai.

---

#### Constraints and uniqueness guarantees

Databases mein **Uniqueness Constraints** bohot aam hain—jaise do users ka username ya email address bilkul same nahi hona chahiye, ya aik hi folder mein aik hi naam ki do files nahi ho sakti.

* **Bacho ki Tarah Samajhein:** Sochein do bache aik hi waqt mein "Ali" naam ka account banana chahte hain. Agar system linearizable nahi hoga, toh dono ki requests alag-alag computers par jayengi, dono computers kahain ge "Ali naam khali hai" aur dono ka account ban jayegi! Yeh toh rules ke khilaf ho gaya.
* **CAS (Compare-and-Set) Ka Khel:** Unique constraint asal mein lock lagane ya atomic CAS operation chalane jaisa hi hai. Jab aik bacha username claim karega, toh system check karega ke agar yeh khali hai toh is par is bache ka nishan laga do. Is ke liye sab nodes ka aik page par hona zaroori hai.
* **Real-World Examples:**
* **Bank Balance:** Aap ke account mein paise zero se kam (negative) nahi hone chahiye. Agar do jagah se aik hi waqt mein paise nikale jayein, toh balance check karne ke liye bilkul up-to-date value chahiye.
* **Warehouse Stock:** Jitna maal godam mein para hai, us se zyada aap bech nahi sakte.
* **Theater/Flight Booking:** Aik seat par do log aik sath booking nahi kar sakte.


* **Dheeli Constraints (Loose Constraints):** Kuch real apps mein thodi bohot galti chalti hai. Jaise agar flight overbook ho jaye (aik seat do logon ko bik jaye), toh company unhein doosri flight de deti hai aur thoda harjana bhar deti hai. Aise mawaqe par linearizability ke bina kaam chalaya ja sakta hai, lekin jahan **Hard Uniqueness Constraint** chahiye (jaise relational database ke rules), wahan iske bina guzara nahi.

---

#### Cross-channel timing dependencies

Hum ne **Figure 10-1** mein dekha tha ke agar Aaliyah munh se score na bolti, toh Bryce ko kabhi pata hi na chalta ke uska data purana hai. Bryce ko masla is liye pata chala kyunke database ke ilawa communication ka aik aur rasta (Aaliyah ki awaaz) maujood tha. Isay kehte hain **Cross-Channel Timing Dependency** (yani do alag alag raston se jankari ka agay-piche hona).

Computer systems mein bhi yeh masla bohot aata hai. Writer ne **Figure 10-5** mein video website (jaise YouTube) ki aik bohot behtareen real-world example di hai.

---

##### Figure 10-5 Ka Breakdown: Video Transcoder Aur Race Condition

Sochein aik system hai jahan user video upload karta hai, aur background mein aik processor (Transcoder) us video ka size chota karta hai taake slow internet par bhi video chal sakay.

<div align="center">
  <img src="./images/05.png" width="700"/>
</div>

**Step-by-Step Dataflow:**

1. **Step 1 (Upload video):** User website par aik bari video upload karta hai.
2. **Step 2 (Store original video):** Web Server us video ko direct Message Queue mein nahi dalta (kyunke queue chote messages ke liye hoti hai). Web server us video file ko **File Storage** (jaise AWS S3) mein save kar deta hai.
3. **Step 3 & 4 (Send & Deliver message):** Jab video file save ho jati hai, toh Web Server **Message Queue** mein aik choti si chit (instruction) bhejta hai ke: *"Bhai, video save ho gayi hai, ab isay transcode (convert) kar do"*. Yeh message queue se hota hua **Video Transcoder** computer tak pohanch jata hai.
4. **Step 5 & 6 (Fetch & Store transcoded video):** Video Transcoder us chit ko parhta hai aur File Storage se original video uthane (**Fetch**) jata hai taake usay convert karke wapis save kar sakay.

**The Dangerous Race Condition (Khatra):**
Agar aap ka File Storage **linearizable nahi hai**, toh yahan aik larai (race condition) shuru ho sakti hai. Ho sakta hai ke Message Queue wala rasta (Steps 3 aur 4) bohot fast ho, aur File Storage ka apna andruni data-copying system (internal replication) thoda slow chal raha ho.

Jab Video Transcoder Step 5 par video uthane pohanchega, toh usay storage par ya toh **khali jagah** milegi ya video ka **purana/aadha version** milega! Agar us ne purani file ko convert kar diya, toh original video aur converted video hamesha ke liye aapas mein kharab (permanently inconsistent) ho jayenge.

* **Wajah:** Yeh masla is liye hua kyunke Web Server aur Transcoder ke darmiyan do alag alag communication channels chal rahe the (File Storage aur Message Queue). Agar real guarantee (linearizability) na ho, toh dono channels ke darmiyan aisi race conditions banti rahengi.
* **Mobile Push Notifications Ki Example:** Bilkul aisa hi tab hota hai jab aap ke mobile par push notification aati hai ke *"Aap ko naya message mila hai"*, aap notification par click karte hain lekin app khulne par naya message gayab hota hai kyunke app jis database node se data parh rahi hai, wahan abhi tak naya message copy hi nahi hua!

---

### Implementing Linearizable Systems

Ab jab hum samajh gaye hain ke linearizability kitni zaroorii hai, toh sawal yeh hai ke isay banayein kaise?

Chunke linearizability ka simple matlab hai: *"Aise behave karo jaise data ki sirf aik copy hai aur sab kaam atomic hain"*, toh sab se asaan tareeqa toh yeh hai ke **sach mein data ki sirf aik hi copy (Single Node) rakhi jaye.** Lekin agar woh akela computer kharab ho gaya ya jal gaya, toh aap ka saara data gayo! Fault tolerance khatam.

Chalein Chapter 6 ke replication ke tareeqon ko dobara check karte hain ke kya unhein linearizable banaya ja sakta hai ya nahi:

* **Single-leader replication (Potentially linearizable):**
Agar aap saare ke saare parhne (reads) aur likhne (writes) ke kaam **sirf aur sirf main leader node** par hi karein, toh yeh system linearizable ho sakta hai. Lekin is mein aik bara pech (catch) hai: Leader ko pakka pata hona chahiye ke wahi asli leader hai. Kabhi kabhi network galti se kisi node ko lagta hai ke woh leader hai (Delusional Leader), jabke piche naya leader chuna ja chuka hota hai. Agar woh pagal leader requests accept karta raha, toh linearizability toot jayegi.
* *Note:* Agar database ko chunks mein baant diya jaye (**Sharding**), tab bhi linearizability par asar nahi parta kyunke yeh guarantee sirf single object (aik key/row) ke liye hoti hai. Cross-shard transactions alag kahani hain.


* **Consensus algorithms (Likely linearizable):**
Consensus algorithms (jaise Raft ya Zab jo etcd aur ZooKeeper mein hote hain) built-in leader election aur split-brain se bachne ki safety ke sath aate hain. Is liye yeh safely linearizable storage bana sakte hain. Lekin yaad rahe, agar koi system consensus use karta bhi hai, tab bhi agar woh bina check kiye kisi bhi node se reads allow kar de, toh data stale ho sakta hai.
* **Multi-leader replication (Not linearizable):**
Yeh systems **hargiz linearizable nahi hote**. Kyunke in mein aik hi waqt mein alag-alag computers par data write ho raha hota hai jo baad mein aapas mein jodaa jata hai. Is mein write conflicts aana pakka hai.
* **Leaderless replication (Probably not linearizable):**
Dynamo-style systems (jaise Cassandra) mein log kehte hain ke agar aap Quorum rule ($w + r > n$) use karein, toh "Strong Consistency" mil jati hai. Lekin distributed systems ki theory ke mutabaq yeh poori tarah sach nahi hai.
* Agar Cassandra ya ScyllaDB mein time-of-day clocks (LWW - Last Write Wins) ke zariye faisla ho raha ho, toh clocks ke agay-piche hone (clock skew) ki wajah se linearizability **pakka toot jati hai**. Even bina clocks ke bhi, network delay ki wajah se quorums mein linearizability toot sakti hai, jaisa niche dikhaya gaya hai.



---

#### Figure 10-6 Ka Breakdown: Quorum Ke Bawajood Consistency Ka Tootna

**Figure 10-6** mein dikhaya gaya hai ke kaise Quorum rule ($w + r > n$) lagane ke bawajood system galti kar baithta hai.

Sochein hamare paas total teen replicas hain ($n=3$). Aik Writer computer $x$ ki value ko $0$ se badal kar $1$ karna chahta hai aur woh teeno nodes par write bhejta hai ($w=3$).

<div align="center">
  <img src="./images/06.png" width="700"/>
</div>

**Step-by-Step Execution:**

1. Writer ki request **Replica 3** par pehle pohanchti hai aur wahan value **1** ho jati hai aur Replica 3 wapis **"OK"** bhej deta hai. Lekin network delay ki wajah se **Replica 1** aur **Replica 2** tak abhi naya data nahi pohancha, wahan abhi bhi **0** para hai.
2. **Reader A Ka Operation (Concurrent Read):** Isi dauran Reader A data parhne aata hai. Uska quorum size $r=2$ hai. Woh **Replica 2** (jo 0 deta hai) aur **Replica 3** (jo 1 deta hai) se data parhta hai. Chunke Replica 3 ke paas naya version hai, Reader A ko nayi value **1** mil jati hai aur uska read operation mukammal (**complete**) ho jata hai.
3. **Reader B Ka Operation (The Violation):** Reader B ka operation Reader A ke mukammal hone ke **BAAD** shuru hota hai. Reader B bhi do nodes ($r=2$) se parhta hai: **Replica 1** aur **Replica 2**. Chunke in dono nodes tak abhi tak Writer ka naya message nahi pohancha tha, dono nodes Reader B ko purani value **0** wapis kar dete hain!

**Nateeja:** Reader B ko purana data (0) mila jabke us se pehle Reader A naya data (1) parh chuka tha! Yeh linearizability ka sab se bara qatal hai (Aaliyah aur Bryce wala scene dobara ho gaya).

* **Ilaaj (The Fix):** Agar Dynamo-style system ko linearizable banana hai, toh do keemtein chukani parengi:
1. **Synchronous Read Repair:** Jab Reader A ko pata chala ke Replica 2 ke paas purana data hai, toh application ko jawab bhejne se PEHLE Reader A ko har haal mein Replica 2 ko update karna parega.
2. **Read Before Write:** Writer ko likhne se pehle saare nodes ka timestamp parhna parega taake naya nishan lagaya ja sakay. Riak database speed bachane ke liye aisa nahi karta, jabke Cassandra parhte waqt wait toh karta hai par clocks kharab hone ki wajah se phir bhi linearizability loose kar deta hai.



Is liye, safety is mein hai ke yeh maan liya jaye ke **Leaderless system quorum lagane ke bawajood linearizable nahi hote.**

---

### The Cost of Linearizability

Ab jab hum ne dekh liya ke kuch tareeqe linearizability de sakte hain aur kuch nahi, toh is ke nafa-nuksan (pros and cons) par baat karte hain. Hum ne dekha tha ke **Multi-Region Replication** (alag alag mulkon mein data rakhna) ke liye multi-leader replication behtar hoti hai. **Figure 10-7** is ki aik behtareen visual misaal hai.

---

#### Figure 10-7 Ka Breakdown: Network Partition Aur Qurbani

<div align="center">
  <img src="./images/07.png" width="700"/>
</div>

Sochein hamare paas do regions hain: **Region 1** aur **Region 2**. Dono ke darmiyan internet ki tar achanak kat jati hai ya communication block ho jati hai. Distributed systems mein is achanak aane wali aafat ko **Network Partition** kehte hain.

1. **Agar Multi-Leader System Ho (AP):** Agar network kat bhi jaye, toh Region 1 ke log Region 1 ke database se baat karte rahenge aur Region 2 wale apne local database se. System chalta rahega (Available rahega). Jab internet wapis aayega, toh dono regions aapas mein data sync kar lenge. Lekin is dauran linearizability nahi hogi.
2. **Agar Single-Leader System Ho (CP):** Sochein main leader Region 1 mein betha hai. Agar Region 2 ke clients koi data write karna chahein ya linearizable read karna chahein, toh unki request ko synchronously Region 1 ke leader ke paas jana parega. Lekin beech mein toh tar kati hui hai!
3. **Nateeja (Outage):** Region 2 ke clients leader se baat nahi kar sakenge. Agar woh local follower se parhein ge toh data stale (purana) milega. Agar aap ki app ko har haal mein linearizable data chahiye, toh system ko Region 2 mein **kaam rokna parega (Unavailability/Outage)**.

---

#### The CAP theorem

Yeh masla sirf single-leader ya multi-leader ka nahi hai. Poori duniya ka koi bhi database agar linearizable banna chahta hai, toh usay is maut ke kooay se guzarna hi parega. Is mushkil trade-off ko hum **CAP Theorem** kehte hain:

* **CP (Consistent under network partitions):** Agar aap ki application ke liye linearizability (C) zaroori hai, aur network mein kharabi (P) aa jaye, toh disconnected nodes ko apna kaam rokna parega ya error dena parega (yani woh **Unavailable** ho jayenge).
* **AP (Available under network partitions):** Agar aap ko linearizability ki chinta nahi hai, toh har node partition (P) ke dauran bhi azaadana kaam karta rahega (yani system **Available** rahega), lekin data naya hone ki koi guarantee nahi hogi.

> **CAP Theorem Ka Asal Sach:** Eric Brewer ne 2000 mein yeh lafz diya tha jis se NoSQL movement shuru hui. Lekin aam zindagi mein log isay galat samajhte hain. Formal definition ke mutabaq CAP bohot choti satah par baat karta hai: Yeh sirf **Consistency mein Linearizability** ko maanta hai, aur **Fault mein sirf Network Partition** ko (jo ke Google ke mutabaq database ke kul haadsaat ka **8% se bhi kam** wajah banti hain). Yeh network delay ya dead nodes ke baare mein kuch nahi batata. Is liye practical system design mein is ki ahmiyat ab sirf tareekhi (historical interest) reh gayi hai.
> Is ko behtar karne ke liye **PACELC principle** banaya gaya: Agar Partition (P) hai toh Choose karo Availability (A) aur Consistency (C) mein se; Else (E) yani agar network theek chal raha hai, toh choose karo Latency (L) aur Consistency (C) mein se.

---

#### The Unhelpful CAP Theorem

Log aksar CAP ko aam zuban mein kehte hain: *"Consistency, Availability, Partition Tolerance: in teeno mein se koi do chun lo!"* **Yeh bilkul galat aur gumrah-kun (misleading) tareeqa hai.**

* **Partition Aap Ki Choice Nahi Hai:** Partition Tolerance koi aisi cheez nahi hai jo aap dukan se khareed sakein. Yeh ek network ki kharabi (fault) hai jo aap ki marzi ke bina **har haal mein hogi**. Aap network ko khatam nahi kar sakte.
* **Asal Lafz:** CAP ko kehne ka sahi tareeqa yeh hai: **"Partition ke waqt Consistent ya Available mein se koi aik chun lo!"** Jab network theek chal raha ho, system consistent bhi hota hai aur available bhi. Galti sirf tab hoti hai jab tar katti hai.

---

#### Linearizability and network delays

Aap ko yeh sun kar hairat hogi ke itni achi guarantee hone ke bawajood, real-world mein bohot hi kam systems linearizable hote hain. Hatta ke aap ke computer ke andar jo **RAM (Memory)** lagi hai, woh bhi multi-core CPU par **linearizable nahi hoti!**

* **CPU Core Ka Cache Buffer:** Agar CPU Core 1 RAM ke kisi address par kuch likhta hai, toh Core 2 ko foran woh badlao nazar nahi aata jab ke tak koi memory barrier ya fence use na kiya jaye.
* **Kyun?** Kyunke har CPU core ke paas apna tez speed cache memory hoti hai. Data pehle cache mein likha jata hai aur aaram se main RAM mein copy hota hai. Agar hum har core ko linearizable banayein, toh computer ki speed bohot slow ho jayegi. Yahan qurbani fault tolerance ke liye nahi balkay **Performance (Speed)** ke liye di gayi hai.

Distributed databases bhi linearizability ko fault tolerance se zyada **Speed (Latency)** ke liye chorte hain.

```
Linearizable Storage Ki Response Time ∝ Network Delay Ki Unpredictability (Ghair-yakeeni surat-e-haal)

```

**Attiya aur Welch Ka Proof:**
In do scientists ne mathematically sabit kiya hai ke agar aap ko linearizability chahiye, toh aap ke read aur write requests ka response time har haal mein network ke sab se bade jhatkay (delays) ke barabar slow ho jayega. Chunke computer networks (jaise internet) mein delays ka koi hisab nahi hota, is liye linearizable systems ka response time hamesha high (slow) rahega.

Is se tez koi algorithm duniya mein banana mumkin hi nahi hai. Is liye agar aap ka system aisa hai jahan speed (low latency) bohot zaroori hai, toh aap ko weaker consistency models ki taraf jana parega. Chapter 13 mein hum dekhenge ke bina linearizability ke bhi kaise system ko sahi chalaya ja sakta hai.

---