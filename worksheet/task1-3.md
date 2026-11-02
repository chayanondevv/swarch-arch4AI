# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:**

**Student ID / รหัสนิสิต:**

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 501 | 605 | 93.5 | 100 |
| t0-small | 11 | 25 | 72.5 | 0 |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> สิ่งที่ได้เพิ่ม (What it buys): ความแม่นยำสูงขึ้นอย่างมาก (High Accuracy)

> สิ่งที่ต้องแลก (What it costs): เวลาประมวลผลที่ช้าลงมาก (High Latency),ค่าใช้จ่ายและทรัพยากรที่สูง (High Compute/Financial Cost)

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 25 | 260 | 413| 14 | 0 | 94.1 |
| t1-after | 9.5 | 393 | 1010 | 0 | 28 | 92.4|

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> p50 มองเห็นเฉพาะ 50% แรกที่ตอบกลับปกติ ไม่สะท้อน Request ที่ล้มเหลว

> Column ไหนที่บอกความจริง: errors,p95_ms

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> ในฝั่ง Review Service: ทุกๆ Request ที่รอค้างอยู่จะกินโควตา Connection/Task ใน Event Loop

> ในฝั่ง API Gateway: Gateway ต้องถือ TCP Socket ค้างไว้เพื่อรอรับคำตอบจาก Downstream Service

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> รีวิวที่ตกอยู่ในสถานะ pending เนื่องจากเกิด Timeout หรือ Error ควรถูกนำไปจัดการด้วยกระบวนการ Asynchronous Offline / Batch Processing

> Asynchronous Work Queue

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> หากตั้ง Timeout เป็น 0.1 วินาที: เวลา 0.1 วินาที สั้นเกินกว่าที่ Large Model จะประมวลผล (Inference) และส่งคำตอบกลับมาได้ทันในสภาวะปกติ ทำให้แทบทุก Request จะถูกตัดเข้า Fallback,ความแม่นยำตกต่ำ,เพิ่มภาระให้ Batch Job

> หากตั้ง Timeout เป็น 10 วินาที: เวลา 10 วินาที นานเกินไปสำหรับการรอตอบกลับแบบ Synchronous Real-time ทำให้ User Experience แย่ลงอย่างมาก

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 6 | 543 | 91.5 | 38 |
| t2-0.7 | 0.7 | 375 | 564 | 93.0 | 53 |
| t2-0.8 | 0.8 | 433 | 574 | 93.5 | 64 |
| t2-0.9 | 0.9 | 477 | 580 | 93.5 | 77 |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> Threshold 0.7 : รักษา Accuracy ได้เกือบสมบูรณ์,ลด Cost ได้เกือบครึ่งหนึ่ง,Latency ดีขึ้น แต่ เสีย Accuracy ไปเล็กน้อย

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> Cascade ช่วยเปลี่ยน Request ส่วนใหญ่ (ฝั่ง p50) ให้วิ่งบน Fast Path (Small Model) ได้สำเร็จ แต่ไม่สามารถช่วยกลุ่ม Worst-case (ฝั่ง p95) ได้ เพราะกลุ่มหลังยังคงต้องวิ่งผ่าน Slow Path (Small + Large Model) ส่งผลให้ p50 ลดลงอย่างเห็นได้ชัด ในขณะที่ p95 แทบไม่ลดลงเลย

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> โมเดลมั่นใจผิดในประโยคประชดประชัน คำคัดสรร หรือภาษากลุ่มเฉพาะ

> ตัวอย่าง: รีวิวที่เขียนด้วยคำประชดประชัน (Sarcasm/Irony) เช่น "อาหารรอนานแค่ 2 ชั่วโมงเอง บริการดีเลิศประทับใจจริงๆ"

> Small Model มักประมวลผลด้วย Feature หรือ Pattern แบบง่าย เมื่อเจอคำเชิงบวกโมเดลจะให้ค่า Confidence สูงมาก ส่งผลให้ร้านอาหารที่ควรถูกตัดคะแนนหรือได้รับการร้องเรียน กลับได้รับ Label เป็น "ดีเยี่ยม"

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 5 | 547 | 93.0 | 38 | 35 |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> รีวิวที่ได้ประโยชน์: รีวิวที่มีข้อความซ้ำกัน หรือพฤติกรรมการพิมพ์คล้ายกันบ่อยๆ

> รีวิวที่ไม่มีวันได้ประโยชน์: รีวิวที่เป็นเอกลักษณ์เฉพาะตัวสูง,รีวิวที่เข้ามาเป็นครั้งแรก

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> ต้องทำการ ล้าง Cache เดิมทิ้งทั้งหมด

> กำหนด TTL (Time-To-Live) หรือ Expiration Strategy

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> Request ในกลุ่ม 5% ที่ช้าที่สุด (p95) คือเคสยากที่ Small Model มี Confidence ต่ำกว่า Threshold ส่งผลให้ Request นั้นต้องเสียเวลา สองต่อ (Sequential Penalty) คือต้องประมวลผลที่ Small Model ก่อน แล้วค่อยโดน Escalate ไปรันที่ Large Model ต่อ ทำให้ latency สะสมเพิ่มขึ้น นอกจากนี้ การวัดผลยังทำให้เห็นการทำงานร่วมกันระหว่าง Cascade และ Cache (Task 3) ว่าเมื่อนำมาต่อกันจริง Cache Hit จะช่วยดึงค่า p50 ดิ่งลงเหลือเพียง 5 ms และช่วยตัด Large Calls ในกลุ่มประโยคซ้ำได้อย่างมีประสิทธิภาพ ซึ่งเป็น Trade-off และ Behavior ทางสถิติของระบบ Real-world Systems ที่เห็นภาพชัดเจนจากการลงมือทดสอบ Load Test จริง
