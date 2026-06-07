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