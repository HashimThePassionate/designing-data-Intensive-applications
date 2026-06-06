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