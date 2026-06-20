# Doing the Right Thing

Feeding AI systems on the world’s beauty, ugliness, and cruelty, but expecting it to reflect only the beauty is a fantasy.
—Vinay Uday Prabhu and Abeba Birhane, “Large Datasets: A Pyrrhic Win for Computer Vision?” (2020)

In quotes ka aasan matlab yeh hai ke agar aap AI computer systems ko dunya ki saari achai, burai, aur na-insafi (data ki shakal mein) khilayenge, aur phir yeh umeed rakhenge ke computer sirf achi aur khubsurat cheezain hi dikhaye, toh yeh aap ka ek khwab ya fantasy hai. Computer wahi seekhta hai jo use dikhaya jata hai.

Is kitab ke aakhri chapter mein, aain thora peeche hat kar poori tasveer ko dekhte hain. Puri kitab mein hum ne data systems ke bohot saray designs (architectures) ko detail mein dekha, unke faide aur nuksaan (pros and cons) ko samjha, aur aisi applications banana seekha jo mazboot (reliable), barhi ki ja sakay (scalable), aur lambe arsay tak chal sakein (maintainable). Lekin hum ne is poori bahas ka ek bohot hi buniyadi aur zaroori hissa chor diya tha, jise ab poora karna lazmi hai.

Har system kisi na kisi maqsad ke liye banaya jata hai; hamare har action ke kuch aise asraat hote hain jo hum chahte hain (intended) aur kuch aise bhi hote hain jo hum ne soche bhi nahi hote (unintended consequences). Kisi system ka maqsad shaayad sirf paisa kamana ho, lekin uske asraat bohot door tak ja sakte hain. Hum engineers, jo ye systems banate hain, hamari yeh zimmedari (responsibility) hai ke hum in asraat ko ghaur se dekhein aur yeh paka karein ke hamare faislo se kisi ko nuksaan na pahunche.

Hum data ke baare mein is tarah baat karte hain jaise woh koi be-jaan cheez (abstract thing) ho, lekin yaad rakhein ke zyada tar datasets asli jeete-jaagte logo ke baare mein hote hain: unka behavior (gussa, khushi, khareedari), unke interests (pasand-napasand), aur unki pehchan (identity). Hamein is data ko insaniyat aur izzat ke sath treat karna chahiye. Users bhi insaan hain, aur insani izzat (human dignity) sab se upar hai.

Software banane mein ab din-ba-din bohot bade ethical (ikhlaqi) faisle shaamil ho rahe hain. Engineers ki madad ke liye kuch guidelines bhi maujood hain, jaise ke **ACM Code of Ethics and Professional Conduct**, lekin in par aam taur par na toh baat hoti hai, na inhein apply kiya jata hai, aur na hi inhein sakhti se nafiz (enforce) kiya jata hai. Nateeja yeh nikalta hai ke software engineers aur product managers aksar logo ki privacy aur apne products ke bure asraat ko bilkul halka lete hain (cavalier attitude).

Koi bhi technology khud se achi ya buri nahi hoti—fark is baat se parta hai ke use kaise istemal kiya ja raha hai aur woh logo par kya asar daal rahi hai. Yeh baat ek software system (jaise search engine) par bhi bilkul waise hi sadiq aati hai jaise kisi hathiyar (jaise bandook) par aati hai. Ikhlaqi zimmedari (ethical responsibility) hamari apni hai; software engineers ke liye yeh kaafi nahi hai ke woh sirf coding aur technology par focus karein aur uske dunya par parne wale asraat ko bhool jayein.

Lekin computer ki baqi cheezon ke ult, ethics (ikhlaqiyat) ke concepts bilkul fix ya saaf nahi hote ke unka ek hi sahi jawab ho; inhein samajhne ke liye insani dimaag chahiye hota hai, aur har bande ki raye alag ho sakti hai (subjective). Kya cheez "achi" hai aur kya "buri", iski koi pakki definition nahi hai, aur computers ke professionals ke darmiyan is subject par koi serious bahas bhi nahi hoti.

Ethics par sochna aur faisle karna mushkil hai, lekin yeh itna zaroori hai ke isay ignore nahi kiya ja sakta. Iska matlab kya hai? Ethics ka matlab yeh nahi hai ke aap ek kagaz par check-list bana lein aur ticks lagate jayein ke "haan mein ne kanoon poora kar liya". Yeh ek aisa process hai jismein aap ko lagatar sochna parta hai, un logo se baat (dialogue) karni parti hai jin par iska asar par raha hai, aur jo nateeja nikle uski zimmedari (accountability) khud uthani parti hai.

---

## Predictive Analytics

Predictive analytics (data ki madad se mustaqbil ka andaza lagana) ek bohot bari wajah hai jiski wajah se log Big Data aur AI ko lekar itne excited hain. Lekin yeh ek aisa ilaqa bhi hai jahan sab se zyada ikhlaqi uljhanein (ethical dilemmas) paida hoti hain. Data analysis ke zariye mausam (weather) ka hal batana, ya kisi bimari ke phelne ka andaza lagana ek alag baat hai; lekin yeh andaza lagana ke:

1. Kya koi qaidi (convict) jail se chhootne ke baad dubara jurm karega?
2. Kya loan (udhaar) ke liye apply karne wala banda paise wapas nahi kar payega (default)?
3. Kya insurance lene wala customer aage chalkar bohot mehnge claims karega?

Yeh bilkul alag baat hai, kyunke in faislo ka seedha asar aam logo ki zindagiyon par parta hai.

Kudarati baat hai ke credit card networks chahte hain ke fraud transactions na hon, banks chahte hain ke unke paise na doobein, airlines chahte hain ke jahaz hijack na hon, aur companies chahte hain ke woh bure ya na-ehal logo ko naukri par na rakhein. Unke point of view se agar unhone ek ache customer ko "no" keh diya toh unka chota sa nuksaan hoga (missed business opportunity), lekin agar unhone kisi galat bande ko haan keh diya toh unka bohot bara nuksaan ho sakta hai. Isliye companies hamesha ahtiyat barat-ti hain: agar thora sa bhi shaq ho, toh woh "NO" keh dena behtar samajhti hain.

Lekin, jaise jaise algorithms (computer programs) ke zariye faisle karne ka ruan barhta ja raha hai, ek ajeeb musibat khari ho gayi hai. Agar kisi bande par computer program ne (chahe sahi ya galti se) "risky" hone ka thappa (label) laga diya, toh use har jagah se "NO" ka samna karna parega.

Us bande ko systematically har cheez se bahar nikal diya jata hai—na use naukri milti hai, na jahaz ka safar, na insurance, na rent par ghar, aur na hi bank ki sahuliyat. Yeh kisi insaan ki aazadi par itni bari pabandi hai ke isay **“Algorithmic Prison” (Algorithms ki Jail)** kaha gaya hai. Insani huqooq ka ahtaram karne wale mumalik mein kanoon ka usool hota hai ke "jab tak jurm sabit na ho, banda begunah hai" (presumed innocence). Lekin automated systems bina kisi saboot ke, bilkul hami taur par (arbitrarily) ek bande ko samaj se kaat kar rakh dete hain, aur us bande ke paas is faisle ke khilaf appeal karne ka rasta bhi nahi hota.

---

## Bias and Discrimination

Computer ke algorithms jo faisle karte hain, woh zaroori nahi ke insaan ke faislo se behtar hon ya badtar hon. Har insaan ke andar koi na koi bhedbhav ya pasand-napasand (bias) hoti hai, chahe woh usay khatam karne ki kitni hi koshish kyun na kare, aur yeh bhedbhav waqt ke sath pure culture ka hissa ban jata hai. Ek umeed yeh thi ke agar hum faisle insani jazbaat ke bajaye "data" par chor dein, toh nizam zyada fair (insaf pasand) ho jayega aur un logo ko bhi mauqa milega jinhein purane nizam mein peeche chor diya jata tha.

Lekin jab hum predictive analytics aur AI systems banate hain, toh hum computer ko khud se rules nahi sikhate ke "kab haan kehna hai aur kab naa". Hum software ko sirf data de dete hain aur computer un patterns ko khud bhaamp leta hai (inferred from data). Musibat yeh hai ke computer ne jo seekha, woh bilkul chupa huwa (opaque) hota hai—hamein yeh toh pata chal jata hai ke data mein do cheezon ka aapas mein talluq (correlation) hai, lekin yeh nahi pata hota ke kyun hai. Agar computer ko diye jaane wale data (input) ke andar pehle se hi koi bhedbhav (systematic bias) maujood hai, toh computer usay seekh lega aur apne jawab (output) mein us bhedbhav ko aur zyada barha (amplify) dega.

Bohot se mumalik ke kanoon mein yeh sakht pabandi hai ke aap kisi ke sath uski zaat (ethnicity), umer (age), gender, sexuality, mazoori (disability) ya aqeeday (beliefs) ki wajah se alag sulook nahi kar sakte (inhein protected traits kehte hain). Lekin kya ho agar computer aap ke baqi data se in protected traits ka andaza laga le?

> **Real-World Example:** Jin shehron ke ilaqon mein nassal (race) ke mutabaq log alag alag rehte hain (racially segregated neighborhoods), wahan kisi bande ka **Postal Code (Zip Code)** ya uska **IP address** hi yeh batane ke liye kafi hota hai ke uski nassal ya rangat kya hai.

Agar computer postal code ko dekh kar faisla kar raha hai, toh woh asal mein rang aur nassal par hi faisla kar raha hai. Yeh sochna bilkul ajeeb hai ke ek algorithm gande aur biased data ko input ke taur par lega aur jadoo se ek bilkul pak-saaf aur unbiased output nikal dega. Lekin data-driven decision making ke chahne wale aksar isi baat par yaqeen rakhte hain—is soch par tanz karte huwe ek mazaq banaya gaya hai:

$$\text{"Machine learning is like money laundering for bias."}$$

*(Yaani jaise money laundering mein kalay dhan ko safaid kiya jata hai, waise hi ML ke zariye samaj ke gande bhedbhav ko mathematical formula ka parda daal kar "saaf aur neutral" dikhaya jata hai).*

Predictive analytics systems sirf maazi (past) ko dekh kar mustaqbil ka andaza lagate hain; agar maazi bhedbhav aur na-insafi se bhara tha, toh computer us na-insafi ko kanoon bana kar aage barha dega. Agar hum chahte hain ke hamara mustaqbil maazi se behtar ho, toh hamein **Moral Imagination (ikhlaqi soch aur zameer)** ki zaroorat hai, aur yeh ek aisi cheez hai jo sirf insaano ke paas hai. Data aur models hamare auzar (tools) hone chahiye, hamare maalik (masters) nahi.

---

## Responsibility and Accountability

Automated decision making (computer ka khud faisle karna) ek bohot bara sawal khara karta hai: Zimmedari kis ki hai (Responsibility and Accountability)? Agar koi insaan galti karta hai, toh aap usay pakar sakte hain aur uske faisle ke khilaf adalat ya senior se rujooh (appeal) kar sakte hain. Algorithms bhi galtiyan karte hain, lekin jab woh galti karein toh zimmedar kaun hai?

* Jab ek self-driving car accident karti hai, toh zimmedari kis ki hai? Driver ki, car banane wali company ki, ya code likhne wale engineer ki?
* Agar credit scoring ka algorithm systematically kisi khass rang ya mazhab ke logo ko reject kar raha hai, toh nuksaan uthane wala banda kahan shikayat kare?
* Agar aap ke Machine Learning system ke faisle ko adalat (court) mein challenge kar diya jaye, toh kya aap judge ke samne khare hokar samja sakte hain ke algorithm ne yeh faisla kis buniyad par kiya?

Insaano ko is baat ki ijazat nahi honi chahiye ke woh apni zimmedari se jaan churane ke liye sara ilzam ek algorithm par daal dein.

**Credit Rating Agencies** iski ek purani misaal hain jo logo ka data jama karke unke baare mein faisle karti hain. Agar aap ka credit score bura ho toh zindagi mushkil ho jati hai, lekin kam az kam woh score aap ki apni loan lene ki asli tareekh (borrowing history) par banta hai. Agar usme koi galti ho, toh aap usay theek karwa sakte hain (chahe agencies yeh kaam kitna hi mushkil kyun na bana dein).

Lekin Machine Learning par chalne wale modern scoring algorithms hazaron qism ke ajeeb-o-ghareeb inputs ka istemal karte hain aur bilkul **opaque (kaalay dibbe/black box)** hote hain. Yeh samajhna takreeban namumkin hota hai ke computer ne kisi bande ko kyun reject kiya aur kya uske sath koi na-insafi hui hai.

| Purana System (Traditional Credit Score) | Modern ML Prediction System |
| --- | --- |
| Sawal poochta hai: **"Aap ne maazi mein kaisa behave kiya?"** | Sawal poochta hai: **"Aap ke jaise doosre log kaun hain, aur unhone maazi mein kaisa behave kiya?"** |
| Asli facts aur aap ki apni history par chalta hai. | Insaano ko ek hi khate mein daalta hai (**Stereotyping**), jaise ke rehaish ki jagah (jo rang aur amiri/ghareebi ka ishara hoti hai). |
| Galti theek karne ka koi na koi rasta hota hai. | Agar ghalat data ki wajah se faisla ghalat ho jaye, toh rujooh (recourse) karna takreeban namumkin hota hai. |

Zyada tar data statistical (shumariyati) hota hai. Iska matlab yeh hai ke agar poori aabadi (population) par computer ka andaza bilkul theek bhi baith raha ho, tab bhi kisi ek akele bande (individual case) par woh andaza bilkul galat ho sakta hai.

> **Simple Example:** Agar aap ke mulk mein insaan ki average umer 80 saal hai, toh iska matlab yeh hargiz nahi hai ke har banda theek apni 80th birthday par mar jayega. Is average se aap kisi ek khass bande ki maut ka din nahi bata sakte. Bilkul isi tarah, prediction system ka jawab sirf ek probability (imkan) hota hai aur akele cases mein woh bilkul galat ho sakta hai.

Data ko aankhein band karke khuda samajh lena na sirf ek deewangi hai balki sakht khatarnak bhi hai. Jaise jaise data-driven faisle barhenge, hamein yeh seekhna hoga ke maujooda bhedbhav ko dubara pakka hone se kaise rokein, algorithms ko kaise transparent (saaf) aur jawabdeh banayein, aur jab woh lazmi taur par galtiyan karein toh unhein theek kaise karein.

Hamein yeh bhi dekhna hoga ke data ki taqat ko ache kaamo ke liye kaise istemal karein aur logo ko nuksaan se kaise bachayein. Misal ke taur par, data analysis se hamein logo ki maashi aur samaji zindagiyon ka pata chalta hai. Is taqat ka ek faida yeh ho sakta hai ke hum un logo tak madad (aid) pahunchayein jinhein iski sab se zyada zaroorat hai. Lekin iska ek bura istemal yeh bhi hota hai ke predatory businesses (shikari karobar) is data ke zariye majboor aur kamzoor logo ko dhoondte hain aur unhein dhokay se mehnge bad-loans ya bekaar college degrees bech dete hain.

---

## Feedback Loops

Agar hum aisi applications ko dekhein jinka asar seedha zindagi aur maut par nahi parta—jaise ke **Recommendation Systems** (jo aap ko YouTube ya Netflix par agli video suggest karte hain)—wahan bhi bade mushkil masail hote hain. Jab yeh systems yeh seekh jaate hain ke user kya dekhna chahta hai, toh woh use sirf wahi cheezain dikhana shuru kar dete hain jin se user pehle se hi agree karta ho.

Is se samaj mein **Echo Chambers** (ek hi aawaz ka goonjna) paida hote hain, jahan purane bhedbhav, jhooti afwahain (misinformation), aur aapas ki nafrat (polarization) parwan चढ़ti hain. Hum aaj kal social media ke inhi echo chambers ka asar election campaigns par saaf dekh rahe hain.

Jab predictive analytics aam logo ki zindagiyon par asar daalti hai, toh sab se khofnak masla **Self-reinforcing Feedback Loops** (khud ko taqatwar banane wali daldal) ki wajah se paida hota hai.

> **The Downward Spiral Example:** Farz karein koi employer (naukri dene wala) nayi bharti ke liye logon ka credit score check karta hai. Aap ek bohot ache aur mehnati worker hain, lekin achanak aap par koi aisi musibat aa jati hai jo aap ke ikhtiyar mein nahi thi (jaise sakht bimari). Bills na dene ki wajah se aap ka credit score gir jata hai. Ab jab aap nayi naukri dhoondne nikalte hain, toh computer algorithm aap ka kharab credit score dekh kar aap ko reject kar deta hai. Naukri na milne ki wajah se aap mazeed ghareeb ho jate hain, baqi bills bhi nahi de paate, aur aap ka credit score mazeed barbad ho jata hai. Ab naukri milna aur mushkil ho gaya hai.

Yeh ek aisi khofnak daldal (downward spiral) hai jo galat assumptions par khari hoti hai, lekin isay maths aur data ke khubsurat parde ke peeche chupa diya jata hai.

> **Another Example:** Economists (mahireen-e-maeshat) ne dekha ke jab Germany mein gas stations (petrol pumps) ne prices tay karne ke liye algorithms ka istemal shuru kiya, toh market mein competition khatam ho gaya. Algorithms ne chalaki se aapas mein milna (collude karna) seekh liya aur aam consumers ke liye petrol mehnga kar diya.

Hum hamesha pehle se andaza nahi laga sakte ke aise feedback loops kab paida ho jayein. Lekin, agar hum poore nizam ko ek sath dekhein—sirf computer ke code ko nahi, balki us se jure tamam insaano aur unke aapas ke talluqaat ko bhi—toh bohot se asraat ka andaza lagaya ja sakta hai. Is soch ko **Systems Thinking** kehte hain. Hum yeh samajhne ki koshish kar sakte hain ke hamara data analysis system alag alag behaviors par kaisa react karta hai. Kya hamara system amir ko mazeed amir aur ghareeb ko mazeed ghareeb bana kar na-insafi ko barha raha hai, ya woh is na-insafi ke khilaf larr raha hai? Hamein hamesha ache maqsad ke bawajood unintended consequences (un-chahe asraat) se hoshiyar rehna chahiye.

---

## Privacy and Tracking

Predictive analytics (yaani data ki madad se logon ke baare mein automated faisle karna) ke maslo ke alawa, khud **data collection (data jama karne)** ke andar bhi bohot bade ikhlaqi (ethical) masle chupe huwe hain. Hamein yeh dekhna hoga ke data jama karne wali companies aur un aam logon ke darmiyan asal rishta kya hai jinka data liya ja raha hai.

* **Jab User Customer Hota Hai:** Jab ek system sirf wahi data store karta hai jo user ne khud apni marzi se enter kiya ho (jaise koi note likhna ya file upload karna) kyunke user chahta hai ke system us par kaam kare, toh system user ke liye ek khidmat (service) kar raha hota hai. Yahan user hi asli **Customer (grahak)** hota hai.
* **Jab User Product Ban Jata Hai:** Lekin jab user ki har ek activity (har click, scroll, aur screen par rukna) ko chupke se track aur log kiya jata hai, toh rishta bilkul badal jata hai. Ab system sirf woh nahi karta jo user use kehta hai, balki system ke apne faide (**interests**) paida ho jate hain, jo aksar user ke faide ke khilaf hote hain.

Online services ke aam features ko behtar banane ke liye logon ke behavior ka data track karna bohot zaroori ho gaya hai. Misal ke taur par:

* Yeh track karna ke logon ne kis search result par click kiya, is se search rankings behtar hoti hain.
* Recommendations dena (jaise "jin logon ko X pasand aaya, unhein Y bhi pasand aaya") users ko nayi kaam ki cheezain dhoondne mein madad deta hai.
* **A/B tests** aur user flow analysis se UI (user interface) ko mazeed asaan banaya jata hai.

In sab ka fayda aam user ko hi hota hai. Lekin, company ka business model kya hai? Baat yahan rukti nahi hai. Agar kisi service ki kamai ka zariya **Advertisements (Ishtiharat)** hain, toh us company ke asli customers woh advertisers hain, jabke aam users ke faide ko doosre nambar par rakh diya jata hai. Tracking mazeed barh jati hai, analysis gehri ho jati hai, aur data ko saalon saal sambhal kar rakha jata hai taake har ek bande ki poori profile banayi ja sakay aur unhein targeted cheezain bechi ja sakein.

Ab company aur user ka rishta bilkul ajeeb ho jata hai. User ko free service ka lalach de kar app par zyada se zyada waqt guzarne par majboor kiya jata hai. Yeh tracking ab user ke liye nahi, balki un advertisers ke liye ho rahi hoti hai jo paisa de rahe hain. Is rishte ke liye ek bohot hi khofnak lafz istemal kiya ja sakta hai: **Surveillance (Jasoosi ya Khufiya Nigrani)**.

---

## Surveillance

Ain ek chota sa **Thought Experiment (zehni mashq)** karte hain. Agar hum rozmarra ki baaton mein lafz "Data" ko hata kar uski jagah lafz "Surveillance" (jasoosi) rakh dein, toh dekhein kya woh jumlay ab bhi sunne mein ache lagte hain?

> "Hamari **surveillance-driven** (jasoosi par chalne wali) organization mein hum real-time **surveillance streams** jama karte hain aur unhein apne **surveillance warehouse** mein store karte hain. Hamare **surveillance scientists** advanced analytics aur **surveillance processing** ka istemal karte hain taake naye raaz nikal sakein."

Yeh sochna is kitab ke naam—*Designing Surveillance-Intensive Applications*—ke liye shaayad thora ajeeb lage, lekin is point par zor dene ke liye ye sakht alfaaz zaroori hain. Software ki madad se dunya par qabza karne ke chakar mein hum ne dunya ka sab se bara mass surveillance (har bande ki jasoosi karne wala) infrastructure khara kar diya hai. Hum tezi se ek aisi dunya ki taraf barh rahe hain jahan har rehne wali jagah mein kam az kam ek internet se jura huwa microphone maujood hai—chahe woh smartphones hon, smart TVs hon, voice assistants hon, baby monitors hon, ya bachon ke woh khilone hon jo cloud-based speech recognition use karte hain. Aur in tamam devices ka security record nihayat hi ganda aur kamzoor raha hai.

Purane zamane ke muqabla mein naya yeh huwa hai ke **Digitization (har cheez ka computer par aana)** ne logon ka barha data jama karna bohot asaan bana diya hai. Hamari location (jagah), hamare aapas ke rishte, hamari baatein, hamari khareedari, hamari payments, aur hamari health ka data track hona ab na-guzer (unavoidable) ho chuka hai. Ek jasoosi karne wali organization aage chal kar ek aam insaan ke baare mein us se khud se zyada jaan sakti hai—jaise kisi bimari ya maashi (economic) masle ka andaza bande ko khud pata chalne se pehle laga lena.

Tareekh ke baray se baray zalim aur dictator (totalitarian regimes) bhi sirf ye khwab hi dekh sakte the ke woh har kamre mein ek mic laga dein aur har bande ko har waqt location track karne wali device uthane par majboor kar dein. Lekin digital technology ke faide itne zyada hain ke aaj hum khud apni khushi se (voluntarily) is poori jasoosi ki dunya ko qabool kar chuke hain. Farq sirf itna hai ke yeh data koi hukumat control ke liye nahi le rahi, balki bari corporations hamein sahuliyat dene ke naam par jama kar rahi hain.

Hum corporations ki is jasoosi ko khushi khushi kyun qabool karte hain? Shaayad aap sochte hon ke "Maine kaun sa kuch chupana hai" (nothing to hide)—yani aap mojooda nizam ke bilkul mutabaq chal rahe hain, aap koi mazloom minority nahi hain, aur aap ko pakray jaane ka darr nahi hai. Lekin har banda itna khush-qismat nahi hota. Ya shaayad aap ko lagta hai ke iska maqsad bohot shareefana hai—yeh koi zabardasti toh nahi kar rahe, sirf achi recommendations aur personalized marketing hi toh de rahe hain. Lekin jab hum predictive analytics ko iske sath milate hain, toh yeh farq khatam ho jata hai.

* **Gaariyon ki Tracking:** Aaj kal gaariyan drivers ki ijazat ke bagair unka driving behavior track karti hain, jiska asar unke insurance premiums (insurance ki qeemat) par parta hai.
* **Health Tracker:** Health insurance companies logon par fitness tracking bands pehn-ne ke liye zor dalti hain.
* **Smartwatch Sensors:** Data analysis itna khatarnak ho chuka hai ke smartwatch ya fitness tracker ke andar laga huwa movement sensor (jo aap ke haath ki harqat naapta hai) yeh tak bilkul theek accuracy se andaza laga sakta hai ke aap keyboard par kya password type kar rahe hain.

Waqt ke sath sensors aur algorithms mazeed tez hote chale jayein ge.

---

## Consent and Freedom of Choice

Hum yeh keh sakte hain ke users apni marzi se in services ko use karte hain, unki **Terms of Service (ToS)** aur privacy policy se agree karte hain, aur data collection ki ijazat (consent) dete hain. Hum yeh bhi keh sakte hain ke data ke badle unhein ek qeemti service mil rahi hai aur service chalane ke liye tracking zaroori hai. No-doubt, social networks aur search engines users ke liye bohot qeemti hain—lekin is argument mein bohot bade masle hain:

1. **Kya tracking waqai zaroori hai?** Kuch tracking features ko behtar karne ke liye zaroori hoti hai (jaise clicks dekh kar search ranking improve karna ya products ki suggestion dena). Lekin content recommendation ke naam par ya ads ke liye logon ki poori profile banana genuinely user ke faide mein nahi hota. Yeh tracking sirf isliye zaroori hai kyunke ads se company ka kharcha nikalta hai, user ke liye iski koi zaroorat nahi hoti.
2. **Users bilkul be-khabar hain:** Zyada tar users ko pata hi nahi hota ke woh databases ko kya data khila rahe hain aur woh kaise process ho raha hai. Privacy policies hamesha aisi mushkil zuban mein likhi jati hain jo sach ko chupane ka kaam karti hain. Iske alawa, ek user ka data un logon ke baare mein bhi bohot kuch bata deta hai jo us service ke user hain hi nahi aur jinhone kabhi kisi policy ko accept nahi kiya. Kal ko jab saare users ka behavioral data aur external sources aapas mein milaye jate hain (**Derived Datasets**), toh aam banda unhein samajh hi nahi sakta.
3. **One-Way Process (Ek Tarfa Nizam):** Data nikalne ka yeh nizam ek tarfa hai, isme koi barabari ya aapas ka mol-tol (negotiation) nahi hota. User ke paas yeh option nahi hota ke woh company se kahe ke "Main itna data doonga aur badle mein mujhe yeh service do". Rules hamesha company set karti hai, rishta bilkul **Asymmetric (gair-mutwazin)** hota hai.

European Union mein **GDPR (General Data Protection Regulation)** ne isko theek karne ke liye sakht rules banaye hain:

* Consent hamesha "freely given (azaad), specific, informed (ba-khabar), aur unambiguous (saaf)" hona chahiye.
* User bina kisi nuksaan ya saza ke apni ijazat wapas (withdraw) le sakay.
* Ijazat lene ka tareeqay bilkul aasan aur saaf zuban mein likha ho.
* Khamoshi, pehle se ticked boxes, ya user ka koi action na lena consent nahi mana jayega.

*(GDPR mein data process karne ki aur bhi buniyadein hain, jaise kanoon par amal karna, kisi ki jaan bachana, ya fraud prevention ke liye legitimate interest use karna, jismein fraud karne wale ki ijazat zaroori nahi hoti).*

Agar koi kahe ke "jis user ko jasoosi pasand nahi, woh service use na kare", toh yeh choice bhi azaad nahi hai. Jab koi service itni mashhoor ho jaye ke samaj ka hissa banne ke liye woh sab ke liye zaroori ho, toh usay chorna na-mumkin ho jata hai. Jab kisi service ke **Network Effects** hote hain (jaise sab ka WhatsApp ya Google par hona), toh usay na use karne ki ek bohot bari samaji qeemat (social cost) hoti hai.

In platforms ko jaan-boojh kar aisa design kiya jata hai ke log inke aadi (addict) ho jayein, bilkul casino mein gambling (jua) ki tarah. Is nizam se door rehna sirf un chand ameer ya privileged (kush-haal) logon ke bas ki baat hai jin ke paas itna waqt aur ilm ho ke woh privacy policies ko samjhein aur jo samaji ya karobari nuqsaan uthana afford kar sakein. Kam-khushhaal logon ke liye koi real choice nahi hoti; unke liye surveillance se bachna namumkin ban jata hai.

---

## Privacy and Use of Data

Kuch log kehte hain ke "Privacy ab mar chuki hai" kyunke log khud hi social media par apni personal zindagiyon ke baare mein sab kuch post kar dete hain. Lekin yeh soch bilkul ghalat hai aur lafz **Privacy** ko na samajhne ki wajah se paida hoti hai.

> **Privacy ki Asli Definition:** Privacy ka matlab sab kuch chupa kar rakhna nahi hai; iska matlab hai **Azaadi (Freedom of Choice)** ke aap khud faisla karein ke kis ko kya dikhana hai, kya public karna hai, aur kya secret rakhna hai. Right to privacy asal mein ek **Decision Right (faisla karne ka haq)** hai, jo har insaan ko apni situation ke mutabaq transparency aur secrecy ke darmiyan chunne ki azaadi deta hai.

| Scenario | Data Share Karne Ki Wajah | Privacy Ka Haq |
| --- | --- | --- |
| **Medical Research** | Ek rare bimari ka mareez researchers ko apna private data khushi se dega taake elaj dhoonda ja sakay. | Haq maujood hai kyunke user ne khud maqsad chunna hai. |
| **Insurance / Job** | Agar wahi data insurance company ya employer ko mil jaye toh mareez ki naukri ya insurance khatam ho sakti hai. | Haq chinn jata hai agar data unke haath lag jaye. |

Jab data surveillance infrastructure ke zariye nikalwaya jata hai, toh privacy rights khatam nahi hote balki user se chin kar data jama karne wali company ke paas transfer ho jate hain. Companies kehti hain "Hum par bharosa rakho", jiska matlab hai ke faisla karne ka haq ab aap ka nahi, company ka hai.

Companies is data ke nateejon ko dunya se chupa kar rakhti hain kyunke agar woh sab samne le aayein toh log darr jayein ge (creepy lagega) aur unka business model tabah ho jayega (jo is baat par chalta hai ke unhein doosron se zyada pata hai). Users ki personal baatein indirect tareeqe se ads target karne ke liye use hoti hain. Faisla user nahi karta, balki company apne profit ko barhane ke liye karti hai.

Bohot si companies sirf user ki **Perception (soch)** ko manage karti hain ke unhein bura na lage, bajaye iske ke woh data collection kam karein. Aur yeh management bhi aksar gandi hoti hai—jaise kisi user ko unki kisi dard-naak maazi ki yaad dila dena jo factual toh theek ho lekin user dekhna na chahta ho. Data ke mamle mein galti ka imkan hamesha rehta hai, aur as an engineer hamein humble (aajiz) hona chahiye aur algorithms mein insani zarooraton ka khayal rakhna chahiye. Privacy settings sirf doosre users se data chupati hain, service ke paas toh mukammal (**unfettered**) access rehta hi hai aur woh privacy policy ke tehat kuch bhi kar sakti hai.

Individuals se corporations ki taraf privacy rights ka itna bara transfer tareekh mein pehle kabhi nahi huwa. Jasoosi pehle bhi hoti تھی۔ Lekin woh mehngi aur manual hoti thi, scalable aur automated nahi thi. Doctor aur mareez, ya wakeel aur client ke darmiyan bhi bharosa hota hai, lekin wahan kanoon aur ethics ki sakht pabandiyan hoti hain. Internet services ne bina kisi meaningful consent ke sensitive data ka pahar jama karna bohot asaan bana diya hai.

---

## Data as Assets and Power

Chunke behavioral data user ke app use karne se khud-ba-khud banta hai, isliye kuch log isay **"Data Exhaust" (data ka dhuan/kachra)** kehte hain, jaise yeh koi be-kaar waste material ho jise recycle karke faida nikala ja raha hai.

Lekin economic point of view se asliyat iske bilkul ult hai:

* Agar targeted ads hi service ka kharcha utha rahe hain, toh user ki activity asal mein ek qism ki **Labor (mazdoori)** hai.
* Hum yeh bhi keh sakte hain ke yeh saari applications sirf ek lalach (**lure**) hain taake aam logon ko jasoosi ke network (surveillance infrastructure) mein phansa kar unka personal data nikala ja sakay. Insani creativity aur rishton ko cynically exploit kiya jata hai.

Personal data ek bohot qeemti asset (asasa) hai. Chup kar kaam karne wale **Data Brokers** logon ka data khareedte, jama karte, aur bechte hain. Startups ki qeemat unke users ke numbers yani unki jasoosi karne ki taqat (**surveillance capabilities**) par lagayi jati hai.

Kyunke data qeemti hai, isliye har koi isay chahta hai—companies bhi aur hukumat (governments) bhi, jo isay khufiya deals, kanoon, ya chori ke zariye haasil karti hain. Jab koi company bankrupt (diwalia) hoti hai, toh uska data asset ke taur par bikta hai. Aur data ko secure rakhna itna mushkil hai ke data breaches (chori) aam baat ban chuki hai.

Isi wajah se critics kehte hain ke data sirf ek asset nahi balki ek **"Toxic Asset" (zehreli cheez)** ya khatarnak material hai, bilkul **Uranium** ki tarah. Agar hum iske galat istemal ko rok bhi lein, tab bhi data jama karne ke faide aur iske galat haathon mein jaane ke risk ko balance karna lazmi hai. Kal ko criminals, dushman mulk ki intelligence, ya koi aisi bad-unwan management company par qabza kar sakti hai jo hamari values ko nahi manti. Hukumat badal sakti hai aur koi aisi dictator hukumat aa sakti hai jo is data ke zariye police state bana le. Bruce Schneier kehta hai:

> "Aisi technologies lagana jo kal ko kisi police state (zalim hukumat) ke kaam aa sakein, nihayat hi gandi civic hygiene (samaji safai) hai."

"Ilm hi taqat hai" (Knowledge is power). Aur sab se bari taqat yeh hai ke aap doosron par nazar rakhein lekin khud par nazar na parne dein. Tech companies aaj overt political power nahi maang rahi hain, lekin unke paas jo data ki taqat hai, woh hamari zindagiyon par bohot bara control rakhti hai.

---

## Remembering the Industrial Revolution

Data is **Information Age (Information ke dor)** ki sab se bari pehchan hai. Internet aur software hamari dunya ko badal rahe hain aur aane wale saalon mein bhi badalte rahein ge. Iska muwazna hum **Industrial Revolution (Sanati Inqilab)** se kar sakte hain.

Industrial Revolution se bohot taraqqi hui, economy barhi, living standards ache huwe—lekin bohot bade masle bhi paida huwe. Hawa aur pani ki pollution (aaloodgi) khofnak thi. Factory ke maalik aish karte the jabke mazdoor gande aur chote gharon mein sakht halat mein ghanton kaam karte the. Child labor (bachon se mazdoori) aam thi.

Saza aur hifazat ke rules (safeguards) banne mein bohot waqt laga—jaise environmental protection regulations, workplace safety protocols, child labor ke khilaf kanoon, aur khane-peene ki cheezon ki checking. Jab factories par kanoon laga ke woh kachra daryon mein nahi phenk sakti, toh business ka kharcha zaroor barha, lekin poore samaj ko bohot faida huwa. Aaj koi bhi us purane gande dor mein wapas nahi jana chahta.

Bilkul isi tarah, hamare Information Age ka bhi ek kaala hissa (dark side) hai jise control karna lazmi hai. Data ka collection aur misuse wahi bara masla hai. Bruce Schneier ke alfaaz mein:

> "Data is information age ka **pollution (kachra/aaloodgi)** hai, aur privacy ki hifazat hamara environmental challenge hai. Halanki aaj hum purane dor ki pollution ko dekh kar hairan hote hain ke hamare baron ne isay kyun ignore kiya, hamare pote-potiyan (grandchildren) hamein is baat par judge karenge ke hum ne data collection aur iske misuse ke challenge ko kaise handle kiya. Hamein unhein proud feel karwana chahiye."

---

## Legislation and Self-Regulation

Data protection laws shayad logon ke huqooq ko bacha sakein. Jaise GDPR kehta hai ke personal data sirf kisi makhsoos aur legitimate (kanooni) maqsad ke liye hi jama kiya jaye aur uske alawa kahin use na ho, aur data kam se kam (**Data Minimization**) ho.

Lekin yeh principle big data ke falsafa ke bilkul khilaf chalta hai. Big data ka maqsad hi yeh hai ke zyada se zyada data jama karo, datasets ko aapas mein milao, aur experiments kar ke naye insights nikalon. Exploration ka matlab hi naye maqsad hain jo pehle se pata nahi hote. Halanki is regulation ka online advertising par thora asar huwa hai, lekin enforcement kamzoor rahi hai aur tech industry ke culture mein koi bara badlao nahi aaya.

Bari companies regulations ki mukhalfat karti hain ke is se innovation rukegi, aur kuch hadd tak yeh baat theek bhi hai. Misal ke taur par, medical data share karne mein privacy ka risk zaroor hai, lekin data analysis se behtar diagnostics aur treatments dhoond kar hazaron jaano ko bachaya bhi ja sakta hai. Risk aur opportunity ko balance karna mushkil kaam hai.

Fundamentally, tech industry mein ek **Culture Shift (soch ka badlao)** zaroori hai. Hamein users ko sirf metrics (numbers) samajhna chorna hoga aur yaad rakhna hoga ke woh jeetay-jaagte insaan hain jo izzat aur azaadi ke haqdaar hain. Hamein khud ko **Self-regulate** karna hoga taake logon ka bharosa jeet sakein, aur end-users ko andhere mein rakhne ke bajaye unhein educate karna hoga.

Surveillance koi aisi cheez nahi hai jise roka na ja sakay; hum ab bhi isay rok sakte hain. Ek engineer ke taur par sab se pehla qadam yeh hai:

* Data ko hamesha ke liye store mat rakho.
* Jaise hi kaam khatam ho, data ko delete (**purge**) kar do.
* Shuru mein hi kam se kam data jama karo (**minimize**).

> **Golden Rule:** Jo data aap ke paas database mein hoga hi nahi, woh na leak ho sakta hai, na chori ho sakta hai, aur na hi koi hukumat usay zabardasti cheen sakti hai.

As tech professionals, agar hum apne kaam ka samaj par asar nahi sochte, toh hum apna kaam theek se nahi kar rahe.

---

## Summary

Yeh hamari kitab ka bilkul aakhri hissa hai. Hum ne is lambe aur haseen safar mein bohot saari zameen naapi hai aur databases, distributed systems, aur data ki dunya ko bohot kareeb se dekha hai. Aain har ek chapter ka bilkul asaan zuban mein nishchay (recap) karte hain:

* **Chapter 1 (Data Systems Ki Buniyad):** Is chapter mein hum ne do bade systems ka farq dekha: **Operational systems (OLTP)** jo rozmarra ke chote aur tezi se hone wale kaamo (jaise dukaan par bill banana ya click karna) ke liye hote hain, aur **Analytical systems (OLAP)** jo bare data par research karne (jaise poore saal ka munafe check karna) ke liye hote hain. Hum ne yeh bhi seekha ke apna server khud lagana (**Self-hosting**) behtar hai ya kisi doosre ka computer rent par lena (**Cloud**). Saath hi, ek akeli computer machine (**Single-node**) aur bohot saare computers ke network (**Distributed systems**) ke faide aur nuksaan dekhe, aur yeh samjha ke apne business ke faide aur users ki zaroorat ke darmiyan balance kaise banaya jata hai.
* **Chapter 2 (Nonfunctional Requirements):** Is hisse mein hum ne software ki aisi khoobiyan par baat ki jo bahar se seedhi nazar nahi aati par system ko mazboot chalane ke liye zaroori hain (inhein **Nonfunctional requirements** kehte hain). Jaise: **Performance** (system kitna tez kaam karta hai), **Reliability** (galti ya kisi computer ke jal jaane ke bawajood software ka zinda rehna), **Scalability** (jab lakho naye users aa jayein tab bhi system ka load sambhal lena aur na baithna), aur **Maintainability** (aane wale waqt mein naye engineers ke liye code ko samajhna aur usme tabdeeli karna asaan hona).
* **Chapter 3 (Data Models Aur Query Languages):** Hum ne data ko database ke andar sajane aur rakhne ke alalag tarike (**Data Models**) dekhe. Jaise table aur khano ki shakal mein data rakhna (**Relational model**), parchi ya file ki shakal mein rakhna (**Document model**), aur aapas ke rishton ka jaal banana (**Graph model**). Hum ne **Event Sourcing** (har badlao ko mitaane ke bajaye ek naye event ki tarah line se save karna) aur **DataFrames** (bade data par calculation ka tarika) bhi samjha. Saath hi, computer se data mangwane ki zubanein (**Query Languages**) seekhi, jaise SQL, Cypher (graphs ke liye), SPARQL, Datalog, aur APIs ke liye GraphQL.
* **Chapter 4 (Storage Engines Aur Indexes):** Hum ne database ke pet ke andar ka engine dekha jo asliyat mein hard drive par data likhta aur mitaata hai. Rozmarra ke kaamo (OLTP) ke liye hum ne do bade engines ka mukabla kiya: **LSM-trees** (jo naye data ko ek nayi file mein tezi se likhte hain) aur **B-trees** (jo purani file ke andar hi jagah dhoond kar tabdeeli karte hain). Analytics ke liye hum ne **Column-oriented storage** dekha (jahan data khare khate/columns mein save hota hai taake calculation tez ho). Hum ne data ko jaldi dhoondne ke liye **Indexes** bhi dekhe, jaise poore text mein se dhoondna (**Full-text search**) aur AI ke liye **Vector search**.
* **Chapter 5 (Data Encoding Aur Data Flow):** Is chapter mein hum ne dekha ke computer hamare likhe huwe data ko binary (0 aur 1 yaani **Bytes**) mein kaise badalta hai taake woh tar ke zariye bheja ja sakay (**Encoding**), aur jab software ka naya version aaye toh purana data kharab na ho (**Schema Evolution**). Hum ne yeh bhi dekha ke do alag computer ya processes aapas mein data kaise share karte hain: ya toh database ke zariye, ya direct phone call ki tarah (**Service calls/APIs**), ya bari machines ke zariye (**Workflow engines**), ya phir achanak hone wale kaamo ke signal par (**Event-driven architectures**).
* **Chapter 6 (Replication - Data Ki Nakal):** Hum ne data ki nakal (**Replication**) banana seekha taake agar ek computer kharab ho jaye toh doosre computer se data mil sakay. Hum ne iske teen bade tarike dekhe: Ek boss wala system (**Single-leader**), bohot saare bosses wala system (**Multi-leader**), aur bina kisi boss ke sab ki barabari wala system (**Leaderless**). Hum ne yeh bhi samjha ke data har computer par barabar rahe (**Consistency models**), jaise jab aap koi post likhein toh aap ko foran naye page par apna hi likha dikhe (**Read-after-write consistency**), aur aisi apps ka nizam dekha jo internet ke bagair (**Offline**) bhi kaam karti hain.
* **Chapter 7 (Sharding - Data Ke Tukre):** Jab data itna bara ho jaye ke dunya ka sab se bara computer bhi usay na sambhal sakay, toh uske tukre-tukre kaise karte hain (**Sharding**). Hum ne dekha ke data ko naye computers par barabar kaise baant-te hain taake kisi ek par bojh na pare (**Rebalancing**), user ki request ko sahi computer tak kaise pahunchate hain (**Request routing**), aur tukron mein bate huwe data par naye indexes (**Secondary indexing**) kaise banate hain.
* **Chapter 8 (Transactions):** Hum ne **Transactions** (yaani computer par hone wale kaamo ka ek aisa pakka group jo ya toh poora ka poora hoga ya phir bilkul nahi hoga, aadha adhura nahi reh sakta) ko samjha. Hum ne data ke pakke save hone (**Durability**), aur do users ke kaamo ke aapas mein na takrane ke darjaat (**Isolation levels**) ko dekha: Jaise sirf pakka data parhna (**Read committed**), maazi ki tasveer dekh kar kaam karna (**Snapshot isolation**), aur sab kaamo ke liye ek hi seedhi line lagana (**Serializable**). Saath hi distributed systems mein sab computers par ek sath kaam pakka karne (**Atomicity**) ka tarika seekha.
* **Chapter 9 (Distributed Systems Ki Musibatein):** Is chapter mein hum ne distributed systems ki khofnak sachaiyon aur unme aane wali rukawaton ka jaiza liya. Jaise tar toot jana ya internet ka sust hona (**Network faults and delays**), alalag computers ki ghariyon ka aage peeche hona (**Clock errors**), computer ka achanak kuch der ke liye so jana (**Process pauses**), aur achanak band ho jana (**Crashes**). In sab ki wajah se ek choti si cheez jaise kisi file par tala lagana (**Lock**) bhi kitna mushkil ho jata hai, hum ne yeh bareeki se samjha.
* **Chapter 10 (Consensus - Aapas Ki Raazi):** Hum ne dekha ke jaise dunya mein log aapas mein larrte hain, waise hi jab computers aapas mein larr rahe hon ya unka rabta toot jaye, toh woh sab mil kar ek sachay faisle par kaise raazi hote hain (**Consensus**). Is se hum ne dunya ka sab se sakht aur saaf nizam (**Linearizability**) banana seekha, jahan poore network mein data bilkul ek hi waqt mein ek jaisa neutral aur sach nazar aata hai.
* **Chapter 11 (Batch Processing - Data Ka Dher):** Hum ne maazi ke jama shuda bare data par ek sath kaam karna (**Batch Processing**) seekha. Hum ne iski shuruat bilkul chote Unix tools (jaise commands ko pipe ke zariye aapas mein jorna) se ki, aur phir dunya ke bare bare networks tak gaye jo bade distributed file systems (DFS) ya cloud object stores par chaltay hain (jaise MapReduce vagaira).
* **Chapter 12 (Stream Processing - Behta Huwa Data):** Hum ne maazi ke data ke bajaye bilkul zinda aur behte huwe data par foran kaam karna (**Stream Processing**) seekha. Hum ne data ke daryon ko sambhalne wale dakiye (**Message Brokers**), database ke badlao ko pakarne wale tools (**CDC - Change Data Capture**), kharabi ke bawajood kaam na rokna (**Fault tolerance**), aur do behte huwe daryon ke data ko aapas mein jorna (**Streaming Joins**) seekha.
* **Chapter 13 (Streaming Architecture Ki Soch):** Is chapter mein hum ne stream processing ke peechay chupi poori firaasat aur falsafa (**Philosophy**) ko samjha. Is se hum ne sekha ke dunya ke bilkul alag alag qism ke computer systems (jaise search engine, cache, aur main database) ko ek sath kaise jora jata hai (**Integration**), software ko waqt ke sath naye zamaane ke mutabaq kaise badla jata hai (**System evolution**), aur bari applications ko asani se bada kaise kiya jata hai (**Scaling**).

---

Aur sab se aakhir mein, is **Aakhri Chapter** mein hum ne thora peeche hat kar poori tasveer ko dekha aur data systems banane ke ikhlaqi pehluon (**Ethical aspects**) par ghaur kiya. Hum ne dekha ke jahan data dunya ka faida kar sakta hai, wahan yeh logon ki zindagiyon ko sakht nuksaan bhi pahuncha sakta hai:

* Aam logon ke khilaf aise automated faisle karna jinki koi appeal na ho.
* Samaj mein bhedbhav (discrimination) aur kamzoor logon ki zyadati (exploitation) ko barhana.
* Jasoosi (**Surveillance**) ko ek aam aur normal baat bana dena.
* Logon ke khufiya aur nihayat personal raaz dunya ke samne khol dena.

Hum ne yeh bhi dekha ke data chori hone (**Data breaches**) ka har waqt khatra rehta hai, aur hamari achi niyat ke bawajood hamare banaye huwe nizam ke un-chahe bure asraat (**Unintended consequences**) nikal sakte hain.

Software aur data ka dunya par bohot bara asar parta hai. Isliye hum engineers ko hamesha yaad rakhna chahiye ke hamare kandhon par ek bohot bari zimmedari hai. Hamein ek aisi dunya banane ke liye kaam karna hai jahan hum khud rehna pasand karein: ek aisi dunya jo har insaan ko insaniyat aur poori izzat (**Humanity and respect**) ke sath treat kare. Aain hum sab mil kar is maqsad ki taraf barhein!