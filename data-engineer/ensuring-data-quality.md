---
description: คุณภาพของข้อมูล
---

# Ensuring Data Quality

ทำไมเราถึงควรให้ความสำคัญกับ คุณภาพของข้อมูลเป็นอันดับต้นๆ เพราะถ้าเราส่งข้อมูลที่ไม่ดีหรือข้อมูลแย่ๆ ไปให้กับผู้ใช้ทางด้าน business ในการตัดสินใจต่างๆ อาจทำให้เกิดการตัดสินใจผิดพลาดส่งผลเสียต่อ ธุรกิจ เพราะฉนั้นเราจึงต้องมีการ validate ข้อมูลให้บ่อยที่สุดเท่าที่ทำได้ในทุกๆขั้นตอนเพื่อให้มั่นใจว่าข้อมูลของเรามีคุณภาพจริงๆ

ตัวอย่างการ validate data&#x20;

* data must be a certain size: ข้อมูลมีขนาดตรงตามที่ระบุไหม
* data must be accurate to some margin of error: ข้อมูลควรมีความแม่นยำไม่มีค่าผิดปกติ(outlier)
* data must arrive within a given timeframe form the start of execution: ข้อมูลควรไหลมาตามเวลาที่เรากำหนด หรือตกลงกัน
* pipeline must run on a particular schedule:  pipeline ควรทำงานตามเวลาที่กำหนด
* data must not contain any sensitive information: ไม่ควรมีข้อมูล sensitive ไหลออกไปให้ผู้ใช้งานได้ใช้

### Dimension of quality

* accuracy: ความถูกต้องของข้อมูลที่ส่งไปและได้รับมา
* completeness: ข้อมูลจากต้นทางไปปลายทางครบหรือป่าว
* uniqueness/deduplication: ข้อมูลที่ไหลไปซ้ำกันหรือป่าว
* validity: มีการ validate ข้อมูลก่อนหรือไม่ ข้อมูลมาจากแหล่งที่ถูกต้องไหม ช่วงเวลาถูกต้องไหม
* consistency\&integrity: การไหลของข้อมูลไปยังปลายทางหลายๆที่ ได้ข้อมูลตรงกันหรือป่าว
* accessibility: ใครสามารถเข้ามาใช้งารข้อมูลได้บ้าง
* timeliness: ข้อมูลไหลเข้ามาตามเวลาที่กำหนดหรือไม่
* compliance: ใครสามารถใช้ข้อมูลชุดนี้ได้บ้างใครเห็นไม่ได้บ้าง



