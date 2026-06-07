# Storage and Retrieval

Richard Feynman ka 1985 ka yeh khayal bohot gehri sachai bayan karta hai: computer asal mein sirf hisab-kitab (arithmetic) nahi karte, balkay yeh bunyadi tor par **filing systems** hain jo data ko manage aur store karte hain. Ek database ka sabse basic aur core maqsad sirf do cheezon par khara hai: jab aap usay data dein toh wo usay mehfooz (store) kar le, aur jab aap baad mein mangein toh wo bina kisi galti ke wo data aapko wapas de de.

Ek application developer ke tor par aapko yeh samajhna kyun zaroori hai ke database andarooni tor par (under the hood) data kaise store aur retrieve karta hai? Halankeh aap apna khud ka storage engine zero se code nahi karenge, lekin jab aapko apni application ke liye koi database select karna hoga, toh uski performance ka inhasar isi baat par hoga ke aapne apni application ke **workload** ke mutabaq sahi storage engine chuna hai ya nahi. Sahi configuration aur tuning ke liye database ke internal mechanism ka rough idea hona laazmi hai.

Storage engines ke darmiyan sabse bada farq unke workloads ke mutabaq hota hai:

* **OLTP (Online Transactional Processing):** Yeh engines transactional workloads ke liye optimize hote hain jahan bohot zyada read/write queries hoti hain aur har query chote data segment ko target karti hai. Is mein do baray khandaan (families) aate hain: *Log-structured* engines (jo immutable data files likhte hain) aur *In-place update* engines (jaise B-trees, jo purane data ke upar hi naya data overwrite karte hain).
* **Analytics (OLAP - Online Analytical Processing):** Yeh engines baray analytical queries ke liye optimized hote hain jahan millions of rows ka data ek sath scan kar ke reports generate ki jati hain.

---

## Storage and Indexing for OLTP

OLTP ke storage mechanism ko bilkul basic level par samajhne ke liye hum dunya ka sabse sadah database design dekh sakte hain jo sirf do simple Bash functions par mushtamil hai.

```bash
#!/bin/bash

db_set () {
  echo "$1,$2" >> database
}

db_get () {
  grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}

```

### Architectural & System Behavior Breakdown

Agar hum is basic code ke data flow aur system behavior ko conceptual level par dekhein, toh iska system architecture is tarah kaam karta hai:

```plaintext
[ Client Request ] ---> db_set("42", "JSON_Data") ---> Append Operations (>>) ---> [ Disk File: database ]
                                                                                           |
[ Client Request ] ---> db_get("42") ---> Full File Scan (grep) ---> Tail -n 1 -----------> [ Latest Value Only ]

```

#### 1. Data Write Flow (`db_set`)

* **Mechanism:** Jab aap `db_set 12 '{"name":"London"}'` call karte hain, toh yeh function data ko `database` naam ki ek plain text file ke aakhir mein **append** (jor) deta hai.
* **Immutability Fact:** Agar aap key `42` ko teen dafa update karenge, toh purana data delete ya overwrite nahi hoga. Wo file mein upar maujood rahega aur naya updated data file ki aakhri line ban jayega.
* **Hardware Efficiency:** Disk par kisi file ke end mein data append karna (Sequential Write) bohot fast hota hai kyunki disk head ko baar baar move nahi karna parta. Yahi wajah hai ke real-world databases bhi internally ek **Log** use karti hain, jo ke ek append-only sequence of records hota hai.

#### 2. Data Read Flow (`db_get`)

* **Mechanism:** Jab aap `db_get 42` call karte hain, toh yeh engine `grep` command ke zariye file ki pehli line se lekar aakhri line tak poori file ko scan karta hai taake jahan jahan key `42` hai, un rows ko nikal sake.
* **Handling Duplicates:** Kyunki humne data overwrite nahi kiya tha, isliye file mein key `42` ke multiple records ho sakte hain. `tail -n 1` ka kaam yeh hai ke wo un saare matches mein se sabse aakhri (sabse late) match ko filter kar ke return karta hai, jo ke user ka current aur updated state hota hai.

#### 3. Real Log vs Application Log

Distrubuted systems aur databases mein **Log** ka matlab wo nahi hota jo hum application debugging (jaise log.info()) ke liye text files banate hain. Database context mein log ka matlab ek **append-only binary ya text file** hai jahan har naya event ya data state sequentially save hota jata hai. Real databases ko is ke sath sath concurrency (ek sath kai writes), disk compaction (purane data ko clean karna taake disk bhaar na jaye), aur crash recovery (agar darmiyan mein system band ho jaye toh data rescue karna) bhi handle karni parti hai.

### Performance Analysis & The Indexing Trade-off

Is simple database ka data storage format bohot badhiya perform karta hai jab baat sirf data write karne ki ho, kyunki write operations sequential hain. Magar jab read operations ki baat aaye, toh iski performance intehai buri ho jati hai.

* **Time Complexity Analysis:** Kyunki har read request par pure file ka scan shuru se aakhir tak laazmi hai, is lookup ki algorithmic cost **$O(n)$** hai. Agar database ka size $n$ records se double ho jaye, toh aapka search time bhi double ho jayega.
* **The Concept of an Index:** Is performace bottleneck ko door karne ke liye hum ek alag data structure introduce karte hain jise **Index** kehte hain. Index asal mein aapke original data se derive kiya gaya ek extra structure hota hai jo data ka rasta (address/pointer) apne paas kisi organized shakal mein save rakhta hai (jaise key ke mutabaq sorted form mein).

#### The Ultimate Storage Trade-off:

1. **Reads Speed-up:** Ek acha index aapke read queries ki time complexity ko $O(n)$ se ghata kar $O(1)$ ya $O(\log n)$ tak le aata hai.
2. **Writes Slow-down:** Har index database ke write operations ko slow kar deta hai. Jab bhi naya data write hoga, database ko na sirf primary log file mein append karna hoga, balkay index structure ke andar ja kar bhi us naye pointer ko update karna hoga.
3. **Space Overhead:** Index extra memory aur disk space consume karta hai.

Isi crucial trade-off ki wajah se koi bhi production-grade database har field par automatic index nahi banata. Ek application developer ko apni query patterns ka pta hona chahiye taake wo sirf unhi fields par index lagaye jinki sabse zyada reads required hain, taake writes par faltu overhead na aaye.

---

## Mockup System Design Scenario (Interview Style)

### Interview Context & Problem Statement

> **Interviewer:** "Hamein ek aisa High-Throughput Activity Logging system design karna hai jahan dunya bhar se millions of IoT devices har second temperature data send kar rahi hain. Writes ka rate intehai zyada hai, aur reads bohot kam hoti hain (sirf debugging ya check-up ke liye). Aap iska storage engine kaise design karenge?"

### Solution Strategy & Conceptual Flow

Hum yahan DDIA ke is core concept ko use karenge: **Append-only log for fast writes + In-Memory Index for quick lookups.**

1. **Write Path:** Har IoT device jab temperature data bhejegi, hum use bina kisi rukawat ke disk file ke end mein append karte jayenge ($O(1)$ write time complexity).
2. **Index Path:** Memory cache (RAM) ke andar hum ek simple Hash Map (Key-Value Key store) maintain karenge. Is Hash Map mein `Key` IoT Device ID hogi, aur `Value` disk file ka wo exact byte-offset (address) hoga jahan us device ka sabse aakhri record maujood hai.
3. **Read Path:** Jab koi engineer kisi specific device ka latest data mangega, system pehle RAM ke andar Hash Map se us device ka exact byte-offset uthayega, aur directly disk se sirf wahi line read kar lega. File ko shuru se end tak scan karne ki zaroorat nahi paregi.

### Architectural Flow Diagram

```plaintext
[ IoT Device 101 ] ---> (Sends Temp: 32°C) ---> [ Load Balancer / Ingestion Service ]
                                                       |
                                        +--------------+--------------+
                                        |                             |
                       (Append Write to End of File)       (Update Address in RAM)
                                        |                             |
                                        v                             v
                       [ Disk File: iot_events.log ]      [ In-Memory Hash Map Index ]
                       | ...                       |      | Key: 101                 |
                       | 101,31°C (Old Byte: 1024) |      | Value: Byte Offset 2048  | --+
                       | 101,32°C (New Byte: 2048) | <----+                          |   |
                                                                                     |   |
[ Engineer Dashboard ] ---> (Request Device 101) ---> [ Lookup Offset: 2048 ] <------+   |
                                                                                         v
                                                  [ Direct Disk Seek at Offset 2048 ] ---+

```

### Trade-off Evaluation in this Design

* **Pros:** Writes ultra-fast hain kyunki disk seek operations zero hain. Reads bhi instant ($O(1)$) hain kyunki RAM se direct file pointer mil jata hai.
* **Cons:** Agar RAM crash ho jaye toh index urr jayega (halankeh log file safe rahegi, aur index ko log scan kar ke dobara build kiya ja sakta hai). Doosra masla yeh hai ke saari keys ko RAM mein fit hona lazmi hai, jo bohot baray data ke liye expensive ho sakta hai.

---

## Log-Structured Storage

Agar hum data ko sirf ek append-only file mein save karte rahein aur reads ko fast karna chahein, toh iska sabse sadah hal yeh hai ke hum memory (RAM) ke andar ek **Hash Map** maintain karein. Is hash map ka kaam har ek `key` ko uske exact **byte offset** ke sath map karna hota hai. Byte offset se muraad disk file mein wo exact location ya point hai jahan us key ka sabse latest value data block shuru hota hai.

### Figure 4-1 Ka Gehrayi Se Mutaala (In-Memory Hash Map aur Log File)

Aap jo Figure 4-1 dekh rahe hain, wo is pure design ke internal data flow ko baray buraheen ke sath dikhati hai:

<div align="center">
  <img src="./images/01.jpg" width="600"/>
</div>

```plaintext
[ RAM: In-Memory Hash Map ]
| Key  | Byte Offset |
| 12   | 0           | ----> Point karta hai Disk par Offset 0 par
| 42   | 60          | ----> Point karta hai Disk par Offset 60 par
        |
        v
[ DISK: Log-Structured File ]
Byte: 0                      10                   20                   30
      |                      |                    |                    |
      1,2,,,{,",n,a,m,e,",:,",L,o,n,d,o,n,",,,",a,t,t,r,a,c,t,i,o,n,s,",:,[,",B,i,g,...
      ^
      Byte 0: Key 12 yahan se shuru ho rahi hai

Byte: 60                     70                   80                   90
      |                      |                    |                    |
      4,2,,,{,",n,a,m,e,",:,",S,a,n, ,F,r,a,n,c,i,s,c,o,",...
      ^
      Byte 60: Key 42 ki nayi entry yahan se shuru hoti hai

```

* **Data Insertion Flow:** Jab bhi aap koi naya key-value pair file mein write karte hain, wo file ke aakhir mein append ho jata hai. Sath hi sath, aap RAM mein maujood Hash Map ko update karte hain ke is key ka naya data ab is naye byte offset par maujood hai.
* **Data Retrieval Flow:** Jab read request aati hai, toh database disk ko blind scan nahi karta. Wo pehle RAM mein hash map se us key ka offset dekhta hai (jaise key `42` ka offset `60` hai). Phir CPU disk ko direct **seek** command bhejta hai ke direct offset `60` par jao aur data read kar lo.
* **Filesystem Cache Advantage:** Agar file ka wo hissa pehle se operating system ke filesystem cache (RAM) mein para hua hai, toh disk par jana hi nahi parta; read operation zero disk I/O ke sath perform ho jata hai.

#### Is Approach Ke Sakht Masail (Limitations)

Halankeh yeh reads ko bohot tez bana deta hai, magar real-world distributed systems mein iske chaar baray nuksaan hain:

* **Disk Space Exhaustion (Data Leak):** Kyunki hum purane records ko kabhi overwrite nahi karte, isliye ek hi key ki bar-bar updates file ka size barhati chali jayengi aur disk space khatam ho jayegi.
* **Slow Crash Recovery:** Hash map sirf RAM mein hota hai. Agar database crash ya restart ho jaye, toh pura index urr jata hai. Isko dobara build karne ke liye poori log file ko shuru se aakhir tak scan karna parega, jo ke baray data sets par ghanto le sakta hai.
* **RAM Constraints (Memory Bottleneck):** Saari keys ka RAM mein fit hona zaroori hai. Agar aapke paas billions of keys hain, toh RAM kam par jayegi. Isko disk par rakhna bohot bura idea hai kyunki disk par hash table rakhne se random access I/O bohot barh jata hai, hash collisions ko handle karna mushkil hota hai, aur jab hash table full ho jaye toh usay grow karna intehai expensive hota hai.
* **No Range Queries Support:** Aap is design mein range queries nahi kar sakte (jaise key `10000` se `19999` tak ka data nikalna). Aapko har ek key ko alag se hash map mein individually search karna parega, kyunki data randomly phelá hua hai.

---

## The SSTable file format

Inhi masail ko hal karne ke liye hum hash tables ke bajaye ek behtar structure use karte hain jise **SSTable (Sorted Strings Table)** kehte hain. SSTable ki do bunyadi shartein hoti hain:

1. File ke andar jitne bhi key-value pairs hain, wo **sorted by key** (keys ke mutabaq tarteeb diye gaye) hone chahiye.
2. Poori segment file mein har ek key sirf **ek hi dafa** aa sakti hai (duplications ko pehle hi khatam kar diya jata hai).

### Figure 4-2 Ka Gehrayi Se Mutaala (Sparse Index aur Compressed Blocks)

Figure 4-2 is format ke internal architecture ko samjhati hai ke kaise hum saari keys ko RAM mein rakhe bina behtareen reads hasil karte hain:

<div align="center">
  <img src="./images/02.jpg" width="600"/>
</div>

```plaintext
[ RAM: Sparse Index ]
| Key      | Byte Offset |
| hammock  | 100491      | ----+
| handbag  | 102134      | ----|----+
| handsome | 104667      | ----|----|----+
                               |    |    |
                               v    v    v
[ DISK: Sorted SSTable File ]
Offset 102134: [ handbag:... | handcuffs:... | handful:... ]  <--- (Compressible Block)
Offset 104667: [ handsome:... | handwaving:... | handwriting:... ]

```

* **The Concept of Sparse Index:** Hamein RAM mein har ek key ka offset rakhne ki zaroorat nahi hai. Hum data ko kuch kilobytes (e.g., 4 KB) ke chunks ya blocks mein divide karte hain aur un blocks ko compress kar dete hain. RAM ke andar hum sirf har block ki **pehli key** (boundary key) ka record rakhte hain. Isay **Sparse Index** kehte hain.
* **Search Execution Flow:** Farz karein aapko key `handiwork` dhoondni hai. Aap Sparse Index mein dekhenge ke `handiwork` alphabetically `handbag` aur `handsome` ke darmiyan aati hai. System direct disk par offset `102134` (handbag) par jump karega aur wahan se agla block sequentially scan karna shuru karega jab tak usay `handiwork` mil na jaye ya block khatam na ho jaye.
* **Compression and I/O Efficiency:** Kyunki data sorted hota hai, isliye sath wali keys milti julti hoti hain (jaise handbag, handcuffs). Aise data ko compress karna bohot efficient hota hai. Yeh hardware level par disk space bhi bachata hai aur disk se data read karte waqt I/O bandwidth ka load bhi kam karta hai, halankeh is mein thoda CPU utilization barh jata hai block ko decompress karne ke liye.

---

## Constructing and merging SSTables

SSTable read karne ke liye toh perfect hai, lekin write karte waqt masla yeh hota hai ke hum file ke darmiyan mein naya data insert nahi kar sakte, kyunki sorting kharab ho jayegi. Agar har write par poori file dobara likhi jaye, toh system baith jayega. Iska hal ek hybrid approach hai jise **LSM-Tree (Log-Structured Merge-tree)** algorithm kehte hain.

### System Architecture aur Step-by-Step Data Flow

```plaintext
[ Client Write ]
       |
       +-------> [ 1. Append to Disk WAL (Crash Recovery Log) ]
       |
       +-------> [ 2. Insert into RAM Memtable (Balanced Sorted Tree) ]
                      |
              (When Memtable fills up, e.g., 4MB)
                      |
                      v
         [ 3. Flush to Disk as Sorted SSTable Segment ] (Immutable)
                      |
                      v
         [ 4. Background Compaction / Mergesort Process ]

```

#### 1. Write Path (Memtable aur WAL)

* Jab bhi koi naye data ki write request aati hai, system usay direct disk par sorted file mein nahi likhta. Wo sabse pehle usay RAM mein ek sorted data structure mein dalta hai jise **Memtable** kehte hain (yeh internally Red-Black tree, AVL tree, ya Skip List hoti hai jo insert hote hi data ko automatic sort rakhti hai).
* **Durability (WAL):** Agar memory crash ho jaye toh data zaya ho sakta hai. Isliye parallel mein ek immediate write disk par ek append-only **Write-Ahead Log (WAL)** file mein bhi save ki jati hai. Is WAL ka maqsad sirf crash recovery hai, yeh sorted nahi hoti aur reads ke liye use nahi hoti.

#### 2. Flushing Memtable to SSTable

* Jab RAM mein Memtable ka size ek certain threshold (jaise ke 4 MB) se barh jata hai, toh database is memtable ko disk par ek **SSTable Segment file** ke shakal mein sequentially write (flush) kar deta hai.
* Kyunki memtable pehle se sorted thi, disk par file likhte waqt koi extra sorting cost nahi aati. Yeh segment ek dafa disk par likh diya jaye toh yeh **Immutable** (un-changeable) ban jata hai. Iske baad purani WAL file ko delete kar diya jata hai kyunki data disk par safe ho chuka hai.

#### 3. Read Path Flow

* Jab koi client kisi key ko read karna chahta hai:
1. System sabse pehle current **Memtable (RAM)** mein dhoondta hai.
2. Agar wahan nahi milti, toh sabse naye on-disk **SSTable Segment** mein dhoondta hai.
3. Agar wahan bhi nahi milti, toh sequential order mein usse purane segments (Segment 2, Segment 3) check karta jata hai jab tak key mil na jaye ya saare segments khatam na ho jayein.



#### 4. Background Compaction (Merging)

Kyunki disk par bohot saari segment files ban jati hain, reads ko fast rakhne aur space reclaim karne ke liye background mein ek thread chalta hai jo **Compaction and Merging** perform karta hai.

### Figure 4-3 Ka Gehrayi Se Mutaala (SSTable Segments Merging Process)

Figure 4-3 dikhati hai ke kaise **Mergesort** algorithm ke zariye background mein multiple segments ko ek single segment mein transform kiya jata hai:

<div align="center">
  <img src="./images/03.jpg" width="600"/>
</div>

```plaintext
[ Segment 1 ] ---> [ handbag:... | handful:... | handicap:... ]
[ Segment 2 ] ---> [ handcuffs:... | handful:... | handiwork:... ]   ---+
[ Segment 3 ] ---> [ handful:... | handicap:... | handlebars:... ]      |
                                                                        v
                                                         ( + Compaction / Mergesort )
                                                                        |
                                                                        v
[ Merged Segment 1,2,3 ] ---> [ handbag:... | handcuffs:... | handful (Latest Version):... ]

```

* **The Core Logic:** Merging process saare files ke pointers ko side-by-side read karta hai. Yeh sabse choti key ko compare karta hai aur naye file mein copy karta jata hai.
* **Handling Duplicates & Tombstones:** Agar ek hi key multiple segments mein maujood hai (jaise `handful`), toh compaction process sirf **sabse naye segment** wali value ko rakhta hai aur baaki purani entries ko drop kar deta hai.
* **Deletions & Tombstones:** Jab user kisi data ko delete karta hai, toh system usay foran disk se erase nahi kar sakta (kyunki files immutable hain). Iske bajaye ek special deletion record append kiya jata hai jise **Tombstone** kehte hain. Compaction ke dauran jab yeh tombstone purane segments se guzarta hai, toh wo us key ke saare purane records ko hamesha ke liye delete kar deta hai.

---

## Bloom filters

LSM-Tree storage engines mein ek bara masla yeh aata hai ke agar koi key database mein maujood hi na ho, ya bohot purani ho, toh system ko saare disk segments aur unke indexes ko check karna parta hai, jo ke bohot zyada disk I/O operations consume karta hai aur reads ko slow kar deta hai. Is bottleneck ko khatam karne ke liye hum **Bloom Filter** use karte hain. Bloom Filter ek probabilistic (imkani) data structure hai jo bohot kam memory mein yeh bata deta hai ke **"Kya yeh key is segment mein maujood hai ya nahi?"**

### Figure 4-4 Ka Gehrayi Se Mutaala (Probabilistic Key Checking)

Aap Figure 4-4 ke bit array aur hashing flow ko is tarah samajh sakte hain:

<div align="center">
  <img src="./images/04.jpg" width="600"/>
</div>

```plaintext
[ Bits Array in Bloom Filter (Size: 0 to 15) ]
Index: 0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
Bits: [0] [0] [1] [0] [1] [1] [0] [0] [0] [1] [0] [0] [0] [0] [0] [1]
               ^       ^   ^               ^                       ^
               |       |   |               |                       |
               +-------+---|---------------+-----------------------+
                           |               |
             Keys Existing |               | Querying Key: "handheld"
             in SSTable:   |               | Hash("handheld") -> (6, 11, 2)
                           |               |
    - Hash("handbag") ----> (2, 9, 4)      | - Bit 6  is 0 (No Match!)
    - Hash("handoff") ----> (9, 15, 5)     | - Bit 11 is 0 (No Match!)
                                           | - Bit 2  is 1
                                           v
                             [ RESULT: "handheld" DEFINITELY NOT PRESENT ]

```

* **How It Is Built:** SSTable segment likhte waqt, us segment ki har key ko multiple cryptographic/non-cryptographic hash functions se guzara jata hai. Jo numbers generate hote hain, bit-array ke un indexes par `0` ko `1` kar diya jata hai. Jaise image mein `handbag` ne bit 2, 9, aur 4 ko `1` kiya, aur `handoff` ne bit 9, 15, aur 5 ko `1` kiya.
* **The Mathematical Evaluation (The Query):** Jab hum `handheld` ko search karte hain, toh iske hash values (6, 11, 2) aati hain. Hum dekhte hain ke bit array mein index 6 aur 11 par `0` para hua hai.
* **The Absolute Rule:** * **If any bit is 0:** Agar ek bhi bit `0` mil jaye, toh iska matlab hai ke yeh key zindagi mein kabhi database mein aayi hi nahi. System bina disk read kiye foran client ke liye `404 Not Found` ya null return kar deta hai. Is tarah faltu disk seek bach jata hai.
* **If all bits are 1:** Agar saari bits `1` milein, toh iska matlab hai ke key shayad segment mein maujood hai. Lekin yahan **False Positive** ka chance hota hai (ho sakta hai mukhtalif keys ke hashes ne mil kar un bits ko coincidentally `1` kiya ho). Aise case mein system disk check karega. Agar data na mila toh koi baat nahi, bas thoda CPU cycle zaya hua, data par koi asar nahi parta.


* **Space Allocation Rule:** Rule of thumb ke mutabaq, agar aap har ek key ke liye Bloom filter mein **10 bits** allocate karein, toh false positive ka rate sirf **1%** reh jata hai. Har extra 5 bits barhane se false-positive ka imkan 10 guna mazeed kam ho jata hai.

---

## Compaction strategies

LSM Engines mein background compaction kis tarah aur kab chalni chahiye, iski do baray maqbool strategies hain jo mukhtalif workloads ke trade-offs ko handle karti hain:

### 1. Size-tiered compaction

* **Mechanism:** Is strategy mein database chote aur naye SSTables ke group hone ka intezar karta hai. Jab aik hi size ke multiple (e.g., 4 segments of 256 MB) files jama ho jati hain, toh unko aapas mein merge kar ke ek bari file (e.g., ~898 MB) bana di jati hai.
* **Workload Optimization:** Yeh **Write-Heavy** workloads ke liye behtareen hai. Kyunki is mein data ko baar baar rewrite nahi karna parta, sequential merges kam hote hain, jis se write throughput bohot high milti hai. Magar iska nuksaan yeh hai ke merging ke dauran bohot zyada temporary disk space chahiye hoti hai aur reads thode slow ho sakte hain kyunki purani files bohot bari ho jati hain.

### 2. Leveled compaction

* **Mechanism:** Is mein database SSTables ke sizes ko fixed (e.g., 16 MB) rakhta hai aur unhein mukhtalif levels (L0, L1, L2...) mein divide karta hai. Level 0 ke ilawa baaki saare levels key-range partitioned hote hain (jaise L1 ki pehli file mein sirf `a-m` keys hain aur doosri mein `n-z`). Jab kisi level ka size limit cross hota hai, toh wahan se kuch files utha kar unke key-range ke mutabaq agle level (i + 1) ke sath merge kar diya jata hai.
* **Workload Optimization:** Yeh **Read-Heavy** workloads ke liye sabse best hai. Kyunki data clean tarah se partitioned hota hai, system ko pata hota hai ke kis file mein jana hai, jis se kam se kam SSTables scan karni parti hain. Yeh disk space bhi kam consume karti hai magar is mein writes thode slow ho sakte hain kyunki system ko levels maintain karne ke liye zyada background writes karne parte hain (**Write Amplification**).

---

## Embedded Storage Engines

Haqeeqi dunya mein har database network service (jaise MySQL, PostgreSQL) ke tor par kaam nahi karti. Kuch databases **Embedded Databases** hoti hain.

* **Architectural Flow:** Yeh koi network socket expose nahi kartiin aur na hi inka alag server processing unit hota hai. Yeh direct aapke application code ke sath ek **library (.dll ya .so file)** ke tor par link ho jati hain aur aapke main application process ke andar hi memory aur threads share karti hain.

```plaintext
[ Client App Process (Your Code) ]
       |
       +---> (Direct Function Call) ---> [ Embedded Storage Engine Library (RocksDB/SQLite) ]
                                                               |
                                                   (Direct Local OS File I/O)
                                                               |
                                                               v
                                                    [ Local Disk Storage ]

```

* **Real-World Examples:** **RocksDB** (LSM-tree based), **SQLite** (B-Tree based transactional), **LMDB**, **DuckDB** (Analytical), aur **KùzuDB** (Graph-based).
* **Use Cases:** Yeh mobile applications mein local user data save karne ke liye use hoti hain. Backend par yeh tab behtareen kaam karti hain jab data ek single machine par fit ho sake ya multitenant architectures mein jahan har ek customer (tenant) ka data completely independent ho aur aap har tenant ke liye alag process memory mein alag isolated embedded instance chala sakein.

---

## Mockup System Design Scenario (Interview Style)

### Interview Context & Problem Statement

> **Interviewer:** "Aap ek Global Financial Transaction Fraud Analytics platform design kar rahe hain. Har second dunya bhar se 500,000 credit card transaction status check aur fraud evaluations ho rahi hain. system ko transactions ko foran store karna hai ($500k\text{ writes/sec}$) aur sath hi sath real-time checks karne hain un accounts par jo system mein exist hi nahi karte taake fake accounts ko block kiya ja sake. Read requests un account IDs par aati hain jo aksar system mein hoti hi nahi hain. Aap reads par disk latency zero kaise karenge aur storage kaise structure karenge?"

### Solution Strategy & Conceptual Flow

Hum yahan DDIA ke concepts (LSM-Tree + Memtable + Bloom Filters + Leveled Compaction) ka solid combination use karenge:

1. **Write Optimisation:** Hum incoming transactions ko direct **Memtable** aur disk **WAL** par sequentially append karenge. Is se $500\text{k writes/sec}$ asani se handle ho jayenge kyunki random disk I/O zero ho chuka hai.
2. **Eliminating Non-Existent Account Reads:** Kyunki bohot si requests un account IDs ki aati hain jo system mein hain hi nahi, agar hum disk scan karenge toh database down ho jayega. Isliye hum har SSTable segment ke sath memory mein ek high-fidelity **Bloom Filter** attach karenge.
3. **The Read Filter Gate:** Jab koi read request aayegi, pehle RAM mein Bloom filter check hoga. Agar filter ne kaha `0`, iska matlab account non-existent hai, hum foran bina disk par jaye client ko fraud/invalid ka trigger bhej denge. Agar filter `1` kahega, tabhi hum **Leveled Compaction** waale sorted blocks mein binary search/sparse lookup chalayenge.

### Architectural Flow Diagram

```plaintext
[ 500k writes/sec Requests ] ----------------------------+
                                                         |
                                                         v
                                           [ In-Memory Memtable (RAM) ]
                                                         |
                                               (Size Threshold Hit)
                                                         |
                                                         v
                                       [ Flush to Disk: Leveled SSTables ]
                                       +---------------------------------+
                                       | L0: [Recent Data]               |
                                       | L1: [a-k.sst]  [l-z.sst]        | <---+
                                       +---------------------------------+     |
                                                                               |
                                                                               |
[ High Rate Fraud Read Queries ]                                               |
  (e.g., Query Account: "acc_999")                                             |
                 |                                                             |
                 v                                                             |
   [ In-Memory Bloom Filter Check ]                                            |
                 |                                                             |
                 +---> (If Bit is 0) ---> [ FORAN RETURN: "Account Not Found" ]|
                 |                         (Zero Disk I/O Overhead!)           |
                 |                                                             |
                 +---> (If Bit is 1) ---> [ Seek Sparse Index ] ---------------+
                                          [ Read Target Compressed Disk Block ]

```

### Trade-off Evaluation in this Design

* **Pros:** Peak write traffic par database crash nahi hoga kyunki flush sequential hai. Non-existent account lookups par disk utilization bilkul zero ho jayegi kyunki Bloom filter unhein RAM level par hi block kar dega.
* **Cons:** Hamein Bloom filters aur Sparse index ke liye achi khasi RAM capital allocate karni paregi. Agar false positive rate ko mazeed kam karna hai, toh memory consumption thodi aur barhegi.

---