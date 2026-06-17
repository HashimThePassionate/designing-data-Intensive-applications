# Transactions

## James Corbett et al., “Spanner: Google’s Globally-Distributed Database” (2012)

Writer ne chapter ka aghaaz Google ki aik mashhoor research paper ke quote se kiya hai. Is quote mein aik bohot bara design decision aur trade-off samjhaya gaya hai:

* **Mushkil Baat:** Kuch authors (developers/researchers) ka manna hai ke **Two-Phase Commit (2PC)**—jo ke aik aisa tarika hai jis se bohot saari alag alag machines par chalne wale database ko aik sath update kiya jata hai—wo bohot mehanga (expensive) parta hai. Mehanga paison ke lihaz se nahi, balke **performance** (speed slow ho jati hai) aur **availability** (agar aik machine ruk jaye to poora system ruk sakta hai) ke maslon ki wajah se.
* **Google ka Point of View:** Google ke engineers kehte hain ke humein is baat par yakeen hai ke database mein transactions ka hona zaroori hai. Agar transactions nahi hogi, to application banane wale programmers ko har waqt darr rahega ke kahin data kharab na ho jaye, aur unhein khud bohot complex code likhna parega. Is se behtar hai ke database transactions ka feature de, aur jab performance ka masla (bottleneck) aaye, to programmers usay baad mein hal kar lein.

> **Bacchon ki Tarah Asaan Samjhein:** Socho aap aik complex lego house bana rahe ho. Agar aap ko har aik block lagane ke baad darr ho ke poora ghar gir sakta hai aur aap ko khud hi safety ropes lagani parein, to aap thak jaoge. Lekin agar aap ko aik jadoo ka glue (glue gun) de diya jaye jo poore ghar ko sambhal le, bhale hi glue lagane mein thoda waqt (performance cost) lage, aap sukoon se ghar bana sakoge. Google bhi yahi keh raha hai ke jadoo ka glue (transactions) hona chahiye!

---

### In the harsh reality of data systems, many things can go wrong

Asli duniya mein computer systems bilkul perfect nahi hote. Writer kehta hai ke jab hum data ke sath kaam karte hain, to bohot saari cheezein aik sath kharab ho sakti hain. Unhon ne in faults (ghaltiyon) ki aik list di hai:

* **Database Software ya Hardware Failures:** Database chalte chalte achanak band ho sakta hai, ya jis computer (hard drive) par data save ho raha hai wo kharab ho sakti hai. Yeh achanak tab bhi ho sakta hai jab database abhi data **write (save)** kar raha ho.
* **Application Crashes:** Jo software (App) database ko use kar raha hai, wo achanak crash ho sakta hai. Farz karein app ne 5 kaam karne thay, aur wo 2 kaam karne ke baad beech mein hi band ho gaya.
* **Network Interruptions:** Internet ya network achanak kat sakta hai. Is se app ka database se raabta toot sakta hai, ya agar database ki do alag alag machines (nodes) hain, to un ka aapsi raabta toot sakta hai.
* **Concurrent Writes:** Aik hi waqt mein aik se zyada users (clients) database mein aik hi jagah par tabdeeli (write) karne ki koshish kar sakte hain. Is se aik user ka data doosre ke data ke upar overwrite (mita) ho sakta hai.
* **Partial Updates (Adhoori Reads):** Aik client aisa data read (parh) sakta hai jo abhi poora badla hi nahi tha (aadha badla tha aur aadha purana tha), jis se data ka koi matlab hi nahi banta.
* **Race Conditions:** Jab do ya do se zyada kaam aik sath aage peeche hote hain, to ajeeb o gareeb kism ke bugs (bimarayan) peda ho jati hain jinhein pakadna mushkil hota hai.

> **Bacchon ki Tarah Asaan Samjhein:** Socho aap aur aap ka dost aik hi drawing paper par aik sath rang bhar rahe ho. Aap chahte ho ke wahan 'Lal' rang ho aur dost chahte hai ke 'Neela' rang ho. Agar aap dono aik dosre ka hath rokay bagair rang bharne lagoge, to paper phat jayega ya rang kharab ho jayega (Concurrent Writes). Ya phir socho aap aik kahani parh rahe ho aur koi beech mein se adhay pahnay phad de (Partial Updates/Crashes). Systems mein bhi yahi sab hota hai.

---

### Transactions as the mechanism of choice

In saari mushkilaat aur kharabiyon se bachne ke liye, pichle kayi saalon se **Transactions** ko sab se behtareen hathiyar (mechanism) mana gaya hai.

* **Transaction Kya Hai?** Yeh aik tarika hai jahan application apne bohot saare reads (data parhna) aur writes (data likhna) ko aik sath jama kar ke **aik logical unit (ek dabba)** bana deti hai.
* **All-or-Nothing Concept:** Concept yeh hai ke is dabbe ke andar jitne bhi kaam hain, wo ya to saare ke saare kamyab honge (**Commit**), ya phir un mein se aik bhi nahi hoga (**Abort** ya **Rollback**). Agar beech mein koi bhi masla aya, to poora operation fail ho jayega aur system aisa ban jayega jaise kuch hua hi nahi tha.
* **Error Handling Bani Asaan:** Is feature ki wajah se application banane wale ko **partial failures** (aadha kaam hua, aadha nahi hua) ki fikar nahi karni parti. Agar transaction fail ho jaye, to app sukoon se usay dubara shuru (**retry**) kar sakti hai.

> **Bacchon ki Tarah Asaan Samjhein:** Socho aap dukan dar ko 100 rupay dete ho aik chocolate aur aik biscuit lene ke liye. Agar dukan dar chocolate de de aur biscuit khatam ho jaye, to aap adha samaan nahi lete. Aap kehte ho "Mera poora soda do ya mere 100 rupay wapas karo". Transactions bhi yahi hain: "Ya to saara kaam karo (Commit), ya kuch bhi mat karo aur pehle jaisa kar do (Abort)".

---

### The purpose of transactions

Writer yahan aik bohot gahri theoretical baat samjhate hain ke transactions koi qudrati qanoon (law of nature) nahi hain ke in ka hona zaroori hi ho.

* **Insani Ijaad:** Inhein insano ne aik khas maqsad ke liye banaya tha: **Programming model ko asaan banana**.
* **Safety Guarantees:** Jab aap transactions use karte ho, to database aap ko kuch kasmein (guarantees) deta hai. Database kehta hai ke "Tum fikar mat karo, network katay ya aag lage, data ko sambhalna meri zimmedari hai". Is wajah se application developer bohot saare ajeeb o gareeb errors aur concurrency (aik sath kaam hone wale) maslon ko ignore kar sakta hai.

---

### Trade-offs and the Post Office Horizon Scandal

Yahan writer aik design decision aur real-world misal dete hain:

* **Weakening Transactional Guarantees:** Har application ko transactions ki sakht zaroorat nahi hoti. Kabhi kabhi hum jaan boojh kar transactions ko kamzoor kar dete hain ya chor dete hain taake humein zyada speed (**performance**) mile ya hamara system har waqt chalta rahe (**higher availability**).
* **Real-world Disaster (Post Office Horizon Scandal):** Writer ne aik haqeeqi misal di hai ke transactions na use karne ka anjaam kitna bura ho sakta hai. UK ke Post Office mein aik accounting system (Horizon) use hota tha. Us software ke peeche jo database tha, us mein **ACID transactions** (safety guarantees) missing thin. Natija kya nikla? System mein mathematical ghaltiyan (bugs) aane lagin, paise gayab dikhne lage, aur un be-kasoor postmasters (mulaazmeen) par chori ka ilzam lag gaya.

---

### How do you figure out whether you need transactions?

Aap ko kaise pata chalega ke aap ko transaction chahiye ya nahi? Is ka jawab janne ke liye pehle humein un safety guarantees aur un ke sath aane wali keemat (**costs/performance trade-offs**) ko samajhna hoga.

Writer batate hain ke **Concurrency Control** (aik sath hone wale kaamon ko sambhalna) dono jagah zaroori hai:

1. **Single-node Databases:** Jo aik hi computer par chalte hain.
2. **Distributed Databases:** Jo bohot saari machines par phailay hote hain.

Is chapter mein agay hum inhi cheezon ko gehrai mein parhenge:

* **Race Conditions:** Jo ajeeb bugs peda karti hain.
* **Isolation Levels:** Database mein alag alag levels ki security/safety hoti hai jaise **read-committed**, **snapshot isolation**, aur **serializability**. (Inhein hum agay bacchon ki tarah asaan kar ke samjhenge).
* **Two-Phase Commit (2PC):** Distributed databases mein har machine par aik sath sab sahi se save (atomicity) karne ka protocol.

---

## What Exactly Is a Transaction?

Duniya ke taqreeban tamaam relational databases (jaise MySQL, PostgreSQL) aur kuch non-relational databases aaj transactions ko support karte hain. Yeh koi naya concept nahi hai. Is style ki shuruat **1975 mein IBM System R** ne ki thi, jo ke duniya ka sab se pehla SQL database tha. Pichle 50 saalon mein is ke andar ke chote-mote badlao zaroor aaye hain, lekin buniyadi idea bilkul wahi hai jo System R ka tha.

* **NoSQL Ka Daur (Late 2000s):** Jab NoSQL databases bohot mashhoor hone lage, to unhon ne shuruat mein transactions ko bilkul khatam kar diya ya phir un ki guarantees ko bohot kamzoor kar diya. Un ka maqsad naye data models lana aur **replication** (data ki copies banana) aur **sharding** (data ko alag alag hisson mein baantna) ko aam banana تھا.
* **Aik Purani Ghalat Fehmi (The Hype):** Log samajhne lage ke transactions aur scalability (bohot bade scale par chalna) aik sath nahi chal sakte. Logon ka khayal tha ke agar system ko bohot tez (high performance) aur har waqt chalta hua (high availability) rakhna hai, to transactions ko chorna parega.
* **NewSQL Ka Aana (The Comeback):** Lekin waqt ke sath yeh khayal ghalat sabit hua. Naye databases jaise **CockroachDB, TiDB, Spanner, FoundationDB,** aur **YugabyteDB** ne yeh sabit kiya ke transactions ko bohot bade scale par chalaya ja sakta hai. Yeh databases **sharding** aur **consensus protocols** (aik aisi taknik jahan bohot saari machines mil kar aik faisle par raazi hoti hain) ka istemal kar ke bohot bade scale par bhi sakht ACID guarantees dete hain.

> **Bacchon ki Tarah Asaan Samjhein:** Socho aik school mein aik bohot bara register hai jahan sab bachon ke naam likhe jate hain. Pehle aik hi teacher register sambhalti thi (Single Node). Phir bache bohot zyada ho gaye, to register ke pahnay phad kar 5 teachers mein baant diye gaye (Sharding). Shuru mein teachers aapas mein larnay lagin ke kis ke paas kaunsa bacha hai (NoSQL era). Lekin phir unhon ne aik monitor banaya jo sab ko aik sath discipline mein rakhta hai (NewSQL Consensus). Ab register bhi banta hua hai aur kaam bhi poore tareeqay se sahi ho raha hai!

---

### The Meaning of ACID

Transactions ki di gayi safety guarantees ko hum aik mashhoor short-name **ACID** se jante hain. Yeh naam **1983 mein Theo Härder aur Andreas Reuter** ne diya tha taake database mein ghaltiyon ko rokne wale tareeqon ko aik proper naam diya ja sake.

Lekin sachai yeh hai ke har database ka "ACID" ka matlab alag hota hai. Aaj kal "ACID" aik marketing ka lafz ban chuka hai. Agar koi company kehti hai ke un ka database ACID-compliant hai, to bhi aap aankhein band kar ke bharosa nahi kar sakte, kyunke is ki gehrai mein bohot saari aisi bareekiyan hain jo har database alag tarah handle karta hai.

* **BASE Systems:** Jo databases ACID ke asoolon par nahi chalte, unhein **BASE** kaha jata hai:
* **B**asically **A**vailable (System chalte rehna chahiye bhale hi thoda kharab ho).
* **S**oft state (Data khud-b-khud thoda badal sakta hai).
* **E**ventual consistency (Kuch waqt baad data sab jagah aik jaisa ho jayega).


BASE ki definition itni ajeeb aur khuli hai ke is ka aasan matlab sirf yeh banta hai: "Jo database ACID nahi hai, wo BASE hai."

---

### Atomicity

Aam tor par "Atomic" ka matlab hota hai aik aisi cheez jis ke mazeed tukde na kiye ja sakein. Lekin computer science ki alag alag branches mein is ka matlab thoda badal jata hai:

* **Multithreaded Programming Mein:** Agar aik thread koi atomic kaam kar rahi hai, to doosri thread usay aadha naya aur aadha purana nahi dekh sakti. System ya to kaam se pehle wali haalat mein hoga ya kaam ke baad wali haalat mein, beech mein kuch nahi hoga.
* **ACID Transactions Mein:** Yahan atomicity ka concurrency (aik sath kaam hone) se koi taluq nahi hai! (Concurrency ko 'I' yaani Isolation handle karta hai). ACID mein Atomicity ka matlab hai: **Agar client bohot saare writes (data save) kar raha hai aur beech mein koi ghalti (fault) aa jaye, to kya hoga?** * **The Abortability Feature:** Farz karein computer crash ho jaye, network kat jaye, ya hard drive full ho jaye. Agar wo saare kaam aik "Atomic Transaction" ke andar hain, to database us transaction ko **Abort (cancel)** kar dega. Aur ab tak jitna bhi aadha-adhura kaam hua tha, database usay **discard (khatam)** ya **undo (wapas pehle jaisa)** kar dega.

> **Bacchon ki Tarah Asaan Samjhein:** Socho aap computer par aik bada folder copy kar rahe ho jis mein 100 photos hain. 50 photos copy huin aur achanak light chali gayi. Agar system atomic nahi hai, to agli baar light aane par wahan sirf 50 photos parhi hongi aur aap ko samajh nahi aayega ke kaunsi bachi hain (Duplicate hone ka khatra). Lekin agar system Atomic hai, to light aane par wo un 50 photos ko bhi mita dega, taake aap sukoon se dobara zero se shuru kar sakein. Isay kehte hain "Ya to poora kaam, ya kuch bhi nahi!"

---

### Consistency

"Consistency" ka lafz poori database duniya mein sab se zyada confuse karne wala lafz hai kyunke is ke kam az kam 5 alag alag matlab hain:

1. **Replica Consistency:** Jo hum ne asynchronous replication lag mein parha (Data ka copies mein der se pahuchna).
2. **Consistent Snapshot:** Backup lete waqt poore database ki aik lamhe ki aisi photo jo purane aur naye kaamon ko sahi timeline (happens-before relation) ke mutabaq dikhaye.
3. **Consistent Hashing:** Sharding mein data ko machines par barabar baantne ka aik tarika.
4. **CAP Theorem Mein Consistency:** Is ka matlab hota hai Linearizability (Sab ko har lamhe bilkul taza data dikhna).
5. **ACID Mein Consistency:** Is ka matlab hai ke aap ka database hamesha aap ke banaye gaye rules ke mutabaq **"Good State" (sahi haalat)** mein rahe.

* **Invariants (Rules):** Farz karein aik bank ka rule (invariant) hai ke poori bank ke saare accounts ke paise jama kar ke jo total banta hai, wo hamesha barabar rehna chahiye (Credits must equal Debits). Agar transaction shuru hone se pehle yeh rule sahi tha, to transaction khatam hone par bhi yeh sahi rehna chahiye.
* **Zimmedari Kis Ki Hai?** Database aap ko kuch tools deta hai jaise **foreign-key constraints, uniqueness constraints, aur check constraints** (jaise age hamesha 18 se badi ho). Lekin complex rules ko database khud check nahi kar sakta. Agar aap ne khud ghalat data transaction mein likh kar bhej diya, to database usay nahi rok sakega. Is liye ACID ka **C (Consistency) asal mein application programmer ki zimmedari hoti hai**, database ki akele ki nahi.

---

### Figure 8-1. A race condition between two clients concurrently incrementing a counter

Chalein ab is di gayi image (Figure 8-1) ka aik aik step bohot detail mein break down karte hain ke kaise bina safety ke data kharab hota hai.

<div align="center">
  <img src="./images/01.png" width="700"/>
</div>

* **The Setup:** Database mein aik counter save hai jis ki shuruati value **42** hai. Do alag alag users (User 1 aur User 2) aik hi waqt mein is counter ko 1 number se barhana (increment karna) chahte hain. Agar dono ka kaam sahi ho, to final value **44** honi chahiye (42 + 1 + 1).
* **Step 1 (User 1 Reads):** User 1 database ko kehta hai `get counter`. Database check karta hai aur User 1 ko batata hai ke value **42** hai.
* **Step 2 (User 2 Reads Concurrently):** Abhi User 1 ne apni taraf se value update nahi ki thi ke isi doran User 2 ne bhi database ko kaha `get counter`. Database ne User 2 ko bhi bataya ke value **42** hai.
* **Step 3 (User 1 Local Calculation & Write):** User 1 apne computer par dimag lagata hai: `[42 + 1 = 43]`. Wo database ko kehta hai `set counter = 43`. Database isay save karta hai aur kehta hai **"OK"**. Database mein value ab 43 ho chuki hai.
* **Step 4 (User 2 Local Calculation & Write):** User 2 ko jo purani value 42 mili thi, wo bhi apne computer par wahi math karta hai: `[42 + 1 = 43]`. Wo database ko kehta hai `set counter = 43`. Database isay bhi save kar leta hai aur kehta hai **"OK"**.
* **The Bug (Lost Update):** Do logon ne counter barhaya, lekin final value **43** reh gayi! User 1 ka kiya hua kaam User 2 ne upar se mita (overwrite) kar diya. Isay computer ki zubaan mein **Race Condition** ya **Lost Update** kehte hain.

---

### Isolation

Upar jo counter ka masla hum ne dekha, usay hal karne ke liye **Isolation** ka concept aata hai. Jab bohot saare users aik hi waqt mein database use kar rahe hon, to wo aik doosre ke kaamon mein tang nahi ada sakte (they cannot step on each other's toes).

* **Serializability (The Ideal Level):** Purani kitabon ke mutabaq, isolation ka sab se top level **Serializability** hai. Is ka matlab yeh hai ke har transaction yeh tasawwur karti hai ke poore database par sirf wahi akeli chal rahi hai. Bhale hi background mein hazaron transactions chal rahi hon, database un ka result aisa nikalega jaise wo aik ke baad aik (serially) chali thin.
* **Performance Trade-off:** Lekin har cheez ko line mein khara karne se system slow ho jata hai (performance cost). Is liye aam zindagi mein databases itna sakht level use nahi karte, balke thode kamzoor levels use karte hain jaise **read-committed** ya **snapshot isolation**.
* **The Oracle Example:** Writer ne aik real-world fact bataya hai ke mashhoor database **Oracle** mein agar aap isolation level ka naam "serializable" set bhi kar dein, to bhi wo background mein asli serializability nahi deta, balke **snapshot isolation** deta hai, jo ke aik kamzoor guarantee hai aur us mein bhi kuch race conditions ho sakti hain.

---

### Durability

Durability ka matlab hai "Hamesha ke liye mahfooz". Yeh database ka aik aisa wada hai ke jab aap ki transaction aik baar **Commit (Kamyab)** ho gayi, to phir bhale hi computer achanak band ho jaye ya us ka hardware crash ho jaye, aap ka data kabhi gayab nahi hoga.

* **Single-node Database Mein:** Data ko memory (RAM) se nikaal kar non-volatile storage (Hard drive ya SSD) par likhna zaroori hota hai. RAM temporary hoti hai, light jate hi saaf ho jati hai.
* **fsync System Call:** Jab hum computer mein koi file write karte hain, to operating system pehle usay memory mein dabake rakhta hai. Databases is se bachne ke liye **fsync** naam ki aik command use karte hain jo operating system ko majboor karti hai ke data ko foran asli disk par pheko.
* **Write-Ahead Log (WAL):** Disk par asli data badlane se pehle database aik diary mein likhta hai ke "Mein yeh kaam karne laga hoon". Agar asli data likhte waqt computer crash ho jaye, to dobara chalne par database us diary (WAL) ko parh kar sab sahi kar deta hai. Is ke sath **checksums** use hote hain jo yeh check karte hain ke disk par likha hua data kahin toot-phoot (corrupt) to nahi gaya.
* **Replicated Database Mein:** Yahan durability ka matlab badal jata hai. Yahan data ko sirf aik disk par nahi likha jata, balke bohot saari alag alag machines (nodes) par copy kiya jata hai. Jab tak tay shuda machines par data nahi pahunch jata, database transaction ko commit nahi manta.

---

### Replication and Durability

Writer yahan aik bohot gahra design trade-off samjhate hain ke asli duniya mein **Perfect Durability (100% guarantee)** jaisi koi cheez nahi hoti. Har taknik ke apne faide aur nuksanat hain:

* **Flaw 1 (Machine Death):** Agar aap ne data disk par likh diya aur wo machine jal gayi, data ghumay ga nahi lekin jab tak machine theek nahi hoti, aap us data ko use nahi kar sakte. Replication is se bachati hai kyunke doosri machine chal rahi hoti hai.
* **Flaw 2 (Correlated Faults):** Agar achanak poore data-center ki banti chali jaye, ya software mein koi aisa bug ho jo har machine ko crash kar de, to agar data sirf memory mein tha to saari replicas se aik sath urh jayega. Is liye replicated systems mein bhi disk par likhna zaroori hai.
* **Flaw 3 (Asynchronous Replication Lag):** Agar data baqi machines par abhi ja hi raha tha aur main machine (leader) mar gayi, to jo naya data abhi raste mein tha wo hamesha ke liye gayab ho jata hai.
* **Flaw 4 (Hardware & SSD Realities):** Research se pata chala hai ke jab achanak light jati hai, to naye SSDs un guarantees ko tor dete hain jo unhon ne di hoti hain. Yahan tak ke `fsync` bhi sahi kaam nahi karta. Disk ke andar chalne wala software (firmware) bhi ghaltiyon se bhara ho sakta hai. Ek research mein dekha gaya ke aik khas firmware bug ki wajah se drives thik **32,768 hours** (taqreeban 3 saal aur 9 mahine) chalne ke baad achanak marna shuru ho gayin.
* **Flaw 5 (The fsync Difficulty):** `fsync` ko code mein sahi tarah use karna itna mushkil hai ke mashhoor database **PostgreSQL** bhi isay 20 saal tak thoda sa ghalat use karta raha aur unhein baad mein pata chala.
* **Flaw 6 (Bit Rot & Corruption):** Hard drive par para para data achanak khud hi kharab (corrupt) ho sakta hai bina kisi ko pata chale. Agar aisa ho jaye, to wo kharab data dheere dheere baqi copies (replicas) aur naye backups mein bhi phail jata hai. Is liye purane tarikhi backups (historical backups) rakhna zaroori hain.
* **Flaw 7 (SSD Bad Blocks):** Ek study ke mutabaq, **30% se 80% SSDs** apne pehle 4 saal ke andar kam az kam aik "bad block" (kharab hissa) peda kar leti hain, jisay un ka apna software bhi theek nahi kar pata. Magnetic hard drives poori tarah marnay mein aage hain, jabke SSDs mein chote chote hissay jaldi kharab hote hain.
* **Flaw 8 (Data Retention in Worn-out SSDs):** Agar aik SSD bohot zyada use ho chuki ho (worn-out) aur aap us ki power kaat dein, to garam mausam mein wo **kuch hafton se mahino ke andar** apna andar ka data bhoolna shuru kar deti hai!

**Natija (The Ultimate Trade-off):** Asli duniya mein koi bhi aik tarika perfect nahi hai. Behtareen software architecture wahi hai jo in saare tarikon—disk par likhna, remote replication karna, aur purane backups rakhna—ko aik sath mila kar use kare taake nuksan ka khatra kam se kam ho sake.

---


## Single-Object and Multi-Object Operations

Hum ne abhi tak dekha ke ACID mein **Atomicity** aur **Isolation** ka asal maqsad yeh hota hai ke agar aik user (client) aik hi transaction ke andar bohot saare writes (data save) kar raha hai, to database ko us haalat mein kaise behave karna chahiye:

* **Atomicity:** Agar bohot saare kaamon ki sequence ke beech mein koi error aa jaye, to transaction ko **Abort (cancel)** ho jana chahiye, aur ab tak jitna bhi kaam hua tha usay kachray mein phenk (discard) diya jana chahiye. Database aap ko "All-or-Nothing" (ya to poora ya kuch bhi nahi) ka wada de kar adhoori nakami (**partial failure**) ki fikar se bacha leta hai.
* **Isolation:** Aik sath chalne wali transactions aik doosre ke kaamon mein dakhal-andazi (interfere) nahi kar saktin. Agar aik transaction bohot saare writes kar rahi hai, to doosri transaction ko ya to wo saare writes aik sath mukammal dikhne chahiye, ya un mein se kuch bhi nahi dikhna chahiye. Aisa kabhi nahi hona chahiye ke usay sirf aadha-adhura badla hua data dikhe.

Yeh definitions yeh tasawwur karti hain ke aap aik sath bohot saare **objects (rows, records, ya documents)** ko badalna chahte hain. Isay **Multi-object transactions** kehte hain aur in ki zaroorat tab parti hai jab data ke mukhtalif hisson ko hamesha aik sath jura (in sync) rakhna ho.

---

#### Figure 8-2. Violating isolation: one transaction reads another transaction’s uncommitted writes (a “dirty read”)

Chalein is image (Figure 8-2) ke poore flow ko step-by-step bohot detail mein samajhte hain ke jab database Isolation ke rule ko torta hai, to kya ajeeb masla peda hota hai:

<div align="center">
  <img src="./images/02.png" width="700"/>
</div>

* **The Scenario:** Writer ne email application ki misal di hai. Agar humein kisi user ke unread emails count karne hon, to aam query yeh hoti hai: `SELECT COUNT(*) FROM emails WHERE recipient_id = 2 AND unread_flag = true`. Lekin agar emails hazaron mein hon, to yeh count lagana slow ho sakta hai. Is se bachne ke liye developers **Denormalization** karte hain—yaani aik alag table/field mein unread emails ka counter save kar dete hain (jaise `mailboxes` table mein `unread` ka column).
* **Step 1 (User 1 Inserts Email):** User 1 aik naya email insert karta hai: `insert into emails (recipient_id, body, unread_flag) values (2, 'Hello', true)`. Database isay save karta hai aur kehta hai **"OK"**. Lekin yaad rahe, abhi transaction commit (final save) nahi hui!
* **Step 2 (User 2 Reads the Email):** Isi dauran User 2 apni screen refresh karta hai aur database se query karta hai: `select body, unread_flag from emails where recipient_id = 2 limit 50`. Database User 2 ko wo email dikha deta hai: `('Hello', true)`. User 2 ko screen par naya email nazar aa jata hai.
* **Step 3 (User 2 Reads the Counter):** Ab User 2 ka software counter display karne ke liye doosri query chalata hai: `select unread from mailboxes where recipient_id = 2`. Database jawab deta hai: **0**.
* **Step 4 (User 1 Updates Counter):** User 2 ke parhne ke *baad*, User 1 ka software counter barhane ki query chalata hai: `update mailboxes set unread = unread + 1 where recipient_id = 2`. Database kehta hai **"OK"**.
* **The Anomaly (Ghalti):** User 2 ko apni screen par aik naya unread email dikh raha hai, lekin upar counter **0** dikha raha hai! Yeh aik adhoori aur galat state hai. Isay **Dirty Read** kehte hain (yaani kisi doosre ka uncommitted data parh lena). Agar yahan database **Isolation** lagata, to User 2 ko ya to email aur counter dono updated dikhte, ya dono purane dikhte, beech ki yeh ajeeb haalat kabhi nazar na aati.

> **Bacchon ki Tarah Asaan Samjhein:** Socho aap ke papa ne aap ke khilone wale dabbe mein aik nayi car daal di (Insert Email). Lekin abhi unhon ne bahar lage paper par car ki ginti (Counter) ko 0 se 1 nahi kiya tha. Aap ne dabba khola, aap ko andar car dikh gayi, lekin jab aap ne bahar paper par ginti parhi to wahan 0 likha tha. Aap confuse ho gaye ke yeh kya jadoo hai! Isolation ka matlab hai ke jab tak papa ginti bhi theek na kar lein, tab tak aap ko dabba kholne ki ijaazat hi na ho.

---

#### Figure 8-3. Atomicity ensures that if an error occurs, any prior writes from that transaction are undone, to avoid an inconsistent state.

Chalein ab is doosri image (Figure 8-3) ke step-by-step flow ko samajhte hain jo Atomicity ki zaroorat ko dikhati hai:

<div align="center">
  <img src="./images/03.png" width="700"/>
</div>

* **Step 1:** User 1 ne successfully email insert kar diya: `insert into emails...`. Database ne kaha **"OK"**.
* **Step 2:** User 1 ab counter barhane laga tha: `update mailboxes set unread = unread + 1...`. Lekin isi lamhe system mein **Error ya Timeout** aa gaya (jaise database ki jor se hard disk full ho gayi, ya network tar toot gaya).
* **The Danger (Bina Atomicity Ke):** Agar system mein atomicity na ho, to email to andar save reh jayega lekin counter hamesha ke liye purana (0) hi reh jayega. Data aapas mein out-of-sync ho jayega.
* **The Solution (Atomicity Ke Sath):** Atomicity database ko majboor karti hai ke chunke doosra kaam (counter update) fail ho gaya hai, is liye pehla kaam (email insert) bhi **Undo (Rollback)** kar diya jaye. Database us email ko mita dega taake system bilkul pehle jaisa saaf ho jaye, aur application bina kisi dar ke dobara poora kaam shuru (**retry**) kar sake.

---

#### Multi-Object Transactions Ki Pehchan

Multi-object transactions mein database ko kaise pata chalta hai ke kaun kaun si queries aik hi dabbe (transaction) ka hissa hain?

* **Relational Databases Mein:** Yeh kaam client ke **TCP Connection** ke zariye hota hai. Aik connection par jab client `BEGIN TRANSACTION` bhejta hai, us ke baad se lekar `COMMIT` bheyjne tak jitni bhi queries aati hain, database un sab ko aik hi transaction maanta hai. Agar beech mein TCP connection toot jaye, to database samajh jata hai ke kuch garbar hai aur wo sab abort kar deta hai.
* **Non-Relational (NoSQL) Databases Mein:** In mein aam tor par kaamon ko aik sath jurnay (group karne) ka aisa koi tarika nahi hota. Agar un ke paas koi `multi-put` API ho bhi (jis se aik sath bohot saari keys update hoti hain), to us ka yeh matlab nahi hota ke wo transaction hai. Ho sakta hai 5 keys update ho jayein aur baqi 5 fail ho jayein, jis se database adhoori haalat mein phans jata hai.

---

### Single-object writes

Atomicity aur Isolation sirf bohot saare records par hi nahi, balke **aik akele record/object** ko badalte waqt bhi utni ہی zaroori hain. Farz karein aap database mein aik bada **20 kB ka JSON document** save kar rahe hain:

1. Agar 10 kB data transfer hone ke baad internet connection kat jaye, to kya database us aadhe-adhure kharab JSON ko save kar ke baith jayega?
2. Agar database disk par purane data ke upar naya data likh raha ho aur achanak light chali jaye, to kya purana aur naya data aapas mein khichdi (spliced/corrupt) ban jayega?
3. Agar aik user us document ko abhi write (save) kar raha ho aur theek usi mili-second mein doosra user usay read kare, to kya usay adha naya aur adha purana data dikhega?

In intehai confuse karne wale maslon se bachne ke liye, taqreeban tamaam storage engines aik akele object (key-value pair ya single row) ke level par Atomicity aur Isolation lazmi dete hain.

* **Implementation Kaise Hoti Hai?** Single-object par Atomicity banane ke liye database recovery log (Crash recovery log) use karta hai, aur Isolation ke liye har object par aik **Lock** laga deta hai (yaani jab tak aik thread kaam kar rahi hai, doosri thread us object ko hath nahi laga sakti).
* **Complex Atomic Operations:** Kuch databases mazeed behtareen features dete hain jaise ke **Atomic Increment** (jo seedha database mein hi counter barha deta hai, jaise Redis mein hota hai, jis se Figure 8-1 wala read-modify-write ka lambay process se jaan choot jati hai). Isi tarah **Conditional Write** (yaani Compare-and-Set / CAS) hota hai, jo kehta hai ke data sirf tabhi save karo agar pichli baar se lekar ab tak kisi aur ne usay badla na ho.

> **Theoretical Nuance:** Writer batate hain ke lafz "Atomic Increment" mein jo "Atomic" hai, wo asal mein multithreaded programming wala hai (yaani concurrency se bachane wala). ACID ke mutabaq isay **Isolated ya Serializable Increment** kehna chahiye, lekin aam zubaan mein log isay atomic increment hi kehte hain.

* **NoSQL Limit:** databases jaise **Aerospike** aur **Cassandra/ScyllaDB** (lightweight transactions ke zariye) single-object level par bohot strong guarantees (linearizable reads aur conditional writes) dete hain, lekin un mein ek se zyada objects (multi-object) par koi guarantee nahi hoti.

---

### The need for multi-object transactions

Kya humein waqai multi-object transactions ki zaroorat hai? Kya hum sirf single-object operations use kar ke poori application nahi bana sakte?

Kuch simple kaamon mein single-object se guzara ho jata hai, lekin bohot saare real-world use cases mein humein multi-object transactions ki sakht zaroorat hoti hai:

1. **Foreign-Key / Graph References:** Relational databases mein aik table ki row doosre table ki row se linked hoti hai (Foreign Key). Graph databases mein aik point (vertex) doosre points se edges ke zariye jura hota hai. Agar aap do records insert kar rahe hain jo aalmi tor par aapas mein linked hain, to dono ka aik sath sahi save hona zaroori hai, warna reference toot jayega aur data ka koi matlab nahi rahega.
2. **Denormalization in Documents:** Document databases (jaise MongoDB) mein data aksar aik hi bade document ke andar hota hai, is liye single-document update se kaam chal jata hai. Lekin chunke un mein `JOIN` karne ki taqat nahi hoti, is liye developers ko data ki copies alag alag documents mein rakhni parti hain (**Denormalization**). Ab jab bhi wo data badlega, aap ko aik sath bohot saare documents ko update karna parega (jaise Figure 8-2 mein email aur counter alag alag documents thay). Agar transactions nahi hongi, to yeh data aapas mein out-of-sync ho jayega.
3. **Secondary Indexes:** Taqreeban har database mein secondary indexes hote hain (taake search tez ho sake). Jab bhi aap kisi record ko badalte hain, database ko background mein us ke saare indexes ko bhi update karna parta hai. Agar yahan transaction isolation na ho, to ho sakta hai aik record aik index mein to dikhe lekin doosre index mein gayab ho, kyunke doosra index abhi update hi nahi hua tha!

---

### Handling errors and aborts

Transactions ki sab se badi taqat hi yeh hai ke agar koi error aaye, to usay **Abort (cancel)** kar ke safely dobara chalaya (**retry**) ja sakta hai. ACID databases ka poora falsafa isi par khara hai: **"Database guarantees torney se behtar samajhta hai ke poore kaam ko hi cancel kar de, bajaye is ke ke aadha kaam chor de."**

Lekin har system is falsafay par nahi chalta:

* **Leaderless Replication Stores (e.g., Cassandra):** Yeh databases "Best Effort" par chalte hain. Un ka asool hota hai ke "Humein jitna ho saka hum ne kiya, agar beech mein error aa gaya to hum pehle se kiya hua kaam wapas undo nahi karenge". Is haalat mein error se nipatnay (recover karne) ki poori zimmedari application programmer par aa jati hai.

#### The Reality of Application Software & ORMs

Afsos ki baat yeh hai ke bohot saare developers sirf "Happy Path" (jab sab theek chal raha ho) ke baare mein sochte hain aur error handling ko ignore kar dete hain.

* **ORMs Ki Ghalti:** Mashhoor frameworks jaise **Rails ActiveRecord** aur **Django** ke ORMs aborted transactions ko khud-b-khud retry **nahi** karte. Jab database error bhejta hai, to yeh frameworks seedha exception (error) throw kar dete hain, jis se user ka enter kiya hua data zaya ho jata hai aur screen par error message aa jata hai. Yeh bohot buri baat hai kyunke rollback ka asal maqsad hi yeh tha ke software chupke se usay dobara chalaye (safe retry kare) aur user ko pata bhi na chale.

#### Retry Karne Ke Masle aur Limitations

Bhale hi retry karna aik behtareen tarika hai, lekin yeh bilkul perfect nahi hai. Is ke 5 bade masle hain:

1. **The Network Timeout Dilemma:** Farz karein database ne transaction ko successfully commit (save) kar diya, lekin confirmation ka signal client ko bhejte waqt network kat gaya. Client ko laga ke kaam fail ho gaya (Timeout). Agar client ab dobara retry karega, to wo kaam **do baar (duplicate)** ho jayega, jab tak ke aap ne application level par deduplication mechanism (jaise unique request IDs) na lagaya ho.
2. **System Overload / Feedback Cycles:** Agar database par pehle hi bohot zyada bojh (high contention/overload) hai aur transactions fail ho rahi hain, to har transaction ka baar baar retry karna bojh ko kam nahi karega, balke system ko mazeed jam kar dega. Is se bachne ke liye **Exponential Backoff** (yaani har fail ke baad rukne ka waqt barhana) aur retry ki aik limit lagana zaroori hai.
3. **Transient vs Permanent Errors:** Retry sirf temporary errors (jaise Deadlock, network ka lamhati jhatka, ya node failover) par hi faida mand hota hai. Agar error pakka (Permanent) hai—jaise koi rule toot raha ho (uniqueness constraint violation)—to 100 baar bhi retry karne se koi faida nahi hoga.
4. **Outside Side Effects:** Agar aap ki transaction ke andar database se baahir ke kaam bhi ho rahe hain—jaise user ko email bhejna—to har retry par user ko aik naya email chala jayega! Agar aap chahte hain ke database aur baahir ke baki systems (jaise email service) aik sath ya to commit hon ya abort hon, to aap ko **Two-Phase Commit (2PC)** use karna parega.
5. **Client Process Crash:** Agar retry karne ke dauran client ka apna computer (application server) hi crash ho jaye, to jo data wo save karne ki koshish kar raha tha, wo hamesha ke liye hawa mein gayab ho jata hai.

---
