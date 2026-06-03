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

---

## Latency and Response Time

Aam taur par log **Latency** aur **Response Time** ko ek hi samajhte hain aur in terms ko aapas mein badal kar use karte hain. Lekin is book mein writer ne inke darmiyan aik bohot hi bareek aur clear technical line draw ki hai, jise **Figure 2-4** mein wazeh dikhaya gaya hai.

Is style ke diagrams mein waqt (time) hamesha left se right flow karta hai. Horizontal lines nodes (Client aur Server) ko represent karti hain, jabke diagonal (tirchay) arrows network par travel karne wali request aur response ko dikhate hain.

---

### Architectural Breakdown of Figure 2-4

Agar hum aik single web request ke poore safar ko micro-level par breakdown karein, toh uske total waqt (Response Time) mein yeh chaar bade components shamil hote hain:

<div align="center">
  <img src="./images/06.jpg" width="700"/>
</div>

* **Response Time (Client's View):** Yeh wo mukammal waqt hai jo client apni aankhon se dekhta hai. Jab user button dabata hai tab se lekar jab tak screen par response poora load nahi ho jata, yeh total duration response time hai. Isme system ke har hissay ka delay shamil hota hai.
* **Service Time (Active Processing):** Yeh wo waqt hai jab server ka CPU, RAM, ya database actually mein us request ko compute ya process kar raha hota hai.
* **Queueing Delay (Waiting Lines):** Yeh system mein line mein lagne ka waqt hai jab aapka data inactive hota hai. Yeh do major points par ho sakta hai:
* **Inbound Queueing:** Request server tak pohnch gayi, lekin CPU pehle se kisi aur kaam mein busy hai, toh request ko CPU available hone tak wait karna parta hai.
* **Outbound Queueing:** Server ne processing poori kar li (Service time khatam), lekin server ka network card (NIC) pehle se doosri heavy files send kar raha hai. Ab response packet ko outbound buffer mein tab tak rukhna parega jab tak network interface free na ho jaye.


* **Latency (The Invisible Flight):** Yeh aik catchall term hai us waqt ke liye jab data par koi active processing nahi ho rahi hoti, balkay data "latent" (soya hua) hota hai. Iska sab se bada hissa **Network Latency** hai—yani request aur response ka wires, fibers, ya routers ke zariye hawa/samandar ke raaste travel karne ka waqt.

---

### Real-World Noise: Response Time Kyun Badal Jata Hai?

Agar aap ek hi server par, ek hi network se, aik hi request baar baar bhejein, tab bhi har dafa response time different aayega. Writer ne iski wajohaat batane ke liye bohot se deep physical aur software level ke real-world examples diye hain:

* **Context Switches:** Server ka Operating System achanak aapki API process ko rok kar kisi background OS process ko priority de deta hai.
* **TCP Retransmission:** Network par koi chota sa glitch aaya, packet drop ho gaya, aur TCP protocol ne usay pehchana aur dubara re-transmit kiya, jis se milliseconds ka delay aaya.
* **Garbage Collection (GC) Pauses:** Agar aapki API Java, Go, ya Node.js mein hai, toh runtime engine memory saaf karne ke liye poori application ko kuch milliseconds ke liye freeze kar deta hai (Stop-the-World pause).
* **Page Faults:** Server ko data RAM mein nahi mila, OS ko majbooran slow hard drive ya SSD (virtual memory storage) se block read karna para.
* **Mechanical Vibrations:** Server room ke rack mein physical hard drives ya cooling fans ki thartharahat (vibrations) ki wajah se drive ka mechanical head apni jagah se hil jata hai aur data read karne mein thora extra waqt leta hai.

---

### Head-of-Line Blocking (HOL)

Queueing delays hi asal wajah hote hain jo response times mein achanak itna bada utaar-chadaao (variability) paida karte hain. Server aik waqt mein parallel mein bohot kam kaam kar sakta hai (limited by CPU cores).

Agar aage khari hui kuch requests bohot slow hain (maslan heavy database read kar rahi hain), toh unke peche khari hui tamam fast requests (jin ka service time sirf 2ms tha) line mein phans jati hain. Is effect ko **Head-of-Line Blocking** kehte hain. Peche kharay client ko bohot slow response time dikhega, halankay server ne uski request par sirf 2ms lagaye thay.

Isi liye system architecture ka sunehra usool yeh hai ke **hamesha performance metrics aur response times ko client-side se measure karein**, na ke sirf server ke service time par khush hote rahein.

---

### System Behavior & Data Flow Diagram

Yeh diagram Figure 2-4 ke conceptual flow aur timeline ko plaintext format mein represent karta hai ke data kis waqt kis state mein hota hai:

<div align="center">
  <img src="./images/08.jpg" width="700"/>
</div>

---

### 💻 Mockup System Design & Interview Scenario

**Scenario:** Aap aik Multi-tenant B2B platform design kar rahe hain jahan ek hi API backend ko light users (jo sirf dashboard dekhte hain - 5ms request) aur heavy corporate users (jo poore mahine ki CSV report download karte hain - 10 seconds request) dono use karte hain. Production mein light users complain kar rahe hain ke achanak unki app 10 seconds ke liye hang ho jati hai. API ka metrics dashboard check karne par server ka *average service time* bilkul normal (20ms) dikhata hai. Masla kahan hai aur isay kaise theek karenge?

**Architectural Diagnosis (The Interview Answer):**
Masla "Head-of-Line Blocking" ka hai. Heavy CSV reports server ke processing threads aur memory buffers ko block kar deti hain, jiski wajah se light queries queue mein phans jati hain (Queueing Delay). Server ka internal metrics register software level par service time toh 5ms hi dikhata hai jab wo process hoti hain, lekin client-side par total response time barh chuka hota hai.

**Architectural Redesign Strategy:**
Hum "Thread/Queue Isolation" technique use karenge. Heavy operations ko light synchronous API calls se bilkul alag algorithms aur workers par shift kiya jayega.

<div align="center">
  <img src="./images/09.jpg" width="700"/>
</div>


**Interview Trade-Off Questions:**

* **Question:** *Agar hum do alag pools (fast aur slow) bana dein, toh kya hardware resources waste nahi honge jab heavy reports koi download na kar raha ho?*
* **Answer:** Yeh aik necessary trade-off hai jahan hum compute density ki qurbani de kar response time ki stabilization khareed rahe hain (Predictable Latency). Is resource wastage ko kam karne ke liye hum slow pool ke workers ko Cloud Native patterns (jaise Kubernetes KEDA ya Serverless FaaS) par rakh sakte hain jo srf tab scale-up hon jab batch queue mein load aaye, warna zero par scale-down rahain.



---

### 📌 Quick Revision Hints

* **Response Time:** Client-side visible time. Total delays ka sum.
* **Service Time:** Server ka actual active processing time.
* **Queueing Delay:** CPU ya network cards free hone ka waiting time. HOL blocking ka zimmedar.
* **Network Latency:** Data ka wire/air mein guzarne wala waqt.
* **Head-of-Line Blocking (HOL):** Kuch slow requests ki wajah se poori queue ka jam ho jana aur peche khari fast requests ka slow ho jana.
* **Metric Location:** Response time hamesha client-side par napa jana chahiye taake queueing aur network latency sahi se capture ho sakay.

---

## Average, Median, and Percentiles

System architecture mein response time ko kabhi bhi ek single number ke taur par nahi dekha jata, balkay yeh values ki aik poori **Distribution** (bikhrao) hoti hai. Jab network delays mein random badlaao aata hai, toh usay hum **Jitter** kehte hain.

### Figure 2-5 Ka Detailed Structural Breakdown

Aapne jo image share ki hai, wo system performance ko samajhne ke liye sab se ahem statistical concept hai. Isme 100 requests (gray bars) ka response time dikhaya gaya hai:

<div align="center">
  <img src="./images/07.jpg" width="700"/>
</div>

* **Mean (Average) Ki Kamzori:** Mean nikalne ke liye hum tamam response times ko jama karte hain aur requests ki tadad se divide kar dete hain. Graph mein waazeh nazar aa raha hai ke **Mean ki line Median se upar hai**. Aisa isliye hai kyun ke system mein kuch thori si requests (outliers) bohot slow hoti hain (jaise GC pause ya disk page fault ki wajah se), aur yeh ginti ki slow requests pooray average ko upar khinch leti hain. Isliye Mean aapko kabhi yeh nahi bata sakta ke ek "typical" user ko kitna delay mila.
* **Median (p50 - Typical User Experience):** Agar aap saari requests ko sab se tez se sab se slow ke order mein sort karlein, toh jo bilkul darmiyan (50th percentile) wali value hogi, wo Median hai. Agar p50 200ms hai, toh iska matlab hai 50% users ko 200ms se kam time mila aur 50% ko us se zyada. Yeh metric aam user ke experience ko sahi tarah bayaan karta hai.
* **Tail Latencies (p95, p99, p99.9):** Apne system ke sab se bure outliers ko dhoondne ke liye hum unche percentiles ko dekhte hain. Graph mein **95th percentile** ka matlab hai ke 100 mein se 95 requests fast thin, aur sirf 5 requests us line se upar thin. Isi tarah **p99** batata hai ke sab se slow 1% requests kitni buri thin.

### Writer Ki Real-World Amazon Example Ka Conceptual Flow

Amazon apne internal microservices ke liye **99.9th percentile (p99.9)** ke strict target rakhta hai, halankay yeh 1,000 mein se sirf 1 user ko affect karta hai. Is decision ke peche aik bohot gahra data flow aur business model hai:

* Jo customers p99.9 wali slow requests hit karte hain, wo aam taur par Amazon ke sab se **Valuable Customers** (heavy buyers) hote hain.
* Unka account purana hota hai, unhone hazaron purchases ki hoti hain, aur unka profile data massive hota hai.
* Jab wo homepage kholte hain, toh backend service ko unke liye database se bohot zyada data fetch karna parta hai, jiski wajah se unka response time tail latency (slowest pool) mein chala jata hai.
* Agar un sab se ameer customers ko website slow dikhegi, toh wo shopping chor denge, jis se direct revenue ka bhari nuksan hoga. Isliye tail latency ko optimize karna business ke liye critical hai.
* *Trade-off Decision:* Amazon ne p99.99 (10,000 mein se 1 request) ko optimize karna chor diya kyun ke us par hardware ka kharcha bohot zyada aa raha hai aur utna financial faida nahi ho raha tha (Diminishing returns).

---

## The User Impact of Response Times

Yeh baat toh taye hai ke fast website users ko pasand hoti hai, lekin latency ka user behavior par asar napna kafi mushkil kaam hai. Writer ne yahan industry ke mukhtalif studies ke tazadaat (paradoxes) ko breakdown kiya hai:

* **Google & Bing Data:** Google ne 2006 mein kaha ke 400ms se 900ms ka delay aane se 20% traffic drop ho gayi. Lekin 2009 ki study mein 400ms delay se sirf 0.6% searches kam huin. Bing ke mutabiq 2 seconds ka delay ad revenue ko 4.3% kam karta hai.
* **The Akamai Paradox (Data Ki Bareeki):** Akamai ki aik study ne dawa kiya ke 100ms latency barhne se conversion rate 7% gir jata hai. Lekin unke data mein aik ajeeb cheez mili: **Jo pages sab se zyada tez load ho rahe thay, unka conversion rate sab se bura tha!**
* *The System Behavior Explanation:* Asal mein jo pages microseconds mein load ho rahe thay, wo koi kaam ke pages nahi thay, balkay wo **404 Error Pages** thay. Kyun ke error page par koi image ya content nahi hota, isliye wo sab se tez load hote hain, aur zahir hai error page par koi user khareedari (conversion) nahi karta. Isliye bina context ke metrics dekhna architect ko gumrah kar sakta hai.
* **Yahoo Study:** Yahoo ne search results ki quality ko control kar ke test kiya aur sabit kiya ke jab responses mein 1.25 seconds ka farq aata hai, toh fast searches par 20% se 30% zyada clicks aate hain.

---

## System Behavior & Data Flow Diagram

Yeh diagram represent karta hai ke percentile monitoring system kaise kaam karta hai aur requests kaise different buckets mein sort hoti hain:

<div align="center">
  <img src="./images/10.jpg" width="700"/>
</div>

---

## 💻 Mockup System Design & Interview Scenario

**Scenario:** Aap aik B2B Fintech application (jaise Stripe) ke Architect hain. Company ke VPs kehte hain ke hamara Datadog monitoring dashboard "Average Response Time" hamesha 100ms dikhata hai, jo ke behtareen hai. Lekin hamare sab se bade enterprise clients (jo har mahine millions of dollars process karte hain) complain kar rahe hain ke unki payment processing API aksar 3 se 4 seconds leti hai. Masla kahan hai aur aap metrics pipeline ko kaise redesign karenge?

**Architectural Diagnosis:**
Masla yeh hai ke aapka monitoring system **Mean/Average** use kar raha hai. Enterprise clients ka data size bada hota hai, unke API payloads bade hote hain, aur unka data relational tables mein complex structures par hit karta hai. Unki requests p99 ya p99.9 (tail latency) mein phansi hui hain, jo ke total traffic ka sirf 1% hain. Average metric unke is bure experience ko baki 99% aam users ke fast data ke peche chupa (hide kar) raha hai.

**Architectural Redesign Strategy:**
Hum Average metric ko monitor karna chor denge aur **Rolling Percentiles Histogram** algorithm implement karenge taake tail latencies ko pakda ja sake.

<div align="center">
  <img src="./images/11.jpg" width="700"/>
</div>

**Interview Trade-Off Questions:**

* **Question:** *Percentiles (p99) ko compute karna Average nikalne se zyada mehanga (resource-intensive) kyun hota hai?*
* **Answer:** Average nikalne ke liye hamen sirf do cheezein save karni parti hain: total sum aur total count. Memory overhead zero hota hai. Lekin exact percentiles nikalne ke liye aapko aik specific time window (maslan 1 minute) ke tamam response times ko memory mein rakh kar sort karna parta hai. High throughput par yeh bohot memory consume karega. Is trade-off ko hal karne ke liye distributed systems mein **T-Digest** ya **HdrHistogram** jaise mathematical approximations use kiye jate hain jo kam memory mein 99% accurate percentile batate hain.


* **Question:** *Agar p99.9 par latency barh rahi hai, toh kya hamen load balancer par round-robin algorithm change karna chahiye?*
* **Answer:** Haan, kyun ke round-robin blindly request bhejta hai. Agar aik node par pehle se p99.9 wali do heavy requests chal rahi hain, toh round-robin teesri request bhi usi par bhej dega jise humne abhi parha ke **Head-of-Line Blocking** paida karegi. Iska behtar trade-off yeh hai ke hum **Least Connections** ya **Peak Ema** algorithm use karein taake requests un nodes par jayein jo free hain.



---

## 📌 Quick Revision Hints

* **Mean (Average):** Outliers se buri tarah skew (mutasir) hota hai; throughput capacity ki calculation ke liye acha hai, user experience ke liye nahi.
* **Median (p50):** Asli "typical" waiting time batata hai.
* **Tail Latency (p99, p99.9):** Sab se slow requests. Inhe optimize karna zaroori hai kyun ke heavy/premium users aksar isi zone mein aate hain.
* **Akamai Paradox:** Sub se fast pages aksar khali ya error (404) pages hote hain, isliye metrics ko context ke sath dekhna lazmi hai.
* **The Cost of Tail Latency:** p99.99 tak jana diminishing returns deta hai, isliye p99.9 par line draw karna aik standard industry practice hai.

---