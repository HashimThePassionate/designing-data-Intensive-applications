# Data Models and Query Languages

Ludwig Wittgenstein ke mashhoor maqoolay ka matlab software engineering ki duniya mein bohot gehra hai. Agar hamare paas kisi cheez ko bayan karne ke liye sahi alfaaz ya model nahi hain, toh hum us problem ka aik behtareen solution soch hi nahi sakte. Data models sirf code likhne ka tareeqa tay nahi karte, balkay yeh hamare zehn mein problem ko solve karne ka poora framework set karte hain.

Most applications ko aik doosray ke upar mukhtalif layers ki surat mein build kiya jata hai. Har layer ki sab se barhi kamyabi yeh hoti hai ke woh apne se neeche wali layer ki complex details ko chhipa deti hai (abstract kar deti hai) aur upar wali layer ko aik saaf aur asan interface deti hai.

### Data Models ki Abstraction Layers

Real-world applications mein data ko higher level se lower hardware level tak le jaane ke liye darja-zail layers kaam karti hain:

1. **Application Developer Layer (The Real World to Objects):**
Aap bator application developer haqiqi duniya ki cheezon (jaise log, products, transactions, ya sensors) ko dekhte hain aur unhein apni programming language ke objects, data structures, aur APIs mein convert karte hain. Yeh layer bilkul aapki application ke business logic ke mutabaq design hoti hai.
2. **General-Purpose Data Model Layer (Objects to Databases):**
Jab aapko un programming objects ko hamesha ke liye store (persist) karna hota hai, toh aap unhein aik general-purpose data model mein express karte hain. Yeh models JSON/XML documents, relational tables, ya phir graph ke vertices aur edges ki shakal mein ho sakte hain. Is layer ka maqsad data ko structured aur queryable banana hota hai.
3. **Storage Engine Layer (Databases to Bytes):**
Database software banane wale engineers yeh faisla karte hain ke un relational tables, documents, ya graphs ko memory, disk, ya network par **bytes** ki shakal mein kaise represent karna hai. Yeh layer tay karti hai ke data ko disk par kis tarah optimize kiya jaye taake search, manipulation, aur processing tez tareen ho sakay.
4. **Hardware Layer (Bytes to Electrical/Physical Signals):**
Sab se lowest level par hardware engineers yeh system design karte hain ke un bytes ko electrical currents, light pulses (fiber optics), ya magnetic fields (hard drives) mein kaise badla aur mahfooz kiya jaye.

<div align="center">
  <img src="./images/01.jpg" width="600"/>
</div>

* **Complexity Hiding:** Har layer apne se neeche wali layer ki pechidegiyon ko hidden rakhti hai. Aik application developer ko is baat se koi sarakkar nahi hota ke hardware par magnetic field kaise change ho rahi hai; unhein sirf apni API aur database model se matlab hota hai.
* **Effective Collaboration:** Is behtareen abstraction ki wajah se database vendor ke engineers (jo C/C++ mein storage engines likhte hain) aur application developers (jo Python ya Java mein business apps banate hain) aik doosray ke kaam mein rukawat bane bina, azadana aur behtareen tareeqay se kaam kar sakte hain.
* **Model Trade-offs:** Har data model ki apni taqat aur kamzori hoti hai. Jo data relational tables mein asani se fit ho jata hai, ho sakta hai usay graph model mein likhna mushkil ho, aur iska ulta bhi mumkin hai. Isliye sahi model ka intekhab poore system ki performance ka faisla karta hai.

---

## Terminology: Declarative Query Languages

SQL, Cypher, SPARQL, aur Datalog jaise query languages **declarative** hoti hain. Iska matlab yeh hai ke aap database ko sirf yeh batate hain ke aapko **kya (WHAT)** data chahiye, uske upar kya conditions honi chahiye, aur output kis tarah transform (sort, group, ya aggregate) hona chahiye. Aap database ko yeh hargiz nahi batate ke yeh goal **kaise (HOW)** hasil karna hai.

### Declarative vs Imperative ka Farq aur Trade-offs

* **The Role of Query Optimizer:** Declarative languages mein database ka aik ahem component jise **Query Optimizer** kehte hain, woh khud yeh faisla karta hai ke kaunsa index scan karna hai, kaunsa join algorithm (jaise Hash Join ya Merge Join) chalana hai, aur query ke mukhtalif hisson ko kis tarah sequence mein execute karna hai.
* **Imperative Approach (Python/Java):** Iske baraks, jab aap traditional programming languages mein code likhte hain, toh aap computer ko aik explicit algorithm dete hain ke pehle step A karo, phir array par loop chalao, phir condition check karo. Yeh *Imperative* approach hai jahan aap "HOW" par focus karte hain.

<div align="center">
  <img src="./images/02.jpg" width="600"/>
</div>

* **Performance and Maintenance Benefits:** Declarative query language ka sab se barha faida yeh hai ke yeh nihayat mukhtasar (concise) hoti hai aur isay likhna asan hota hai. Is se bhi barh kar, yeh query engine ke andarooni implementation details ko chupati hai. Agar database company apne execution engine ko mazeed behtar ya tez bana deti hai, toh aapko apna SQL query code badalne ki bilkul zaroorat nahi padti. Aapki queries khud-ba-khud tez ho jati hain.
* **Automatic Parallelism:** Aik complex database declarative query ko backend par multiple CPU cores aur alag-alag machines par parallel mein distribute karke execute kar sakta hai, aur developer ko is parallel processing ki ratti barabar bhi fikar nahi karni padti. Agar aap yahi kaam khud manually coding (imperative) ke zariye karte, toh aapko multi-threading, race conditions, aur data sharding ka poora manual control sambhalna padta, jo ke nihayat mushkil aur error-prone kaam hai.

---

## Interview aur Mockup System Design Scenario

### Scenario (The Problem)

Aap aik bade Scale ka **E-commerce Analytics Dashboard** design kar rahe hain jahan business users har ghante ki sales, category-wise trends, aur high-value customers ka data real-time mein filter aur analyze karte hain. Pehle yeh system aik custom imperative data script (Python loop-based filtering) par chal raha tha, jo ke scale nahi ho pa raha aur jab bhi data ka volume badhta hai ya storage format change hota hai, poora pipeline break ho jata hai. Aapko is system ko high availability aur performance ke liye redesign karna hai using Proper Abstraction and Query Models.

### System Design Core Decisions & Trade-offs

1. **Abstraction Layer Separation:** Hum Analytics API aur Raw Storage ke darmiyan aik clear separation layer rakhenge taake agar hum backend storage par Row-oriented format se Column-oriented format (e.g., Parquet/ClickHouse) par switch karein, toh dashboard ka dashboard code break na ho.
2. **Declarative Querying over Imperative Loops:** Data filtering aur aggregation ke liye custom loops likhne ke bajaye hum declarative querying use karenge. Is se query optimizer ko mouqa milega ke woh data sharding aur parallel execution ko handle kare bina application layer ko disturb kiye.

### Architectural Flow Diagram

<div align="center">
  <img src="./images/03.jpg" width="600"/>
</div>

### Interview Talk (Key Takeaways)

* **Interviewer Question:** Agar hum data processing ka logic application layer mein Python worker threads ke zariye likhein, toh kya nuksan hai?
* **Your Answer:** Agar hum imperative tareeqay se application layer mein data filter karenge, toh pehle hazaaron rows ko network ke zariye database se app server par pull karna padega (High Network I/O overhead). Mazeed yeh ke agar database ke indexes ya partition schemes change hoteri hain, toh hamara application code break ho jayega. Declarative model use karne se database engine query ko data ke bilkul qareeb (compute pushdown) parallelize kar sakta hai aur sirf final computed result network par bhejta hai, jo system ko scale karne ke liye behtareen design pattern hai.


---

## Relational Versus Document Models

Aaj ki software engineering ki duniya mein SQL sab se zyada jana-pehchana aur istemal hone wala data model hai, jis ki buniyaad Edgar Codd ne 1970 mein **Relational Model** ke roop mein rakhi thi. Is model ke mutabaq, data ko **Relations** (jise hum tables kehte hain) mein organize kiya jata hai, aur har relation un-ordered **Tuples** (yaani database rows) ka aik majmua hota hai.

### Relational Model Ka Irtiqa Aur Dominance

* **Theoretical Origins to Market Dominance:** Jab Edgar Codd ne yeh model pesh kiya, toh yeh sirf aik math-based theoretical proposal tha. Us daur ke engineers ko shaq tha ke kya is tarah ke model ko computer disk par efficiently implement kiya bhi ja sakta hai ya nahi, kyun ke is mein explicit pointers ke bajaye logical joins par rely kiya jata tha. Magar 1980s ke darmiyan tak, Relational Database Management Systems (RDBMS) aur SQL itne behtar ho chuke thay ke yeh regular ya structured data ko store aur query karne ke liye industry ka standard ban gaye.
* **Long-Term Success in Analytics:** Decades guzarne ke baad bhi, business analytics aur data warehousing jaiseuse cases mein relational model ka hi raj hai, jahan data ko *Star* aur *Snowflake* schemas mein organize karke complex reporting ki jati hai.

### Muqabla Aur Tareekhi Data Models

Relational model ne market par qabza karne ke liye lambi ladaas ladi hai. Iske raaste mein kayi competitors aaye:

* **Hierarchical and Network Models (1970s - 1980s):** Yeh relational model ke sab se pehle aur bade competitors thay. Hierarchical model (jaise IBM ka IMS) data ko aik tree structure mein dekhta tha, jabke Network model (CODASYL) explicit memory pointers ke zariye records ko aapas mein jorhta tha. Relational model ne in dono ko isliye shikast di kyun ke is ne developer ko physical pointer paths ke jhanjhat se azad kar diya.
* **Object Databases (Late 1980s - Early 1990s):** Jab Object-Oriented Programming (OOP) ka raaj shuru hua, toh laga ke object databases relational ka khatma kar dein gi taake programming objects ko direct save kiya ja sakay. Magar yeh commercial level par nakaam rahin aur bohot jaldi gayab ho gaein.
* **XML Databases (Early 2000s):** Internet ke anay par XML format ka shor macha, aur khususat XML databases bani hain, magar inka adoption sirf kuch makhsoos (niche) industries tak mahdood raha.

<div align="center">
  <img src="./images/04.jpg" width="600"/>
</div>

* **SQL's Adaptation Strategy:** Relational model ki kamyabi ka aik bada sabab yeh bhi hai ke is ne waqt ke saath khud ko badla. Jab XML aur JSON ka trend aaya, toh SQL databases (jaise PostgreSQL aur MySQL) ne in formats ko apne andar hi support karna shuru kar diya, jis se competitors ki zaroorat khatm ho gayi.

---

### NoSQL Aur NewSQL Ka Daur

2010s mein **NoSQL** ka lafz aik naye inqilab ke roop mein ubhra. Iska maqsad relational databases ki badshahat ko radd karna tha.

* **NoSQL Ke Core Pillars:** NoSQL koi aik single technology nahi hai, balkay yeh chand ahem makhsoos goals ka naam hai:
1. **Schema Flexibility:** Data store karne ke liye pehle se rigid tables aur columns design karne ki zaroorat nahi.
2. **Horizontal Scalability:** Data ko aik hi bade super-computer par rakhne ke bajaye commodity servers ke pure cluster par divide (shard) karna.
3. **Open Source Adoption:** Proprietary software ke bajaye open-source community-driven tools ka istemal.


* **The Rise of NewSQL:** Jab logo ne dekha ke NoSQL scale toh accha karta hai magar relational databases jaisi transactional guarantees (ACID properties) nahi deta, toh **NewSQL** databases wajood mein aaein. Unka maqsad NoSQL ki scalability aur Relational ka transactional trust aik sath dena tha. Waqt ke saath jab yeh saare principles aam databases mein merge ho gaye, toh NoSQL aur NewSQL jase words ka trend ab dheema ho chuka hai.

---

### Document Model Ki Kamyabi

NoSQL movement ka jo sab se mazboot aur mustaqil asar baqi raha, woh hai **Document Model** ki popularity, jo aam taur par data ko **JSON** format mein represent karta hai.

* **MongoDB aur Couchbase ka Kirdar:** Is model ko specialized document databases ne aam banaya. JSON documents ko unke flexible nature ki wajah se pasand kiya jata hai. Relational tables mein agar aapko aik naya column add karna ho, toh poore table par *ALTER TABLE* chalana padta hai jo bade production systems mein risk hota hai. Iske mukable mein, JSON documents mein har record (document) ka apna alag structure ho sakta hai bina kisi global schema ki pabandi ke.

```plaintext
Relational Storage (Rigid Normalized Structure):
[ Table: Users ] ───(Needs Join)───> [ Table: Contact Info ]

Document Storage (Self-Contained JSON Document):
{
  "user_id": 101,
  "name": "Hashim",
  "contact": { "email": "hashim@email.com", "phone": "12345" }
}
```

---

## Interview aur Mockup System Design Scenario

### Scenario (The Problem)

Aap aik modern **Content Management System (CMS)** aur User Profile service design kar rahe hain jahan alag-alag qism ke users (e.g., Authors, Clients, Guest Users) apna data save karte hain. Har user type ke attributes bilkul mukhtalif hain (Authors ke paas unki publications aur bio hai, jabke Clients ke paas company details aur billing history hai). Mazeed yeh ke marketing team har hafte naye custom fields profile mein add karti rehti hai. Agar aap aik traditional relational database use karte hain, toh har schema change par heavy migration karni padti hai jo zero-downtime deployement mein rukawat banti hai.

### System Design Core Decisions & Trade-offs

* **Relational Core Data vs Document Profile Data:** Hum core application features (jaise billing, accounting, subscriptions) ke liye relational storage engine use karenge kyun ke wahan multi-table transactional consistency zaroori hai. Magar User Profile aur Custom Fields ke liye hum **Document Model (JSON)** ka intekhab karenge.
* **Trade-off (Schema-on-Write vs Schema-on-Read):** Relational database *Schema-on-write* hota hai (database check karta hai ke data format sahi hai ya nahi save karte waqt). Document database *Schema-on-read* hota hai (data jaisa bhi ho save ho jata hai, application read karte waqt uski validation handle karti hai). Hum flexibility ke liye schema-on-read ka trade-off select kar rahe hain taake product agility barhi rahe.

### Architectural Flow Diagram

<div align="center">
  <img src="./images/05.jpg" width="600"/>
</div>

### Interview Talk (Key Takeaways)

* **Interviewer Question:** Kya hum document model ka maza kisi relational database mein nahi le sakte? Aaj kal toh PostgreSQL bhi JSONB support karta hai.
* **Your Answer:** Bilkul le sakte hain, aur yeh aik behtareen hybrid approach hai. Agar hum PostgreSQL ka `JSONB` data type use karte hain, toh humein relational integrity (Foreign Keys for main user ID) bhi milti hai aur sath hi aik column ke andar dynamic JSON data store karne ki flexibility bhi mil jati hai. Magar agar hamara poora application workflow hi completely document-centric hai aur humein bohot bade scale par data ko cluster par write-heavy partitions mein distribute karna hai, toh MongoDB jaisa dedicated document database native horizontal sharding aur behtar scaling automation provide karega jo RDBMS ke upar JSON chalane se zyada efficient hoti hai.

---

## The Object-Relational Mismatch

Software engineering ki dunya mein aik bohot bada structural gap hai: jis tarah data ko application code (Object-Oriented Programming like Java, Python) mein represent kiya jata hai, aur jis tarah data ko Relational Databases (SQL tables, rows, columns) mein mahfooz kiya jata hai, un dono mein aik wazeh tazaad (disconnect) hai. Is disconnect ko distributed systems ki zabaan mein **Impedance Mismatch** kaha jata hai.

### Impedance Mismatch Ka Concept Aur Analogy

Yeh term asal mein electronics se udhaar li gayi hai. Kisi bhi electric circuit ke inputs aur outputs ki aik makhsoos impedance (alternating current ke khilaf resistance) hoti hai. Agar aap do circuits ko aapas mein jorhte hain, toh maximum power transfer tabhi hota hai jab dono ki impedance aapas mein match karein. Agar mismatch ho, toh signal reflect hota hai aur energy ka zaya hota hai.

Bilkul isi tarah, software mein jab aap application ke rich object graphs (jo memory mein pointers ke zariye connected hote hain) ko database ke flat 2D relational tables mein save karne ki koshish karte hain, toh aapko aik complex translation layer likhni padti hai. Is translation layer ki wajah se development velocity dheemi ho jati hai aur system ki performance par asar padta hai.

---

## Object-relational mapping

Is translation layer ke bohot bade boilerplate code (repetitive SQL statements) ko kam karne ke liye **Object-Relational Mapping (ORM)** frameworks (jaise Java mein Hibernate aur Ruby/Rails mein ActiveRecord) banaye gaye. ORM ka kaam programming objects ko database tables ke sath automatically map karna hai, magar is approach par bohot architectural criticisms hain.

### ORMs Ke Core Problems Aur Architectural Trade-offs

* **Abstraction Leakage aur Complexity:** ORMs bohot complex hote hain aur yeh relational model aur object model ke darmiyan ke farq ko poori tarah chhipa nahi sakte. Aik developer ko barh-e-raast dono models ke baare mein sochna padta hai, kyun ke agar database level par data normalized hai, toh ORM ko batana padta hai ke objects ko kis tarah map kiya jaye.
* **OLTP vs Analytics (OLAP) ka Pechida Masala:** ORMs ko aam taur par sirf OLTP (Online Transaction Processing) applications yaani daily operations ke liye design kiya jata hai. Data engineers jo analytics aur business intelligence par kaam karte hain, unhein direct raw relational structures aur tables par queries chalani hoti hain. Isliye, agar ORM ne backend par ajeeb o ghareeb schema generate kiya hai, toh data warehousing aur analytics ke liye bohot mushkil paida ho jati hai.
* **Polyglot Persistence Ki Kami:** Zyada tar ORMs sirf relational SQL databases ke sath hi chalte hain. Agar kisi complex organization mein diverse data systems use ho rahe hain—jaise Search Engines (Elasticsearch), Graph Databases (Neo4j), ya NoSQL document stores—toh wahan traditional ORMs fail ho jate hain aur unka support khatam ho jata hai.
* **Inefficient Schema Generation:** Kuch ORMs database schema ko khud-ba-khud generate karte hain. Yeh auto-generated schemas aksar inefficient hote hain aur queries ko database par slow kar dete hain. Jab aap unhein customize karne baithte hain, toh customization itni complex ho jati hai ke ORM use karne ka a असल faida hi khatam ho jata hai.

### The N+1 Query Problem (Performance Bottleneck)

Yeh ORM frameworks ka sab se mashhoor aur khatarnak performance trap hai.

Misaal ke taur par, aapko aik page par $N$ comments display karne hain, aur har comment ke sath uske author ka naam dikhana hai.

1. **ORM Approach:** ORM pehle aik query chalaye ga jo $N$ comments le kar aayegi. Ab har comment ke andar author ki `user_id` maujood hogi. Author ka naam nikalne ke liye ORM loop ke andar har single comment ke liye alag se database query chalaye ga. Is tarah pehle comment ke liye 1 query, aur phir $N$ comments ke authors ke liye $N$ mazeed queries chalengi. Total **$N+1$ queries** database par hit karengi, jo network I/O overhead ko be-tahasha badha dengi.
2. **Handwritten SQL Approach:** Agar aap khud SQL likhein, toh aap aik single query ke andar hi `JOIN` operations perform kar ke comments aur users ka data aik hi dafa mein database se nikal sakte hain.

```plaintext
[ Handcoded SQL Solution (1 Query) ]
User Request -> SELECT * FROM comments JOIN users ON comments.user_id = users.id -> Returns complete data in 1 Roundtrip

[ ORM Inefficient Solution (N+1 Queries) ]
User Request -> 1. SELECT * FROM comments (Returns N rows)
             -> 2. SELECT * FROM users WHERE id = 1
             -> 3. SELECT * FROM users WHERE id = 2  ... up to N times (N Roundtrips!)

```

* **ORMs Ke Advantages:** In sab nuksanat ke bawajood, ORMs ka faida yeh hai ke yeh simple CRUD operations ke liye boilerplate code ko khatam karte hain. Yeh database query results ko cache karne mein madad dete hain aur database schema migrations (tables ke structure ko upgrade/downgrade karna) ko manage karna asan banate hain.

---

## The document data model for one-to-many relationships

Har qism ka data relational tables mein fit nahi baithta. Is ko samajhne ke liye hum LinkedIn Profile (Resume) ki real-world example ko dono models ke lehaz se dissect karte hain.

<div align="center">
  <img src="./images/06.jpg" width="600"/>
</div>

### Relational Model Ka Approach (Figure 3-1)

Agar hum LinkedIn profile ko relational database mein design karein (jise Figure 3-1 mein dikhaya gaya hai), toh humein pooray data ko normalize karna padega:

* **users table:** Is mein user ki unique identity `user_id (251)`, `first_name`, `last_name`, aur profile picture ka URL save hoga, kyun ke yeh pure profile mein sirf aik hi dafa aate hain.
* **regions table:** Location (`region_id`) ko alag table mein rakha jata hai taake data duplicate na ho (jaise `us:91` ka matlab "Washington, DC, United States" hai).
* **positions, education, aur contact_info tables:** Chunke aik user ke paas multiple jobs (positions) aur multiple degrees (education) ho sakti hain, isliye yeh **One-to-Many relationship** banata hai. In sab ko alag-alag tables mein rakha jata hai aur har record mein aik Foreign Key (`user_id = 251`) lagayi jati hai jo `users table` ke primary record ko point karti hai.

**Problem:** Jab bhi aapko Barack Obama ki profile ka page load karna hoga, system ko ya toh 4 se 5 alag-alag SQL queries chalani padengi ya phir aik bohot hi messy aur heavy multi-way `JOIN` query execute karni padegi, jo disk par se alag-alag jagah se data dhoond kar layegi.

### Document Model Ka Approach (Example 3-1 aur Figure 3-2)

<div align="center">
</div>

Document model isi information ko aik self-contained **JSON document** ki shakal mein store karta hai (jise Example 3-1 mein dikhaya gaya hai).

```plaintext
[ Document Store Logical Disk Layout ]
{
 "user_id": 251,
 "first_name": "Barack",
 "last_name": "Obama",
 "headline": "Former President of the United States of America",
 "region_id": "us:91",
 "photo_url": "/p/7/000/253/05b/308dd6e.jpg",
 "positions": [
 {"job_title": "President", "organization": "United States of America"},
 {"job_title": "US Senator (D-IL)", "organization": "United States Senate"}
 ],
 "education": [
 {"school_name": "Harvard University", "start": 1988, "end": 1991},
 {"school_name": "Columbia University", "start": 1981, "end": 1983}
 ],
 "contact_info": {
 "website": "https://barackobama.com",
 "x": "https://x.com/barackobama"
 }
}
===> Stored together as a single contiguous block/string on disk.

```

* **Data Locality (Read Optimization):** JSON document ka sab se bada faida yeh hai ke is mein **Data Locality** behtareen hoti hai. Profile ka poora data disk par aik hi jagah, aik hi contiguous block mein save hota hai. Jab application profile request karti hai, toh database bina kisi `JOIN` ke, sirf aik single disk read operation se poora tree-structure data utha kar le aata hai, jo read latency ko bohot kam kar deta hai.


  <img src="./images/07.jpg" width="600"/>

* **Tree Structure Explicit (Figure 3-2):** One-to-many relationships asal mein data ka aik tree structure banati hain. Figure 3-2 ke mutabaq, Root node `user 251` hai. Is root node se branches nikal rahi hain: aik branch `positions` ki taraf jati hai jo aage sub-branches `job 1`, `job 2` mein divide hoti hai. Doosri branch `education` ki taraf jati hai. JSON model is tree structure ko naturally aur explicitly apnata hai, jis se programming codes ke objects ke sath impedance mismatch khatam ho jata hai.

### The Bounded vs Unbounded Trap (One-to-Many vs One-to-Few)

Yahan aik bohot critical architectural nuance hai: LinkedIn resume mein positions aur education hamesha limited hoti hain (aam taur par aik insan 5 ya 10 jobs badalta hai). Isko hum **One-to-Few** kehte hain. Aise data ke liye document model mein data ko embed karna perfect hai.

Magar, agar relationship sach mein **One-to-Many** ke bajaye **One-to-Unbounded** ho—misaal ke taur par, kisi celebrity ki social media post par aane wale lakhon comments—toh aap un saare comments ko single post document ke andar embed **nahi** kar sakte. Is se document ka size hadd se zyada badh jayega (jaise MongoDB ki 16MB per document limit hai), read/write magnification ka masla hoga, aur system crash ho jayega. Aise unbounded scenarios ke liye hamesha relational table approach (jahan har comment alag row hoti hai) behtar hoti hai.

---

## Interview aur Mockup System Design Scenario

### Scenario (The Problem)

Aap aik bade scale ka **Professional Networking Platform (LinkedIn Clone)** design kar rahe hain. Is platform par har mahine millions of active users profiles dekhte hain (Read Heavy). Purana architecture aik traditional RDBMS par chal raha tha jahan profile load karne ke liye 6 tables par custom joins lagte thay. Peak hours mein heavy joins aur ORM ke generate kiye huay inefficient queries ki wajah se Database CPU 95% touch kar jata hai, aur profile loading mein latency 3 seconds tak chali jati hai. Aapko is profile rendering pipeline ko sub-100ms latency par lekar aana hai.

### System Design Core Decisions & Trade-offs

1. **Migration to Document Model for Profiles:** Hum user profiles ke read-heavy section ko relational DB se hata kar aik Document Store (ya PostgreSQL JSONB column with indexing) par shift karenge taake multi-table joins ka jhanjhat hi khatam ho jaye aur Data Locality ka bharpoor faida uthaya ja sakay.
2. **Trade-off (Write Cost vs Read Speed):** Document model select karne se profile update (write operation) thoda costly ho sakta hai kyun ke poora document dobara write karna padta hai, magar hamara system 95% Read-Heavy hai, isliye hum ne reads ko optimize karne ke liye writes par thoda compromise (trade-off) kiya hai.

### Architectural Flow Diagram

<div align="center">
  <img src="./images/08.jpg" width="600"/>
</div>

### Interview Talk (Key Takeaways)

* **Interviewer Question:** Agar hum document model use karte hain aur kal ko kisi school ya company ka naam change hota hai, toh humein har user ke profile document ke andar ja kar usay update karna padega. Is data duplication aur consistency ke maslay ko kaise handle karenge?
* **Your Answer:** Yeh document model ka aik ahem trade-off hai jise *Denormalization* kehte hain. Agar school ya company ka naam badalta hai, toh updates asynchronous background workers ke zariye batch processing mein chalaye ja sakte hain, kyun ke profiles par thodi si eventual consistency (kuch der baad naam update dikhna) acceptable hai. Magar iske badlay mein jo humein read speed mein data locality ka faida mil raha hai, woh production system ko scale karne ke liye zyada zaroori hai. Agar transaction consistency 100% zaroori ho (jaise user payments), toh hum strictly relational path hi use karenge.

---