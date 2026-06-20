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

---

## Batch Processing in Distributed Systems

Hum ne jo single-machine par Unix tools (cat, awk, sort wagera) ki misaal dekhi, us mein log data ko process karne ke liye computer ke yeh components aapas mein mil kar kaam kar rahe تھے:

* **Storage Devices (Hard Drive/SSD):** Jinhein operating system ke filesystem interface ke zariye access kiya jata hai.
* **A Scheduler:** Jo yeh tay karta hai ke kaun sa process kab chalega aur CPU ki takat kisko kitni milegi.
* **Unix Programs:** Jo pipes (`|`) ke zariye aik doosre se jude hote hain (aik ka output doosre ka input banta hai).

Bilkul yahi saare components **Distributed Data Processing Frameworks** (hazaron computers par chalne wale systems) mein bhi maujood hote hain. Asal mein, aap in frameworks ko **Distributed Operating Systems** maan sakte hain; inke paas bhi apna filesystem, apna job scheduler, aur aise programs hote hain jo filesystems ya doosre raabta raston ke zariye aik doosre ko data bhejte hain.

---

### Distributed Filesystems

Aap ke computer ka operating system jo filesystem (jaise ext4 ya XFS) deta hai, woh andar se kayi parton (layers) se mil kar banta hai:

* **Block Device Drivers:** Yeh sab se niflay level par direct disk se baat karte hain aur raw data blocks ko read/write karte hain.
* **Page Cache:** Yeh block layer ke upar banti hai aur hal hi mein use huye data blocks ka hissa memory (RAM) mein rakhti hai taake speed tez rahay.
* **Filesystem Layer:** Yeh block API ko lapet (wrap) kar ke barri files ko chote blocks mein torti hai aur files ka metadata (jaise inodes, directories, aur file names) ka hisab rakhti hai. Linux mein ext4 aur XFS iski aam misalein hain.
* **Virtual Filesystem (VFS):** Yeh sab se upar hota hai jo applications ko aik jaisa common API deta hai. VFS ki wajah se application ko is baat ki fikar nahi hoti ke piche ext4 chal raha hai ya XFS, uske liye file parhna aur likhna aik jaisa hota hai.

**Distributed Filesystems (DFS)** bhi bilkul isi tarah kaam karte hain. Files ko chote chunks ya **Blocks** mein tor diya jata hai aur unhein hazaron alag-alag machines par baant (distribute) diya jata ہے۔

* **Blocks Ka Size:** Distributed filesystem ke blocks aam local filesystem se **bohot baray** होते hain. Jahan Linux ka ext4 filesystem sirf 4,096 bytes (4 KB) ka block use karta hai, wahan Hadoop ka HDFS by default **128 MB** ka block use karta hai, aur JuiceFS ya baqi object stores **4 MB** ke blocks use karte hain.
* **Baray Blocks Ka Faida:** Jab data Petabytes (lakhoon GBs) ki tadad mein ho, toh agar blocks chote honge toh unka hisab kitab (metadata) rakhna azab ban jayega. Baray blocks hone se metadata bohot chota ho jata hai aur disk par data dhoondne ka kharcha (**seek overhead**) bhi parhne ke muqable mein bohot kam ho jata hai.
* **Aadha Block Likhna (Partial Blocks):** Computer ki physical hard drives kabhi bhi aadha block nahi likh saktin, is liye operating system ko poora block use karna parta hai chahe data thoda hi kyun na ho. Lekin distributed filesystems operating system ke filesystems ke upar bante hain, is liye un par yeh pabandi nahi hoti. Misaal ke tor par, agar ek 900 MB ki file hai aur aap 128 MB ka block size use kar rahe hain, toh system 128 MB ke 7 pure blocks banayega aur aakhri block bilkul exact **4 MB** ka banayega (khamkhah space zaya nahi karega).

#### Data Nodes Aur Unka Kaam

DFS ke blocks ko parhne ke liye network ke zariye us makhsoos machine ko request bheji jati hai jahan woh block save hota hai. Cluster ka har computer background mein aik worker program chala raha hota hai (ek daemon process) jo remote requests ko qubool karta hai aur blocks ko apne local disk se utha kar de deta hai.

* HDFS mein in worker programs ko **DataNodes** kaha jata hai.
* GlusterFS mein inhein **glusterfsd** processes kehte hain.
* Hum is book mein inhein aam zuban mein **Data Nodes** kahain ge.

#### Distributed Page Cache

Distributed filesystems mein bhi page cache ka distributed roop hota hai. Chunke DFS ke blocks data nodes par aam files ki shakal mein save hote hain, is liye jab bhi koi data parha ya likha jata hai, woh data node ke operating system ki **In-Memory Page Cache** se guzarata hai. Is se baar baar parha jane wala data data node ki RAM mein hi rehta hai aur speed fast milti hai. Kuch advance filesystems (jaise JuiceFS) mazeed caching tiers bhi dete hain, jaise client-side par ya local disk par cache banana.

#### Metadata Management

Jaise ext4 file ki locations aur directory structure ka hisab rakhta hai, waise hi distributed filesystems ko bhi yeh yaad rakhna parta hai ke kaun sa block kis machine par para hai.

* Hadoop (HDFS) mein is kaam ke liye aik makhsoos service hoti hai jisay **NameNode** kehte hain.
* DeepSeek ka **3FS** filesystem ek metadata service use karta hai jo apna saara data **FoundationDB** jaise key-value store mein paka save (persist) karti hai.

#### Protocols Aur VFS Ka Jora

Operating system mein jo kaam VFS karta hai, distributed processing mein wahi kaam DFS ka **Protocol ya Interface** karta hai. Yeh aik pluggable interface hota hai; jab tak koi distributed filesystem is protocol ke rules ko follow karega, batch processing systems usay asani se use kar sakenge.

* Misaal ke tor par, **Amazon S3 ka API** aaj kal ek industry standard ban chuka hai. MinIO, Cloudflare R2, Tigris, aur Backblaze B2 saare isi S3 API ko implement karte hain. Iska faida yeh hai ke jo batch processing system S3 ko support karta hai, woh in mein se kisi bhi storage system par bina code badle chal sakta hai.

NFS (Network File System) ya FUSE (Filesystem in Userspace) ke zariye in distributed filesystems ko operating system ke VFS ke sath aise jor diya jata hai ke user ko lagta hai ke woh apne hi computer ki koi local drive khol kar betha hai. Amazon ka **EFS (Elastic File System)** aur **Archil** isi NFS protocol ka use karte hain jo scale mein bohot baray distributed systems hote hain. Bahar se client sirf aik single endpoint par connect hota hai, lekin background mein yeh systems hazaron data nodes se data read/write kar rahe hote hain.

---

#### Distributed Filesystems and Network Storage

Distributed filesystems poori tarah se **Shared-Nothing Principle** (yani har computer azad hai, koi memory ya disk aam tor par aapas mein share nahi hoti) par chalte hain. Yeh network storage ke purane tareeqon jaise **NAS** (Network Attached Storage) aur **SAN** (Storage Area Network) ke **Shared-Disk** approach se bilkul alag hain.

* **Shared-Disk (NAS/SAN):** Is mein ek bohot barri central storage appliance (machinery) hoti hai, jis ke liye custom mehanga hardware aur special network infrastructure (jaise Fibre Channel) lagana parta hai.
* **Shared-Nothing (DFS):** Is mein kisi khas ya mehangay hardware ki zaroorat nahi hoti. Bas aam computers (**Commodity Hardware**) ko aam datacenter network ke zariye aapas mein jor diya jata hai.

Chunke distributed filesystems saste aur aam computers par bante hain, is liye in mein hardware kharab hone ka khatra (failure rate) enterprise hardware se zyada hota hai. Is musibat se bachne ke liye aur fault tolerance haasil karne ke liye blocks ko aik se zyada machines par copy (**Replicate**) kiya jata hai. Iska aik aur faida yeh hota ke scheduler ke paas choice hoti hai ke woh kaam ko us machine par chalaye jahan pehle se data ki copy pari ho, jis se workload barabar badhta hai.

Replication do tarah se ho sakti hai:

1. **Full Replication:** Ek hi data ki 3 ya 4 bilkul saaf copies alag machines par rakhna (Chapter 6).
2. **Erasure Coding:** Jaise **Reed–Solomon codes**. Is mein data ko mathematical formulas ke zariye aise toda jata hai ke agar koi machine mar bhi jaye, toh bache hue data se gumshuda data wapis nikal aata hai. Is mein full replication ke muqable mein storage ka kharcha bohot kam hota hai. Yeh bilkul computer ke andar lagne wale **RAID** jaisa hai, bas farq yeh hai ke distributed filesystem mein yeh redundancy poore network par phaili hoti hai.

---

### Object Stores

**Object Storage** services—jaise Amazon S3, Google Cloud Storage (GCS), Azure Blob Storage, aur OpenStack Swift—aaj kal batch processing ke liye distributed filesystems ka ek bohot hi popular maseeha ban chuki hain. Hatta ke distributed filesystem aur object store ke darmiyan ka farq ab bohot kam ho gaya hai. FUSE drivers ke zariye log S3 ko bhi local filesystem ki tarah mount kar lete hain, aur Ceph ya JuiceFS jaise systems filesystem aur object storage dono ke APIs aik sath dete hain. Lekin inke APIs, performance, aur consistency guarantees mein zameen-asman ka farq hota hai.

Object store ke andar har file (object) ka ek unique URL hota hai, jaise:
`s3://my-photo-bucket/2025/04/01/birthday.png`

* **Bucket:** Host ka hissa (`my-photo-bucket`) bucket ka naam hota hai jo poori duniya mein unique hona chahiye.
* **Object Key:** Uske baad ka saara hissa (`/2025/04/01/birthday.png`) us object ki unique key kehlata hai.

#### Object Stores Ke Rules (Bacho ki tarah samajhein)

* **Immutable (Na-badalne wale):** Object stores mein data ko **GET** call se parha jata hai aur **PUT** call se likha jata hai. Ek dafa jo object likh diya, aap uske andar ghuss kar badlao (update) nahi kar sakte. Agar aap ko file badalni hai, toh aap ko poori ki poori file dobara PUT call se naye siray se re-write karni paregi (halanqe Azure Blob aur S3 Express One Zone ab data aage jorna support karte hain, baki stores mein yeh facility nahi hai). Yahan traditional coding ke functions jaise `fopen` ya `fseek` (file ke beech mein jump karna) bilkul nahi hote.
* **Directories Ka Dhoka:** Jab aap S3 kholte hain toh aap ko folders aur directories nazar aati hain, lekin **object stores mein directories ka koi concept hota hi nahi hai!** Yeh slashes (`/`) sirf ek string (text) ka hissa hote hain. Jab aap directory listing mangte hain, toh system asal mein makhsoos **Prefix** (shuruati lafz) match karke objects ki list dikha deta hai.

Prefix listing aam filesystem se do tarah se alag hoti hai:

1. **Recursive By Default:** Agar aap `s3://my-photo-bucket/2025/04` ki list mangenge, toh yeh Unix ke `ls -R` ki tarah uske andar ke sub-folders ke saare objects bhi nikal kar le aayega.
2. **Khali Folder Mumkin Nahi:** Agar aap `s3://my-photo-bucket/2025/04/01` ke andar ke saare objects delete kar dein, toh `01` naam ka folder hi duniya se gayab ho jayega! Kyunke piche koi key bachi hi nahi jiske andar yeh text ho. Is se bachne ke liye log galti se wahan ek 0-byte ka khali object bana dete hain taake folder ka nishan bacha rahay.

Distributed filesystems mein hard links, symbolic links, file locking, aur atomic renames (aik jhatkay mein folder ka naam badalna) jaise features hote hain, jabke **object stores mein yeh sab missing hota hai**. S3 mein folder rename karne ka matlab hai ke uske andar maujood hazaron files ko aik aik karke naye naam par copy karo aur phir purani files ko delete karo!

#### Key-Value Stores Vs Object Stores

Chapter 4 wale key-value stores chote data (Kilobytes) aur har waqt tez low-latency reads/writes ke liye design hote hain. Jabke Object stores baray data (Megabytes se Gigabytes) aur aaram se barri reads ke liye optimize hote hain. (Halanqe ab **S3 Express One Zone** ne single-digit millisecond latency dena shuru kar di hai jo key-value store ke kareeb tar hai).

* **Data Locality Ka Farq:** HDFS ka sab se bara faida yeh tha ke woh computing task (code) ko us machine par chalata tha jahan file ka block pehle se para ho (**Data Locality**). Is se network ka bandwidth bachta hai kyunke code ka size file se chota hota hai.
* Doosri taraf, Object stores storage aur computation (servers) ko bilkul alag alag (**decouple**) rakhte hain. Is mein data network par travel karta hai, lekin chunke modern datacenter networks had se zyada fast hain, is liye yeh nuksan ab bardasht kar liya jata hai. Iska faida yeh hai ke aap storage aur CPU/RAM ko aik doosre se azaad scale kar sakte hain.

---

### Distributed Job Orchestration

Operating system ka jo asool hum ne parha, woh **Job Orchestration** (kaam ko tarteeb dena) par bhi apply hota hai. Jab aap single machine par Unix pipeline chalate hain, toh computer ka kernel piche `awk`, `sort`, `uniq` wagera ko chalane, memory baantne, aur CPU par schedule karne ka zimma uthata hai. Distributed system mein yahi kaam **Job Orchestrator** karta hai.

Jab koi batch processing framework kisi orchestrator ke paas job chalane ki request bents hai, toh us request mein yeh metadata shamil hota hai:

* Total kitne tasks chalane hain.
* Har task ko kitni Memory, CPU, aur Disk chahiye.
* Job ki unique ID aur credentials (ijazat nama).
* Input aur output data kahan para hai.
* Hardware ki requirements (jaise makhsoos GPUs ya SSDs).
* Job ka executable code (executable binary ya Docker image) kahan para hai.

**Kubernetes** aur **Hadoop YARN** (Yet Another Resource Negotiator) jaise orchestrators is jankari ko cluster ke metadata ke sath jor kar kaam chalate hain. Inke andar teen bunyadi components hote hain:

1. **Task Executors (Worker Daemons):**
Cluster ke har computer par ek worker program chal raha hota hai (jaise YARN ka **NodeManager** ya Kubernetes ka **kubelet**). Inki zimmedari hoti hai tasks ko chalana, central manager ko apni zinda hone ki dharkan (**Heartbeats**) bhejte rehna, aur resource utilization ka hisab rakhna. Jab inhein request milti hai, yeh code download karte hain aur task shuru kar dete hain. Yeh operating system ke sath mil kar **Linux cgroups** use karte hain taake aik task kisi doosre task ki memory mein ghuss na sakay aur na hi uske resources chura sakay (**Performance Isolation**).
2. **Resource Manager:**
Yeh pooray cluster ka main dimaagh hota hai jiske paas har node ki hardware capacity, khali jagah, chalte hue tasks, aur network locations ka poora global view hota hai. Chunke saari jankari is ke paas hoti hai, is liye yeh cluster ka bottleneck ya SPOF bhi ban sakta hai. YARN is state ko save karne ke liye **ZooKeeper** aur Kubernetes **etcd** ka istemaal karta hai.
3. **Scheduler:**
Yeh resource manager ke andar ka aik subsystem hota hai jo jobs ko start, stop ya check karne ki requests qubool karta hai. Sochein usay request mili ke *"10 tasks chalao jinhein makhsoos GPU chahiye"*. Scheduler global view check karega aur faisla karega ke yeh 10 tasks kis kis computer par chalenge, aur phir un computers ke Task Executors ko hukum bhej dega.

> **Subschedulers:** Sometime kuch kaamo ke liye application-specific scheduling chahiye hoti hai (jaise jab queries barhein toh read replicas ko auto-scale karna). Central scheduler aur yeh subschedulers mil kar kaam karte hain. YARN mein in subschedulers ko **ApplicationMasters** kehte hain aur Kubernetes mein inhein **Operators** kaha jata hai.

---

#### Resource allocation

Orchestration mein scheduler ka kaam sab se zyada challenging aur dimaagh ghumane wala hota hai. Usay cluster ke limited resources ko un saari jobs mein baantna parta hai jo aapas mein pehle chalne ki larai kar rahi hoti hain. Yahan do cheezon ka balance farz hai: **Fairness (Insaf)** aur **Efficiency (Karkardagi)**.

Bacho ki tarah aik simple misaal se samajhein: Sochein hamare paas ek chota cluster hai jiske 5 nodes hain aur total **160 CPU cores** khali hain. Achanak do alag alag jobs ki requests aati hain, aur **dono ko apna kaam khatam karne ke liye 100 Cores chahiye**. Ab scheduler kya karega?

* **Strategy 1 (Barabari):** Scheduler dono jobs ko 80-80 cores de deta hai. Aur bache hue 20-20 tasks tab shuru honge jab pehle wale tasks khatam honge.
* **Strategy 2 (Gang Scheduling):** Scheduler pehle aik job ko pure 100 cores de deta hai aur poori task ek sath chalata hai. Doosri job ko tab tak wait karwaya jata hai jab tak pehle wali ke cores azaad na ho jayein.
* **Strategy 3 (Incomplete Info):** Agar doosri job pehli job ke chalne ke kafi dair baad aaye, toh scheduler ko pehle se nahi pata tha. Woh pehli job ko saare cores de chuka hoga. Ab usay faisla karna hoga ke kya woh pehli job ke chalte hue tasks ko jaan se maar de (**Preemption**) taake naye ke liye jagah banay, ya naye wale ko bhookha (**Starvation**) rakhay.

Is choti si misaal mein hi kitne baray trade-offs hain! Agar gang scheduling mein scheduler cores reserve karke baith jaye jab tak pure 100 khali na hon, toh servers khali bethe rahenge aur cluster ki utilization gir jayegi, ya systems aapas mein **Deadlock** ho jayenge. Agar woh wait karwaye, toh starvation ho sakti hai. Agar tasks ko kill kare (preempt kare), toh pehle ki hui mehnat zaya ho jayegi aur efficiency kharab hogi.

Asal zindagi mein jab scheduler ke paas hazaron ya lakhon requests aa rahi hon, toh perfect math nikalna na-mumkin ho jata hai (Yeh ek **NP-hard problem** hai). Is liye practical schedulers perfect solution ke bajaye **Heuristics** (tajurbaati asoolon) ka sahara lete hain. Is ke liye mukhtalif algorithms use hote hain:

* **FIFO** (First-In First-Out)
* **DRF** (Dominant Resource Fairness)
* **Priority Queues**
* **Capacity / Quota-based scheduling**
* **Bin-packing algorithms**

---

#### Scheduling workflows

Unix tools ke example mein hum ne dekha ke kaise commands pipes ke zariye aik doosre se connect the. Distributed batch processing mein bhi bilkul yahi hota hai: aik job ka output doosri job ka input banta hai, aur aik job ke piche kayi inputs ho sakte hain jo alag alag jobs ne banaye hon. Is lambay silsilay ko hum **Workflow** ya **DAG (Directed Acyclic Graph)** kehte hain.

> **Durable Execution Vs Batch Workflows:** Chapter 8 mein jo workflow engines hum ne parhay the, woh step-by-step RPC calls (web requests) ke zariye kaam karte hain aur chota data process karte hain. Jabke Batch Processing mein workflow ka matlab hota hai baray baray data processes ki ek sequence, jo files parhti hain aur nayi files banati hain, external APIs par RPC calls nahi kartin.

Ek pipeline mein bohot saari jobs ka workflow banana kyun zaroori hota hai?

1. **Multi-Team Sharing:** Agar aik job ka output 5 alag alag teams ki jobs ke liye input hai, toh behtar yeh hai ke pehli job apna output storage mein save kar de. Baqi teams ki jobs jab bhi data update ho, wahan se parh sakti hain.
2. **Tools Ka Milan:** Aap alag alag tools ko jor sakte hain. Misaal ke tor par, aik **Spark** job ne data process karke HDFS par phenka, wahan se aik **Python** script ne trigger dabaya aur **Trino SQL** query chala kar us data ko mazeed makhsoos karke S3 par save kar diya.
3. **Multi-Stage Resharding:** Agar aik stage mein data ko *User_ID* ke mutabaq shard karna hai, aur aglay stage mein *Region_ID* ke mutabaq, toh pehli stage data ko naye roop mein shard karke save karegi taake doosri stage apna kaam asani se shuru kar sakay.

Unix pipeline mein do commands ke beech ka pipe memory ka aik chota sa buffer use karta hai aur disk par file nahi banata. Agar buffer bhar jaye, toh pehla process ruk jata hai jab tak doosra data parh na le (**Backpressure**). Spark aur Flink bhi aisa network-based streaming model support karte hain jahan data direct aik task se doosre task ko network par pass ho jata hai.

Lekin aam distributed workflows mein zyadatar tareeqa yeh hota hai ke har job apna final output **Distributed Filesystem ya Object Store** par likhti hai. Is se jobs aapas mein azad (**decouple**) ho jati hain. Agar aik job chal rahi hai, toh agli job ke liye workflow scheduler tab tak wait karega jab tak pehli job safely $100\%$ khatam na ho jaye aur data disk par paka save na ho jaye.

YARN ya Spark ka apna scheduler sirf aik individual job ke andar ke tasks ko sambhalta hai, pooray workflow ke dependency ko nahi jaanta. Is liye pooray workflows ko schedule aur manage karne ke liye alag se **Workflow Schedulers** use kiye jate hain, jaise **Airflow, Dagster, aur Prefect**. Barri companies mein 50 se 100 jobs ke workflows aam baat hain, is liye in tools ke bina data pipelines ka hisab rakhna namumkin hai.

---

#### Handling faults

Batch jobs bohot lambay waqt tak chalti hain. Jab aik job hazaron computers par ghantong chali ho, toh beech mein kisi aik task ka fail ho jana bilkul aam baat hai (chahe hardware kharab ho ya network ka jhatka lage).

Sometime scheduler khud bhi jaan-booch kar aap ke task ko kill (**preempt**) kar deta hai. Yeh tab hota hai jab aap saste virtual machines use kar rahe hon jinhein Amazon EC2 par **Spot Instances**, Azure par **Spot VMs**, aur Google Cloud par **Preemptible Instances** kehte hain.

* Yeh machines cloud providers ke paas bachi hui faltu computing power hoti hain jo woh bohot sasti (70-80% discount par) de dete hain.
* Batch processing jobs time-sensitive nahi hotin (yani agar thoda dair se bhi khatam hon toh chalta hai), is liye log kharcha bachane ke liye spot instances par batch jobs chalate hain. Lekin iska nateeja yeh hota hai ke jaise hi cloud provider ke paas koi mehanga regular customer aayega, scheduler aap ke chalte hue saste task ko foran kill kar dega. Hardware faults se zyada spot instances par preemption ki wajah se tasks fail hote hain.

Batch jobs mein kharabiyon se nipatna online systems se lakh darja asaan hai. Chunke batch jobs input ko cherti nahi hain aur output shuru se naya banati hain, is liye agar koi task fail ho jaye, toh system us aadhay-adhuray kharab output ko delete karta hai aur us task ko kisi doosre computer par **dobara restart (retry)** kar deta hai.

Aik task ke marne par poori 10 ghantay ki job ko shuru se chalana bohot barri bewakoofi hogi. Is liye MapReduce aur uske baqi naye bhai saare parallel tasks ko aik doosre se bilkul azad (independent) rakhte hain taake galti hone par sirf us aik chote task ko hi dobara chalaya jaye.

Workflow ke andar jab aik task ka data doosre ke liye input ho, toh fault tolerance thodi barik ho jati hai:

* **MapReduce Ka Tareeqa:** Yeh har stage ka beech ka data (**Intermediate Data**) har haal mein distributed filesystem (disk) par paka likhta hai aur aglay task ko tab tak touch nahi karne deta jab tak pehla paka complete na ho jaye. Yeh bohot safe hai lekin baar baar disk par likhne se slow ho jata hai.
* **Spark Ka Tareeqa:** Spark is beech wale data ko disk par likhne ke bajaye **RAM (Memory)** mein hi rakhta hai (agar memory bhar jaye tab hi local disk par spill karta hai). Yeh poora rasta yaad rakhta hai (**Lineage Tracking**) ke data kis formula se bana tha. Agar koi beech ka data zaya ho jaye, toh Spark poori job chalane ke bajaye sirf us khoye hue hissay ko formula dekh kar dobara compute kar leta hai.
* **Flink Ka Tareeqa:** Flink aik alag tareeqa apnaata hai jahan woh chalte hue tasks ka thodi thodi dair baad chupke se snapshot (**Checkpointing**) leta rehta hai taake galti hone par pichlay checkpoint se kaam shuru kiya ja sakay.

---

