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

## ID Generators and Logical Clocks

Bohot saari applications mein jab database mein koi naya record banta hai, toh hamein usay aik unique ID (shanaas) deni parti hai taake hum us record ko as a **Primary Key** use kar sakein.

Agar database sirf **aik single computer (single-node)** par chal raha ho, toh wahan aik **Autoincrementing Integer** (yani aik aisa number jo har naye record par 1, 2, 3 karke khud ba khud barhta jaye) istemaal karna bohot aam aur asaan hai. Iska faida yeh hai ke yeh memory mein bohot kam jagah leta hai—sirf 64 bits (ya agar aap ko pakka yakeen ho ke aap ke records 4 billion se zyada nahi honge, toh 32 bits, lekin aam tor par aisa sochna khatarnak hota hai).

Autoincrementing IDs ka aik aur sab se bara faida yeh hai ke **IDs ki tarteeb (order) se aap ko yeh pata chal jata hai ke kaun sa record pehle bana tha.**

---

#### Figure 10-8 Ka Breakdown: Chat Application Mein Autoincrementing IDs

Writer ne **Figure 10-8** mein aik chat app ki example di hai jahan ek single-node ID generator messages ko auto-incrementing numbers assign kar raha hai taake chat ki baatein tarteeb mein rahein.

<div align="center">
  <img src="./images/08.png" width="700"/>
</div>

**Step-by-Step Breakdown:**

1. **Aaliyah Ka Message:** Aaliyah sab se pehle aik sawaal poochti hai: *"Where shall we go for lunch today?"*. Yeh request ID Generator node ke paas jati hai. Node counter ko barha kar **ID: 1** karta hai aur message ko save karke Aaliyah ko wapis bhej deta hai.
2. **Caleb Ka Message (Concurrent):** Bilkul isi dauran Caleb bhi aik message bhejta hai: *"Hi everybody"*. Chunke yeh Aaliyah ke message ke sath hi chal raha hai (concurrent hai), ID generator counter ko mazeed barha kar isay **ID: 2** de deta hai. Linearizability yahan yeh kehti hai ke jab tak dono messages unique hain, inki aapas ki tarteeb kuch bhi ho, farq nahi parta.
3. **Bryce Ka Jawab:** Bryce Aaliyah ka sawaal parhta hai aur uske BAAD jawab likhta hai: *"@Aaliyah I vote falafel shop"*. Chunke Bryce ka action Aaliyah ke message ke mukammal hone ke strictly baad hua hai, is liye database counter ko barha kar isay **ID: 3** deta hai.

Jab app in messages ko screen par dikhaye gi, toh woh IDs ki tarteeb (1, 2, 3) ke mutabaq dikhaye gi, jis se poori chat ki kahani bilkul sahi samajh aayegi.

**Single-Node Generator Kaise Kaam Karta Hai?**
Aik computer ke andar isay banana bohot asaan hai. Aap CPU ki **Atomic Increment Instruction** (jaise fetch-and-add) use karte hain, jis se agar aik se zyada threads bhi counter ko barhana chahein toh woh safely aik doosre ko kharab kiye bina counter barha deti hain. Agar computer crash ho jaye, toh counter zero na ho (duplicate IDs se bachne ke liye), is liye is counter ko disk par save (persist) karna parta hai.

Lekin asli distributed systems mein is single-node design ke teen baray maslay hain:

* **Single Point of Failure (SPOF):** Agar yeh akela computer kharab ya down ho gaya, toh poora system ruk jayega kyunke kisi ko ID nahi milegi (Fault-tolerant nahi hai).
* **Sust Speed (Network Latency):** Agar aap duniya ke doosre kone (region) mein baithe hain aur aap ko aik record banane ke liye har dafa duniya ke is kone wale single node se ID mangni paray, toh poori duniya ka chakkar (round-trip) lagane mein bohot waqt zaya hoga.
* **Bottleneck (Rukawat):** Agar aap ki website par aik hi second mein lakhoon log writes kar rahe hain, toh yeh akela node saari requests ka bojh nahi sambhal sakay ga.

In masail se nipatne ke liye log alternative tareeqe use karte hain, jin ke design decisions aur trade-offs niche table mein samjhaye gaye hain:

| ID Generator Approach | Yeh Kaise Kaam Karta Hai? | Iska Bara Nuksan (Trade-off) |
| --- | --- | --- |
| **Sharded ID assignment** | Database ko chunks (shards) mein baant diya jata hai. Misaal ke tor par Node 1 sirf Even numbers (2, 4, 6) dega aur Node 2 sirf Odd numbers (1, 3, 5) dega. Ya ID ke kuch bits shard number ke liye rakh diye jate hain. | **Ordering khatam ho jati hai.** Agar do messages ki IDs 16 aur 17 hain, toh aap daaway se nahi keh sakte ke 16 pehle aaya tha, kyunke dono alag nodes ne diye hain aur ho sakta hai aik node ka time aage ho. |
| **Preallocated blocks of IDs** | Main node se har computer blocks utha leta hai. Jaise Node A ne 1 se 1000 tak numbers rakh liye, aur Node B ne 1001 se 2000 tak. Jab numbers khatam hone lagein, toh naya block mang liya jata hai. | **Is mein bhi tarteeb kharab hoti hai.** Ho sakta hai aik naya message Node B se ID 1005 utha le aur uske baad aane wala message Node A se ID 5 utha le, jo ke dekhne mein purani lagegi. |
| **Random UUIDs (v4)** | Universally Unique Identifiers (128-bit lamba random number) jo har computer local level par bina kisi se pooche khud generate kar leta hai. | **Order bilkul random hota hai.** Do UUIDs ko compare karke aap kabhi nahi jaan sakte ke kaun si naye waqt mein bani hai. Space bhi zyada (128 bits) leti hai. |
| **Wall-Clock Timestamps (Unique banaya hua)** | Computer ki aam gari (Physical Clock) ka time sab se upar (most significant bits) rakha jata hai, aur baki bache bits mein shard ID ya random number dal diya jata hai taake ID unique rahe. (Jaise UUID v7, Twitter Snowflake, ULID, MongoDB ObjectID). | **Linearizable nahi hote.** Agar aik computer ki clock thodi fast chal rahi ho aur doosre ki slow (Clock Skew), toh baad mein aane wale event ko purana timestamp mil sakta hai. Non-monotonic jumps ki wajah se aik hi node par bhi order ulta ho sakta hai. |

---

### Logical Clocks

Upar hum ne dekha ke physical clocks (jo aam zindagi ke ghantay, seconds, ya microseconds naapti hain) distributed systems mein order ke liye dhoka de sakti hain. Is liye distributed systems mein aik aur tarah ki clock use hoti hai jisay **Logical Clock** kehte hain.

Physical clock ke baraks, logical clock koi samandar ya suraj ka waqt nahi batati, balkay yeh **sirf events (kaam hone ke numbers) ko ginti hai.** Is ka timestamp aap ko yeh nahi batayega ke abhi dopehr ke 3 bajay hain ya shaam ke 5, lekin agar aap do logical timestamps ko aapas mein compare karenge, toh aap $100\%$ sahi bata sakenge ke kaun sa kaam pehle hua aur kaun sa baad mein.

Logical Clock ki teen bunyadi shartain hoti hain:

1. Timestamps size mein chote (compact) aur bilkul unique hon.
2. Aap kisi bhi do timestamps ko compare karke unki aik pakki tarteeb (**Total Ordering**) nikal sakein.
3. Unka order **Causality (Wajah aur Asar)** ke mutabaq ho. Yani agar Operation A pehle hua tha aur uski wajah se Operation B hua ($\text{A} \to \text{B}$), toh A ka timestamp har haal mein B se chota hona chahiye.

Distributed ID generators (jaise Snowflake) uniqueness toh dete hain lekin causal ordering ki teesri shart poori nahi karte.

---

#### Lamport timestamps

Leslie Lamport ne 1978 mein aik bohot hi sasta aur simple tareeqa diya jo causal ordering ki shart ko poora karta hai aur isay distributed ID generator ke tor par use kiya jata hai. Isay **Lamport Clock** kehte hain.

> **Zaroori Bareeki (Caveat):** Lamport clocks total ordering toh deti hain lekin yeh **Linearizability faraham nahi karti n**. Yani yeh aap ko yeh guarantee nahi de sakti ke data bilkul fresh ya up-to-date hai ya nahi. Yeh sirf aur sirf events ko aisi IDs deti hain ke agar Event A pehle hua tha Event B se, toh A ki ID hamesha B se choti hogi.

---

#### Figure 10-9 Ka Breakdown: Lamport Clocks In Action

**Figure 10-9** mein dikhaya gaya hai ke Lamport clock isi chat wali example mein kaise chalegi.

<div align="center">
  <img src="./images/09.png" width="700"/>
</div>

* **Timestamp Ki Shakal:** Lamport timestamp do cheezon ka jora (pair) hota hai: `(counter, node ID)`. Misaal ke tor par `(1, "Aaliyah")`. Agar do alag computers ka counter same ho bhi jaye, toh unke naam (node ID) alag hone ki wajah se poora timestamp unique ho jata hai.
* **Algorithm Ke Do Rules (Bacho ki tarah samajhein):**
1. Jab bhi koi node apna koi kaam karega ya message bhejega, woh apne local counter mein $+1$ karega aur naya number attach karega.
2. Jab bhi koi node kisi doosre node ka message receive karega, woh us message ke andar ka counter dekhega. **Agar samne wale ka counter mere local counter se bara hai, toh main apna counter jump karwa kar uske barabar kar loonga** aur phir apna counter $+1$ kar loonga.



**Diagram Ke Steps Ka Breakdown:**

1. Shuru mein Aaliyah aur Caleb dono ka counter $0$ hai. Dono aik doosre se be-khabar (concurrently) messages bhejte hain.
* Aaliyah apna counter 1 karti hai. Message banta hai: `id: (1, "Aaliyah")`
* Caleb apna counter 1 karta hai. Message banta hai: `id: (1, "Caleb")`


2. Bryce ke paas jab yeh dono messages pohanchte hain, toh Bryce dekhta hai ke dono ka counter 1 hai, jo ke uske apne local counter (0) se bara hai. Bryce apna counter badha kar 1 kar leta hai.
3. Ab Bryce Aaliyah ke sawaal ka jawab bhejta hai. Rule 1 ke mutabaq, Bryce apne counter ko $+1$ karke 2 karta hai. Jawab ka message banta hai: `id: (2, "Bryce")`.

**Timestamps Ko Compare Karne Ka Rule:**
Jab do Lamport timestamps ko compare karna ho, toh pehle unke **Counter** ko dekha jata hai. Jis ka counter bara, woh naya. Agar counter barabar ho jayein (jaise Aaliyah aur Caleb ka counter 1 hai), toh unke **Node ID** (strings) ko ABC ke mutabaq (lexicographically) compare kiya jata hai. Chunke "Aaliyah" alphabet mein "Caleb" se pehle aati hai, is liye:


$$\text{(1, "Aaliyah")} < \text{(1, "Caleb")} < \text{(2, "Bryce")}$$


Is tarah system ko aik mukammal aur saaf tarteeb (Total Ordering) mil jati hai.

---

#### Hybrid logical clocks (HLC)

Lamport timestamps bohot acche hain, lekin un mein do baray maslay hain:

* Inka asli physical time (ghari ke waqt) se koi lena dena nahi hota. Aap database mein yeh query nahi chala sakte ke *"Mujhe 20 June ko aane wale saare messages dikhao"*. Uske liye physical time alag se save karna parta hai.
* Agar do nodes aapas mein kabhi baat hi na karein, toh aik ka counter bohot aage nikal sakta hai aur doosre ka piche reh sakta hai, halanqe dono aik hi waqt par kaam kar rahe hon.

Is maslay ka hal **Hybrid Logical Clock (HLC)** hai. Yeh physical clock (seconds/microseconds) aur Lamport clock ke counters ko aapas mein jor deti hai.

* Yeh physical clock ki tarah seconds ginti hai, lekin jab aik node kisi doosre node se aisa timestamp dekhta hai jo uske apne time se aage hai, toh yeh apni clock ko **aage khainch (jump) leti hai**.
* Agar physical clock galti se piche chali jaye (NTP adjustment ki wajah se), tab bhi HLC ka counter hamesha aage hi barhta hai (monotonically forward), jis se time kabhi piche nahi murta.
* Is wajah se HLC ke timestamp ko aap aam ghari ka waqt bhi maan sakte hain aur is mein causality ($\text{happens-before}$) ki guarantee bhi mil jati hai. Is ke liye kisi expensive hardware (jaise GPS) ki zaroorat nahi hoti. **CockroachDB** is ki zinda misaal hai.

---

#### Lamport/hybrid logical clocks versus vector clocks

Snapshot Isolation (Chapter 2) implement karne ke liye transactions ko unique IDs deni parti hain taake har transaction sirf apne se choti ID wale writes dekh sakay. Lamport aur Hybrid clocks yeh IDs banane ke liye bohot behtareen hain kyunke yeh causality barkarar rakhti hain.

Lekin in dono mein aik cheez yaad rakhna zaroori hai: **Jab do concurrent events hote hain, toh yeh algorithms unki tarteeb ka faisla tuke (arbitrary order, jaise string comparison) se karte hain.** Aap do timestamps ko dekh kar yeh nahi bata sakte ke yeh dono sach mein aik hi waqt chal rahe the ya aik pehle hua tha. (Figure 10-9 mein Aaliyah aur Caleb ka counter same tha toh pata chal gaya, par agar counter alag ho toh guessing mumkin nahi).

* **Vector Clocks:** Agar aap ko har haal mein yeh pata karna ho ke **kaun se records concurrent hain**, toh aap ko **Vector Clock** use karni paregi. Vector clock mein har node ka alag counter hota hai aur har write ke sath saare nodes ke counters ki poori list (vector) save karni parti hai.
* **Trade-off:** Vector clock ka sab se bara nuksan yeh hai ke yeh memory mein bohot zyada jagah leti hai—jitne system mein nodes honge, utne integers har ID ke sath save karne parenge ($O(N)$ space complexity).

---

### Linearizable ID Generators

Hamein lagta hai ke Lamport ya Hybrid clocks kaafi hain, lekin inki ordering **Linearizability se kamzor hoti hai.** Linearizability ki shart yeh hai ke agar Request A duniya ke kisi bhi kone mein Request B ke shuru hone se pehle khatam ho chuki thi, toh B ki ID har haal mein A se bari honi chahiye, chahe un dono ne aapas mein kabhi communication na bhi ki ho. Lamport clock sirf un nodes ka order sahi karti hai jo aapas mein data share kar chuke hon.

Agar generator linearizable na ho, toh kitna bara hadsa ho sakta hai, yeh **Figure 10-10** mein dikhaya gaya hai.

---

#### Figure 10-10 Ka Breakdown: Privacy Ka Qatal (Non-linearizable ID Generator)

Sochein aik social media website (jaise Facebook ya Instagram) hai jahan ek User A apni aik embarrassment (sharminda karne wali) photo sirf apne doston ke sath share karna chahta hai.

<div align="center">
  <img src="./images/10.png" width="700"/>
</div>

**Step-by-Step Scenario:**

1. **Step 1 (Account setting change):** User A ka account pehle public tha. Woh apne laptop se setting badal kar **"Private"** karta hai. Yeh request **Accounts DB** par jati hai, jahan counter 14 hota hai aur is write ko timestamp milta hai: **write ts = 15**.
2. **Step 2 (Photo upload):** Account private karne ke FORAN BAAD, User A apne mobile phone se woh photo upload kar deta hai. Yeh request **Photos DB** ke paas jati hai.
3. **The Flaw (Masla):** Chunke dono databases alag hain aur un mein Lamport/Hybrid clock chal rahi hai, aur Photos DB ka counter thoda piche chal raha tha (counter = 11), toh photo upload ko naya timestamp mila: **write ts = 12**.
4. **Step 3 (The Unauthorized Viewer):** Ab aik bacha (Viewer) jo User A ka dost nahi hai, thodi der baad profile kholta hai. Uska parhna (Read) MVCC snapshot isolation use kar raha hai aur usay timestamp milta hai: **read at ts = 13**.
5. **Nateeja:** System jab check karega ke kya viewer ko photo dikhani hai, toh woh Accounts DB mein dekhega ke *ts = 13* par account ka kya status tha. Chunke account private *ts = 15* par hua tha, toh database kahega ke *ts = 13* par account **"Public"** tha! Aur doosri taraf photo *ts = 12* par upload ho chuki thi (13 > 12), is liye system viewer ko woh khufia photo dikha dega! User ki privacy leak ho gayi!

**Ilaaj:** Is maslay ka sab se asaan hal yeh hai ke aik **Linearizable ID Generator** use kiya jaye, jo har haal mein photo upload ko account setting se bari ID (ts) assign karega, chahe dono databases alag hi kyun na hon.

---

#### Implementing a linearizable ID generator

Isay implement karne ka sab se seedha tareeqa yeh hai ke sach mein **aik single node** ko pooray cluster ke liye ID generator bana diya jaye. Us node ke paas sirf teen zimmedariyan hoti hain:

1. Counter ko atomically barhana aur return karna.
2. Crash se bachne ke liye counter ko disk par save (persist) karna.
3. Fault tolerance ke liye data ko **Single-Leader Replication** ke zariye doosre nodes par copy karna.

Distributed systems mein is design ko **Timestamp Oracle** kehte hain (jaise TiDB/TiKV mein use hota hai, jo Google ke Percolator se inspired hai).

**Optimization (Speed barhane ka tareeqa):**
Har aik ID request par disk par likhna aur replicate karna system ko bohot slow kar dega. Is liye naye design mein ID generator aik sath **IDs ka pura batch (jaise 1 se 1000 tak)** disk par aur replicas par save kar leta hai. Phir local memory se clients ko jaldi jaldi IDs deta rehta hai. Agar computer crash ho jaye, toh agla computer bacha hua batch chor kar naye batch (1001) se shuru karega. Kuch IDs zaya zaroor hongi, lekin kabhi duplicate ya out-of-order IDs nahi nikalengi.

> **Limitations (Hudood):** Aap is ID generator ko sharding (tukron mein) nahi baant sakte kyunke alag-alag shards linearizable order barkarar nahi rakh sakte. Aap isay alag regions mein bhi nahi phailaye sakte, is liye geographic distributed databases mein bhi saari duniya se requests ko aik hi main region ke node par ana parta hai.

---

#### Spanner Ka Alag Tareeqa: TrueTime

Agar aap single node par depend nahi karna chahte, toh Google Spanner wala tareeqa apna sakte hain. Spanner kisi single node se ID nahi mangta, balkay hardware support (**Atomic Clocks** aur **GPS receivers**) par bharosa karta hai.

* Spanner ki clock sirf aik naya time nahi batati, balkay aik **Uncertainty Interval (shak ka waqt)** batati hai: $[t_{\text{earliest}}, t_{\text{latest}}]$. System ko pata hota hai ke asal time is range ke andar hi kahin hai.
* Spanner ka rule yeh hai ke jab bhi koi naya write hota hai, database tab tak rukta hai (**Wait** karta hai) jab tak us uncertainty ka poora waqt guzar na jaye.
* Is wait karne ka faida yeh hota hai ke agar Request B, Request A ke mukammal hone ke baad shuru hui hai, toh B ka timestamp har haal mein A se bara hi hoga. Is tarah bina kisi cross-region communication ke pooray globe mein linearizable IDs mil jati hain. Lekin iske liye bohot mehangay hardware aur software ki zaroorat hoti hai.

---

### Enforcing constraints using logical clocks

Hum ne pehle parha tha ke linearizable CAS (Compare-and-Set) operation ke zariye hum locks aur uniqueness constraints (jaise unique username) safely implement kar sakte hain. Toh kya hum yahi kaam kisi logical clock ya linearizable ID generator se nahi kar sakte?

**Jawab hai: Poori tarah nahi!**

Sochein do log aik hi username "Ali" register karna chahte hain. Hum dono requests ko logical clock ke zariye timestamps de dete hain. Phir hum kehte hain ke jis ka timestamp chota hoga, wahi winner hoga. Chunke clock linearizable hai, hamein pata hai ke future mein aane wali requests ka timestamp hamesha bada hoga, toh winner badal nahi sakta.

Lekin asli masla abhi bhi bacha hua hai: **Kisi aik node ko yeh kaise pata chalega ke uska timestamp hi poori duniya mein sab se chota (lowest) hai?**

* Yeh pakka karne ke liye us node ko poore cluster ke **har aik node se baat karni paregi** taake woh confirm kar sakay ke kisi aur ke paas is se chota timestamp toh nahi para.
* Agar un nodes mein se aik computer bhi mar gaya (fail ho gaya) ya network partition ki wajah se us se rabta toot gaya, toh poora system wahin ruk (freeze) jayega, kyunke hum baki nodes ke timestamps ke bina faisla nahi kar sakte.

Aisa system fault-tolerant nahi kehla sakta. Is liye distributed systems mein locks, leases, aur unique constraints ko kharabiyon ke bawajood safely chalane ke liye sirf logical clocks ya IDs kaafi nahi hain. Hamein is se zyada takatwar cheez chahiye, aur us ka naam hai: **Consensus (Ikhtilaf-e-rai ko khatam karna)**, jisay hum agay mazeed gehrai se parhenge.

---

