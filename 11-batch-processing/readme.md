# Batch Processing

A system cannot be successful if it is too strongly influenced by a single person. Once the initial design is complete and fairly robust, the real test begins as people with many different viewpoints undertake their own experiments.
—Donald Knuth, “The Errors of TeX” (1989)

---

### Introduction: Online Aur Offline Systems Ka Farq

Ab tak is kitaab mein hum ne jitne bhi designs parhay hain, un mein zyada tar **Requests aur Queries** aur unke badlay mein milne wale **Responses (jawab)** ki baat ki gayi hai. Aaj kal ke zyadatar modern data systems isi tarah kaam karte hain: aap system se kuch maangte hain ya usay koi kaam karne ka hukum dete hain, aur system koshish karta hai ke aap ko jitna jaldi ho sakay jawab day day.

Misaal ke tor par:

* Jab aap browser mein koi website kholte hain (Web request).
* Jab aik service doosri service ko internet ke zariye call karti hai (Remote API call).
* Databases, caches, aur search indexes par chalne wali queries.

In sab systems ko hum **Online Systems** kehte hain.

* **Bacho ki Tarah Samajhein:** Yeh bilkul aisa hai jaise aap kisi dukan par gaye, dukandar se aik chocolate mangi, aur us ne foran aap ko chocolate thama di. Yahan sab se zaroori cheez **Response Time** (kitni jaldi jawab mila) hoti hai. Aur agar dukan ka computer kharab ho jaye toh dukan band ho jayegi, is liye in systems mein **High Availability** (har waqt chalte rehna) bohot zaroori hoti hai.

Lekin har kaam itna jaldi aur aam query se nahi ho sakta. Sochein agar aap ko:

* Aik bohot bara AI (Artificial Intelligence) model train karna ho.
* Hazaron gigabytes data ka roop badal kar usay kisi doosri shakal mein save karna ho.
* Bohot baray dataset par business analytics nikalni ho.

Aise baray kaamo ko hum interactively (aik hi request mein) nahi kar sakte. In baray kaamo ko **Batch Processing Jobs** kaha jata hai, aur jo systems inhein chalate hain, unhein hum **Offline Systems** kehte hain.

---

### Batch Processing Ki Khususiyaat Aur Faide

Ek **Batch Processing Job** ka asool bohot seedha hota hai: yeh pehle se maujood **Input Data** ko uthata hai (jo ke **Read-Only** hota hai, yani us mein koi radd-o-badal nahi kiya jata) aur us par poora process chala kar ek naya **Output Data** bilkul zero se (from scratch) naya generate karta hai.

Yeh aam read/write transactions ki tarah purane data ke andar ghuss kar changing (mutate) nahi karta. Is ka matlab hai ke aap ka output poori tarah input se nikalta (derive hota) hai. Agar aap ko naya output pasand nahi aaya, toh aap usay delete karein, apne code ka logic theek karein, aur job ko dobara chala dein!

Input data ko **Immutable** (na-badalne wala) rakhne aur side effects (jaise external databases mein sath sath changes karna) se bachne ke wajah se batch jobs ke bohot saare shaandar faide hote hain:

* **Human Fault Tolerance (Insaani Galtiyon Se Bachao):** Sochein agar aap ne code mein koi bug (galti) chor di aur naya output kharab ya corrupt generate ho gaya. Aap ko ghabrane ki bilkul zaroorat nahi! Aap bas code ka purana sahi version wapis layein (roll back karein) aur job ko dobara chala dein—aap ka output phir se bilkul sahi ho jayega.
* *Bacho ki Tarah Samajhein:* Yeh aisa hai jaise aap ne whiteboard par marker se drawing banayi. Agar galat bani, toh aap ne usay mittaya aur dobara shuru se sahi drawing bana li. Lekin aam databases mein aisa nahi hota; agar transaction database mein galat data write kar de, toh code roll back karne se database ka kharab hua data khud theek nahi hota.


* **Time Travel (Waqt Mein Piche Jana):** Boxt se object stores aur open table formats is feature ko support karte hain jahan aap purane output ko aik alag directory mein mehfooz rakh sakte hain aur agar naya output kharab ho toh foran purane par switch kar sakte hain.
* **Agile Software Development:** Chunke is mein galti hone par nuksan ko hamesha ke liye reversible (wapis theek) kiya ja sakta hai, is liye developers bina daray jaldi jaldi naye features bana sakte hain kyunke unhein pata hai ke unki kisi galti se live data hamesha ke liye tabah nahi hoga.
* **Input Files Ka Reusability:** Aap aik hi input file ko bohot saari alag alag jobs ke liye use kar sakte hain. Hatta ke aap aisi monitoring jobs bhi chala sakte hain jo naye output ko purane output se compare karke check karein ke kahin naye data mein koi aisi kharabi toh nahi jo pehle nahi thi.
* **Resources Ka Behtareen Istemaal:** Batch processing frameworks computers ki takat (CPU aur Memory) ko bohot tameez aur efficiency se use karte hain. Agar aap yahi bara data aam OLTP databases ya application servers par process karne baithenge, toh computer ke bohot zyada resources zaya honge aur kharcha bohot barh jayega.

---

### Batch Processing Ke Challenges Aur Kam bahiyaan

Itne saare faidong ke sath sath, batch processing mein kuch mushkilat (challenges) bhi hain:

1. **Aakhri Lamhay Tak Intezar:** Zyadatar frameworks mein naya output tab tak kisi doosri job ke istemaal ke qabil nahi hota jab tak **poori ki poori job khatam na ho jaye**. Aap ko aakhri byte ke process hone tak wait karna parta hai.
2. **Inefficiency (Susti):** Agar aap ke input data mein sirf aik chota sa naya badlao (even single byte change) bhi aaye, toh poori batch job ko shuru se saara ka saara input dataset dobara reprocess karna parta hai.
3. **Waqt Ki Keemat:** Ek batch job chalne mein minutes, ghantay, ya kayi din bhi le sakti hai. Log aam tor par inhein periodic schedule (jaise har raat 12 bajay) par chalne ke liye set kar dete hain.

* **Performance Ka Scale:** Is mein performance naapne ka scale response time nahi hota, balkay **Throughput** hota hai—yani system ne ek makhsoos waqt mein kitna zyada data process kiya.
* **Fault Handling:** Kharabi se nipatne ke liye kuch batch systems poori job ko abort (khatam) karke shuru se dubara restart karte hain, jabke kuch modern systems itne fault-tolerant hote hain ke agar kuch computers (nodes) crash bhi ho jayein, tab bhi job bina ruke safely mukammal ho jati hai.

> **Stream Processing (Aik Alternative):** Batch processing ka ek behtareen alternative **Stream Processing** hai. Is mein job kabhi chal kar khatam nahi hoti, balkay woh har waqt input data par nazar rakhti hai aur jaise ہی koi naya badlao aata hai, usay foran (kuch hi seconds mein) process kar leti hai. Is ko hum aglay Chapter 12 mein deeply parhenge.

---

### Batch Processing Ki Tareekh Aur Evolution (MapReduce Se Spark Tak)

Modern batch processing par sab se gahra asar **MapReduce** ka hai, jo Google ne 2004 mein ek algorithm ke tor par publish kiya tha. Baad mein isay open-source databases aur systems jaise **Hadoop, CouchDB, aur MongoDB** mein implement kiya gaya.

MapReduce ek kafi low-level programming model tha (yani is mein code likhna thoda mushkil aur basic hota tha) aur yeh distributed data warehouses ke parallel query execution engines jitna advance nahi tha. Lekin us zamane mein saste computers (**Commodity Hardware**) ko aapas mein jor kar bohot baray paimane (scale) par data process karne ke liye yeh ek bohot bara qadam tha. Aaj ke daur mein MapReduce purana ho chuka hai aur ab Google ke andar bhi is ka istemaal band ho gaya hai.

Aaj ke daur mein modern batch processing ke liye naye aur advance frameworks use kiye jate hain, jin ki tabdeeli ko hum is table ke zariye breakdown karte hain:

| Khasiyat (Feature) | Purana Daur (Hadoop/MapReduce Era) | Modern Daur (Modern Batch Frameworks) |
| --- | --- | --- |
| **Main Tools / Frameworks** | Apache Hadoop, MapReduce Core. | **Apache Spark, Apache Flink**, Cloud Data Warehouses. |
| **Execution Strategy** | Low-level coding, data ko baar baar disk par likhna parta tha. | Sophisticated **Caching** (Memory ka behtareen use) aur advance parallel execution. |
| **Usability / APIs** | Mushkil aur lamba custom Java code likhna parta tha. | **Dataflow APIs, Query Languages, aur DataFrame APIs** (Code likhna bohot asaan ho gaya hai). |
| **Workflow Schedulers** | Hadoop-centric tools jaise **Oozie** aur **Azkaban**. | Generalized modern solutions jaise **Airflow, Dagster, aur Prefect** (Jo cloud aur data warehouses ko asani se sambhaltay hain). |
| **Storage Layer** | Distributed Filesystems jaise **HDFS** (Hadoop Distributed File System), GlusterFS, CephFS. | Cloud **Object Storage** jaise **Amazon S3** ya Google Cloud Storage. |

Aaj kal **BigQuery** aur **Snowflake** jaise scalable cloud data warehouses ne data warehousing aur batch processing ke darmiyan ka farq bilkul khatam kar diya hai.

---

### Is Chapter Ka Roadmap

Batch processing ke is pure jahan ko achi tarah samajhne ke liye hum is chapter mein in steps ke mutabaq aage barhenge:

1. Hum sab se pehle ek single computer par standard **Unix Tools** (jaise grep, awk, sort) ka istemaal karke batch processing ki buniyadi samajh (intuition) banayenge.
2. Phir hum dekhenge ke isi data processing ko distributed system mein **aik se zyada machines** par kaise phailaya jata hai.
3. Hamein pata chalega ke ek normal operating system ki tarah distributed batch processing frameworks ke paas bhi apna ek **Scheduler** aur apna ek **Filesystem** hota hai.
4. Uske baad hum un alag alag **Processing Models** ko explore karenge jin ke zariye batch jobs ka code likha jata hai.
5. Aur aakhir mein, hum batch processing ke aam aur mashhoor **Real-World Use Cases** par gehrai se baat karenge.

---