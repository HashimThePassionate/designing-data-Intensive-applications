# Sharding

Grace Murray Hopper ne 1962 mein ek bohot gehri baat kahi thi ke humein computers ko sequential (ek ke baad ek kaam karne) ki limit mein nahi bandhna chahiye. Humein data ke aapas ke relationships aur uski priorities ko define karna chahiye, na ke sirf purane ruttay-rattaye tareeqon (procedures) par chalna chahiye.

Jab data bohot bada ho jata hai, toh ek akela computer usay sambhal nahi pata. Ek distributed database data ko alag-alag nodes (machines) par do bade tareeqon se phelata (distribute) hai:

* **Replication:** Iska matlab hai ke **ek hi data ki exact copies** ko alag-alag machines par save karna. Iska faida yeh hota hai ke agar ek machine kharab bhi ho jaye, toh data doosri machine par safe rehta hai.
* **Sharding (ya Partitioning):** Agar data itna zyada ho jaye ya us par likhne ka kaam (write throughput) itna barh jaye ke ek akeli machine use bardasht na kar sakay, toh hum data ke **chote-chote tukde (shards)** kar dete hain. Phir har ek tukde (shard) ko alag machine par bhej diya jata hai.

> **Asaan Alfaaz Mein (ELI5):** > Farz karein aap ke paas ek bohot bari library hai.
> * **Replication** yeh hai ke aap ek hi kitaab ki 3 copies banakar 3 alag alag kamron mein rakh dein taake agar ek kamre mein aag lag jaye toh kitaab gum na ho.
> * **Sharding** yeh hai ke kitaab itni moti aur bhari hai ke koi usay utha nahi sakta, toh aapne us kitaab ke chapters ko alag-alag kar diya (Chapter 1 ek bande ko diya, Chapter 2 doosre ko) taake sab mil kar asani se parh sakein.
> 
> 

### Sharding Ka Basic Rule

Normally, shards ko is tarah design kiya jata hai ke data ka **har ek piece (yaani har ek row, record, ya document) sirf aur sirf ek hi shard ka hissa hota hai**. Har shard apne aap mein ek complete, chota independent database hota hai, halankay kuch databases aise advanced operations bhi support karte hain jo aik sath bohot se shards ko touch kar sakte hain.

### Sharding Aur Replication Ka Milap (Combination)

Sharding ko aam tor par replication ke sath milakar chalaya jata hai taake data gum na ho. Iska matlab yeh hai ke bhale hi ek record sirf **ek hi shard** ka hissa ho, lekin fault tolerance (system kharab na hone) ke liye us shard ki **multiple copies** alag-alag nodes par majood hoti hain.

Ek akela node (computer) aik se zyada shards ko apne andar store kar sakta hai. Agar hum **Single-Leader Replication Model** (jahan ek leader hota hai aur baqi followers) use kar rahe hain, toh sharding aur replication ka milap bilkul waisa dikhega jaisa niche diye gaye diagram mein hai.

---

### Figure 7-1: Combining replication and sharding ka Deep Breakdown

Chalein is diagram ke ek-ek hisse aur workflow ko bohot hi asaan tarah se samajhte hain:

<div align="center">
  <img src="./images/01.png" width="700"/>
</div>


* **System ka Structure:** Is architecture mein total **4 Nodes (Machines)** hain: `Node 1`, `Node 2`, `Node 3`, aur `Node 4`. Aur pure data ko **4 Shards** mein divide kiya gaya hai: `Shard 1`, `Shard 2`, `Shard 3`, aur `Shard 4`.
* **Leader aur Follower ka Setup:** Har shard ka sirf **ek hi Leader** hota hai (jahan naya data write hota hai) aur baqi **Followers** hote hain (jo leader se data copy karte hain). Ek hi node par kuch shards ke leaders ho sakte hain aur kuch ke followers:
* **Node 1:** Yeh `Shard 1` ka **Leader** hai, lekin `Shard 2` aur `Shard 3` ka **Follower** hai.
* **Node 2:** Yeh `Shard 3` ka **Leader** hai, lekin `Shard 2` aur `Shard 4` ka **Follower** hai.
* **Node 3:** Yeh `Shard 2` ka **Leader** hai, lekin `Shard 1` ka **Follower** hai aur `Shard 4` ka bhi **Follower** hai.
* **Node 4:** Yeh `Shard 4` ka **Leader** hai, lekin `Shard 1` aur `Shard 3` ka **Follower** hai.



#### Replication Streams (Data flow kaise ho raha hai?):

Diagram mein jo rang-birangi arrows (teer) hain, woh **Replication Streams** ko dikha rahi hain (yaani data leader se nikal kar followers tak kaise ja raha hai):

1. **Red Arrow (Shard 1 Stream):** `Node 1` par majood `Shard 1 Leader` naya data receive karta hai aur usay `Node 3` aur `Node 4` par majood `Shard 1 Followers` ki taraf bhejta hai.
2. **Gold Arrow (Shard 2 Stream):** `Node 3` par majood `Shard 2 Leader` data ko `Node 1` aur `Node 2` ke `Shard 2 Followers` ko copy karta hai.
3. **Green Arrow (Shard 3 Stream):** `Node 2` par majood `Shard 3 Leader` apna data `Node 1` aur `Node 4` ke `Shard 3 Followers` ko bhejta hai.
4. **Blue Arrow (Shard 4 Stream):** `Node 4` par majood `Shard 4 Leader` apna data `Node 2` aur `Node 3` ke `Shard 4 Followers` ko bhejta hai.

#### Real-World Workflow Example (Writing to Shard 4):

Diagram ke bottom-right corner mein ek user (`Writing to shard 4`) ko dikhaya gaya hai.

* Jab is user ne koi aisa data write karna chaha jo **Shard 4** se belong karta hai, toh system ne us request ko seedha **Node 4** par bheja, kyunke **Shard 4 ka Leader Node 4 par baitha hai**.
* Node 4 is write request ko pakray ga, apne paas save karega, aur phir blue arrows ke zariye usay Node 2 aur Node 3 ke followers tak pohncha dega.

---

## Sharding and Partitioning

Aap jo software use kar rahe hain, uske mutabaq is "Shard" ke alag-alag naam hote hain. Writer ne yahan poori industry ke tools ki terminologies ko clear kiya hai:

| Software / Tool | Shard Ka Industry Name |
| --- | --- |
| **Apache Kafka** | Partition |
| **CockroachDB** | Range |
| **HBase / TiDB** | Region |
| **Couchbase** | vBucket |
| **Riak** | vnode |
| **Apache Cassandra** | token-range |
| **Bigtable / YugabyteDB / ScyllaDB** | tablet |

### PostgreSQL Ka Special Case (Partitioning vs Sharding)

Kuch databases mein "Partitioning" aur "Sharding" ko do bilkul alag concepts mana jata hai.

* **PostgreSQL Partitioning:** Is mein ek bohot bari table ko split karke **ek hi machine** ke andar alag-alag files mein store kiya jata hai. Iska sab se bada faida yeh hota hai ke agar aapko koi poora partition delete karna ho, toh files ko delete karna bohot fast ho jata hai.
* **PostgreSQL Sharding:** Iska matlab hota hai pooray dataset ko split karke **alag-alag physical machines (multiple machines)** par distribute karna.

Lekin baqi zyadatar database systems mein *Partitioning* aur *Sharding* ko ek hi cheez ke do naam samjha jata hai.

### Word "Shard" Ki Dilchasp Tareekh (History)

Yeh "Shard" ka lafay kahan se aaya? Iske peeche do theories hain:

1. **Ultima Online Game Theory:** Ek mashhoor online role-playing game thi jiska naam tha *Ultima Online*. Is game ki kahani mein ek magic crystal (shisha/kanch) toot kar tukde-tukde ho jata hai, aur har ek tukda (shard) us game ki duniya ki ek copy ko refract (dikha) raha hota hai. Wahan se yeh lafaz databases mein aya jiska matlab bana "Parallel servers ka ek set".
2. **Acronym Theory:** Ek aur theory yeh hai ke yeh 1980s ke ek database ka short form (acronym) tha jiska matlab tha **S**ystem for **H**ighly **A**vailable **R**eplicated **D**ata, lekin is database ki baki details tareekh mein kahin kho chuki hain.

> **Zaroori Warning (Crucial Concept):** > Database ke context mein jo hum **Partitioning** (data ke tukde karna) parh rahe hain, iska **Network Partitions (Netsplits)** se door door tak koi taluq nahi hai. Network Partition ek fault (masla) hota hai jahan machines ke darmiyan ka network network tar toot jata hai ya slow ho jata hai.

Database replication ke jitne bhi rules aur concepts hote hain, woh sab shards ki replication par bhi bilkul waise hi apply hote hain. Chunke sharding scheme ka choice aur replication scheme ka choice aapas mein independent hain (ek doosre par depend nahi karte), is liye concept ko mazeed simple rakhne ke liye hum aage replication ko discuss nahi karenge aur sirf sharding par focus karenge.

---