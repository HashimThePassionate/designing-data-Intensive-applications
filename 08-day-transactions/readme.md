# Transactions

## James Corbett et al., “Spanner: Google’s Globally-Distributed Database” (2012)

Writer ne chapter ka aghaaz Google ki aik mashhoor research paper ke quote se kiya hai. Is quote mein aik bohot bara design decision aur trade-off samjhaya gaya hai:

* **Mushkil Baat:** Kuch authors (developers/researchers) ka manna hai ke **Two-Phase Commit (2PC)**—jo ke aik aisa tarika hai jis se bohot saari alag alag machines par chalne wale database ko aik sath update kiya jata hai—wo bohot mehanga (expensive) parta hai. Mehanga paison ke lihaz se nahi, balke **performance** (speed slow ho jati hai) aur **availability** (agar aik machine ruk jaye to poora system ruk sakta hai) ke maslon ki wajah se.
* **Google ka Point of View:** Google ke engineers kehte hain ke humein is baat par yakeen hai ke database mein transactions ka hona zaroori hai. Agar transactions nahi hogi, to application banane wale programmers ko har waqt darr rahega ke kahin data kharab na ho jaye, aur unhein khud bohot complex code likhna parega. Is se behtar hai ke database transactions ka feature de, aur jab performance ka masla (bottleneck) aaye, to programmers usay baad mein hal kar lein.

> **Bacchon ki Tarah Asaan Samjhein:** Socho aap aik complex lego house bana rahe ho. Agar aap ko har aik block lagane ke baad darr ho ke poora ghar gir sakta hai aur aap ko khud hi safety ropes lagani parein, to aap thak jaoge. Lekin agar aap ko aik jadoo ka glue (glue gun) de diya jaye jo poore ghar ko sambhal le, bhale hi glue lagane mein thoda waqt (performance cost) lage, aap sukoon se ghar bana sakoge. Google bhi yahi keh raha hai ke jadoo ka glue (transactions) hona chahiye!

---

### In the harsh reality of data systems, many things can go wrong

Asli duniya mein computer systems bilkul perfect nahi hote. Writer kehta hai ke jab hum data ke sath kaam karte hain, to bohot saari cheezein aik sath kharab ho sakti hain. Unhon ne in faults (ghaltiyon) ki aik list di hai:

* **Database Software ya Hardware Failures:** Database chalte chalte achanak band ho sakta hai, ya jis computer (hard drive) par data save ho raha hai wo kharab ho sakti hai. Yeh achanak tab bhi ho sakta hai jab database abhi data **write (save)** kar raha ho.
* **Application Crashes:** Jo software (App) database ko use kar raha hai, wo achanak crash ho sakta hai. Farz karein app ne 5 kaam karne thay, aur wo 2 kaam karne ke baad beech mein hi band ho gaya.
* **Network Interruptions:** Internet ya network achanak kat sakta hai. Is se app ka database se raabta toot sakta hai, ya agar database ki do alag alag machines (nodes) hain, to un ka aapsi raabta toot sakta hai.
* **Concurrent Writes:** Aik hi waqt mein aik se zyada users (clients) database mein aik hi jagah par tabdeeli (write) karne ki koshish kar sakte hain. Is se aik user ka data doosre ke data ke upar overwrite (mita) ho sakta hai.
* **Partial Updates (Adhoori Reads):** Aik client aisa data read (parh) sakta hai jo abhi poora badla hi nahi tha (aadha badla tha aur aadha purana tha), jis se data ka koi matlab hi nahi banta.
* **Race Conditions:** Jab do ya do se zyada kaam aik sath aage peeche hote hain, to ajeeb o gareeb kism ke bugs (bimarayan) peda ho jati hain jinhein pakadna mushkil hota hai.

> **Bacchon ki Tarah Asaan Samjhein:** Socho aap aur aap ka dost aik hi drawing paper par aik sath rang bhar rahe ho. Aap chahte ho ke wahan 'Lal' rang ho aur dost chahte hai ke 'Neela' rang ho. Agar aap dono aik dosre ka hath rokay bagair rang bharne lagoge, to paper phat jayega ya rang kharab ho jayega (Concurrent Writes). Ya phir socho aap aik kahani parh rahe ho aur koi beech mein se adhay pahnay phad de (Partial Updates/Crashes). Systems mein bhi yahi sab hota hai.

---

### Transactions as the mechanism of choice

In saari mushkilaat aur kharabiyon se bachne ke liye, pichle kayi saalon se **Transactions** ko sab se behtareen hathiyar (mechanism) mana gaya hai.

* **Transaction Kya Hai?** Yeh aik tarika hai jahan application apne bohot saare reads (data parhna) aur writes (data likhna) ko aik sath jama kar ke **aik logical unit (ek dabba)** bana deti hai.
* **All-or-Nothing Concept:** Concept yeh hai ke is dabbe ke andar jitne bhi kaam hain, wo ya to saare ke saare kamyab honge (**Commit**), ya phir un mein se aik bhi nahi hoga (**Abort** ya **Rollback**). Agar beech mein koi bhi masla aya, to poora operation fail ho jayega aur system aisa ban jayega jaise kuch hua hi nahi tha.
* **Error Handling Bani Asaan:** Is feature ki wajah se application banane wale ko **partial failures** (aadha kaam hua, aadha nahi hua) ki fikar nahi karni parti. Agar transaction fail ho jaye, to app sukoon se usay dubara shuru (**retry**) kar sakti hai.

> **Bacchon ki Tarah Asaan Samjhein:** Socho aap dukan dar ko 100 rupay dete ho aik chocolate aur aik biscuit lene ke liye. Agar dukan dar chocolate de de aur biscuit khatam ho jaye, to aap adha samaan nahi lete. Aap kehte ho "Mera poora soda do ya mere 100 rupay wapas karo". Transactions bhi yahi hain: "Ya to saara kaam karo (Commit), ya kuch bhi mat karo aur pehle jaisa kar do (Abort)".

---

### The purpose of transactions

Writer yahan aik bohot gahri theoretical baat samjhate hain ke transactions koi qudrati qanoon (law of nature) nahi hain ke in ka hona zaroori hi ho.

* **Insani Ijaad:** Inhein insano ne aik khas maqsad ke liye banaya tha: **Programming model ko asaan banana**.
* **Safety Guarantees:** Jab aap transactions use karte ho, to database aap ko kuch kasmein (guarantees) deta hai. Database kehta hai ke "Tum fikar mat karo, network katay ya aag lage, data ko sambhalna meri zimmedari hai". Is wajah se application developer bohot saare ajeeb o gareeb errors aur concurrency (aik sath kaam hone wale) maslon ko ignore kar sakta hai.

---

### Trade-offs and the Post Office Horizon Scandal

Yahan writer aik design decision aur real-world misal dete hain:

* **Weakening Transactional Guarantees:** Har application ko transactions ki sakht zaroorat nahi hoti. Kabhi kabhi hum jaan boojh kar transactions ko kamzoor kar dete hain ya chor dete hain taake humein zyada speed (**performance**) mile ya hamara system har waqt chalta rahe (**higher availability**).
* **Real-world Disaster (Post Office Horizon Scandal):** Writer ne aik haqeeqi misal di hai ke transactions na use karne ka anjaam kitna bura ho sakta hai. UK ke Post Office mein aik accounting system (Horizon) use hota tha. Us software ke peeche jo database tha, us mein **ACID transactions** (safety guarantees) missing thin. Natija kya nikla? System mein mathematical ghaltiyan (bugs) aane lagin, paise gayab dikhne lage, aur un be-kasoor postmasters (mulaazmeen) par chori ka ilzam lag gaya.

---

### How do you figure out whether you need transactions?

Aap ko kaise pata chalega ke aap ko transaction chahiye ya nahi? Is ka jawab janne ke liye pehle humein un safety guarantees aur un ke sath aane wali keemat (**costs/performance trade-offs**) ko samajhna hoga.

Writer batate hain ke **Concurrency Control** (aik sath hone wale kaamon ko sambhalna) dono jagah zaroori hai:

1. **Single-node Databases:** Jo aik hi computer par chalte hain.
2. **Distributed Databases:** Jo bohot saari machines par phailay hote hain.

Is chapter mein agay hum inhi cheezon ko gehrai mein parhenge:

* **Race Conditions:** Jo ajeeb bugs peda karti hain.
* **Isolation Levels:** Database mein alag alag levels ki security/safety hoti hai jaise **read-committed**, **snapshot isolation**, aur **serializability**. (Inhein hum agay bacchon ki tarah asaan kar ke samjhenge).
* **Two-Phase Commit (2PC):** Distributed databases mein har machine par aik sath sab sahi se save (atomicity) karne ka protocol.

---

