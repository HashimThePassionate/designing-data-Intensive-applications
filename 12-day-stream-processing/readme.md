# Stream Processing

A complex system that works is invariably found to have evolved from a simple system that works. The inverse proposition also appears to be true: A complex system designed from scratch never works and cannot be made to work.
—John Gall, Systemantics (1975)

---

### Introduction: Bounded Aur Unbounded Data Ka Asli Farq

Chapter 11 mein hum ne **Batch Processing** par bohot tafseel se baat ki thi. Batch processing kya karti hai? Yeh pehle se maujood files ke ek poore set ko uthati hai (input), us par calculations chalati hai, aur files ka ek naya set tayaar karti hai (output). Yeh output ek tarah ka **Derived Data** (nikala hua data) hota hai, jise agar zaroorat paray toh batch process ko dobara chala kar naye siray se re-create kiya ja sakta hai. Hum ne dekha ke is asaan lekin takatwar soch se hum search indexes, recommendation systems, aur analytics jaisi barri barri cheezein bana sakte hain.

Lekin pooray Chapter 11 mein hum aik bohot barri shart (assumption) par chal rahe the: **Input hamesha Bounded (mehdood) hota hai.** Bounded ka matlab hai ke data ka size pehle se pata hota hai aur woh ek jagah ruka hua hota hai, is liye batch process ko saaf pata hota hai ke kab input khatam ho gaya hai aur kab kaam rokna hai.

* **Bacho ki Tarah Samajhein (MapReduce Aur Sorting Ka Masla):** Sochein aap ke paas bacho ke khilono ka ek dabba hai aur aap ne unhein unke size ke mutabaq chote se bade ki tarteeb (sort) mein rakhna hai. MapReduce mein jo sorting hoti hai, uski shart yeh hai ke usay **pehle poora dabba khali karke saare khilonay dekhne parenge**, uske baad hi woh pehla sab se chota khilone bahar nikal sakta hai. Kyun? Kyunke ho sakta hai jo khilona dabbe mein sab se aakhir mein para ho, wahi poore dabbe mein sab se chota ho! Agar aap ne poora input parhne se pehle hi output nikalna shuru kar diya, toh tarteeb galat ho jayegi. Is liye batch processing mein kaam beech mein shuru karne ka option nahi hota.

Lekin asli zindagi (reality) mein data aisy ruka hua nahi hota. Asli data **Unbounded (na-khatam hone wala)** hota hai, kyunke yeh waqt ke sath dheere dheere, thoda thoda karke aata hai. Aap ke users ne kal bhi data generate kiya, aaj bhi kar rahe hain, aur kal bhi karte rahenge. Jab tak aap ki company band nahi ho jati, yeh silsila kabhi khatam nahi hoga! Data ka dataset kabhi bhi poori tarah "complete" nahi hota.

Is liye, batch processors ko majbooran is na-khatam hone wale data ko artificially (zabardasti) chote chote jhatkay wale tukron mein baantna parta hai. Jaise:

* Pore din ka data ikhta karo aur har raat 12 bajay batch job chalao (Daily Batch).
* Ya har ghante ke aakhir mein batch job chalao (Hourly Batch).

---

### Batch Se Stream Processing Ki Taraf Safar

Rozana chalne wale batch processes ke sath sab se bara masla yeh hota hai ke agar input data mein abhi koi badlao (change) aaya hai, toh uska nateeja output report mein **ek din baad** nazar aayega. Aaj kal ke tez daur mein impatient (be-sabar) users ke liye itna lamba intezar karna bohot slow hai.

Is delay (deri) ko kam karne ke liye hum kya kar sakte hain?

1. Hum processing ki frequency barha sakte hain—jaise har ek second ke aakhir mein aik second ka data process karein.
2. Ya phir, hum waqt ke in artificial tukron (time slices) ko poori tarah chhor dein aur **jaise hi koi event (kaam) ho, usay foran ussi waqt process kar lein!**

Isi soch ko hum **Stream Processing** kehte hain.

Aam zuban mein **Stream** ka matlab hota hai aisa data jo waqt ke sath dheere dheere aur lagatar (incrementally) milta rehta hai. Yeh concept computer ki duniya mein bohot purana hai aur kayi jagah dikhta hai:

* Unix operating system mein `stdin` (standard input) aur `stdout` (standard output) mein pipes ke zariye data ka behna.
* Programming languages mein `lazy lists` (jo tabhi naya element banati hain jab manga jaye).
* Filesystem ke APIs (jaise Java ka `FileInputStream`).
* Internet par chalne wale `TCP connections`.
* Online dekhi jaane wali audio aur video (jaise YouTube ya Netflix par video stream hona, jahan poori video ek sath download nahi hoti balkay sath sath chalti rehti hai).

---

### Is Chapter Ka Roadmap

Is chapter mein hum **Event Streams** ko data management ke ek naye nizam (mechanism) ke tor par dekhenge. Yeh batch data ka hi ek aisa roop hai jo na-khatam hone wala (unbounded) hai aur lagatar process hota hai. Hamara safar in teen marhalon se guzray ga:

* Hum sab se pehle yeh parhenge ke streams ko computer mein kaise dikhaya (represent), save (store), aur network par ek jagah se doosri jagah bheja (transmit) jata hai.
* Phir hum explore karenge ke streams aur aam databases ka aapas mein kya gehra rishta hai.
* Aur aakhir mein, **"Processing Streams"** ke section mein hum un tareeqon aur tools ko deeply check karenge jin ke zariye in streams ko har waqt (continually) process kiya jata hai aur application ki building blocks tayaar ki jati hain.

---

