# Apache Airflow

Apache Airflow เป็น open source  ที่นิยมนำมาใช้ทำ data pipeline  หรือสร้าง work flow  ถูกพัฒนาโดย Airbnb&#x20;

### Airflow Principles

* dynamic: สามารถเขียน code เพื่อ generate pipeline ขึ้นมาได้
* extensible:  สามารถพัฒนา operator ใหม่ๆขึ้นมาต่อกับตัว core ได้เพื่อให้ตอบโจทธ์ กับแต่ละองค์กรได้
* elegant: สามารถส่ง ค่า หรือ ข้อมูลบ้างอย่างเข้าไปที่ data pipeline ในขณะที่ data pipeline กำลังทำงานอยู่ด้วย template engine
* scalable: รองรับการ scalable&#x20;

### Why Airflow?

* Modern & friendly UI: ui ใช้ง่าย
* Python to define workflow: สามารถเขียน python ในการจัดการ pipeline และ workflow ได้
* General purpose: สามารถนำ pipeline ไปทำอย่างอื่นได้, ไปทำ automage อย่างอื่นได้ ไม่จำเป็นต้องทำแค่ data pipeline&#x20;
* Easy to add new functionality: ง่ายต่อการเพิ่ม function ใหม่ๆ&#x20;
* Easy to monitor the state of a workflow: ง่ายต่อการ monitor การทำงานของ workflow&#x20;
* Build to scale: scale ได้ง่าย
* Big community: community ใหญ่

### When to use Ariflow

* pipeline and workflow as a code: python
* schedule task and pipeline at regular interval:  month, day, week
* integration with many different types of database: mysql, nosql, oracle
* rich web interface: friendly ui

### When not to use Ariflow

* streaming and real-time data
* processing big data
* dynamic pipeline&#x20;
* run precisely schedule time

### Architecture of Airflow

<figure><img src="../.gitbook/assets/arch-diag-basic.png" alt=""><figcaption><p><a href="https://airflow.apache.org/docs/apache-airflow/stable/concepts/overview.html">https://airflow.apache.org/docs/apache-airflow/stable/concepts/overview.html</a></p></figcaption></figure>

* web server: provide a dashboard for user
* scheduler: orchestrate execution of job on a tigger or schedule
* worker: execution the operation define in a DAG
* queue: hold the state of running  DAG and  task
* metastore: contain the status of the DAG run and task instance

### Process in Airflow

<figure><img src="../.gitbook/assets/arch-diag-basic (1).png" alt=""><figcaption><p><a href="https://airflow.apache.org/docs/apache-airflow/2.0.0/start.html">https://airflow.apache.org/docs/apache-airflow/2.0.0/start.html</a></p></figcaption></figure>

### Airflow concepts to code

<figure><img src="../.gitbook/assets/Screen Shot 2565-09-27 at 12.49.07.jpg" alt=""><figcaption><p><a href="https://www.skooldio.com/courses/data-pipelines-with-airflow">https://www.skooldio.com/courses/data-pipelines-with-airflow</a></p></figcaption></figure>

* directed acyclic graph (DAG): collection of all tasks
* operator: wrapper that encapsulates logic to do
* task: unit of execution, initiate(นำเข้า) for operator
