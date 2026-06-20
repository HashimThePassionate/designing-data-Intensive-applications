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


## Batch Processing with Unix Tools

Sochein aap ke paas ek web server hai jo har dafa kisi user ki request handle karte waqt log file ke aakhir mein ek nayi line jor (append) deta hai. Agar hum NGINX server ka default log format use karein, toh log file ki aik akeli line dekhne mein aisi lagegi:

```text
216.58.210.78 - - [27/Jun/2025:17:55:11 +0000] "GET /css/typography.css HTTP/1.1" 200 3377 "https://martin.kleppmann.com/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36"

```

Is aik akeli line mein jankari (data) ka ek samandar chhupa hua hai. Isay samajhne ke liye hamein NGINX ke log format ki definition ko dekhna hoga, jo ke yeh hai:

```text
$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"

```

Chalein is mushkil line ko bacho ki tarah asaan karke breakdown karte hain ke is mein kya likha hai:

* **`216.58.210.78` (`$remote_addr`):** Yeh us user (client) ka IP address hai jo website par aaya.
* **`-` (`$remote_user`):** Yeh user authenticated nahi tha (yani us ne login nahi kiya hua hai), is liye yahan nishan (-) laga hai.
* **`[27/Jun/2025:17:55:11 +0000]` (`$time_local`):** Yeh woh exact tareeq aur waqt hai jab request server tak pohanchi.
* **`"GET /css/typography.css HTTP/1.1"` (`$request`):** User ne server se kaha ke *"Mujhe `/css/typography.css` naam ki file laa kar do (GET request)"*.
* **`200` (`$status`):** Iska matlab hai kaam kamyab (HTTP 200 OK) raha.
* **`3377` (`$body_bytes_sent`):** Jo file bhejii gayi, uska size 3,377 bytes tha.
* **`"https://martin.kleppmann.com/"` (`$http_referer`):** User is se pehle is website par betha tha, jahan se click karke ya link ke zariye woh is file tak pohancha.
* **`"Mozilla/5.0..."` (`$http_user_agent`):** User Chrome browser version 137 use kar raha tha ek Mac computer par.

Hamein lag sakta hai ke logs ko parhna aur parse karna ek aam sa boring kaam hai, lekin sach yeh hai ke aaj kal ki barri barri tech companies (jaise ad networks ya payment processors) ke poore dhanday (pipelines) isi par chalte hain. Hatta ke MapReduce aur Big Data ki poori tehreek (movement) shuru hi isi wajah se hui thi ke logon ne in logs ko process karna tha.

---

### Simple Log Analysis

Bohot se naye aur mehangay tools aate hain jo in log files ko utha kar pyari pyari reports aur graphs bana dete hain. Lekin apni samajh ko pakka karne ke liye hum yeh report bilkul zero se khud banayenge, woh bhi Unix (Linux) ke bilkul bunyadi commands ka istemaal karke.

Sochein aap ka boss aap se kehta hai ke *"Mujhe apni website ke **top 5 sab se zyada chalne wale pages (URLs)** nikal kar do"*. Aap Linux terminal par aik single line ka command pipeline likhenge:

```bash
cat /var/log/nginx/access.log | awk '{print $7}' | sort | uniq -c | sort -r -n | head -n 5

```

Yeh command kaise kaam karta hai, iska step-by-step breakdown bina kisi mushkil ke samajhte hain:

1. **`cat /var/log/nginx/access.log`**
Yeh command aap ki poori log file ko parh kar uska sara text aage bhej deta hai. (Sahi baat toh yeh hai ke yahan `cat` lagana zaroori nahi tha, hum seedha file ka naam `awk` ko bhi de sakte the, lekin is tarah aik seedhi line ya factory ki assembly line asani se samajh aati hai).
2. **`awk '{print $7}'`**
`awk` ek bohot hi pyara worker hai. Yeh har line ko uthata hai aur jahan jahan khali jagah (whitespace) hoti hai, wahan se data ke tukray kar deta hai. Hum ne usay kaha `{print $7}`, yani har line ka **saatwan (7th) tukra** bahar nikalo. Hamari log line mein saatwan tukra wahi URL (`/css/typography.css`) hai jo user ne maanga tha. Ab is step ke baad baqi saari jankari (IP, Browser, Time) phenk di gayi hai, sirf URLs ki ek lambi list agay ja rahi hai.
3. **`sort`**
Yeh command saare URLs ko ABC ke mutabaq (alphabetically) tarteeb de deta hai. Iska faida yeh hota hai ke agar ek hi URL website par 100 dafa maanga gaya tha, toh woh saare 100 ke 100 nishan aik doosre ke upar-niche (aik sath) tarteeb mein aa jayenge.
4. **`uniq -c`**
`uniq` ka kaam hai duplicate cheezon ko khatam karna. Lekin `uniq` thoda bholu hai; yeh sirf tabhi duplicate pehchanta hai agar do bilkul ek jaisi lines **aik doosre ke bilkul barabar (adjacent)** pari hon (isi liye hum ne pehle step mein `sort` chalaya tha taake ek jaise URLs sath aa sakein). Is ke sath laga **`-c` (counter)** option database ko kehta hai ke duplicate line ko delete karte waqt sath mein ginti (count) bhi likh do ke yeh URL kitni dafa aaya tha.
5. **`sort -r -n`**
Ab hamare paas list aisi ban chuki hai jahan shuru mein ek number (ginti) likhi hai aur agay URL likha hai. Hum dobara `sort` chalate hain. Is dauran **`-n`** ka matlab hai ke alphabet ke bajaye **Number (ginti)** ke mutabaq tarteeb do, aur **`-r`** ka matlab hai **Reverse (ulta)** tarteeb do—yani sab se bara number sab se upar aaye aur chote numbers niche chale jayein.
6. **`head -n 5`**
Aakhir mein `head` command aata hai jo upar se sirf pehli **5 lines (`-n 5`)** ko pakarhta hai aur baqi ka kachra ya choti list ko phenk deta hai.

Is poore process ka final nateeja terminal par aisa dikhata hai:

```text
4189 /favicon.ico
3631 /2016/02/08/how-to-do-distributed-locking.html
2124 /2020/11/18/distributed-systems-and-elliptic-curves.html
1369 /
 915 /css/typography.css

```

Aap dekh sakte hain ke sab se upar `/favicon.ico` hai jo 4,189 dafa load hua. Agar aap Unix tools se waqif nahi hain, toh yeh line dekhne mein thodi jadu jaisi lagay gi, lekin yeh had se zyada takatwar hai. Yeh kuch hi seconds mein **kayi Gigabytes (GBs)** ki log files ko ragar (process kar) sakti hai.

Agar aap ka mood badal jaye aur aap report badalna chahein, toh badlao bohot asaan hai:

* Agar aap CSS files ko report se nikalna chahte hain, toh awk ka argument badal kar `$7 !~ /\.css$/ {print $7}` kar dein.
* Agar aap dekhna chahte hain ke kaun se Users (IP addresses) sab se zyada aaye hain, toh url ke bajaye pehla field `{print $1}` kar dein.

`awk`, `sed`, `grep`, `sort`, `uniq`, aur `xargs` ka milap kuch hi minto mein aap ke baray se baray analysis ko chutkiyon mein hal kar deta hai aur inki speed bohot kamaal hoti hai.

---

### Chain of Commands Versus Custom Program

Ab kuch software developers keh sakte hain ke *"Yaar, yeh Linux terminal ki ajeeb si chain likhne ke bajaye hum khud ka ek saaf suthra code kyun na likh lein?"*. Bilkul likh sakte hain! Agar hum yahi poora kaam modern **Python** mein karna chahein, toh code kuch aisa dikhega:

```python
from collections import defaultdict

# 1. Counter ki dictionary banayein jahan shuruati ginti 0 hogi
counts = defaultdict(int)

# 2. Log file ko aaram se read karne ke liye open karein
with open('/var/log/nginx/access.log', 'r') as file:
    for line in file:
        # Har line ke tukray karein aur saatwan element (index 6) uthayein
        fields = line.split()
        if len(fields) > 6:
            url = fields[6]
            # 3. Us URL ke counter mein +1 ka izafa karein
            counts[url] += 1

# 4. Dictionary ko counter value ke mutabaq ulta (descending) sort karein aur top 5 lein
top5 = sorted(counts.items(), key=lambda item: item[1], reverse=True)[:5]

# 5. Top 5 results ko screen par print karwa dein
for url, count in top5:
    print(f"{count} {url}")

```

Yeh Python program Unix pipeline jitna chota toh nahi hai, lekin parhne mein bohot saaf hai. Kaun sa tareeqa behtar hai, yeh thoda sa aap ke zaati shauq (taste) par depend karta hai. Lekin upar-upar se dikhne wale is farq ke ilawa, jab isay aik **bohot barri file** par chalaya jaye, toh dono ke kaam karne ke tareeqe (execution flow) mein ek zameen-asman ka farq samne aata hai.

---

### Sorting Versus In-Memory Aggregation

Python script aur Unix pipeline ke darmiyan asal architectural farq yeh hai:

* **Python Ka Tareeqa (In-Memory Aggregation):** Python script computer ki memory (RAM) ke andar aik hash table (dictionary) banati hai, jahan har URL ke samne uski ginti save hoti rehti hai.
* **Unix Pipeline Ka Tareeqa (Sorting):** Unix pipeline mein koi hash table nahi banti! Woh saare ke saare URLs ko uthata hai aur unki aik lambi list ko direct sort (tarteeb) karna shuru kar deta hai taake ek jaise URLs aapas mein jor jayein.

Chalein bacho ki tarah iska nafa-nuksan samajhte hain ke kaun sa approach kab king hai:

**Case 1: Jab data chota ya darmiyana ho (In-Memory Jet)**
Agar aap ki website aam size ki hai, toh ho sakta hai ke poori website par total distinct (alag alag) URLs sirf kuch hazar hon. Un saare URLs aur unke counters ko RAM mein rakhne ke liye mushkil se **1 GB RAM** chahiye hogi.

Is surah-e-haal mein Python ka tareeqa bohot fit hai, kyunke us ka **Working Set** (woh memory jis par computer ko baar baar jaldi se hath marna parta hai) bohot chota hai. Agar ek hi URL 10 lakh dafa bhi aaye, hash table mein jagah sirf aik hi URL ki rehti hai, bas counter ka number barhta jata hai. Yeh kaam aap ke laptop par bhi chutkiyon mein ho jayega.

**Case 2: Jab data memory se bara ho jaye (Disk Spilling Power)**
Lekin sochein agar aap ke paas alag alag URLs itne zyada hain ke un ka counter RAM mein fit hi nahi ho raha (Working set RAM se bara ho gaya hai). Ab agar Python script chalayenge toh computer **Out of Memory (OOM Error)** de kar crash ho jayega.

Yahan par Unix ka **Sorting approach** baazi le jata hai, kyunke yeh disk (hard drive/SSD) ka behtareen use karna jaanta hai. Yeh bilkul wahi asool use karta hai jo hum ne Chapter 3 mein **Log-Structured Storage (LSM-Tree)** mein parha tha:

* **External Merge Sort:** Linux ka `sort` command data ke chote chote tukron ko pehle RAM mein sort karta hai, phir un chote tukron ko disk par temporary files (**Segment files**) bana kar save kar deta hai.
* Aakhir mein, woh in saare sorted segments ko aapas mein jor kar (**Merge** karke) ek barri sorted file bana deta hai.
* Is tareeqe ka faida yeh hai ke data ko hamesha sequential access (aik line se) parha aur likha jata hai, jo ke hard drives aur SSDs par bohot zyada tez chalta hai.

Linux ke GNU Coreutils mein jo `sort` utility aati hai, usay pehle se hi itna aqalmand banaya gaya hai ke agar data RAM se bara ho jaye, toh woh khud-ba-khud disk ka istemaal shuru kar deti hai (**spill to disk**) aur sath hi sath processor ke saare cores (**multiple CPU cores**) par voting aur sorting ko parallel kar deti hai. Iska matlab hai ke Unix commands bina crash hue bohot baray datasets ko hazam kar sakte hain. Wahan par bottleneck RAM nahi hoti, balkay sirf yeh hota hai ke aap ka disk kitni tezi se file ko read kar raha hai.

> **Sab Se Badi Limitation:** Unix tools ka sab se bara nuksan yeh hai ke yeh sirf **aik akeli machine (single machine)** par chalte hain. Agar data itna barh jaye ke woh aik computer ki memory aur disk dono se bahar nikal jaye, toh yahan par single machine Unix tools haar jaate hain—aur yahin se shuruat hoti hai **Distributed Batch Processing Frameworks** ki (jo data ko hazaron computers par baant kar process karte hain).

