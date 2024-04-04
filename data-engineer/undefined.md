# คำศัพย์ที่เจอบ่อยๆ

### **ETL (**Extract-Transform-Load**)**

กระบวนการที่ออกแบบเอาไว้ดึงข้อมูลออกมาจากหลายๆ ที่ รวมถึงการตรวจสอบคุณภาพของข้อมูลที่จะนำมาประยุกต์ โดยมีการปรับข้อมูลให้เป็นไปในรูปแบบเดียวกันเพื่อให้ ข้อมูลจากหลายๆ แหล่งสามารถใช้งานร่วมกันได้&#x20;

โดยถูกแบ่งออกเป็น 3 กระบวนการ

* Extract การดึงข้อมูลจากแหล่งข้อมูลภายนอกต่างๆ
* Transform แปลงข้อมูลเพื่อให้ตรงตามกับความต้องการ
* Load นำข้อมูลเข้าสู่ Data Warehouse หรือ ฐานข้อมูลอื่นๆ

> ETL จะถูกนำมาใช้ก็ต่อเมื่อ&#x20;
>
> * กรณี ปลายทางของเรา(destination) require specific data format หรือ require fix schema เท่านั้น ทำให้เราต้องทำการ transform ก่อนนำเข้าข้อมูล
> * กรณี ที่ไม่สามารถเก็บ privacy data ไว้ใน destination ได้ ก็ต้องทำการ transform ก่อน

### ELT (Extract-Load-Transform)

กระบวนการที่คล้ายกับ ETL แต่จะสลับจาก  Transform ไปเป็นการ Load ก่อน ส่วนใหญ่จะนิยมใช้คู่กับ Cloud Platform เพราะตัว Cloud Platform มีการสเกลและการแยกทรัพยากรที่เหมาะสมกับการคำนวณข้อมูลที่มีจำนวนมาก

> ELT จะถูกนำมาใช้ก็ต่อเมื่อ&#x20;
>
> * destination เป็น cloud native data warehouse หรือ data warehouse ที่รองรับข้อมูลใหญ่ๆ
>
> ข้อดี ทำให้ Data Analyst สามารถทำงานได้อย่างรวดเร็วกว่า ETL

### Data Pipeline

เป็กระบวนการ "ย้ายข้อมูลจากต้นทาง ไปยังปลายทาง"&#x20;

แบ่งได้ 2 ประเภท

* Batch Processing: ประมวลผลเป็นช่วงเวลา เช่น รายเดือน, รายวัน, รายปี
* Stream Processing: ประมวลผล real time

> ETL, ELT เป็น 1 ใน data pipeline pattern ที่นิยมนำมาใช้

### DAG (Directed Acyclic Graph)

เป็น conceptual framework ที่ช่วยจัดการ data engineer task และ data work flow&#x20;

ประกอบด้วย 3 อย่าง

* directed: มีทิศทาง
* acyclic: ไม่เกิด loop ขึ้น
* graph: แผนผังที่ประกอบด้วย line และ node

### Data Source

คือแหล่งข้อมูลต่างๆ เช่น

1. Database: Structure (SQL),  UnStructured(No SQL), Semi Structured(JSON)&#x20;
2. File: Microsoft Office , G Suite , Image , Voice
3. Application: CRM , ERP , POS
4. Website and API
5. IOT Device

### Dataset

คือชุดข้อมูลที่ถูกต้องตามลักษณะโครงสร้างข้อมูล และเพียงพอที่จะนำไปใช้ประมวลผลได้

### Data Lake

แหล่งเก็บข้อมูลทุกรูปแบบที่ไม่มีการเปลี่ยนแปลงรูปแบบใดๆ หรือ ข้อมูลดิบ(Raw Data) ที่มาจากต้นทาง

### Data Warehouse

เป็นศูนย์กลางในการรวบรวมข้อมูลจำนวนมากจากหลายแหล่งข้อมูล ที่ผ่านการคัดกรอง และมีประโยชน์ เพื่อนำไปใช้ประกอบการตัดสินใจ หรือทำการวิเคราะห์ทางธุรกิจ

### Data Mart

คลังข้อมูลขนาดเล็ก ข้อมูลใน Data Mart เป็น subset ของ Data warehouse ที่มุ่งเน้น เฉพาะเจาะจง ไปในเรื่องใดเรื่องหนึ่ง

### **Data Architecture**

วิธีการที่แต่ละองค์กรวางแผนไว้เพื่อจัดการกับข้อมูลในองค์กร โดยเป็นการอธิบาย Flow ของข้อมูล เพื่อกำหนดแนวทางและเพื่อให้แน่ใจว่าสุดท้ายแล้วข้อมูลที่เลือกเก็บ จะตรงกับความต้องการขององค์กรและจะเกิดประโยชน์ต่อองค์กรอย่างแท้จริง

### Ingest Data

ขั้นตอนในการนำข้อมูลจาก Data Source เข้ามาเก็บใน Data Lake เพื่อนำไปใช้ในกระบวนการต่อไป

แบ่งเป็น 2 แบบ

* Batch: การประมวลผลของกลุ่มข้อมูลในช่วงระยะเวลาหนึ่ง(Day, Week, Month)
* Streaming: การประมวลผลแบบเรียลไทม์(Real Time)

### Store Data

ขั้นตอนจัดเก็บข้อมูลต่างๆ ไม่ว่าจะเป็น Object, Unstructured, Structure, Semi Structure เอาไว้ทำ Data Lake, Data Warehouse และ Data Mart

### Transform Data

เป็นขั้นตอนการแปลงข้อมูล เช่น Convert data, Clean data เป็นต้น

### Analyze

การนำข้อมูลที่ผ่านการ Process แล้ว หรือ ยังไม่ผ่าน ก็ได้  ไปใช้งานในการวิเคราะห์

### Visualize

การนำข้อมูลไปแสดงเป็น Report, Dashboard, Graph หรือ Table ไว้ให้ดูง่ายและเห็นเป็นภาพ

### **OLTP(Online Transaction Processing)**

ฐานข้อมูลเชิงสัมพันธ์ ที่ถูกออกแบบเพื่อให้รองรับการใช้งานในแต่ละวัน มีการเปลี่ยนแปลข้อมูล เพิ่ม ลบ แก้ไข เป็นข้อมูลที่เกิดขึ้นในธุรกิจในแต่ละวัน ลักษณะการเก็บข้อมูลจะใช้วิธีการ Normalized สามารถเก็บข้อมูลจำนวนมากได้อย่างมีประสิทธิภาพ

### **OLAP(Online Analytical Processing)**

เป็นเทคโนโลยีที่ใช้ข้อมูลจาก  Data Warehouse พื่อนำไปใช้ในการวิเคราะห์และตัดสินใจทางธุรกิจทางธุรกิจ สามารถค้นหาคำตอบที่ต้องการ และสามารถแก้ปัญหาที่มีความซับซ้อนโดยใช้ระยะเวลาสั้น ๆ

### **IaaS (Infrastructure as a Service)**

**บริการบนคลาวด์จ่ายตามการใช้งานสำหรับบริการต่างๆ เช่นพื้นที่เก็บข้อมูล ระบบเครือข่าย ตัวอย่างเช่น Azure IaaS services, AWS EC2, Google Compute Engine (GCE)**

### **PaaS (Platform as a Service)**

**เครื่องมือฮาร์ดแวร์และซอฟต์แวร์ที่มีอยู่ทางอินเทอร์เน็ต มักใช้เพื่อพัฒนาแอปพลิเคชัน ตัวอย่างเช่น AWS Elastic Beanstalk, Windows Azure, Google App Engine (GAE)**

### **SaaS (Software as a Service)**

**ซอฟต์แวร์ที่พร้อมใช้งาน เป็นบริการแอปพลิเคชันบนระบบคลาวด์ ตัวอย่างเช่น Microsoft O365, Google Apps, Salesforce, Dropbox, ZenDesk**

