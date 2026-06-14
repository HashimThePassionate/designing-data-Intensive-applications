# Distributed Systems Replication

Replication ka buniyadi maqsad data ki identical (huba-hu) copies ko mukhtalif network machines par distribute karna hai taake system hardware faults ko jhel sake, geographically data ko users ke qareeb rakh kar latency kam kar sake, aur massive read queries ko scale out kar sake. Is waqt dunya bhar ke databases teen main replication models par kaam karte hain.

---

## 1. Single-Leader Replication (Active/Passive)

Is architecture mein cluster ke andar sirf ek makhsoos node ko **Leader** designate kiya jata hai jo naye writes accept karta hai. Baqi saare nodes **Followers** hote hain jo strictly read-only hote hain aur leader ke data change log ko parh kar apna local data sync karte hain.

**Pros (Fawaid):**

* **Strict Consistency:** Data ka flow ek hi line (queue) mein leader se guzarta hai, jisse concurrent writes aapas mein takrate nahi hain aur operations sequential rehte hain.
* **Read-Scaling:** Read queries ko multiple asynchronous followers par distribute karke read traffic ka massive bojh leader se hataya ja sakta hai.

**Cons (Nuksanat):**

* **Availability Hit (Failover Penalty):** Agar leader crash ho jaye, toh system ko naya leader elect karne ke liye ek process chalana parta hai (Failover). Failover poora hone tak poora system naye writes accept karna band kar deta hai.

**Architectural Flaws & Limitations (Defined):**

* **Read-Your-Own-Writes Anomaly:** Asynchronous replication mein lag (delay) ki wajah se, jab user apna data update karke furan page refresh karta hai, toh request lagging follower par chali jati hai aur user ko apna naya data gayab (purana) dikhta hai.
* **Monotonic Reads Anomaly:** User jab lagataar queries chalata hai toh load balancer usay pehle ek fresh follower par aur phir ek lagging follower par bhej sakta hai. Is se user ko lagta hai jaise waqt piche chala gaya ho (e.g., pehle comment dikhna aur refresh karne par gayab ho jana).
* **Consistent Prefix Reads Anomaly:** Sharded databases mein causally related baatein (jaise sawaal aur uska jawab) network lag ki wajah se aage-piche ho jati hain, aur parhne wale ko jawab pehle aur sawaal baad mein dikhta hai.
* **Split-Brain Catastrophe:** Network partition ke waqt agar purana leader aur naya elect hone wala leader dono khud ko active master samajh baithein, toh dono aazadana writes accept karne lagte hain, jisse database hamesha ke liye corrupt ho jata hai.

**Algorithms & Patterns (Defined):**

* **Semi-Synchronous Pattern:** Saare followers ko synchronous lock karne se system ruk jata hai, isliye sirf ek follower ko strictly synchronous rakha jata hai aur baqi sab ko asynchronous. Agar sync node gir jaye toh foran kisi async node ko promote kar diya jata hai taake data loss zero rahe.
* **Logical (Row-based) Log Replication:** Storage engine ke physical disk blocks (WAL) bhejne ke bajaye data ko decoupled logical rows (e.g., `Table: Users, Row: 5, Action: Update`) mein bheja jata hai. Yeh abstraction database ko bina downtime ke naye software versions par upgrade hone ki ijazat deti hai (Zero-Downtime Rolling Upgrades).
* **Lease-Based Fencing Engine:** Split-brain anomaly se bachne ke liye leader ko `etcd` ya `ZooKeeper` se ek temporary lock (lease) leni parti hai. Agar network toot jaye toh bina lease ke node writes accept nahi kar sakta, jisse double-leader corruption ruk jati hai.

**Architecture Solutions (Defined):**

* **Sticky Routing:** Monotonic reads se bachne ke liye load balancer user ke ID ka hash nikal kar uski saari requests strictly ek hi makhsoos follower par lock kar deta hai taake data sequence kabhi piche na jaye.
* **Client-Side LSN Watermarking:** Read-your-writes anomaly ko hal karne ke liye client update ke waqt database se receipt ke taur par Log Sequence Number (LSN) save kar leta hai. Read ke waqt agar koi follower is LSN se piche ho, toh router query ko block kar deta hai ya direct leader par bhej deta hai.

**Use Cases & Real-World Applications (2026 Landscape):**

* **Primary Use:** Financial ledgers, banking systems, aur identity management jahan data loss aur conflict bilkul na-qabil-e-qubool hai.
* **Tech Stack:** PostgreSQL, MySQL, aur modern **NewSQL databases** (jaise CockroachDB aur Google Spanner) jo strict ACID guarantees aur linearizability dete hain.


---

## 2. Multi-Leader Replication (Active/Active)

Jab system globally mukhtalif continents mein phel jaye, toh har geographic region (maslan US aur Asia) ko apna ek aazad Leader assign kar diya jata hai. Har leader apne local users se direct writes accept karta hai aur background mein doosre regions ke leaders ko asynchronous stream bhejta hai.

**Pros (Fawaid):**

* **Zero-Latency Writes:** Har user ka write request direct uske local datacenter par commit ho jata hai, jisse inter-continent internet latency mukammal chhup (hide) jati hai.
* **Tolerance of Regional Outages:** Agar poora ek datacenter (e.g., Asia) offline ho jaye, toh US ka leader bina ruke aazadana writes process karta rehta hai.

**Cons (Nuksanat):**

* **Massive Write Conflicts:** Chunke do alag leaders ek doosre se bina pooche parallel writes accept karte hain, isliye aik hi record par concurrent tabdeeliyan aapas mein buri tarah takrati (conflict) hain.

**Architectural Flaws & Limitations (Defined):**

* **The Overtaking Anomaly (Causality Breakdown):** All-to-all topologies mein agar network line slow ho, toh naye insert ka data traffic mein phans sakta hai aur uspar depend karne wala update packet kisi doosre node par pehle land kar sakta hai, jisse database node confuse ho kar data corrupt kar deta hai.
* **The Set Union Bug:** Amazon jaisay purane systems mein jab concurrent conflicts ko resolve karne ke liye dono lists (Siblings) ko blindly merge (Set Union) kiya gaya, toh delete ki hui items dobara zinda ho kar cart mein wapas aane lagin.

**Algorithms & Patterns (Defined):**

* **CRDTs (Conflict-free Replicated Datatypes):** Yeh mathematical data types string ya object ke har hissay ko ek unique, immutable ID (jaise `Character 'e' = ID 3A`) de dete hain. Is tarah data nodes par bina index errors ke mathematically automatically merge ho jata hai.
* **Operational Transformation (OT):** Real-time text editors (jaise Google Docs) mein use hone wala algorithm jo concurrent edits ke index arrays ko live transform (shift) kar deta hai taake characters aage piche na hon.
* **Version Vectors:** Physical ghadiyon (clocks) par clock-drift ka masla hota hai. Ise hal karne ke liye har node ek logical counter rakhta hai, aur pure cluster ke counters ka array list (e.g., `[L1: 1, L2: 0]`) har packet ke sath bheja jata hai taake exact "Happens-Before" (sabab aur nateeja) history track ho sake.

**Architecture Solutions (Defined):**

* **Conflict Avoidance Sharding:** Data ko is tarah divide karna ke kisi makhsoos user ka data strictly usi ke local home region wale leader par hi write ho. Jab data doosre region jayega hi nahi, toh conflict paida hi nahi hoga.
* **Tombstones:** Collections mein set union bug se bachne ke liye item ko physically delete karne ke bajaye uspar "Tombstone" (maut ka thappa) laga diya jata hai. Merge ke waqt tombstone wali items automatically drop ho jati hain.

**Use Cases & Real-World Applications (2026 Landscape):**

* **Primary Use:** Collaborative environments, globally distributed content management, aur Local-First offline architectures.
* **Tech Stack:** Real-time canvas apps (Figma, Miro), collaborative docs (Google Docs, Notion), aur edge-compute local databases (Realm, SQLite via Sync Engines).


---

## 3. Leaderless Replication (Dynamo-Style)

Is architecture mein leader ka wajood mukammal khatam kar diya jata hai. Client apne write aur read requests cluster ke bohot saare replicas (nodes) ko ek sath parallel fire karta hai. Amazon Dynamo, Cassandra, aur ScyllaDB is model par chalte hain.

**Pros (Fawaid):**

* **Failover-Free Extreme Availability:** Chunke koi single master nahi hota, kisi bhi node ke crash hone par system ki write capability block nahi hoti.
* **Gray Failures Resilience (Request Hedging):** Agar koi node dead nahi hai par resource strain ki wajah se intahai slow chal raha hai, toh client multiple nodes ko parallel request bhej kar sab se fast aane wale response ko pick kar leta hai (Hedging), jisse system fast rehta hai.

**Cons (Nuksanat):**

* **Eventual Consistency:** System background mein sync hota hai isliye is bat ka hamesha khatra rehta hai ke client ko stale (purana) data milega.

**Architectural Flaws & Limitations (Defined):**

* **Partial Write Persistence (No Rollback):** Agar client naya data write kare jo Node 1 par save ho jaye, lekin Node 2 par disk full error se fail ho jaye, toh system error toh dega par Node 1 ka data automatically rollback (delete) nahi karega. Agli query mein wahi incomplete/failed data user ko nazar aa sakta hai.
* **Sloppy Quorum Disconnection:** Agar network itna buri tarah toot jaye ke lazmi nodes tak rasai na ho, toh database writes ko kisi bilkul un-assigned (random) node par udhaar rakhwa deta hai. Is halat mein writes toh safe rehte hain, lekin strict read guarantees toot jati hain.
* **Last-Write-Wins (LWW) Data Loss Trap:** Concurrent writes ko blindly time ke hisab se sort karne se, agar kisi node ki ghadi fast chal rahi ho, toh database lower timestamp wale naye writes ko silently hamesha ke liye delete kar deta hai.

**Algorithms & Patterns (Defined):**

* **Quorum Overlap Condition ($w+r>n$):** Yeh ek strict mathematical guarantee hai. Agar total replicas $n$ hain, toh successful writes ($w$) aur reads ($r$) ki tadaad is tarah set ki jati hai ke unka majmua $n$ se bara ho. Is pigeonhole principle se yeh pakka ho jata hai ke read karte waqt kam az kam ek node aisa lazmi milega jiske paas latest write mojud ho.
* **Read Repair:** Jab client parallel read karta hai aur use maloom hota hai ke kisi ek node ka version (e.g., v6) baqi nodes (v7) se purana hai, toh client background mein furan latest data us stale node par overwrite karke usay theek kar deta hai.
* **Hinted Handoff:** Agar koi node offline hai, toh zinda nodes uske writes ko apne paas ek hint (parchi) bana kar udhaar rakh lete hain. Jaise hi offline node wapas aata hai, wo udhaar writes usay hand-over kar diye jate hain.
* **Anti-Entropy Engine:** Yeh ek daimi background process hai jo replicas ki hashes ko Merkle Trees (Cryptographic Hash Trees) ke zariye compare karta hai. Yeh missing data dhoond kar nodes ko eventually sync karta hai.

**Architecture Solutions (Defined):**

* **Causal Context Tracking:** Siblings ko safely merge karne ke liye database client ko data ke sath poora Version Vector array (Causal Context) return karta hai. Client agla write bhejte waqt array wapas bhejta hai taake server concurrent edits ko pehchan sake.
* **Zero-Disk Architecture (ZDA):** Modern leaderless deployments jahan compute nodes totally stateless hote hain aur persistent data caching storage ke liye directly saste aur immutable cloud object stores (jaise Amazon S3) par flush karte hain.


**Use Cases & Real-World Applications (2026 Landscape):**

* **Primary Use:** High-throughput streaming data, continuous IoT telemetry, user activity tracking, aur multi-region caching.
* **Tech Stack:** Apache Cassandra, ScyllaDB, DynamoDB (NoSQL use cases), aur Riak jo high fault-tolerance aur fast ingestion mangte hain.

---

## 4. The 2026 Architect's Guide: Use Cases & Standard Implementations

| Feature Requirement | Recommended Architecture | 2026 Standard Implementation | System Behavior & Trade-off |
| --- | --- | --- | --- |
| **Financial Ledgers & Strict Billing** | **Single-Leader / NewSQL** | CockroachDB, Google Spanner | Ensures Linearizability. Trades off availability during network partitions to guarantee absolute zero data corruption. |
| **High-Scale IoT Telemetry Logs** | **Leaderless** | Cassandra, ScyllaDB | Tunes $w=1, r=3$ for maximum write throughput. Allows partial stale reads but ensures zero dropped telemetry metrics. |
| **Cross-Region Real-time Collaboration** | **Multi-Leader (CRDTs)** | Yjs, Automerge, WebSockets | Removes WAN latency via Local-First Sync engines. Requires complex application-level conflict resolution (Siblings/Tombstones). |
| **Heavy Analytical Workloads (OLAP)** | **Decoupled CDC Pipelines** | PostgreSQL -> Kafka -> Snowflake | Utilizes Logical Row-based replication to stream data without locking production read/write engines. |
| **Cost-Optimized Cloud Storage** | **Zero-Disk Architecture (ZDA)** | S3 Object Stores, SlateDB | Replaces expensive EBS volumes by deploying stateless nodes. Trades off sub-millisecond local reads for limitless S3 durability. |


---