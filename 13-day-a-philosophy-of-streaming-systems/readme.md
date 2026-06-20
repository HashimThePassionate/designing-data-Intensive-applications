# A Philosophy of Streaming Systems

If a thing be ordained to another as to its end, its last end cannot consist in the preservation of its being. Hence a captain does not intend as a last end, the preservation of the ship entrusted to him, since a ship is ordained to something else as its end, viz. to navigation. (Often quoted as: If the highest aim of a captain was the preserve his ship, he would keep it in port forever.)
—St. Thomas Aquinas, Summa Theologica (1265–1274)

---

Is sadiyon purani misaal ka aasan matlab yeh hai ke agar kisi jahaaz ke kaptaan ka maqsad sirf apne jahaaz ko bacha kar surakshet rakhna hota, toh woh usay kabhi samandar mein lekar hi na jata, balkay hamesha dukan (port) par khara rakhta. Lekin jahaaz ka asli maqsad bandargah par kharay rehna nahi, balkay samandar ke thapedon ko jhelte hue safar (**Navigation**) mukammal karna hai.

Bilkul isi tarah, software systems ka maqsad sirf khamkhah chalte rehna nahi, balkay mushkilat ke bawajood business ke kaamo ko sahi tareeqay se chalana hai.

Chapter 2 mein hum ne baat ki thi ke hamara asli maqsad aise data systems banana hai jo **Reliable** (bharosay mand), **Scalable** (bojh sambhalne wale), aur **Maintainable** (paidaish se aakhir tak chalane mein asaan) hon. Poori kitaab mein hum ne inhi baaton par focus kiya hai: kharabiyon se bachne ke algorithms (fault-tolerance), data ko tukron mein baantna (sharding), aur system ko naye roop mein dhalna (evolution).

Is naye chapter mein hum in saare bikhray hue ideas ko aik sath jorrein ge. Khaas tor par Chapter 12 ke **Streaming aur Event-Driven Architecture** ka sahara lekar hum software development ka ek naya falsafa (philosophy) tayaar karenge jo in saare maqsadon ko poora karega. Yeh chapter baqi chapters se thoda alag aur *opinionated* (aik makhsoos soch par paka zor dene wala) hai, jahan hum mukhtalif tareeqon ka muqabla karne ke bajaye aik behtareen tareeqay ki gehrai mein utrein ge.

---

## Data Integration

Poori kitaab mein hum ne baar baar ek hi baat seekhi hai ke computer ki duniya mein kisi bhi aik maslay ko hal karne ke kayi tareeqay hote hain, aur har tareeqay ka apna ek faida aur nuksan (trade-off) hota hai:

* Chapter 4 mein storage engines parhte waqt hum ne dekha ke LSM-Trees alag kaam ke liye hain, B-Trees alag kaam ke liye, aur Column-Oriented storage bilkul alag kaam ke liye.
* Chapter 6 mein replication ke andar Single-Leader, Multi-Leader, aur Leaderless architectures ke apne apne fayde hain.

Agar aap ka aik simple masla hai ke *"Mujhe data save karna hai aur baad mein parhna hai"*, toh poore jahan mein iska koi aik 'perfect' solution nahi hai. Halat ke mutabaq faisla badal jata hai. Kisi aik software ke andar saare jahan ke features thonse nahi ja sakte. Agar koi company aik hi software mein sab kuch daalna chahegi, toh uske saare features ghatiya karkardagi (poor implementation) ka shikaar ho jayenge compared to specialized tools.

Is liye har tool—chahe naya "general-purpose" database hi kyun na ho—ek makhsoos kaam ke dhabbe (**Usage Pattern**) ke liye design kiya jata hai.

Hum software engineers ke samne pehla challenge yeh aata hai ke hum pehchanein ke kaun sa tool kis halat mein fit bahta hai. Software bechne wali companies (vendors) kabhi aap ko yeh nahi batayengi ke unka software kahan nakaam ho jata hai, lekin pichlay chapters ko parhne ke baad aap ke paas itni technical samajh aa chuki hai ke aap un ke daaway ke piche chhupe nuksanat (trade-offs) ko asani se parh sakein.

Lekin agar aap tools ki selection samajh bhi jayein, tab bhi aik doosra bara challenge baqi rehta hai. Ek barri aur complex application mein data kayi alag alag kaamo ke liye aik sath use hota hai, aur poore jahan ka koi aik akela software un saare kaamo ko akele perfect handle nahi kar sakta. Is liye, aap ko majbooran **kayi alag alag softwares ko aprop mein jorr kar** apni application faraham karni parti hai.

---

### Combining Specialized Tools by Deriving Data

Is ki ek bohot hi aam aur simple misaal dekhte hain. Aap ko apni website par ek aam transactional database (**OLTP DB**) chahiye jahan user ka data paka save ho, lekin sath hi aap ko ek **Full-Text Search Index** (jaise Elasticsearch) bhi chahiye taake users website par koi bhi lafz likh kar cheezein search kar sakein.

Halanqe PostgreSQL jaise modern databases mein search ka feature built-in hota hai jo choti apps ke liye kaafi hai, lekin agar aap ko Amazon ya Netflix jaisi advance search facilities chahiye, toh aap ko makhsoos search tools use karne parenge. Is ke ulat, search indexes data ko hamesha ke liye durable save rakhne ke liye acche nahi hote. Is liye aap ko do alag tools ko aprop mein milana hi parta hai.

Jaise jaise data ke roop (representations) barhte jaate hain, data integration ka yeh masla mushkil hota jata hai. Database aur search index ke ilawa, ho sakta hai aap ko data analytical systems (Data Warehouses/Spark) mein bhejni ho, speed barhane ke liye caches maintain karni hon, ya machine learning recommendation engines ko feed dena ho, ya data badalne par live notifications bhejni hon.

---

#### Reasoning about dataflows

Jab aik hi data ko alag alag dhabbo (access patterns) ke liye kayi storage systems mein rakhna paray, toh aap ke dimaagh mein **Inputs aur Outputs** ka poora naksha bilkul crystal clear hona chahiye:

* Data sab se pehle kis database mein write ho raha hai?
* Kaun sa naya view kis source se nikala (**derive**) ja raha hai?
* Hum data ko sahi format mein sahi jagah par kaise pohnchayein?

Misaal ke tor par, behtareen tareeqa yeh hota hai ke data sab se pehle main **System-of-Record** database (asli paka store) mein write ho. Uske baad wahan se badlao ko pakra jaye (**Change Data Capture - CDC**) aur bilkul usi exact order mein search index par apply kar diya jaye. Agar index ko update karne ka akela rasta sirf yeh CDC stream ho, toh aap ka search index hamesha main database ke sath perfect sync (consistent) rahega. New input dene ka akela rasta sirf main database hoga.

Agar aap application ke code ko ijazat de dein ke woh database mein bhi khud likhe aur search index mein bhi direct write kare, toh pichlay chapter wala hadsa (**Figure 12-4 wala dual write masla**) dobara ho jayega, jahan do concurrent clients aapas mein lar parenge aur dono storage systems requests ko alag alag order mein process karke permanently out-of-sync ho jayenge.

> **Bacho ka Asool:** Agar aap saare user inputs ko aik single channel (leader) se guzarlein jo unhein ek pakki tarteeb (**Total Ordering**) de day, toh baqi saare systems mein data jorrna bacho ka khel ban jata hai (State Machine Replication approach). Chahe aap CDC use karein ya Event Sourcing, sab se ahem asool pooray system ka ek single timeline (total order) tay karna hai.

Naye derived systems ko event log ke zariye update karna hamesha **Deterministic aur Idempotent** (aik jaisa asar dene wala) banaya ja sakta hai, jis se faults se recover karna bohot simple ho jata hai.

---

#### Derived data versus distributed transactions

Do alag systems ko sync rakhne ka purana aur classic tareeqa distributed transactions (Two-Phase Commit - 2PC) chalana tha. Toh naye zamane ka yeh **Derived Data Systems** wala tareeqa distributed transactions ke muqable mein kaisa hai?

Theoretical level par dono aik hi maqsad hal karte hain lekin unka rasta alag hai:

* Distributed Transactions aik atomic commit protocol use karti hain jo sab ko aik jhatkay mein save karta hai (Immediate read-your-own-writes guarantee).
* Log-Based Derived Data systems kamyabi haasil karne ke liye **Deterministic Retry aur Idempotence** ka sahara lete hain (Asynchronous nature, eventual consistency).

**Sub se Bara Farq:** Transaction wale systems guaranteed naya data foran dikha dete hain, jabke derived data systems background mein aaram se (asynchronously) update hote hain, is liye un mein reads thodi dair se up-to-date hoti hain.

Distributed transactions (XA protocol) ka fault tolerance bohot ghatiya hota hai aur yeh speed ko slow kar deti hain, jis se inka scale bohot limited ho jata hai. Jab tak poori duniya mein koi naya behtareen transaction protocol mashhoor nahi ho jata, tab tak **Log-Based Derived Data** hi alag alag systems ko jorrne ka sab se kamyab aur umeed-warr tareeqa hai.

Engineering mein yeh keh dena kafi nahi hai ke *"Bhai, eventual consistency toh honi hi hai, bas bardasht karo"*. Hamein developers ko sahi rasta dikhana hoga. Hum is chapter mein agay chal kar dekhenge ke kaise asynchronous logs ke upar bhi strong guarantees haasil ki ja sakti hain.

---

#### The limits of total ordering

Chote systems ke liye poore cluster ke kaamo ko aik seedhi tarteeb (**Totally Ordered Log**) dena bilkul feasible hai (jaise single-leader databases aaram se karte hain). Lekin jaise hi aap workload ko bohot baray paimane (scale) par le jaate hain, is total order ki deewar (limitations) samne aane lagti hain:

* **Throughput Ki Hadd:** Total order banane ke liye saare events ko aik hi single leader node se guzarna parta hai jo faisla karta hai. Agar data ki raftar aik computer ki capacity se barh jaye, aap ko log ko tukron mein baantna (**Shard**) parega. Jaise hi do alag shards banay, unke aapas ke messages ka order aage-piche (ambiguous) ho jata hai.
* **Geographic Distributed Datacenters:** Agar aap ke servers alag alag mulkon (regions) mein hain taake aik datacenter tabah hone par bhi app chalti rahay, toh har datacenter ka apna alag leader hota hai. Network latency ki wajah se alag mulkon se aane wale events ka aprop mein koi fixed order nahi reh pata.
* **Microservices Architecture:** Is design mein har microservice ka apna azaad database hota hai jo state share nahi karta. Jab do alag services se do alag events nikalte hain, toh unka aprop mein koi timeline tay nahi hota.
* **Offline Client-Side State:** Agar koi mobile app user ka click foran local screen par bina server ke ijazat ke save kar leti hai aur offline kaam karti hai, toh clients aur servers ko events hamesha alag alag order mein dikhenge.

Formal theory mein total order tay karne ko **Total Order Broadcast** kehte hain, jo ke **Consensus (Chapter 10)** ke barabar hai. Zyadatar consensus algorithms sirf tab tak kaam karte hain jab tak aik node poore load ko sambhal sakay, un mein hazaron nodes par voting baantne ka koi sasta nizam maujood nahi hai.

---

##### Ordering events to capture causality

Agar do events ke beech aprop mein koi rishta ya connection nahi hai, toh order aage-piche hone se koi farq nahi parta, unhein tuke se kisi bhi tarteeb mein rakha ja sakta hai. Ek hi row ke updates ko hum same partition key de kar aik shard mein lock kar dete hain. Lekin kabhi kabhi **Causality (Wajah aur Asar ka rishta)** bohot barik tareeqay se samhne aata hai, jise samajhna bohot zaroori hai.

**Bacho ki Tarah Ek Real-World Example Samajhein:**
Sochein ek social network (jaise Facebook) par do users hain jo pehle aprop mein relationship mein the lekin ab unka break-up (talooq khatam) ho gaya hai.

1. **Event 1 (Unfriend):** Pehla user gusse mein ex-partner ko unfriend (block) kar deta hai.
2. **Event 2 (Send Message):** Unfriend karne ke FORAN BAAD, woh user apne baki bache doston ko ek rude (bura) message bhejta hai jis mein ex-partner ki burayi likhi hoti hai.

User ka maqsad bilkul saaf tha: Chunke us ne ex-partner ko pehle block kar diya tha, is liye usay yeh bura message har haal mein **nazar nahi aana chahiye**.

Lekin sochein agar system friendship status ko aik alag database shard mein save karta hai aur messages ko doosre database shard mein. Agar un dono shards ke beech **Total Order kho jaye**, aur aage notifications bhejne wali service ke paas **Message-Send Event pehle pohanch jaye aur Unfriend Event 2 second baad pohanche**—toh system galti se ex-partner ke mobile par push notification bhej dega ke *"Aap ke ex ne ek naya message bhejha hai!"* Privacy ka qatal ho gaya!

Yahan notification asal mein messages aur friend list ka beech ka ek distributed join tha. Is time dependency ke maslay ka koi aik simple hal software engineering mein abhi tak nahi mila, lekin teen (3) shuruati raaste maujood hain:

* **Logical Timestamps:** Yeh bina kisi central coordination ke total ordering de sakte hain (Lamport clocks), lekin is mein consumer code ko out-of-order events handle karne ke liye extra dimaagh aur metadata lagana parta hai.
* **Causal Reference ID:** Jab user koi naya action kare, toh app purane dekhe hue system state ki unique ID naye event ke andar reference ke tor par paki likh kar bhej day taake causality ka nishan jura rahay.
* **Conflict Resolution Algorithms:** Yeh unexpected order mein aane wale data ke state ko sambhal toh letay hain, lekin agar action ka koi external side effect ho chuka ho (jaise user ke mobile par notification chali jana), toh yeh usay wapis hawa mein urrah nahi sakte.

Future mein shayad distributed computing mein aise naye patterns aayenge jo bina kisi central bottleneck (total order broadcast) ke causality ko safely aur efficiently capture kar sakenge.

---

## Batch and Stream Processing

Data integration ka asli maqsad yeh hai ke data apny sahi roop mein company ki saari sahi jagahong par safely pohanch jaye. Is ke liye data ko ingest karna, badalna (transform), do tables ko jorrna (join), filter karna, aggregates nikalna, aur AI models train karna parta hai. **Batch aur Stream Processors** hi is maqsad ko hal karne ke hathiyar hain. Unke final outputs search indexes, materialized views, aur recommendation reports bante hain.

Jaise hum ne pichlay do chapters mein parha, batch aur stream processing ke andar ke zyadatar asool bilkul same hote hain. Sab se bunyadi farq sirf itna hai ke **Stream processors Unbounded (na-khatam hone wale) data par har waqt chalte hain, jabke batch processing ka input finite (makhsoos size) ka hota hai.**

---

### Maintaining materialized views

Batch processing ka poora dhabba **Functional Programming** jaisa hota hai. Yeh pure aur deterministic functions ko promote karta hai, jahan output sirf aur sirf input par depend karta hai. Input read-only hota hai aur output append-only, jis se system mein koi side effects nahi aate. Stream processing bhi bilkul aisi hi hai, bas yeh operators ko ek managed aur fault-tolerant **State** (yaad-dasht) rakhne ki extra ijazat deti hai.

Deterministic functions ka faida sirf fault tolerance mein nahi hota, balkay is se company ke andar data ke bahao (**Dataflows**) ko samajhna bohot simple ho jata hai. Chahe aap ka derived data koi search index ho, statistical model ho, ya koi cache ho—hamesha yeh sochna fayda-mand hota hai ke yeh aik **Data Pipeline** hai jahan aik cheez se doosri cheez nikaali ja rahi hai. Main system ke state changes functional code se guzar kar derived systems par apply ho rahe hain.

Sochein agar hum secondary indexes ki tarah derived systems ko bhi main database transaction ke andar hi **Synchronously** update karna shuru kar dein? Kar toh sakte hain, lekin **Asynchrony (background mein aaram se kaam karna) hi log-based systems ko robust aur majboot banati hai.** * **Failure Containment (Kharabi ko rokna):** Asynchronous log ka faida yeh hai ke agar system ka aik hissa (jaise search index server) mar bhi jaye, toh kharabi wahi local level par ruk jati hai, main database chalta rehta hai. Iske baraks, distributed transactions mein agar aik bhi participant fail ho jaye, toh poori transaction abort ho jati hai, jis se kharabi pooray system mein aag ki tarah phail jati hai.

* **Cross-Shard Scale:** Sharded systems mein secondary indexes hamesha shard ki boundaries ko cross karte hain. Aise cross-shard raabton ko agar scalable aur reliable banana hai, toh index ko asynchronously maintain karna hi sab se behtareen solution hai.

---

### Reprocessing data for application evolution

Derived data ko zinda rakhne ke liye batch aur stream processing dono ka milap kamaal hai:

* **Stream Processing** input ke badlao ko low delay (kuch hi milliseconds) mein derived views tak pohnchati hai.
* **Batch Processing** pichlay kai saaloon ke jama hue historical data ko naye siray se reprocess karke ek bilkul naya view tayaar karne ke kaam aati hai.

Data ko naye siray se **Reprocess** karna system ko zinda rakhne aur naye features/requirements ke mutabaq badalney ka sab se shaandar mechanism hai. Agar aap data reprocess nahi kar sakte, toh aap database ke schema mein sirf choti moti changings hi kar sakenge (jaise naya optional column jorrna). Lekin reprocessing ke zariye aap poore ke poore dataset ka huliya badal kar usay aik bilkul naye data model mein dhal sakte hain taake naye requirements poori ho sakein.

---

#### Schema Migrations on Railways

Barri barri "Schema Migrations" sirf computers mein nahi hoti, balkay asli zindagi ke non-computer systems mein bhi sadiyon se ho rahi hain. Is ki ek bohot behtareen tareekhi misaal 19th-century ke England mein **Railways (train ki patriyon)** ki building mein milti hai.

Shuru shuru mein rail ki do patriyon ke beech ka fasla (**Gauge size**) tay karne ke liye mukhtalif companies ke alag alag standards the. Ek company ki train doosri company ki patri par nahi chal sakti thi, jis se poora train network aprop mein jorrna namumkin ho gaya tha.

Aakhir-kar, saal 1846 mein hukumat ne faisla kiya ke poore mulk mein aik hi **Standard Gauge** chalega. Ab purane chalte hue tracks ko naye size mein badalna tha—lekin aap live train lines ko mahino ya saaloon ke liye band toh nahi kar sakte the! Iska hal unho ne yeh nikala ke tracks ko **Dual Gauge / Mixed Gauge** mein badal diya:

* **The Third Rail Solution:** Unho ne purani do patriyon ke sath aik **Teesri Patri (Third Rail)** jorr di. Yeh kaam chote chote tukron mein aaram se chalta raha. Jab teesri patri lag gayi, toh standard train aur non-standard train dono aik hi track par safely chal sakti thin.
* Jab waqt ke sath saari purani trains standard size mein convert ho gayein, toh unho ne woh temporary teesri patri nikal kar phenk di!

**Asli Khushnuma Soch (The Software Connection):**
Database ke derived views hamein bilkul isi tarz par bina downtime ke **Gradual Schema Migration** (aaram se data badalna) ki ijazat dete hain. Agar aap ne poore dataset ka roop badalna hai, toh aap ko live database ko achanak band karke switch karne ka khatra mol lene ki bilkul zaroorat nahi hai!

Aap purane schema (View A) aur naye schema (View B) dono ko aik sath **Side-by-Side** chalayein, jo aik hi event log se azaadana data parh rahe hon.

1. Shuru mein aap $1\%$ users ko naye view (View B) par shift karein taake bugs aur karkardagi check ho sakay. Baqi $99\%$ log purane view par hi chalte rahenge.
2. Jaise jaise aap ka confidence barhay, aap naye view par users ki percentage barhate jayein ($10\%, 50\%, 100\%$).
3. Jab saare users naye view par safely shift ho jayein, toh purane view ke database ko shut down karke uske resources free karwa lein.

Is gradual migration ki khoobsurat baat yeh hai ke **har step poori tarah se Reversible (wapis palatne ke kabil) hota hai**. Agar naye code mein koi kharabi aayi, aap aik second mein router se traffic wapis purane view par shift kar dein, aap ke paas hamesha aik chalta hua sahi system backup mein khara hota hai. Is se nuqsan ka darr khatam ho jata hai aur engineering team tezi se naye features deploy kar sakti hai.

---

### Unifying batch and stream processing

Batch aur stream processing ko aik hi chat ke niche jorrne ke liye shuru mein **Lambda Architecture** ka proposal laya naya tha, lekin us mein code ko do alag engines par double likhna parta tha jis se bohot maslay aaye aur ab us ka istemaal band ho gaya hai.

Naye modern designs mein historical data ko replay karna (batch) aur live aane wale events ko handle karna (stream) aik hi single system ke andar jorr diya gaya hai, jisay hum **Kappa Architecture** kehte hain.

Batch aur Stream processing ko aik hi system mein mukammal tarah jorrne (unify karne) ke liye teen (3) technical khususiyaat ka hona farz hai:

* **Log Replay Power:** Processing engine ke paas yeh taqat honi chahiye ke woh naye live events parhne ke sath sath purane historical events ko bhi usi raftar se dobara replay kar sakay (jaise log-based message brokers ya cloud object stores se data read karna).
* **Exactly-Once Semantics:** Engine ke paas paki fault tolerance honi chahiye ke agar task ke darmiyan koi computer crash bhi ho jaye, toh nateeja aisa niklay jaise koi galti hui hi na thi (partial outputs ko automatic discard karna).
* **Event-Time Windowing Tools:** System ke andar windows hamesha **Event Time** (file ke andar ke asli waqt) par chalni chahiye, Processing Time par nahi. Kyunke jab aap 3 saal purana data reprocess kar rahe honge, toh machine ki maujooda ghari bilkul be-maani (meaningless) ho chuki hogi. **Apache Beam API** is ki behtareen misaal hai jo is mathematical calculation ko model karti hai aur background mein isay **Apache Flink** ya Google Cloud Dataflow par run kiya jata hai.

---


