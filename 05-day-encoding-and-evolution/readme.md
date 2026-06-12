# Encoding and Evolution

## Evolvability aur Data Schema Changes

Duniya ka nizam badalta rehta hai, aur yahi usool software applications par bhi lagu hota hai. Waqt ke sath-sath applications mein naye features add hote hain, purane modify hote hain, aur business ki zarooriyat badal jati hain. Jab application ke features badalte hain, toh unke sath-sath **data format aur schemas** ko bhi badalna padta hai. Maslan, kisi user profile mein naya field (jaise profile picture ka URL) add karna ho, ya purane data ko kisi naye andaz mein dikhana ho.

Alag-alag data models is badlao (schema evolution) ko handle karne ke liye mukhtalif tareeqay istemal karte hain:

* **Relational Databases:** Yeh databases is baat par yaqeen rakhte hain ke poore database mein sirf **ek hi strict schema** nafiz (enforce) hona chahiye. Agar schema badalna ho, toh migration (yaani `ALTER TABLE` statements) ke zariye poore table ka structure badla jata hai, aur har waqt sirf ek hi schema active hota hai.
* **Schema-on-read ("Schemaless") Databases:** Document databases (jaise MongoDB) koi strict schema enforce nahi karte. Iska matlab yeh hai ke database ke andar ek hi waqt mein purane format ka data aur naye format ka data sath-sath reh sakta hai.

---

## Code Deployments aur Coexistence ki Realities

Jab database ka schema badalta hai, toh application code ko bhi badalna parta hai taake wo naye fields ko read aur write kar sake. Lekin bade distributed systems mein yeh badlao ek jhatkay mein (instantaneously) har jagah apply nahi kiya ja sakta. Iski do bari wajoohat hain:

### Server-Side Applications aur Rolling Upgrades

Servers par naya code deploy karne ke liye **Rolling Upgrade (ya Staged Rollout)** ka tareeqa apnaya jata hai. Ismein saare server nodes ko ek sath band karke naya code nahi dala jata (jis se downtime aaye), balkay kuch nodes par naya version deploy kiya jata hai, unhe monitor kiya jata hai, aur phir aahista-aahista baqi nodes ko upgrade kiya jata hai. Iska faida yeh hai ke bina kisi service downtime ke releases aasan ho jati hain, lekin iska nateeja yeh nikalta hai ke system mein **ek hi waqt mein purana code aur naya code dono sath-sath chal rahe hote hain**.

### Client-Side Applications

Mobile apps ya desktop applications ke mamle mein developer ka user par koi control nahi hota. User apni marzi se dino, hafton ya mahino baad app update karta hai. Iska matlab hai ke purana client code lambe arse tak naye backend services ke sath communicate karta rahega.

Inhi wajoohat ki bina par, system ko smooth chalane ke liye do tarah ki compatibility ki zaroorat hoti hai:

* **Backward Compatibility:** Jab **naya code** purane code ke likhe huay data ko aasani se read kar sake. Yeh achieve karna aasan hota hai kyunke naye code ke developer ko purane data format ka pehle se pata hota hai.
* **Forward Compatibility:** Jab **purana code** naye code ke likhe huay data ko read kar sake aur crash na ho, balkay naye fields ko chup-chaap ignore kar de. Yeh thoda mushkil hota hai kyunke purane code ko nahi pata hota ke future mein kya naye badlao aane wale hain.

---

## APIs ke Context mein Compatibility Dynamics

Web services, REST APIs, aur RPCs mein compatibility ka rule dono directions mein apply hota hai. Isko samajhne ke liye data flow ko request aur response ke lehaz se dekhna zaroori hai:

* **Old Client calling a New Service:** Agar ek purana app version kisi naye upgraded backend service ko call karta hai, toh naye service ko request samajhne ke liye **backward compatibility** chahiye, aur purane client ko naye service ka reply handle karne ke liye **forward compatibility** chahiye (taake wo response mein aane wale naye fields ko dekh kar crash na ho).
* **New Client calling an Old Service:** Agar naya app version kisi purane backend ko call kare, toh purane backend ko naye request fields ignore karne ke liye **forward compatibility** chahiye, aur naye client ko purane response ko parhne ke liye **backward compatibility** chahiye.

---

## Data Loss Dilemma (Figure 5-1)

Forward compatibility ka ek bohot bada aur khamosh khatra image_af849c.png mein dikhaya gaya hai. Jab system mein rolling upgrade chal raha ho aur purana code aur naya data aapas mein takraen, toh data silently delete ho sakta hai. Chaliye is poore flow ko step-by-step samajhte hain:

```plaintext
[ New Version Code ] ---> Writes Record with new field ("photoURL") ---> [ Database ]
                                                                               |
                                                                               | (Read & Decode)
                                                                               v
[ Old Version Code ] <--- In-Memory Model Object (Drops "photoURL") <----------+
       |
       | (Modifies "favoriteNumber" & Encodes back to JSON)
       v
[ Database ] <----------- Overwrites original record WITHOUT "photoURL" (DATA LOSS!)

```

---

<div align="center">
  <img src="./images/01.png" width="600"/>
</div>

### Architectural Step-by-Step Flow:

1. **Naye Code ka Write Operation:** Ek upgraded node database mein ek record write karta hai jisme ek naya field `photoURL` mojood hai:
`{"userName": "Martin", "favoriteNumber": 1337, "interests": ["hacking"], "photoURL": "http://..."}`
2. **Purane Code ka Read aur Decode Operation:** Rolling upgrade ke dauran, ek purana node (jo `photoURL` ke baare mein nahi jaanta) is record ko database se read karta hai. Jab wo is JSON ko memory mein decode karta hai (jaise image_af849c.png mein Java class `Person` dikhai gayi hai), toh us class mein `photoURL` ka koi variable nahi hota. Nateeja yeh hota hai ke parser us anjaan field (`photoURL`) ko **ignore karke drop** kar deta hai. Memory mein object sirf `userName`, `favoriteNumber`, aur `interests` ko hold karta hai.
3. **Data Update:** Purana code application logic ke mutabaq sirf ek field ko update karta hai, jaise `person.setFavoriteNumber(42)`.
4. **Re-encode aur Write Back (Data Loss):** Ab purana code is object ko dubara JSON mein badalta hai (`person.toJSON()`) aur database mein overwrite kar deta hai. Chunke memory mein `photoURL` pehle hi drop ho chuka tha, is liye naya JSON jo database mein jata hai wo yeh hota hai:
`{"userName": "Martin", "favoriteNumber": 42, "interests": ["hacking"]}`

> **Khatra (The Gotcha):** Database mein bina kisi error ke data overwrite ho gaya, lekin naye code ka likha hua `photoURL` hamesha ke liye **mizh (lost)** ho gaya. Agar aapka serialization framework unknown fields ko as-it-is preserve nahi karta, toh aisi khamosh tabahi distributed systems mein aam baat hai. Is chapter mein hum yahi seekhenge ke JSON, XML, Protocol Buffers, aur Avro is mushkil ko kaise hal karte hain.

---

## Mockup System Design Scenario (Interview Prep)

### Scenario Context

Aap ek High-Traffic User Profile Service ke System Architect hain. Service par rolling upgrades chalte rehte hain. Interviewer aap se poochta hai:
*"Hum users ke table mein ek naya field `verificationBadge` add kar rahe hain. Rolling upgrade ke dauran purane nodes bhi chal rahe hain aur naye bhi. Hum kaise ensure karein ke purane nodes jab kisi profile ko update karein, toh naya `verificationBadge` field delete na ho, jaisa ke image_af849c.png ke case mein hota hai?"*

### Architectural Design Implementation

Hum system ko is tarah design karenge ke hamara data access layer unknown fields ko preserve kare, ya hum ek schema-driven binary format use karenge. Niche iska conceptual architectural flow diya gaya hai:

```plaintext
                               +-----------------------------------+
                               |        Load Balancer              |
                               +-----------------------------------+
                                         /               \
                                        /                 \
                                       v                   v
                        +--------------------+       +--------------------+
                        |  Old App Node (v1) |       |  New App Node (v2) |
                        +--------------------+       +--------------------+
                                       \                   /
                         (Read/Write)   \                 / (Read/Write)
                                         v               v
                               +-----------------------------------+
                               |       Distributed Database        |
                               +-----------------------------------+
                               | Record: [Known Fields]            |
                               |         [Unknown_Fields_Map] <----+--- Stores "verificationBadge"
                               +-----------------------------------+

```

### Comprehensive Architectural Explanation

1. **Unknown Fields Preservation Strategy:**
Agar hum JSON use kar rahe hain, toh hum purane code (v1) ke parser ko configure karenge ke wo kisi bhi unmapped field (jaise `verificationBadge`) ko drop karne ke bajaye ek fallback generic map (`private Map<String, Object> unknownFields`) mein store kar le.
2. **Data Flow Control:**
* **Step A:** New Node (v2) database mein user record write karta hai jisme `verificationBadge: "gold"` shamil hai.
* **Step B:** Old Node (v1) us record ko read karta hai. JSON parser `userName` aur dusre purane fields ko unke variables mein dalta hai, lekin `verificationBadge` ko dekh kar use `unknownFields` ke map mein save kar leta hai.
* **Step C:** Old Node jab `favoriteNumber` update karke data dubara serialize karega, toh serialization logic `unknownFields` map ke data ko bhi wapas JSON string mein merge karega.


3. **Alternative Solution (Binary Formats):**
Interviewer ko batayein ke real-world production mein hum Protocol Buffers ya Avro jaise formats use karte hain, jahan har field ka ek unique tag/id hota hai. Purana code naye tag numbers ko decode karte waqt automatically binary buffer mein preserve rakhta hai aur bina chhede wapas database mein write kar deta hai, jis se data loss ka khatra zero ho jata hai.

---