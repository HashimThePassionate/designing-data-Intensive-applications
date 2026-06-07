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

## B-Trees

LSM-tree (Log-structured storage) ke ilawa, databases mein data store aur read karne ka sabse maqbool aur purana tareeqan **B-Tree** hai. 1970 mein introduce hone wale B-trees aaj bhi dunya ke taqreeban tamaam Relational Databases (jaise MySQL, PostgreSQL) aur kai Non-relational databases ke default indexing engines hain.

### Design Philosophy: B-Trees vs LSM-Trees

Agache B-trees bhi SSTables ki tarah data ko **sorted by key** rakhte hain taake range queries aur fast lookups ho sakein, lekin dono ki andarooni design philosophy bilkul mukhtalif hai:

* **LSM-Trees:** Yeh database ko mukhtalif size ke segments (kuch megabytes) mein torta hai. Yeh segments hamesha **Immutable** hote hain (sirf naya data append hota hai, purana change nahi hota).
* **B-Trees:** Yeh database ko **fixed-size blocks ya pages** mein torta hai. Har page ka size fixed hota hai (traditionally 4 KiB, jabki PostgreSQL mein 8 KiB aur MySQL mein 16 KiB default hota hai). Yeh disk par ja kar purane data ke upar hi naya data overwrite karte hain, jise **In-place update** kehte hain.

Har page ka ek unique **Page Number** hota hai. Ek page doosre page ka reference (pointer) is page number ke zariye rakhta hai. Agar saare pages ek hi file mein hain, toh system `Page Number $\times$ Page Size` kar ke disk par us page ka exact byte offset nikal leta hai.

---

### Figure 4-5 Ka Gehrayi Se Mutaala (B-Tree Index Lookup)

Aap jo Figure 4-5 dekh rahe hain, wo yeh samjhati hai ke jab humein koi key dhoondni ho (jaise `user_id = 251`), toh B-tree ka pointer-based look up architecture kaise kaam karta hai:

<div align="center">
  <img src="./images/05.jpg" width="600"/>
</div>

```plaintext
[ Root Page ] -> Contains Keys: [ 100 | 200 | 300 | 400 | 500 ]
                       |
             (251 lies between 200 and 300)
                       |
                       v
[ Child Page ] -> Contains Keys: [ 210 | 230 | 250 | 270 | 290 ]
                                         |
                               (251 lies between 250 and 270)
                                         |
                                         v
[ Leaf Page ]  -> Contains Actual Data: [ 250:val | 251:val | 252:val ]

```

1. **The Root Page:** Tree ke sabse upar ek **Root Page** hota hai. Har lookup yahin se shuru hota hai. Is page mein keys aur unke sath child pages ke references (`ref`) hote hain. Har child page ek khas continuous key range ke liye zimmedar hota hai.
2. **The Traversal Path:** Hamein `251` search karna hai. Root page par hum dekhte hain ke 251 kis range mein aata hai. Yeh $200 \le \text{key} < 300$ ke darmiyan hai, toh hum us range ke niche wale specific `ref` pointer ko follow karte hain.
3. **The Leaf Page:** Hum agle child page par pohanchte hain jahan range mazeed choti sub-ranges mein divide hoti hai. Wahan hum dekhte hain ke 251 ab $250 \le \text{key} < 270$ ke darmiyan hai. Is pointer ko follow karne par hum aakhri page par pohanchte hain jise **Leaf Page** kehte hain. Leaf page par actual keys aur unki values (ya wo address jahan value pari hai) inline maujood hoti hain.
4. **Branching Factor:** Ek single page ke andar kitne child references aa sakte hain, usay **Branching Factor** kehte hain (Figure 4-5 mein yeh factor 6 hai). Real-world mein yeh factor keys aur pointers ke size par depend karta hai, magar aam tor par yeh kai sainkro (hundreds) mein hota hai.

---

### Figure 4-6 Ka Gehrayi Se Mutaala (Page Splitting aur Tree Growth)

B-tree mein jab data update karna ho, toh system leaf page dhoond kar naye data ko purane data ke upar overwrite kar deta hai. Magar jab naya data **Insert** karna ho, toh system ko page splitting ka sahara lena parta hai, jise Figure 4-6 mein buraheen ke sath dikhaya gaya hai:

<div align="center">
  <img src="./images/06.jpg" width="600"/>
</div>

```plaintext
[ BEFORE INSERTION ]
Parent Page Pointer -> [ 310 | 333 | 345 ]
                         |
                         v
Leaf Page (Full)    -> [ 333:val | 335:val | 337:val | 340:val | 342:val ] (No Space!)

-----------------------------------------------------------------------------------------

[ AFTER ADDING KEY 334 (Page Split) ]
Parent Page Updated -> [ 310 | 333 | 337 | 345 ]  <-- Boundary key 337 pushed up!
                               |     |
            +------------------+     +------------------+
            v                                           v
New Leaf Page 1 -> [ 333:val | 334:val | 335:val ]   New Leaf Page 2 -> [ 337:val | 340:val | 342:val ]

```

* **The Problem:** Hum ne key `334` insert karni hai, magar range `333–345` wala leaf page pehle hi full ho chuka hai (us mein mazeed space nahi hai).
* **The Action (Page Split):** System is full page ko do aadhe-aadhe (half-full) pages mein tor deta hai. Pehla page `333–337` range ka banta hai (jis mein naya key 334 fit ho jata hai) aur doosra page `337–345` range ka ban jata hai.
* **Cascading Effect to Parent:** Kyunki ab ek ke bajaye do pages ban chuke hain, isliye inka jo **Boundary Key** hai (jo ke `337` hai), usay upar wale **Parent Page** mein push kar diya jata hai taake parent page ab dono naye child pages ko target kar sake. Agar parent page bhi pehle se full hota, toh wo bhi split ho jata. Yeh split ka silsila upar root tak ja sakta hai. Agar root page split ho jaye, toh tree ki depth (levels) ek darja barh jati hai.
* **Balanced Tree Guarantee:** Is split algorithm ki wajah se B-tree hamesha **Balanced** rehta hai. $n$ keys wale B-tree ki depth hamesha **$O(\log n)$** hoti hai. Zyadatar production databases ka data sirf 3 se 4 levels deep B-tree mein fit ho jata hai (4 KiB page size aur 500 branching factor ke sath ek 4-level tree taqreeban **250 TB** data store kar sakta hai), yani sirf 4 disk reads mein aapko aapka record mil jata hai.

---

## Making B-trees reliable

B-tree ka bunyadi rule yeh hai ke yeh disk par maujood pages ko direct overwrite karta hai. Yeh approach LSM-trees se bilkul ulti hai (jo sirf naye files append karte hain aur purani files ko touch nahi karte). Direct disk overwrite karne mein bohot baray hardware risks hote hain.

### Hardware Risks & Solutions

* **The Danger of Corruption:** Agar page split ke dauran (jab database multiple pages disk par write kar raha ho) suddenly light chali jaye ya system crash ho jaye, toh kuch pages write ho jayenge aur kuch reh jayenge. Is se tree ka structure kharab ho jayega aur kuch pages **Orphan Pages** ban jayenge (yani unka koi parent pointer nahi rahega).
* **Torn Pages:** Agar hardware level par disk ek pure 4 KB ya 16 KB ke page ko ek hi jhatke mein atomicity ke sath write na kar paye, toh page ka aadha hissa naya aur aadha purana reh jata hai, jise **Torn Page** kehte hain.

#### 1. The Write-Ahead Log (WAL) / Journaling

In sab khatraat se bachne ke liye har B-tree implementation disk par ek extra data structure maintain karti hai jise **Write-Ahead Log (WAL)** ya *Journal* kehte hain.

* **The Protocol:** Jab bhi B-tree mein koi modification (insert/update/delete) karni ho, database tree ke pages ko hath lagane se pehle us event ko is append-only WAL file mein write karta hai.
* **Crash Recovery:** Agar system mid-page write par crash ho jaye, toh restart hone par database is WAL file ko dubara parhta (replay karta) hai aur B-tree ko ek consistent aur sahi state mein wapas le aata hai.

#### 2. Memory Buffering aur Durability

Performance ko tez karne ke liye databases har write ko foran disk page par overwrite nahi karte, balkay B-tree pages ko RAM mein buffer (cache) kar lete hain. WAL file is baat ki guarantee hoti hai ke data safe hai. Jab tak naya transaction WAL mein write ho kar OS ki `fsync` system call ke zariye disk par physically flush ho jata hai, tab tak data durable mana jata hai, bhale hi main B-tree pages abhi RAM mein hi kyun na pare hon.

---

## Using B-tree variants

B-trees ke itne saal purane safar mein developer community ne iske kai advanced variants banaye hain taake mukhtalif use-cases ko handle kiya ja sake:

* **Copy-on-Write (COW) Scheme:** Kuch databases (jaise **LMDB**) in-place overwrite aur WAL ka jhanjhat hi khatam kar deti hain. Jab kisi page ko modify karna ho, toh use purane address par overwrite karne ke bajaye disk par ek **bilkul naye location** par likha jata hai. Phir uske parent pages ka ek naya version generate kiya jata hai jo is naye location ko point karta hai. Yeh scheme concurrency control aur Snapshot Isolation ke liye intehai mufeed hai.
* **Key Abbreviation (Prefix Compression):** Tree ke andarooni (interior) pages mein poori poori text keys store karne ke bajaye unke sirf shuruati lafz (prefixes) store kiye jate hain jo ranges ki boundary batane ke liye kafi hon. Is se page mein space bachti hai, branching factor barhta hai, aur tree ki depth kam hoti hai.
* **Sequential Leaf Layout:** Koshish ki jati hai ke disk par leaf pages ek sequential tarteeb mein hon taake range scans karte waqt disk heads ko zyada move na karna pare. Magar tree ke barhne sath sath is order ko maintain rakhna bohot mushkil ho jata hai.
* **Sibling Pointers (B+Tree Links):** Har leaf page apne barabar wale left aur right leaf pages ka direct reference (pointer) apne paas rakhta hai. Iska faida yeh hota hai ke agar aapko sorted order mein scan chalana ho, toh aapko baar baar parent nodes par wapas ja kar niche aane ki zaroorat nahi parti; aap direct leaves ke darmiyan traverse kar sakte hain.

---

## Mockup System Design Scenario (Interview Style)

### Interview Context & Problem Statement

> **Interviewer:** "Hamein ek Core Banking Ledger System design karna hai jahan users ke account balances store honge. Requirements yeh hain ke point lookups (jaise kisi user ka exact balance check karna) hamesha instant ho, memory overhead kam se kam ho, aur critical requirement yeh hai ke agar server ka power failure (crash) ho jaye, toh balance corrupt nahi hona chahiye aur system hamesha consistent state mein rahay. Aap LSM-tree use karenge ya B-Tree?"

### Solution Strategy & Conceptual Flow

Is scenario ke liye hum **B-Tree with WAL** select karenge. Reasons aur tradeoffs yeh hain:

1. **Why Not LSM-Tree?** LSM-trees mein ek key ke multiple versions alag alag segments mein hotae hain, jis ki wajah se point lookups par read amplification hoti hai (multiple segments scan karne par sakte hain). Ledger system mein point lookups fast aur predictable ($O(\log n)$) hone chahiye.
2. **In-Place Update Advantage:** Ledger mein rows fixed hoti hain (Account ID $\rightarrow$ Balance). B-tree unhi fixed pages par data overwrite karega, jis se space predictability bani rehti hai aur background compaction ka wait nahi karna parta.
3. **Crash Resilience Path:** Jab koi transaction (e.g., Transfer $100) aayegi, hum pehle usay **WAL (Write-Ahead Log)** mein append karenge aur `fsync` chalayenge. Uske baad RAM Buffer mein B-tree page update hoga. Agar disk par actual page update ke dauran light chali bhi jaye, toh restart par WAL se balance recover ho jayega.

### Architectural Flow Diagram

```plaintext
[ User Transaction Request ] ---> (Deduct $100 from Acc: 501)
                                         |
                                         v
                     [ 1. Append to Append-Only Disk WAL ] 
                                         |  (Guarantees Durability via fsync)
                                         v
                     [ 2. Update Page in RAM Buffer Cache ]
                                         |
                       (Periodic Background Page Flush)
                                         |
                                         v
                     [ 3. In-Place Overwrite on Disk File ]
                     +---------------------------------------+
                     | Root Page -> Child Page -> Leaf Page  |
                     | [Acc 501: Old Bal] -> [Acc 501: $900] | (Overwritten!)
                     +---------------------------------------+
                                         |
                (If Crash Happens Mid-Flush: Replay WAL to Recover)

```

### Trade-off Evaluation in this Design

* **Pros:** Point lookups intehai tez hain kyunki tree balanced hai aur max 3-4 steps mein target node mil jata hai. Multi-versioning na ہونے ki wajah se disk space optimal rehti hai. Strong consistency aur data integrity zero corruption ke sath milti hai.
* **Cons:** Writes thodi slow ho sakti hain LSM-tree ke muqable mein, kyunki page split ke dauran multiple disk pages par random overwrites karne parte hain aur WAL par double-writing ka overhead hota hai.

---

### Comparing B-Trees and LSM-Trees

Ek aam asool (rule of thumb) ke mutabaq, **LSM-trees** un applications ke liye behtareen hain jahan **writes** bohot zyada hoti hain (write-heavy workloads), jabki **B-trees** un workloads ke liye tez hain jahan **reads** ka kaam zyada hota hai (read-heavy workloads). Lekin real-world benchmarks hamesha workload ki barikiyaon par depend karte hain, isliye sahi muqabla karne ke liye aapko apni application ke specific traffic pattern par test karna parta hai.

Yeh koi strict strict either/or choice nahi hai; aam tor par modern storage engines dono approaches ki khubiyon ko aapas mein blend (mix) karte hain—jaise multiple B-trees rakhna aur unhein LSM-style mein background mein merge karna.

---

### Read performance

B-tree aur LSM-tree dono ka read mechanism hardware aur computational level par bilkul alag behave karta hai.

* **B-Tree Read Mechanics:** B-tree mein kisi bhi single key (point lookup) ko dhoondna bohot predictable hota hai. System tree ke har level par sirf **ek page** read karta hai. Kyunki B-tree ki depth (levels) hamesha bohot choti hoti hai (aam tor par 3 se 4 levels), isliye reads intehai tez aur mushtaqil (predictable performance) hote hain.
* **LSM-Tree Read Mechanics:** LSM storage engine mein ek single read request ko dhoondne ke liye background mein chalne wale mukhtalif compaction stages ke **multiple SSTables** ko check karna par sakta hai. Halankeh **Bloom filters** is disk I/O operations ko bohot had tak kam kar dete hain, phir bhi iska path B-tree ke muqable mein thoda complex hota hai.

#### Range Queries Ka Bara Trade-off:

* **B-Trees:** Range queries (jaise key `1000` se `2000` tak ka data nikalna) B-trees par intehai asaan aur fast hoti hain kyunki iska pura structure disk pages par ordered aur sorted shakal mein linked hota hai.
* **LSM-Trees:** LSM par range queries bohot expensive ho jati hain. Kyunki data alag alag segment files mein phelá hua hota hai, system ko **saare disk segments ko parallel mein scan** karna parta hai aur unke results ko merge kar ke output dena hota hai. Sabse aham baat yeh hai ke **Bloom filters range queries mein koi madad nahi kar sakte**, kyunki range ke andar aane wali har possible key ka hash nikalna practically namumkin hai.

```plaintext
[ LSM Range Query Request ] 
            |
            v
   +--------+--------+
   |                 |
   v                 v
[ Scan Segment 1 ]  [ Scan Segment 2 ]  --> Parallel Disk Scans Required
   |                 |
   +--------+--------+
            |
            v
[ Merge & Filter Results ] ---> [ Return to Client ]

```

#### High Write Traffic aur Memtable Backpressure:

Agar application se aane wali writes ka rate itna zyada ho ke RAM mein maujood **Memtable** foran full ho jaye, aur background mein disk par chalne wala compaction process us speed se segments ko merge na kar paa raha ho, toh log-structured engines mein latency spikes aate hain.

Is situation ko handle karne ke liye RocksDB jaise engines **Backpressure** apply karte hain: wo temporary tor par saari read aur write requests ko **suspend (freeze)** kar dete hain jab tak RAM ki memtable completely disk par flush na ho jaye.

---

### Sequential versus random writes

Database writes ki do bunyadi patterns hoti hain jinka disk performance par bara asar parta hai:

1. **Random Writes (B-Trees):** Agar aapka application aisi keys write kar raha hai jo pure key-space mein bikhri hui hain, toh B-tree engine ko disk par maujood un pages ko dhoond kar overwrite karna parega jo disk par kahin bhi ho sakte hain. Yeh small aur scattered writes ka pattern **Random Writes** kehlata hai.
2. **Sequential Writes (LSM-Trees):** Iske baraks, LSM-trees memory (memtable) se poora ka poora segment file ek hi dafa mein sequential tarah se disk par write karta hai. Yeh big aur continuous chunks ka write pattern **Sequential Writes** kehlata hai.

Hardware ke lehaas se disk storage hamesha sequential writes par zyada throughput deti hai ba-nisbat random writes ke. Purane spinning hard drives (HDDs) par mechanical head ki movement ki wajah se random write miliseconds leta hai jo computing dunya mein ek sadi ke barabar hai. Aaj ke dauran use hone wale modern SSDs par yeh farq mechanical head na hone ki wajah se kam zaroor hai, magar phir bhi bohot noticeable hai.

---

### Sequential Versus Random Writes on SSDs

SSDs (Solid State Drives) aur NVMe flash memory ka internal architecture mechanical limitations se pak hota hai, magar inke andar flash cells ke lehaas se ek alag physical constraint hota hai:

* **The Page vs Block Constraint:** SSD ke andar flash memory ko read ya write ek **Page (typically 4 KiB)** ki shakal mein kiya jata hai, lekin agar data ko erase (delete) karna ho, toh wo sirf ek bare **Block (typically 512 KiB)** ki shakal mein hi ho sakta hai.

#### Garbage Collection (GC) Mechanism:

Jab database kisi page ko overwrite ya delete karta hai, toh SSD controller purane page ko direct wipe nahi kar sakta. Jab ek pure 512 KiB block ko khali karna hota hai, toh SSD controller pehle us block mein maujood valid pages ko utha kar doosre naye block mein shift karta hai, aur phir purane pure block ko erase karta hai. Is mechanical step ko **Garbage Collection (GC)** kehte hain.

* **LSM Sequential Advantage on SSD:** LSM-tree sequential writes ke zariye bade chunks write karta hai, jis se poora 512 KiB ka block aksar ek hi file ke data se bhar jata hai. Jab wo file baad mein compaction ke zariye delete hoti hai, toh SSD controller poore block ko bina kisi Garbage Collection ke aik jhatke mein erase kar deta hai.
* **B-Tree Random Disadvantage on SSD:** B-tree jab randomly 4 KiB ke pages overwrite karta hai, toh SSD blocks ke andar valid aur invalid pages ka ek mix khichdi ban jata hai. Garbage collector ko valid data ko baar baar copy-paste karna parta hai, jis se disk ki write bandwidth application ke bajaye GC internal tasks mein zaya ho jati hai. Yeh cheez SSD ko jaldi ghisa deti hai aur uski life line (**SSD Wear Out**) ko fast karti hai.

---

### Write amplification

Application se aane wali ek single write request jab underlying physical disk par multiple physical I/O operations mein convert ho jaye, toh is phenomenan ko **Write Amplification** kehte hain.

$$\text{Write Amplification} = \frac{\text{Total bytes written to disk}}{\text{Logical bytes requested by application}}$$

#### Write Amplification In Both Engines:

* **LSM-Trees Path:** Data pehle durability ke liye WAL (Write-Ahead Log) mein likha jata hai $\rightarrow$ Phir memtable se disk par immutable segment banta hai $\rightarrow$ Phir jitni dafa background compaction chalegi, wo data baar baar naye segments mein rewrite hota rahega. (Agar values barri hon, toh keys aur values ko alag rakh kar is overhead ko kam kiya jata hai).
* **B-Trees Path:** Data kam se kam do dafa laazmi likha jata hai: ek dafa WAL log file mein, aur doosri dafa actual tree page par. Iske ilawa, agar page ke andar sirf 2 bytes ka data bhi change hua ho, tab bhi crash recovery ko asani se handle karne ke liye database ko poora ka poora fixed page (4KB to 16KB) disk par dobara overwrite karna parta hai.

Aam workloads ke liye **LSM-trees ka write amplification factor kam hota hai** kyunki unhein har choti write par poore bare pages overwrite nahi karne parte aur wo data ko efficiently compress kar dete hain.

> **Important Testing Fact:** Jab aap kisi khali LSM database par test chalaayenge, toh shuru mein koi compaction nahi ho rahi hogi aur write throughput asman par dikhegi. Hamesha benchmark ko lambe waqt tak chalayein taake jab database grow ho, toh compaction aur live writes ke darmiyan hone wali disk bandwidth ki sharing ka asli high-amplification rate samne aa sakay.

---

### Disk space usage

Data storage footprint aur disk space efficiency ke lehaas se dono engines ke darmiyan fragmentation aur snapshot compression ka bara farq hota hai:

* **B-Tree Fragmentation:** B-trees mein waqt ke sath sath fragmentation barh jati hai. Jab bohot saari keys delete hoti hain, toh file ke andar maujood pages khali (holes) ho jate hain. Naye additions un pages ko use toh kar sakte hain, magar un khali spaces ko operating system ko wapas return karna namumkin hota hai kyunki wo file ke darmiyan mein phase hote hain. Isliye PostgreSQL jaise databases mein **Vacuum** jaisa heavy background process chalana parta hai jo pages ko reshuffle kar ke space reclaim karta hai.
* **LSM-Tree Space Efficiency:** LSM mein fragmentation ka masla nahi hota kyunki compaction process har thodi der baad files ko zero se sequentially rewrite karta hai. Iske ilawa sorted blocks hone ki wajah se compression ratios bohot high milti hain aur disk file sizes choti rehti hain.

#### Regulations (GDPR Deletions) aur Snapshots:

* **Data Deletion Challenge:** Agar regulatory compliance (jaise data protection laws) ke mutabaq data ko disk se hamesha ke liye mitana ho, toh LSM mein masla aata hai. Deleted record ka **Tombstone** jab tak saare compaction levels par cross ho kar sabse purane level tak nahi pohanchta, data physically disk par exist karta rehta hai.
* **Instant Snapshots Advantage:** LSM ka immutable segment design live production backups aur **Database Snapshots** lene ke liye behtareen hai. Kyunki disk files kabhi change nahi hotiin, system bina data copy kiye sirf un current segment files ko freeze (pin) kar deta hai aur snapshot ready ho jata hai. B-tree mein jahan live pages par in-place overwrites ho rahe hon, wahan bina read/write block kiye consistent snapshot lena intehai mushkil aur resource-heavy task hota hai.

---

## Mockup System Design Scenario (Interview Style)

### Interview Context & Problem Statement

> **Interviewer:** "Aap ek High-Throughput Audit Logging aur Compliance Verification platform design kar rahe hain. System par 24/7 continuous append-only transaction logs aa rahe hain ($150\text{k writes/sec}$). Business requirement yeh hai ke har raat bina system ko stop kiye production ka ek exact **Point-in-Time Snapshot (Backup)** liya jaye taake compliance testing ho sake. Plus, hamein hardware cost kam rakhni hai aur cloud storage par chalne wale **SSDs ki life span (wear-out)** ko maximize karna hai. Aap niche konsa storage architecture engine use karenge?"

### Solution Strategy & Conceptual Flow

Is distributed compliance logging system ke liye hum **LSM-Tree Storage Engine** select karenge. Reasons ko hum core hardware aur software parameters ke sath is tarah evaluate karenge:

1. **Minimizing SSD Wear-Out & GC:** B-tree use karne se $150\text{k writes/sec}$ par random page writes honge jo SSD blocks ke andar valid/invalid split barha denge, jis se SSD background Garbage Collection tight ho jayegi aur drive jaldi wear out ho jayegi. LSM engine continuous memtable flush ke zariye sequential 512 KiB blocks likhega, jis se physical write amplification kam hogi aur SSD ki life barh jayegi.
2. **Zero-Copy Instant Snapshots:** Kyunki LSM ke segment files (SSTables) **Immutable** hain, jab raat ko backup snapshot trigger hoga, hamein terabytes of data physically copy karne ki zaroorat nahi hai. System sirf un files ke references ko save kar lega jo us point par exist karti thin. Live traffic nayi memtables aur naye segment files par chalti rahegi bina kisi performance drop ke.
3. **High Storage Compression:** Audit logs text heavy hote hain, sorted LSM data blocks par high compression apply hogi jo local/cloud disk space overhead ko B-tree ke muqable 40% tak ghata degi.

### Architectural Flow Diagram

```plaintext
[ 150k/sec Audit Writes ] ---> [ In-Memory Memtable ] 
                                      |
                           (Sequential Page Flushes)
                                      |
                                      v
                       [ Disk: Immutable SSTables ]
                       | - file_01.sst (Locked by Snapshot) <---+
                       | - file_02.sst (Locked by Snapshot) <---|---+
                       | - file_03.sst (New Active Segment)     |   |
                                                                |   |
[ Daily Snapshot Trigger ] -------------------------------------+   |
         |                                                          |
         v                                                          |
[ Metadata Pointer Manifest ] --------------------------------------+
(Saves state: "Snapshot_01 includes file_01 and file_02")
(Zero-copy backup completed instantly without performance degradation!)

```

### Trade-off Evaluation in this Design

* **Pros:** Writes par predictable flat lines miltiin hain, SSD lifespan high rehti hai, aur snapshots background backup pipelines ko choke kiye bina zero-copy mechanism par seconds mein ban jate hain.
* **Cons:** Agar kabhi compliance auditor aakar bohot purani dates ki range-query maang le (e.g., scan all records from March to April), toh system par read latency barh jayegi kyunki usay background ke saare cascading segments ko parallel scan karna parega.

---


### Multicolumn and Secondary Indexes

Ab tak humne jitni bhi guftagu ki hai wo primary key-value indexes ke mutaliq thi, jo relational model mein primary-key indexes ki tarah kaam karte hain. Primary key ka maqsad relational table mein kisi ek row, ya document database mein kisi ek document, ya graph database mein kisi ek vertex ko uniquely (sabse alag) identify karna hota hai. Database ke baaki records is primary key (ya ID) ke zariye us row/document/vertex ko refer karte hain, aur index ka kaam un references ko jaldi resolve (talaash) karna hota hai.

Lekin real-world applications mein **Secondary Indexes** ka istemaal bhi bohot zyada aam hai. Relational databases mein aap `CREATE INDEX` command ka istemaal kar ke ek hi table par kayi secondary indexes khare kar sakte hain. Yeh aapko primary key ke ilawa doosre columns par data search karne ki sahulat dete hain.

* **The Core Structural Difference (Non-Uniqueness):** Primary index aur secondary index mein sabse bada farq yeh hai ke secondary index mein values **unique nahi hotiin**. Yani ek hi index entry ke andar multiple rows, documents, ya vertices aa sakte hain. For example, agar aapne `user_id` par secondary index banaya hai, toh ek hi user ke multiple transaction rows ya activity records us single index entry ke niche jama ho sakte hain.

Database engines is non-uniqueness ke maslay ko do tarah se hal karte hain:

1. **Postings List Approach:** Index ki har entry ke agay matching row identifiers (IDs) ki ek poori list bana di jaye (bilkul waise hi jaise full-text search index mein *postings list* hoti hai).
2. **Key Appending Approach:** Index ki har entry ke sath uski unique primary row identifier/ID ko append (jor) diya jaye taake har entry technical level par automatic unique ban sake.

Storage engines jo in-place updates use karte hain (jaise B-trees) aur jo log-structured approach use karte hain (jaise LSM-trees), dono hi secondary indexes ko ba-asani implement karne ki salahiyat rakhte hain.

```plaintext
[ Search Request: user_id = 42 ] ---> [ Secondary Index (B-Tree/LSM) ]
                                                   |
                                       (Finds Non-Unique Matches)
                                                   |
                                      +------------+------------+
                                      |                         |
                                      v                         v
                           [ Row ID: 101 (Match 1) ]  [ Row ID: 504 (Match 2) ]
                                      |                         |
                                      +------------+------------+
                                                   |
                                                   v
                                [ Fetch Actual Rows from Primary Storage ]

```

---

### Storing Values Within the Index

Index ke andar jo `key` hoti hai, queries uske mutabaq search perform kartiin hain. Lekin us key ke agay `value` ki shakal mein kya data store hoga, yeh index ke design aur architecture par depend karta hai. Iski teen barri iqsaam (types) hain:

#### 1. Clustered Index

* **System Behavior:** Agar database actual data row, document, ya vertex ko direct index structure ke andar hi inline store kar le, toh usay **Clustered Index** kehte hain.
* **Real-World Implementation:** MySQL ke **InnoDB** storage engine mein table ki primary key hamesha ek clustered index hoti hai. SQL Server mein bhi aap har table par ek clustered index specify kar sakte hain.
* **Data Flow:** Iska faida yeh hota hai ke jab search query key dhoond leti hai, toh data wahan pehle se maujood hota hai. Alag se kisi doosri file ya storage block mein lookup nahi karna parta.

#### 2. Heap File Approach (Non-Clustered Reference)

* **System Behavior:** Is approach mein index ke andar actual data nahi hota, balkay actual data ka ek **Reference (Pointer)** hota hai. Yeh reference ya toh us row ki primary key hoti hai (jaise InnoDB apne secondary indexes ke liye karta hai) ya phir disk par us data ki exact physical location ka direct pointer hota hai.
* **The Heap File:** Jis jagah table ki saari rows bina kisi khaas order ke random ya append-only shakal mein store ki jati hain, usay **Heap File** kehte hain.
* **Real-World Implementation:** **PostgreSQL** actual data store karne ke liye isi heap file approach ko use karta hai.
* **Handling Updates in Heap Files:** Agar aap kisi record ki value update karte hain aur uski key change nahi hoti, toh heap file approach mein us row ko uski purani jagah par hi **in-place overwrite** kiya ja sakta hai—magar shart yeh hai ke naye data ka byte-size purane data se bada na ho. Agar naya data bada hai, toh system usay heap file mein kisi naye location par move karega jahan space ho. Aise case mein database ko do kaam karne par sakte hain:
* Ya toh dunya ke saare secondary indexes ko update kar ke naye heap address par point karwaya jaye.
* Ya phir purani heap location par ek **Forwarding Pointer** chor diya jaye jo aane wali requests ku naye address par redirect kar de.



#### 3. Covering Index (Index with Included Columns)

* **System Behavior:** Yeh clustered index aur heap file ka ek darmiyani rasta (middle ground) hai. Is mein index structure ke andar key ke sath table ke kuch extra selective columns ko bhi **Include** kar diya jata hai, bhale hi poori full row heap file mein hi kyun na pari ho.
* **Query Covering Fact:** Agar aapki query sirf unhi columns ka data maang rahi hai jo index ke andar pehle se maujood hain, toh database heap file ko touch kiye bina direct index node se hi result return kar deta hai. Is condition mein hum kehte hain ke **Index ne query ko cover kar liya**. Yeh reads ko extreme fast banata hai, magar data duplication ki wajah se disk space zyada leta hai aur writes par overhead barhata hai.

```plaintext
Approach 1: Clustered Index
[ Index Node: Key=12 ] ---> [ Full Row Data Inline: (Name: London, Attractions: [...]) ]

Approach 2: Heap File Reference
[ Index Node: Key=12 ] ---> [ Direct Reference/Pointer ] ---> [ Disk Heap File (Unordered Data) ]

Approach 3: Covering Index
[ Index Node: Key=12 | Included: Name="London" ] ---> (If query only asks for Name, return from Index!)

```

---

### Keeping Everything in Memory

Ab tak is chapter mein humne jitne bhi structures aur indexes parhay hain, wo sab disk (HDDs/SSDs) ki physical limitations aur unki slow speed ko counter karne ke liye design kiye gaye the. Hum disk ki is complex management ko sirf do baray faidon ki wajah se bardasht karte hain: pehla yeh ke wo **Durable** hain (power off hone par data delete nahi hota), aur doosra yeh ke RAM ke muqable unka cost-per-gigabyte bohot kam hota hai.

Lekin ab jaise jaise RAM sasti hoti ja rahi hai, yeh cost ka argument kamzor par raha hai. Bohot se datasets ka size itna bada nahi hota ke unhein disk par patka jaye; unhein poora ka poora memory (RAM) mein rakha ja sakta hai, yahan tak ke multiple machines par distribute bhi kiya ja sakta hai. Is maqsad ke liye **In-Memory Databases** banaye gaye hain.

#### Durability vs Caching Mechanics in Memory

* **Cache-Only Stores:** **Memcached** jaise stores sirf caching ke liye hote hain, jahan agar machine restart ho jaye aur data urr jaye, toh system par koi aanch nahi aati.
* **Durable In-Memory Databases:** Yeh databases poora data RAM mein rakhne ke bawajood full durability provide kartiin hain. Yeh maqsad special hardware (jaise battery-powered RAM) se hasil hota hai, ya phir aam tor par disk par **changes ka ek append-only log (WAL)** write kar ke, periodic snapshots save kar ke, ya data ko doosri machines par replicate kar ke hasil kiya jata hai.

> Jab yeh databases disk par log likhti hain, tab bhi inhein *In-Memory Database* hi mana jata hai. Kyunki disk ka istemaal sirf aur sirf crash recovery ke liye ek append-only backup ke tor par ho raha hota hai, jabki **saari read queries 100% direct RAM memory se serve hoti hain**.

#### Real-World Products & Technology Models

* **Relational In-Memory:** **VoltDB**, **SingleStore**, aur **Oracle TimesTen** relational model par chalne wale in-memory databases hain. Unka dawa hai ke wo disk-based data structures ke software management overheads ko khatam kar ke bohot barri performance gains dete hain.
* **Key-Value Store:** **RAMCloud** ek open-source durable in-memory store hai jo RAM aur disk dono par log-structured approach use karta hai. **Redis** aur **Couchbase** asynchronous disk writing ke zariye *weak durability* provide karte hain.

#### The Counterintuitive Performance Truth (Asli Sachai)

Aam tor par log yeh samajhte hain ke in-memory databases ki tezi ki wajah sirf yeh hai ke unhein disk se read nahi karna parta. **Yeh ek bohot bara mukhalta (misconception) hai.** Agar aapke paas ek standard disk-based database hai aur aapke paas itni RAM hai ke pura data us mein fit ho jaye, toh Operating System automatic saare disk blocks ko memory mein cache kar leta hai. Phir wo disk-based database bhi disk se zero reads karta hai.

In-memory databases ke zyada tez hone ki asli wajah yeh hai ke wo memory ke internal data structures ko disk-serialized byte formats mein encode/decode karne ke **software overheads, memory-packing, aur concurrency locks ke code complexity se bach jaate hain**.

Sath hi, RAM ka poora control hone ki wajah se yeh databases aise complex data models asani se implement kar leti hain jo disk-based indexes par namumkin hain. For example, **Redis** memory ke andar priority queues (ZSETs) aur sets ka behtareen database interface bohot hi simple code implementation ke sath chalata hai.

```plaintext
[ Client Read Request ] ---------------------> [ Served 100% from RAM Data Structures ]
                                                              ^
[ Client Write Request ] ---> [ RAM Data Structure Update ] --+ (Fast Acknowledgment)
                                      |
                                      v (Asynchronous Background Thread)
                        [ Disk Append-Only Log / Snapshot ] (Only for Durability)

```

---

## Mockup System Design Scenario (Interview Style)

### Interview Context & Problem Statement

> **Interviewer:** "Hamein ek Real-Time Ride-Hailing Platform (jaise Uber/Careem) ka **Driver Matching & Surge Pricing engine** design کرنا ہے۔ Dunya bhar se drivers har 2 seconds baad apni exact GPS Coordinates (Latitude, Longitude) aur Availability Status send kar rahe hain ($200\text{k pings/sec}$). System ko instant multidimensional geospatial search chalani hai taake customer ke paas wale top 5 closest available drivers dhoonde ja sakein aur real-time analytics ke mutabaq surge calculate ho sake. Reads aur writes dono ultra-low latency ($<5\text{ms}$) demand karte hain. Aap kaisa storage indexing structure use karenge?"

### Solution Strategy & Conceptual Flow

Is ultra-low latency geospatial matching system ke liye hum **Durable In-Memory Database Architecture (Redis with Spatial Indexing)** choose karenge. Relational disk databases ya standard B-trees/LSM-trees yahan fail ho jayenge kyunki continuous random spatial updates coordinates ko disk encoding block formats mein convert karne par disk I/O bottleneck paida kar denge.

1. **Why In-Memory Engine:** Data ka size (Active Drivers Location) itna bara nahi hota ke ek machine ki RAM mein fit na ho sake. In-memory engine use karne se hum disk block encoding ka serialization overhead zero kar denge.
2. **Advanced Data Structure (Geohash / Quadtrees):** Redis memory ke andar direct primitive arrays aur sorted sets use karta hai. Hum driver ki location ko Geohash (ek unique string/number string) mein convert karenge aur use memory ke sorted set structure mein insert karenge. Memory indexing $O(\log n)$ par instant update aur point queries response karegi.
3. **Data Protection & Durability Flow:** Kyunki live drivers ka location data transient (kuch dair ka) hota hai, agar system crash ho bhi jaye toh drivers 2 seconds baad dubara apni location bhej denge. Isliye hum strict disk synchronization ke bajaye **Asynchronous Append-Only File (AOF) logging** setting use karenge taake write path block na ho aur low-latency maintain rahe.

### Architectural Flow Diagram

```plaintext
[ 200k/sec Driver GPS Pings ] ---> [ Ingestion Gateway ]
                                           |
                                           v
                        [ In-Memory Redis Cluster (RAM) ]
                        +---------------------------------------+
                        | Spatial Sorted Set (ZSET) Index       |
                        | Key: "driver_locations"               |
                        | Value: [Driver_A : Geohash_9v1b]      | <---+
                        +---------------------------------------+     |
                                           |                          |
                     (Instant 100% Memory Lookup & Matching)          |
                                           |                          |
                                           v                          |
[ Customer Request: Find Nearest ] ---> [ Geospatial Radius Query ] --+
                                           |
                                           v (Async Flush for Durability)
                             [ Disk: Background AOF Log File ]

```

### Trade-off Evaluation in this Design

* **Pros:** Processing speed bemisaal ($<2\text{ms}$) hogi kyunki database software layers mein koi disk-format formatting logic nahi chal raha. Spatial range queries RAM algorithms ke zariye instant response karengi.
* **Cons:** Agar poora cluster ek sath crash ho jaye, toh un 2 seconds ke darmiyan hone wale data updates temporary loss ho sakte hain jab tak drivers dubara ping na bhej dein (Weak Durability Trade-off for Performance). Sath hi, system ko RAM cost optimization ka khayal rakhna parega.


---