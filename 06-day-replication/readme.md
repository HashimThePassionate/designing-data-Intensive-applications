# Replication

Distributed systems mein **Replication** ka matlab hota hai aik hi data ki bilkul huba-hu (identical) copies ko mukhtalif machines par rakhna, jo network ke zariye aaps mein connected hon.

Douglas Adams ka aik bohot mashhoor jumla hai ke *"Aik aisi cheez jo kharab ho sakti hai aur aik aisi cheez jo kabhi kharab nahi ho sakti, un mein sabsay bada farq yeh hai ke jab kabhi na kharab hone wali cheez galti se kharab ho jaye, toh use repair karna ya us tak pohanchana namumkin ho jata hai."* Distributed systems par bhi yahi baat lagu hoti hai; hum jitna bhi resilient system bana lein, network aur hardware faults aane hi aane hain. Isliye hum replication ka sahara lete hain.

Data ko replicate karne ke **3 bade architectural fawaid (reasons)** hote hain:

* **Latency Kam Karna (Geographical Proximity):** Data ko geographic taur par aapke users ke jitna mumkin ho qareeb rakha jata hai. Agar user Pakistan mein hai, toh request US bhejne ke bajaye qareeb ke server (e.g., Singapore) se serve ki jati hai, jis se app bohot fast chalti hai.
* **High Availability aur Durability (Fault Tolerance):** Agar system ka aik hissa (node) kisi hardware crash ya datacenter ki bijli janay se down ho jaye, toh baqi bache huay nodes bina kisi downtime ke kaam ko sambhal lete hain.
* **Read Throughput ko Scale Out Karna:** Agar system par sudden parhne wali queries (read traffic) ka bohot zyada bojh aa jaye, toh hum replication ke zariye multiple machines khari kar dete hain, jo us traffic ko aaps mein baant (distribute) leti hain.

> **The Core Assumption:** Is poore context mein hum yeh maan kar chal rahe hain ke aapka dataset itna chota hai ke har single machine poore ke poore data ki copy ko apni disk par hold kar sakti hai. Agar data aik machine se bada ho jaye, toh uske liye hum **Sharding (Partitioning)** karte hain, jo hum agay seekhenge.

Agar aapka data kabhi badalta nahi (Static Data), toh replication dunya ka sabsay aasan kaam hai—aik dafa copy karo aur bhool jao. **Replication ki asli mushkil tab shuru hoti hai jab data continuously badal raha ho (Writes aur Updates ho rahe hon).**

Distributed databases mein is badlao ko saare nodes par synch karne ke liye 3 main algorithms ki families use hoti hain, aur poori dunya ke databases inhi teen tareeqon par chalte hain:

1. **Single-Leader Replication**
2. **Multi-Leader Replication**
3. **Leaderless Replication**

In teeno approaches ke apne fayde aur nuksan (trade-offs) hain, jaise yeh chunna ke replication **Synchronous** (foran confirm hone wali) ho ya **Asynchronous** (background mein aahista chalne wali), aur un replicas ko kaise handle kiya jaye jo temporary offline ho chuke hain.

Inhi trade-offs ki wajah se distributed databases mein **Eventual Consistency** (aik na aik waqt data har jagah barabar ho jana) ka concept paida hota hai, jo aksar developers ko confuse karta hai. Is lag/delay ko manage karne ke liye hum agay chal kar *Read-Your-Writes* aur *Monotonic Reads* jaisi strict guarantees ko deeply breakdown karenge.

---

```plaintext
[ Master/Leader Node (Write) ] ---> Synchronous Sync ---> [ Follower Node 1 (Read) ]
              |
              +------------------> Asynchronous Sync ----> [ Follower Node 2 (Read) ]

```

### Comprehensive Diagram Explanation

Is basic dataflow diagram mein single-leader replication ka core model dikhaya gaya hai. Jab client koi naya data write karta hai, toh wo hamesha `Master/Leader Node` par jata hai. Leader node us naye write ko baqi nodes tak pohanchane ke liye do tareeqay chun sakta hai: `Follower Node 1` ko wo strictly real-time (**Synchronous**) data bhejta hai, yaani jab tak node 1 haan nahi kahega, write complete nahi mana jayega. Jabke `Follower Node 2` ko background mein (**Asynchronous**) data copy kiya jata hai, jis se leader par load kam hota hai par node 2 par thoda replication lag/delay aa sakta hai.

---

## Backups and Replication

Aksar developers ke dimaag mein aik sawaal aata hai: *"Agar hamare paas data ki multiple copies (Replicas) maujud hain jo har waqt live chal rahi hain, toh kya humein alag se Backups rakhne ki zaroorat hai?"* Writer iska bohot saaf jawab deta hai: **Haan! Backup aur Replication do bilkul alag maqsad ke liye banaye gaye hain, aur dono aik doosre ka badal nahi hain.**

Chaliye inke farq aur rishte ko bilkul bacho ki tarah breakdown karke samajhte hain:

* **Replication ka Kaam (Real-time Mirror):** Replication ka maqsad yeh hai ke jaise hi aik node par koi write operation ho, wo miliseconds ke andar baqi saare live nodes par copy ho jaye. Yeh live system ki availability barkarar rakhne ke liye hota hai.
* **Backup ka Kaam (Time Machine):** Backup ka maqsad data ke purane snapshots (historical history) ko kisi safe jagah save karna hota hai, taake aap waqt mein piche (go back in time) ja sakein.

### The Fatal Threat: Accidental Deletion (Galti se data delete hona)

Farz karein aapki application ke production database par aik junior developer se galti se aik destructive command chal jati hai: `DROP TABLE Users;` ya saare users ka data update ho kar kachra ban jata hai.

* **Replication ka Behavior:** Chunke replication ka kaam hi updates ko har jagah foran phejna hai, wo is destructive command ko aik jhatkay mein saare live replicas par chala degi. Dekhte hi dekhte data har server se saaf ho jayega! Replication aapko is inshani galti (human error) se nahi bacha sakti.
* **Backup ka Behavior:** Is tabahi ke waqt aapka backup aapki jaan bachayega. Aap raat ke 12 baje liye gaye purane backup snapshot ko uthayenge aur database ko dobara us purani sahi state par restore kar lenge.

### Ek Doosre ke madat (How they complement each other)

Production architectures mein backups aur replication aaps mein mil kar kaam karte hain:

1. **Bootstraping New Followers:** Jab aapko system mein aik bilkul naya khali replica node add karna hota hai, toh aap live database par load dalne ke bajaye aik purane consistent backup snapshot se data utha kar us naye node par load karte hain, aur phir bacha hua fresh data replication log se sync karwa lete hain.
2. **Archiving Logs for Backup:** Databases ke andar behne wale replication change logs ko archive karke dur-daraz ke storage mein save karna hi backup process ka hissa hota hai.

### Storage aur Cost Optimization Rule

Kuch modern databases memory ke andar hi purani states ka snapshot sambhal kar rakhte hain (Immutable Snapshots), jo aik tarah ka internal backup hota hai. Lekin iska bada trade-off yeh hai ke aap barson purana data usi mehnge aur fast primary storage hardware (SSD/NVMe) par rakh rahe hain jahan aapka live database chal raha hai.

Bade systems mein cost optimize karne ka sunehra usool yeh hai ke live production database mein sirf **Current Active State** ko rakha jaye, aur barson purane backup snapshots ko saste Cloud Object Stores (jaise Amazon S3 ya Google Cloud Storage) mein dump kar diya jaye, jo kam accessed data (cold storage) ke liye bohot saste parte hain.

---

## Mockup System Design Scenario (Interview Prep)

### Scenario Context

Aap aik multi-million dollar Fintech Wallet application ke Infrastructure Architect hain. System highly available hona chahiye (zero downtime). Ek din, high traffic ke dauran, aik bug ki wajah se users ka ledger balance database mein randomly overwrite ho jata hai, aur replication ki wajah se wo galat balance Singapore, US aur Europe ke saare nodes par live copy ho jata hai.
*Interviewer aap se poochta hai:* "Aap is catastrophic data corruption se system ko kaise recover karenge? Aur aisa disaster recovery system design karein jo data loss aur downtime dono ko minimum kare."

### Architectural Design Implementation

Hum is disaster recovery challenge ko hal karne ke liye **Point-In-Time Recovery (PITR)** aur **Cross-Region Object Storage Backup** ka decoupled design model apply karenge. Niche iska flow structure diya gaya hai:

```plaintext
[ Corrupted Live DB (All Regions) ] <--- 3. Replays Clean Write-Ahead Logs (Up to 10:59 AM)
                 ^
                 | 2. Restores Base State
+-----------------------------------+
| S3 Cold Storage (Backup System)   |
+-----------------------------------+
| Snapshot: 10:00 AM (Base Data)    | <--- 1. Fetch 10:00 AM Backup Snapshot
| Write-Ahead Logs (Continuous)     |
+-----------------------------------+

```

### Comprehensive Architectural Explanation

1. **The Fallacy of Multi-Region Replication:**
Interviewer ko batayein ke chunke data corrupt ho chuka hai aur replication ne use global level par phela diya hai, live failover nodes par shift karne ka koi faida nahi hoga kyunke har jagah kachra data copy ho chuka hai. Live systems ko foran read-only mode par dalna padega.
2. **Point-In-Time Recovery (PITR) Execution:**
Hum production database ko bilkul shuru se khara karne ke liye do cheezein utilize karenge jo Object Storage (S3) mein hourly back up ho rahi thin:
* **Base Snapshot:** Hum disaster se theek pehle ka sasta base snapshot (e.g., 10:00 AM ka snapshot) primary database par restore karenge. Ab database 10:00 AM ki state par aa gaya.
* **Write-Ahead Log (WAL) Replay:** Database engine ke paas aik sequence log hota hai jisme har transaction ka record hota hai. Hum 10:00 AM se lekar exact 10:59 AM (corrupted bug chalne se exact aik second pehle) tak ke saare transaction logs ko database par dobara **Replay** karenge.


3. **The Clean State Recovery:**
Is execution ke nateeje mein database bina kisi data corruption ke exact 10:59 AM par zinda ho jayega. Live replication engine dobara chalte hi is saaf aur correct data ko baqi saare regions (US, Europe) mein push kar dega, aur system bina bade data loss ke safely recover ho jayega.

---

## Quick Revision & Key Takeaways

* **Core Summary:** Replication ka maqsad live system ko latency, availability aur read scaling faraham karna hai jabke data change ko dynamic sync karna iska sabsay bada challenge hai. Backup ka maqsad history mein piche ja kar galti se delete ya corrupt huay data ko dubara zinda karna hota hai.
* **The Architectural Rule:** Kabhi bhi replication ko backup ka badal mat samjhein. Replication aapko hardware failure se bacha sakti hai lekin human validation errors, application bugs, ya accidental data drops se sirf continuous backup snapshots hi bacha sakte hain.
* **Flash-Card Points:**
* **Replication Latency:** Data ko user ke physical location ke qareeb rakh kar network delay ko kam karna.
* **Data Outlives Code Assumption:** Is chapter mein dataset itna chota mana gaya hai ke har single machine poore database ki copy ko hold kar sake.
* **Accidental Propagation:** Replication system live updates ko copy karta hai, isliye agar production par data drop ho jaye toh wo replicas se bhi foran delete ho jata hai.
* **Cold Object Store Optimization:** Mehnge primary database storage par load kam karne ke liye purane historical backups ko saste, kam accessed storage (jaise Amazon S3) mein archive karna.

---

## Single-Leader Replication

Distributed systems mein jab hum data ki miltiple copies alag-alag machines par rakhte hain, toh un machines ko **Replica** kaha jata hai. Lekin yahan aik sab se bada sawal yeh paida hota hai ke hum yeh kaise pakka (ensure) karein ke jo data aik machine par write hua hai, wo baqi saare replicas par bhi sahi salamti se pohanch jaye?

Agar har replica ka data bilkul aik jaisa rakhna hai, toh har write query ko har single replica par chalna padega. Is masle ko hal karne ka sabsay aam aur mashhoor tareeqa **Leader-Based Replication** (jise Primary-Backup ya Active/Passive Replication bhi kehte hain) hai. Yeh nizam teen bacho jaise aasan steps par chalta hai:

1. **The Leader (Sardar Node):** System mein maujud saare replicas mein se aik node ko **Leader** (Primary ya Source) ghoshit (designate) kar diya jata hai. Jab bhi kisi client ko database mein koi naya data likhna (Write) ya badalna (Update/Delete) ho, toh wo apni request strictly sirf aur sirf leader node ko bhej sakta hai. Leader us data ko pehle apne local storage (disk) par write karta hai.
2. **The Followers (Cheelay Nodes):** Baqi saare nodes ko **Followers** (Read Replicas, Secondaries, ya Hot Standbys) kaha jata hai. Jab bhi leader apne paas koi naya data write karta hai, wo us badlao ko aik **Replication Log ya Change Stream** (tabdeeli ka rasta) ki shakl mein apne saare followers ko network par bhej deta hai. Har follower us log ko parhta hai aur bilkul usi exact order (tartoob) mein un writes ko apne local database par apply karta jata hai jis order mein leader ne execute kiya tha.
3. **Read/Write Rules:** Clients jab bhi database se data parhna (Read) chahein, wo leader ya kisi bhi follower se parh sakte hain. Lekin naya data likhne (Write) ki izazat sirf aur sirf leader ke paas hoti hai. Followers clients ke liye strictly **Read-Only** hote hain.

Agar aapka database **Sharded (Partitioned)** hai, yaani data ke baray baray hissay kiye huay hain, toh har single shard ka apna aik makhsoos leader hota hai. Ho sakta hai Shard A ka leader Machine 1 par ho aur Shard B ka leader Machine 2 par, lekin har shard ke paas aik waqt mein sirf aik hi active leader node ho sakta hai.

### Real-World Technology Usage

Yeh single-leader model distributed systems ki dunya mein har jagah raaj kar raha hai. Yeh relational databases (PostgreSQL, MySQL, Oracle Data Guard, SQL Server Always On) ka built-in feature hai. NoSQL document databases (MongoDB, DynamoDB) aur message brokers (Apache Kafka) bhi isi par chalte hain. Hatta ke modern automatic leader election consensus protocols (jaise Raft jo CockroachDB, TiDB, aur etcd mein use hota hai) bhi isi single-leader paradigm par base karte hain.

---

### Figure 6-1 ka Deep Breakdown

<div align="center">
  <img src="./images/01.png" width="600"/>
</div>

Chaliye ke architectural graph ko step-by-step aur component level par poori bareeki se samajhte hain:

```plaintext
[ User 1234 ] ---> (Writes: picture_url='me-new.jpg') ---> [ Leader Replica ]
                                                                   |
                                          +------------------------+ (Fires Stream)
                                          |
                                          v
                                  [ Data Change Log ]
                                  table:       users
                                  primary key: 1234
                                  column:      picture_url
                                  old_value:   me-old.jpg
                                  new_value:   me-new.jpg
                                  transaction: 987654321
                                          |
                     +--------------------+--------------------+
                     |                                         |
                     v                                         v
            [ Follower Replica 1 ]                    [ Follower Replica 2 ]
                                                               ^
                                                               | (Read-Only Query)
                                                      [ User 2345 ]

```

#### Comprehensive Components Explanation:

* **The Write Transaction:** Bai'n (left) taraf **User 1234** apni profile picture badal raha hai. App SQL query chalati hai: `UPDATE users SET picture_url = 'me-new.jpg' WHERE user_id = 1234;`. Yeh write query seedha **Leader Replica** par land karti hai.
* **The Data Change Log Struct:** Leader is query ko chala kar local disk par save karta hai aur aik structured binary packet generate karta hai jise diagram mein **Data Change** dikhaaya gaya hai. Is packet mein metadata hota hai:
* `table`: `users` (kis table mein tabdeeli hui).
* `primary key`: `1234` (kis record ko chheda gaya).
* `column`: `picture_url` (kaun sa field badla).
* `old_value`: `me-old.jpg` (purana data kya tha).
* `new_value`: `me-new.jpg` (naya data kya save hua).
* `transaction`: `987654321` (unique log ID).


* **The Replication Streams:** Leader is data change packet ko network pipes ke zariye upar aur neechay mojud dono **Follower Replicas** ki taraf push kar deta hai. Followers is log ko dekh kar apne paas data update kar lete hain.
* **The Read Path:** Da'in (right) taraf **User 2345** jab User 1234 ki profile open karta hai, toh uski `SELECT *` read query leader par load dalne ke bajaye neechay wale **Follower Replica** par chali jati hai. Is tarah leader par se read ka bojh hat jata hai.

---

## Synchronous Versus Asynchronous Replication

Distributed database design ka aik bohot hi critical theoretical aspect yeh taye karna hai ke data leader se followers tak **Synchronously** (foran pakka ho kar) jaye ya **Asynchronously** (background mein aaram se) jaye.

### Figure 6-2  ka Timing Breakdown

<div align="center">
  <img src="./images/02.png" width="600"/>
</div>
mein timing charts ke zariye dono replication types ka aaps mein muqabla dikhaya gaya hai. Chaliye is timing flow ko step-by-step dekhte hain:

1. **Client to Leader Call:** User 1234 profile picture update ki request bhejta hai. Leader ko request milti hai aur wo data change process karta hai.
2. **The Synchronous Flow (Follower 1):** Leader data change ka packet **Follower 1** ko bhejta hai. Followers 1 use apni disk par write karta hai aur leader ko wapas **`OK`** (confirmation) signal bhejta hai. Diagram mein aap dekh sakte hain ke leader ne tab tak user ko **`OK`** ka response nahi bheja jab tak Follower 1 ka confirmation nahi aa gaya. Is duration ko diagram mein **"Waiting for follower's OK"** dikhaya gaya hai.
3. **The Asynchronous Flow (Follower 2):** Parallel mein, leader wahi data change packet **Follower 2** ko bhi bhejta hai, lekin is dafa leader uske reply ka **wait nahi karta** (non-blocking). Leader Follower 1 se `OK` milte hi user ko kamyabi ka signal bhej deta hai. Diagram ke aakhir mein mojud lambi tirchi line (substantial delay) yeh dikhati hai ke Follower 2 ne network congestion ya kisi aur wajah se kafi der baad ja kar us message ko process kiya aur apna `OK` leader tak pohanchaya.

---

### Synchronous Replication ke Theoretical Trade-offs

* **Fawaid (Pros):** Iska sabsay bada faida data consistency aur safety hai. Follower 1 ke paas har lamhe bilkul up-to-date data hota hai jo leader se 100% match karta hai. Agar leader achanak crash ho jaye ya uski hardware tabah ho jaye, toh hum aankhein band karke Follower 1 ko naya leader bana sakte hain kyunke aik bit ka data bhi loss nahi hua.
* **Nuksanat (Cons):** Iska nuksan system ki availability par parta hai. Agar Follower 1 crash ho jaye, ya network wire toot jaye, toh leader naye writes process **nahi kar sakega**. Leader block ho jayega aur waiting state mein baith jayega jab tak synchronous replica wapas zinda nahi hota. Poora system grind ho kar ruk jayega.

### Asynchronous Replication ke Theoretical Trade-offs

* **Fawaid (Pros):** Leader bilkul aazad hota hai. Chahe saare followers network problem ki wajah se ghanton piche (replication lag) kyun na chale जाएं, leader bina ruke high throughput ke sath naye writes confirm karta rehta hai.
* **Nuksanat (Cons):** Durability kamzoor ho jati hai. Agar user ko `OK` milne ke baad aur Follower 2 tak data pohanchne se pehle leader ka hardware fail ho jaye, toh wo un-replicated writes **hamesha ke liye gum (lost)** ho jayenge. Client ko laga data save ho gaya, par asal dunya mein wo gayab ho chuka hota hai.

---

### In-Practice Configurations: Semi-Synchronous & Quorums

Production environments mein operational simplicity aur data safety ko balance karne ke liye do darmiyanay raste (hybrid models) nikale jate hain:

```plaintext
[ Client Write ] ---> [ Leader ] === (Strict Sync Waiting) ===> [ Follower 1 (Sync Node) ]
                          |
                          +.......... (Background Async) .......> [ Follower 2 (Async Node) ]
                          |
                          +.......... (Background Async) .......> [ Follower 3 (Async Node) ]

```

#### 1. Semi-Synchronous Architecture

Saare followers ko synchronous rakhna bewaqufi hai kyunke aik node down hone se poora system baith jayega. Isliye real-world databases mein **Semisynchronous** pattern use hota hai:

* Poore cluster mein sirf **aik follower ko synchronous** rakha jata hai aur baqi sab ko asynchronous.
* Agar wo synchronous follower down ya slow ho jaye, toh topology management engine automatically kisi aik asynchronous follower ko pakad kar use synchronous mein **promote** kar deta hai.
* Yeh guarantee deta hai ke data kam az kam do active nodes (Leader + One Follower) par har waqt surakshit mojud hai.

#### 2. Quorum-Based Majority Pattern

Consensus protocols (jaise Raft/Paxos) mein majority quorum ka rule chalta hai. Agar aapke paas 5 replicas hain, toh leader naye write ko tabhi successful declare karega jab kam az kam **3 out of 5 nodes** (including the leader) us write ko synchronously confirm kar dein. Baqi bache huay 2 nodes background mein asynchronously sync hote rehte hain.

---

## Mockup System Design Scenario (Interview Prep)

### Scenario Context

Aap aik high-scale Photo Sharing Application (jaise Instagram) ka data tier design kar rahe hain jahan har second hazaron users photos upload (Write) karte hain aur millions of users unhein timeline par dekhte (Read) hain.
*Interviewer aap se poochta hai:* "Agar hum 10 distributed read replicas lagate hain aur un sab ko fully synchronous configure kar dete hain taake data strict consistent rahe, toh high availability aur performance par kya asar parega? Aur as an architect aap is system ko gRPC endpoints ke sath kaise optimize karenge?"

### Architectural Design Implementation

Hum is architecture ko completely synchronous rakhne ke bajaye **Semi-Synchronous Leader-Based Edge Topology** par design karenge taake scaling aur safety dono achieve hon.

```plaintext
[ Client Mobile App ] ---> POST /upload_photo ---> [ API Gateway (gRPC Router) ]
                                                            |
                                                   (Directs Write Only)
                                                            v
                                                   [ Shard 1 Leader Node ]
                                                   (Writes Local Log)
                                                            |
                             +------------------------------+------------------------------+
                             | (Strict Sync Block)                                         | (Async Change Stream)
                             v                                                             v
                  [ Follower 1 (Sync Replica) ]                                 [ 9 Asynchronous Followers ]
                  (Confirms Write to Leader)                                    (Serve Global Read Traffic)

```

### Comprehensive Architectural Explanation

1. **Why 10 Synchronous Followers is a Disaster:**
Interviewer ko batayein ke agar 10 ke 10 replicas synchronous kar diye gaye, toh system ka write throughput aur availability zero ke qareeb pohanch jayegi. Probability ke mutabaq, 10 distributed nodes mein se network jitter ya garbage collection pause ki wajah se har dusre second koi na koi node slow hoga. Leader har write par us slowest node ke liye block hoga, jis se saare clients ke uploads timeout ho jayenge.
2. **The Optimized Production Architecture:**
* **The Dynamic Routing:** Hum API gateway par gRPC client stubs use karenge jo user ke upload traffic ko strictly direct karenge `Shard 1 Leader Node` ki taraf.
* **The Deployment Model:** Hum database configuration mein **Semisynchronous** mode active karenge. Leader sirf `Follower 1` ke `OK` ka wait karega. Jaise hi Follower 1 acknowledge karega, user ko image uploaded ka response mil jayega (Latency drops drastically).
* **The Scale-Out Strategy:** Baqi bache huay **9 Asynchronous Followers** par global timelines ka bhari read traffic divert kar diya jayega. Agar replication lag ki wajah se kisi user ko photo 500 miliseconds baad bhi dikhe, toh wo business logic ke mutabaq acceptable trade-off hai, lekin system completely crash-proof aur highly available ho jayega.



---

## Quick Revision & Key Takeaways

* **Core Summary:** Single-leader replication mein saare writes aik sardar node (Leader) par jate hain jo data change log stream ke zariye followers ko sync karta hai. Synchronous replication strict consistency deti hai par availability tabah karti hai, jabke asynchronous replication high speed deti hai par leader fail hone par data loss ka khatra lati hai.
* **The Architectural Rule:** Distributed systems mein scalability aur resilience ke liye kabhi bhi saare nodes ko synchronous lock mat karein. Hamesha **Semisynchronous (1 Sync + N Async)** ya **Majority Quorum (3 out of 5)** ka architectural pattern deploy karein.
* **Flash-Card Points:**
* **Replica:** Database ki identical copy hold karne wali individual network machine.
* **Replication Log (Change Stream):** Leader par badalne wale data ke metadata aur values ka sequence packet (table, PK, old/new values).
* **Semisynchronous Configuration:** Data loss se bachne ka hybrid model jahan kam az kam do nodes par data har waqt real-time synced rehta hai.
* **Replication Lag:** Asynchronous followers ka leader ke naye data se seconds ya minutes piche reh jane ka network duration time delay.



---
