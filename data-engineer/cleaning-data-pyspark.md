# Cleaning Data ด้วย pyspark กัน

บทความนี้จะมาทำ basic cleaning data  บน google colab  ด้วย pyspark กัน

### Step1: Create Colab file

เริ่นต้นก็ไปสร้าง file ที่ google colab ตาม url นี้เลย

```url
https://colab.research.google.com
```

เสร็จแล้วก็กด ไปที่ menu file > new notebook

### Step2: Install Spark

ขั้นตอนนี้จะทำการ install เจ้า spark ใน colab กัน code ก็ตามนี้เลย

```python
!apt-get update
!apt-get install openjdk-8-jdk-headless -qq > /dev/null
!wget -q https://archive.apache.org/dist/spark/spark-3.3.0/spark-3.3.0-bin-hadoop3.tgz
!tar xzvf spark-3.3.0-bin-hadoop3.tgz
!pip install -q findspark
```

### Step3: Set enviroment variable

ทำการ enviroment variable ไว้ใช้งานในภายภาคหน้า

```python
import os
os.environ["JAVA_HOME"] = "/usr/lib/jvm/java-8-openjdk-amd64"
os.environ["SPARK_HOME"] = "/content/spark-3.3.0-bin-hadoop3"
```

### Step4: install Pyspark

ติดตั้ง pyspark ด้วย command ข้างล่างนี้เลย

```python
!pip install pyspark==3.3.0
```

### Step5: Create Spark Session

ทำการ สร้าง spark session&#x20;

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.master("local[*]").getOrCreate()
spark.conf.set("spark.sql.legacy.timeParserPolicy", "legacy")
```

> ใช้ `local[*]` เพื่อเปิดการใช้งานการประมวลผลแบบ multi-core ให้กับ spark

### Step6: Get data file

ขั้นตอนนี้เราจะมา dowload data file ที่จะนำมาใช้ในการ clean data กัน&#x20;

สามารถ download ได้ข้างล่างเลย

{% file src="../.gitbook/assets/bank_term_deposit.csv" %}

> เดี๋ยวมาเขียนต่อ >.\<!
