# การทำ Data Pipeline ที่ดี

การทำ data pipeline ให้ดีนั้นเราควรจะคำนึง คุณสมบัติ(characteristic) 4 ประการณ์ ดังนี้

* reproducible: สามารถ run ซ้ำๆได้โดยที่ ผลลัพธ์ ที่คาดหวังนั้นต้องเหมือนเดิม (idempotent)
* future proof:  สามารถ ย้อนกลับไปแก้ไข ข้อมูลที่ผิดได้ และสามารถย้อนกลับไปเก็บข้อมูลในอดีตได้ (backfilling)
* fault tolerance: ทนทานต่อข้อผิดพลาด หากเกิดข้อผิดพลาดสามารถจะทำงานต่อได้ (automatic retry of failed task)
* transparent: สามารถดูการไหลของข้อมูลได้ ดูความผิดพลาดที่เกิดขึ้นระหว่างทางได้ (clarity of where data are)

### Idempotent

เป็นคุณสนบัติทางคณิตศาสตร์  คือ เวลาที่กำหนด input เข้าไปใน function ผลลัพธ์ที่ได้ควรเหมือนเดิมทุกครั้ง

> common practice ง่ายๆของ idempotent ก็คือการกำหนดเวลาว่าจะเริ่มต้ังแต่เมื่อไหร่และไปสิ้นสุดเมื่อไหร่ เพื่อให้ได้ข้อมูลในเฉพาะช่วงเวลานั้นเสมอ

### Designing Pipelines Tasks

แนวทางการ ออกแบบ pipeline task ควรคำนึงถึง เรื่อง boundaries หรือ single well-defined purpose คือ task ควรทำงานอย่างใดอย่างนึงเท่านั้น การ design แบบนี้จะช่วยให้ task สามารถทำงานแบบ parallel ได้ง่าย หาข้อผิดพลาดที่เกิขึ้นระหว่าง task ได้ง่ายด้วย



