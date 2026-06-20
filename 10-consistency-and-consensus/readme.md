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