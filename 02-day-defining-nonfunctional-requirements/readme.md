# Defining Nonfunctional Requirements

## Defining Nonfunctional Requirements

Jab bhi koi application banayi jati hai, toh list mein sab se upar **Functional Requirements** hoti hain (yani app kya kaam karegi, buttons kaise kaam karenge, screens kaisi hongi).
Lekin iske sath sath kuch **Nonfunctional Requirements** bhi hoti hain jo utni hi ahem hoti hain. Agar aapki app kaam toh theek karti hai lekin bohot slow hai ya baar baar crash hoti hai, toh aisi app ka hona na hona barabar hai.

Is chapter mein hum in chaar main nonfunctional requirements ko measure karna seekhenge:

1. **Performance:** System kitni tezi se response deta hai?
2. **Reliability:** Jab system mein kuch galat (error) ho, tab bhi wo theek se kaam karna jaari rakhe.
3. **Scalability:** Jab load (traffic) barhe, toh system naya hardware efficiently add kar ke load kaise manage karega?
4. **Maintainability:** System ko lumbay arsay tak theek aur update rakhna kitna asaan hai?

In concepts ko clear karne ke liye writer ne ek real-world case study (X / Twitter) ki example di hai.

---

## Case Study: Social Network Home Timelines

Farz karein hum Twitter jaisa ek social network bana rahe hain jahan log posts (tweets) karte hain aur ek dusre ko follow karte hain.
System ka aam (average) load:

* **Posts:** 5,800 posts per second (PPS). Peak time par yeh 150,000 PPS tak ja sakta hai.
* **Followers:** Average user 200 logon ko follow karta hai aur uske 200 followers hain. Lekin kuch celebrities ke 100 million se zyada followers bhi hote hain.

### Representing Users, Posts, and Follows

<div align="center">
  <img src="./images/01.jpg" width="700"/>
</div>

Agar hum is system ka data ek aam Relational Database (jaise MySQL) mein rakhein toh schema kuch is tarah hoga jaisa **Figure 2-1** mein dikhaya gaya hai:

* **Users Table:** User ki details (ID, name, profile image).
* **Posts Table:** Kisne post ki, kya text hai, aur kab ki.
* **Follows Table:** Kaun kisko follow kar raha hai.

Sab se zaroori read operation hai **Home Timeline** (woh screen jahan aapko un sab logon ki posts nazar aati hain jinhe aap follow karte hain). Ise laane ke liye aam taur par yeh SQL query chalayi jayegi:

```sql
SELECT posts.*, users.* FROM posts
 JOIN follows ON posts.sender_id = follows.followee_id
 JOIN users ON posts.sender_id = users.id
 WHERE follows.follower_id = current_user
 ORDER BY posts.timestamp DESC
 LIMIT 1000

```

**The Polling Problem (Bohot Bura Idea):**
Agar hum chahte hain ke users ko naye posts fauran (5 seconds ke andar) nazar aayein, toh unki app har 5 seconds baad yeh query DB par bhejegi (isay polling kehte hain). Agar 10 million users online hain, toh yeh DB par **2 million queries per second** banega!
Is se bhi buri baat yeh hai ke ek query mein DB ko us user ke follow kiye gaye 200 logon ki posts alag alag dhoond kar merge karni parti hain (2 million * 200 = **400 million lookups per second**). Yeh kisi bhi relational database ke liye handle karna namumkin hai.

---

### Materializing and Updating Timelines

Is maslay ko hal karne ke liye humein "Pull" (polling) se nikal kar **"Push"** architecture par aana hoga aur system ko pre-compute karna hoga.

**The Solution: Materialization (Precomputing)**
Hum har user ke liye ek data structure (cache / materialized view) pehle se bana kar rakhenge jismein uski timeline ready hogi. Jab user log in karega, usay heavy query ke bajaye seedha yeh ready-made cache (mailbox) pakra diya jayega.

Is architecture ka data flow **Figure 2-2** mein dikhaya gaya hai:

<div align="center">
  <img src="./images/02.jpg" width="700"/>
</div>


* **Fan-out:** Jab bhi koi user post karta hai, backend API us user ke sab followers ki list nikalegi aur us ek post ko un tamam followers ke "Mailbox" (Timeline Cache) mein dal (push kar) degi. Ek input request ka multiple downstream requests mein taqseem ho jana **Fan-out** kehlata hai.

**Performance Calculation:**
Agar 5,800 posts/sec hain aur har post ko 200 followers tak push (fan-out) karna hai, toh humein DB par **1.1 million writes per second** karni hongi. Yeh bohot bara number hai, lekin pichli approach ke 400 million reads/sec se kahin behtar aur sasta hai. Hum reads slow hone ke bajaye writes par thora extra kaam kar lenge.

**The Extreme Cases (Celebrity Problem):**
Yeh materialized view 99% users par perfect chalega, lekin edge cases mein architecture fail ho jayega:

* **High-Follower Accounts (The Justin Bieber Problem):** Agar kisi celebrity ke 100 million followers hain, aur wo post kare, toh hamara system ek second mein 100 million logon ke mailbox (cache) update karne ki koshish karega. Is fan-out se system crash ho sakta hai.

**The Hybrid Architectural Solution:**
Isay hal karne ke liye Twitter jaisi companies **Hybrid Approach** (mixed approach) use karti hain:

* Aam users ke posts unke followers ke mailbox (cache) mein push (fan-out) hote hain.
* Celebrities ke posts alag (separate) store hote hain. Jab user apni app kholta hai, toh system uske personal cache mein maujood posts aur celebrity ke alag para data, dono ko run-time par (pull) merge kar ke dikhata hai. Is se 100 million writes karne ki zaroorat nahi parti.

---

### 💻 Mockup System Design & Interview Scenario

**Scenario:** Aap WhatsApp jaisi ek chat app ka backend design kar rahe hain. Group chats mein jab koi user message bhejta hai, toh wo message sab members ko forward hota hai. Ek user ne 50,000 members wala ek bara broadcast group banaya hai. Jab wo message bhejta hai, toh server ke CPU spikes aate hain aur message deliver hone mein minutes lag jate hain. Isko architecturally kaise theek karenge?

**Architectural Flow (Plaintext Diagram):**

<div align="center">
  <img src="./images/04.jpg" width="700"/>
</div>

**Interview Trade-Off Questions:**

* **Question:** *Timeline banate waqt aapne "Read-Heavy" query se jaan churanay ke liye Fan-out ("Write-Heavy") approach kyu choose ki?*
* **Answer:** Kyun ke social networks hamesha Read-Heavy hote hain. Ek post likhi ek dafa jati hai lekin parhi hazaron dafa jati hai. Agar hum har read par complex JOIN lagayenge toh DB crash ho jayega. "Materialization" ka concept yahi hai ke reads ko bijli ki tezi par rakhne ke liye, hum writes ke dauran thora extra kaam aur storage (cache) qurban (trade-off) karne ko tayar hain.


* **Question:** *"Fan-out" architecture mein agar peak load (jaise football world cup final) aaye toh kya hoga?*
* **Answer:** Agar spikes aate hain toh hum timeline updates ko strictly synchronous nahi rakhte. Hum background queues (jaise RabbitMQ ya Kafka) ka istemal karte hain. Agar load barh jaye, toh posts aam dinon ke 5 seconds ke bajaye 30 seconds baad deliver hongi (Eventual Consistency), lekin hamara frontend fast load hota rahega kyun ke wo hamesha ready cache read kar raha hoga.


---

## Describing Performance

Software performance ko samajhne aur measure karne ke liye do main metrics ka istemal kiya jata hai, jinhe aapas mein integrate kar ke system ki capacity ka andaza lagaya jata hai:

* **Response Time:** Yeh wo total waqt hai jo user ke request bhejne se lekar usay screen par output (response) milne tak lagta hai. Isay milliseconds (ms) ya seconds mein napa jata hai. Yaad rahe ke response time aur service time mein farq hota hai; response time mein network delay aur queue mein khray rehne ka waqt bhi shamil hota hai.
* **Throughput:** Aik second mein aapka system kitni requests process kar raha hai, ya kitna data volume handle kar raha hai (Requests Per Second - RPS / Queries Per Second - QPS). Iski unit hamesha "somethings per second" hoti hai.

Writer ne pichli Twitter case study se inko is tarah map kiya hai:

* **Throughput Metrics:** "Posts per second" (5,800 PPS) aur "Timeline writes per second" (1.1 million).
* **Response Time Metrics:** "User ki home timeline load hone mein kitna time laga" aur "Celebrity ki post followers tak kitni der mein pohnchi".

### Figure 2-3 Ka Detailed Architectural Breakdown

<div align="center">
  <img src="./images/03.jpg" width="700"/>
</div>

Aapne jo image share ki hai, wo darasal Throughput aur Response Time ka aik bohot hi critical relational graph hai. Is graph ke curve ko technical andaz mein samajhna zaroori hai:

* **Flat Line (Service time on unloaded system):** Graph ke shuru mein jab throughput (load) kam hota hai, toh response time bilkul flat aur stable rehta hai. Iska matlab hai hardware farigh hai, request aati hai aur CPU usay fauran process kar ke wapas bhej deta hai.
* **The Curve Upwards (Queueing Delay):** Jaise jaise throughput barhta hai aur hardware ki maximum capacity ke kareeb pohnchta hai, response time achanak seedha upar (dramatically high) bhagta hai.
* **Wajah (The Core Reason):** Is exponential barhauti ki wajah **Queueing** hai. Jab system par load bohot zyada hota hai, toh jab nayi request aati hai, CPU pehle se purani requests process kar raha hota hai. Nayi request ko memory queue (waiting line) mein khara hona parta hai. Hardware limit ke paas pohnchkar yeh queue itni lambi ho jati hai ke waiting time (queueing delay) asli processing time se kahin zyada barh jata hai.

---

## When an Overloaded System Won't Recover

Jab system apni maximum hardware capacity ko touch karne lagta hai, toh wo aik aisi khatarnak halat mein dakhil ho sakta hai jahan se uska khud se theek hona namumkin ho jata hai. Isay distributed systems mein **Metastable Failure** kehte hain.

* **The Vicious Cycle (Retry Storm):** Jab queueing delay ki wajah se response time had se zyada barh jata hai, toh frontend clients (mobile apps/browsers) samajhte hain ke request gum ho gayi hai. Wo timeout ho jate hain aur **wahi request dobara (retry) bhej dete hain**.
* **The Explosion:** Is se server par traffic ka load achanak double ya triple ho jata hai. Pehle se duba hua server mazeed dab jata hai. Is tabahi ko **Retry Storm** kehte hain. Is halat mein agar asli users traffic kam bhi kar dein, tab bhi system tab tak theek nahi hota jab tak usay reboot ya reset na kiya jaye, kyun ke memory queues pehle hi crash ho chuki hoti hain.

### Architectural Remedies (Halat ko control karne ke tareeqe)

Is metastable failure se bachne ke liye distributed software architectures mein yeh patterns lagaye jate hain:

1. **Client-Side Solved Layers:**
* **Exponential Backoff:** Agar client ki request fail ho jaye, toh wo fauran retry na kare. Pehli retry 1 second baad, dusri 2 second, teesri 4 second baad kare (waqt barhata jaye). Sath mein **Jitter** (randomness) shamil kare taake saare clients aik hi sath attack na karein.
* **Circuit Breaker:** Agar koi backend service lagatar errors ya timeouts de rahi hai, toh Gateway level par circuit "open" ho jata hai. Ab mazeed requests us service tak bhej kar usay mazeed dubaane ke bajaye client ko fauran "Service Unavailable" ka fallback response de diya jata hai.


2. **Server-Side Solved Layers:**
* **Load Shedding:** Server jab dekhta hai ke uska CPU 95% touch ho raha hai aur queue full hai, toh wo nayi aane wali requests ko chup-chap proactively drop (reject) karna shuru kar deta hai (HTTP 503 Service Unavailable) taake jo purani requests andar hain unhe kam az kam safely poora kiya ja sake.
* **Backpressure:** Server downstream services ya clients ko signal bhejta hai ke "meri capacity full hai, data bhejna slow karo" (Token Bucket algorithms ke zariye rate limiting lagayi jati hai).



---

### 💻 Mockup System Design & Interview Scenario

**Scenario:** Aap aik Ticket Booking platform (jaise PSL Finals ya Concert tickets) ke Architect hain. Flash sale shuru hote ہی 100,000 users aik sath "Buy Ticket" par click karte hain. Server ka database bottleneck ban jata hai, response time 20 seconds par chala jata hai. Clients timeout ho kar lagatar "Retry" button daba rahe hain, jis se system freeze ho gaya hai. Aap is "Retry Storm" ko rokne ke liye system kaise redesign karenge?

**Architectural Strategy:**

* Hum Gateway level par **Token Bucket Rate Limiter** lagayenge taake throughput hardware limit se aagay na jaye.
* Backend par **Load Shedding** active karenge aur clients mein **Exponential Backoff** implement karwayenge.

**Architectural Flow (Plaintext Diagram):**

<div align="center">
  <img src="./images/05.jpg" width="700"/>
</div>

**Interview Trade-Off Questions:**

* **Question:** *Load Shedding lagane se toh users ko error (HTTP 429/503) milega, kya yeh bura user experience nahi hai?*
* **Answer:** Yeh aik zaroori trade-off hai. Do options hain: (1) Ya toh system ko open chora jaye jahan Retry Storm se poora system crash ho jaye aur 100% users down ho jayein. (2) Ya phir Load Shedding use ki jaye jahan 20% users ko safely ticket mil jaye aur baqi 80% ko clean "Server Busy" ka message dikhe. Distributed systems mein full crash se partial degradation hamesha behtar hoti hai.


* **Question:** *Throughput barhane ke liye kya hum sirf hardware barhate jayein (Vertical Scaling)? Is graph ka hardware se kya talluq hai?*
* **Answer:** Vertical scaling (bada server lagana) ki aik absolute physical limit hoti hai. Graph batata hai ke hardware chahe jitna bada ho, aik point par aakar throughput queueing wall ko touch karega hi. Asal scalability yeh hai ke hum system ko horizontally scale karein taake jab throughput limit ke paas pohnche, toh naye stateless nodes add ho kar graph ki line ko dubara flat (stable) kar dein.



---

### 📌 Quick Revision Hints

* **Response Time vs Throughput:** Response time user ka wait time hai; throughput server ka processed work per second hai.
* **Queueing Effect (Figure 2-3):** Max capacity ke paas response time isliye dामatially barhta hai kyun ke requests ko CPU farigh hone tak queue mein sarna parta hai.
* **Metastable Failure / Retry Storm:** Slow system par clients ke bar bar retry karne se peda hone wali tabahi jo reboot ke baghair theek nahi hoti.
* **Defenses:** Client side par *Exponential Backoff + Jitter*, server side par *Load Shedding, Circuit Breakers, aur Backpressure*.

