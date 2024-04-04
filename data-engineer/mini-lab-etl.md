# มาลองทำ Mini lab ETL กัน

บทความนี้จะมาลองทำ ETL ด้วย Airflow บน docker กัน เป็น minilab แบบ ง่ายๆ มั่ง >.\<!

### Step 1:  Install Airflow

เริ่มจากไป download  docker-compose.yaml จาก link ข้างล่างนี้มาก่อน

```
https://airflow.apache.org/docs/apache-airflow/2.4.1/docker-compose.yaml
```

หลังจากนั้นก็ทำการสร้าง Dockerfile ใช้ code ตามนี้

```docker
FROM apache/airflow:2.4.0 #airflow docker image

RUN pip install ccxt==1.66.32 #CryptoCurrency eXchange Trading Library
```

กลับมาที่ file  docker-compose.yaml  ทำการแก้ไข  code นิดหน่อยโดยจะลบบาง service ที่ไม่ใช้ออกไปก่อน และ config ค่าอีกนิดหน่อย

```yaml
version: '3'
x-airflow-common:
  &airflow-common
  image: my-airflow:latest
  build:
    context: .
    dockerfile: Dockerfile
  environment:
    &airflow-common-env
    AIRFLOW__CORE__EXECUTOR: LocalExecutor
    AIRFLOW__CORE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@airflow-postgres/airflow
    AIRFLOW__CORE__FERNET_KEY: ''
    AIRFLOW__CORE__DAGS_ARE_PAUSED_AT_CREATION: 'true'
    AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
    AIRFLOW__API__AUTH_BACKENDS: 'airflow.api.auth.backend.basic_auth'
    AIRFLOW__SCHEDULER__DAG_DIR_LIST_INTERVAL: 15
    AIRFLOW__SMTP__SMTP_HOST: mailhog
    AIRFLOW__SMTP__SMTP_STARTTLS: 'false'
    AIRFLOW__SMTP__SMTP_SSL: 'false'
    AIRFLOW__SMTP__SMTP_PORT: 1025
    _PIP_ADDITIONAL_REQUIREMENTS: ${_PIP_ADDITIONAL_REQUIREMENTS:-}
  volumes:
    - ./dags:/opt/airflow/dags
    - ./logs:/opt/airflow/logs
    - ./plugins:/opt/airflow/plugins
  user: "${AIRFLOW_UID:-50000}:0"
  depends_on:
    &airflow-common-depends-on
    airflow-postgres:
      condition: service_healthy

services:
  airflow-postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    volumes:
      - postgres-db-volume:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "airflow"]
      interval: 5s
      retries: 5
    restart: always

  airflow-webserver:
    <<: *airflow-common
    command: webserver
    ports:
      - 8080:8080
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://localhost:8080/health"]
      interval: 10s
      timeout: 10s
      retries: 5
    restart: always
    depends_on:
      <<: *airflow-common-depends-on
      airflow-init:
        condition: service_completed_successfully

  airflow-scheduler:
    <<: *airflow-common
    command: scheduler
    healthcheck:
      test: ["CMD-SHELL", 'airflow jobs check --job-type SchedulerJob --hostname "$${HOSTNAME}"']
      interval: 10s
      timeout: 10s
      retries: 5
    restart: always
    depends_on:
      <<: *airflow-common-depends-on
      airflow-init:
        condition: service_completed_successfully

  airflow-triggerer:
    <<: *airflow-common
    command: triggerer
    healthcheck:
      test: ["CMD-SHELL", 'airflow jobs check --job-type TriggererJob --hostname "$${HOSTNAME}"']
      interval: 10s
      timeout: 10s
      retries: 5
    restart: always
    depends_on:
      <<: *airflow-common-depends-on
      airflow-init:
        condition: service_completed_successfully

  airflow-init:
    <<: *airflow-common
    entrypoint: /bin/bash
    command:
      - -c
      - |
        function ver() {
          printf "%04d%04d%04d%04d" $${1//./ }
        }
        airflow_version=$$(AIRFLOW__LOGGING__LOGGING_LEVEL=INFO && gosu airflow airflow version)
        airflow_version_comparable=$$(ver $${airflow_version})
        min_airflow_version=2.2.0
        min_airflow_version_comparable=$$(ver $${min_airflow_version})
        if (( airflow_version_comparable < min_airflow_version_comparable )); then
          echo
          echo -e "\033[1;31mERROR!!!: Too old Airflow version $${airflow_version}!\e[0m"
          echo "The minimum Airflow version supported: $${min_airflow_version}. Only use this or higher!"
          echo
          exit 1
        fi
        if [[ -z "${AIRFLOW_UID}" ]]; then
          echo
          echo -e "\033[1;33mWARNING!!!: AIRFLOW_UID not set!\e[0m"
          echo "If you are on Linux, you SHOULD follow the instructions below to set "
          echo "AIRFLOW_UID environment variable, otherwise files will be owned by root."
          echo "For other operating systems you can get rid of the warning with manually created .env file:"
          echo "    See: https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html#setting-the-right-airflow-user"
          echo
        fi
        one_meg=1048576
        mem_available=$$(($$(getconf _PHYS_PAGES) * $$(getconf PAGE_SIZE) / one_meg))
        cpus_available=$$(grep -cE 'cpu[0-9]+' /proc/stat)
        disk_available=$$(df / | tail -1 | awk '{print $$4}')
        warning_resources="false"
        if (( mem_available < 4000 )) ; then
          echo
          echo -e "\033[1;33mWARNING!!!: Not enough memory available for Docker.\e[0m"
          echo "At least 4GB of memory required. You have $$(numfmt --to iec $$((mem_available * one_meg)))"
          echo
          warning_resources="true"
        fi
        if (( cpus_available < 2 )); then
          echo
          echo -e "\033[1;33mWARNING!!!: Not enough CPUS available for Docker.\e[0m"
          echo "At least 2 CPUs recommended. You have $${cpus_available}"
          echo
          warning_resources="true"
        fi
        if (( disk_available < one_meg * 10 )); then
          echo
          echo -e "\033[1;33mWARNING!!!: Not enough Disk space available for Docker.\e[0m"
          echo "At least 10 GBs recommended. You have $$(numfmt --to iec $$((disk_available * 1024 )))"
          echo
          warning_resources="true"
        fi
        if [[ $${warning_resources} == "true" ]]; then
          echo
          echo -e "\033[1;33mWARNING!!!: You have not enough resources to run Airflow (see above)!\e[0m"
          echo "Please follow the instructions to increase amount of resources available:"
          echo "   https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html#before-you-begin"
          echo
        fi
        mkdir -p /sources/logs /sources/dags /sources/plugins
        chown -R "${AIRFLOW_UID}:0" /sources/{logs,dags,plugins}
        exec /entrypoint airflow version
    environment:
      <<: *airflow-common-env
      _AIRFLOW_DB_UPGRADE: 'true'
      _AIRFLOW_WWW_USER_CREATE: 'true'
      _AIRFLOW_WWW_USER_USERNAME: ${_AIRFLOW_WWW_USER_USERNAME:-airflow}
      _AIRFLOW_WWW_USER_PASSWORD: ${_AIRFLOW_WWW_USER_PASSWORD:-airflow}
      _PIP_ADDITIONAL_REQUIREMENTS: ''
    user: "0:0"
    volumes:
      - .:/sources

  airflow-cli:
    <<: *airflow-common
    profiles:
      - debug
    environment:
      <<: *airflow-common-env
      CONNECTION_CHECK_MAX_COUNT: "0"
    command:
      - bash
      - -c
      - airflow

  minio:
    image: minio/minio:RELEASE.2022-09-25T15-44-53Z
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: miniopass
    ports:
      - 9000:9000
      - 9001:9001
    volumes:
      - minio-data-volume:/data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
    restart: always
    command: server /data --console-address ":9001"

  postgres:
    image: postgres:13
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    volumes:
      - postgres-data-volume:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-q", "-d", "postgres", "-U", "postgres"]
      timeout: 30s
      interval: 5s
      retries: 5
    restart: always

  mailhog:
    image: mailhog/mailhog
    ports:
      - 1025:1025
      - 8025:8025

volumes:
  postgres-db-volume:
  minio-data-volume:
  postgres-data-volume:
```

เสร็จแล้วก็ มาสร้าง directories สำหรับให้ airflow เรียกใช้งาน file ต่าง ตามนี้

* `./dags` -  ไว้ใส่ file dags ของเรา
* `./logs` - ไว้เก็บ logs ของ tasks
* `./plugins` - ไว้ใส่ file  [custom plugins](https://airflow.apache.org/docs/apache-airflow/stable/plugins.html)  หรือ custom operator

สร้าง  AIRFLOW\_UID .env file&#x20;

```batch
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

ถัดมาก็มาลอง start ตัว docker-compose กันเลยดีกว่า ด้วยคำสั่ง

```
docker-compose up -d
```

หลังจาก build ตัว airflow ของเราเสร็จแล้ว  ตัว airflow ของเราก็จะพร้อมใช้งานที่ url ตานข้างล่างนี้เลย

```
http://localhost:8080
```

เข้า url มาแล้วก็จะเจอกับ หน้า login ของ ariflow แบบนี้

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 125155.png" alt=""><figcaption><p>airflow login page</p></figcaption></figure>

ส่วนรหัสสำหรับเข้าใช้ airflow  ก็ตามนี้เลย

```
username: airflow 
password: airflow
```

login เข้ามาก็จะเจอกับ airflow main page แบบนี้

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 130429.png" alt=""><figcaption><p>airflow main page</p></figcaption></figure>

### Step2: Create DAG

ขั้นตอนนี้เราจะมาลอง สร้าง file dag ง่ายกัน

เริ่มจาก create file ชื่อ btc\_pipeline.py ไว้ใน directory  dags

code ของ file btc\_pipeline.py  ก็ตามนี้เลย

```python
from airflow import DAG
from airflow.operators.dummy import DummyOperator
from airflow.utils import timezone

default_args = {
    "owner": "mumaroi",
    "start_date": timezone.datetime(2022, 7, 1),
}
with DAG(
    "btc_pipeline",
    default_args=default_args,
    schedule_interval=None,
) as dag:

    sampleTask = DummyOperator(
        task_id="sample_task",
    )
```

เสร็จแล้วลอง refresh  airflow ก็จะเห็น dag  ของเราในรายการ dags

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 140824.png" alt=""><figcaption><p>DAGs list page</p></figcaption></figure>

ถ้า click เข้าไปใน file dag ของเราก็จะสามารถดูรายละเอียดของ dag ได้ด้วย ตามภาพคือรายละเอียดของ task ใน dag

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 141252.png" alt=""><figcaption><p>task in dag file</p></figcaption></figure>

ลอง run dag กันดู  เริ่มจาก unpause dag ก่อน

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 152025.png" alt=""><figcaption><p>unpause dag</p></figcaption></figure>

จากนั้น click ปุ่ม play แล้วเลือก trigger dag

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 152202.png" alt=""><figcaption><p>trigger dag</p></figcaption></figure>

สามารถเข้าไปดูการทำงาน ของ task ภายใน dag ได้ที่ หน้ารายละเอียดของ dag

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 153038.png" alt=""><figcaption></figcaption></figure>

### Step 3:  Create connection minio for data lake

#### Minio data-lake and connections

ขั้นตอนนี้เราจะมาสร้าง bucket ของเราเพื่อเอาไว้พักข้อมูลที่เราได้มา โดย bucket ที่เราจะใช้จะเป็นตัวของ minio&#x20;

เราสามารถเข้าใช้งาน minio ได้ตาม url นี้เลย

```
http://localhost:9001
```

เข้ามาแล้วก็จะเห็นหน่าตาแบบนี้

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 193705.png" alt=""><figcaption><p>minio login</p></figcaption></figure>

ทำการ login ด้วยรหัสตามนี้

```
usernam: minio
password: miniopass
```

เข้ามาแล้วทำการสร้างตัว bucket ของเรากัน

เริ่มจากเข้าไปที่ Menu bucket แล้วเลือก create bucket

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 194220.png" alt=""><figcaption><p>minio bucket</p></figcaption></figure>

ใส่ชื้อ bucket ให้เรียบร้อย

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 194927.png" alt=""><figcaption><p>create bucket</p></figcaption></figure>

ใส่ชื่อให้เรียบร้อย เสร็จแล้วกด create bucket&#x20;

ถัดมา ทำการ create service account กัน ไปที่ menu  identity > service account

&#x20;แล้วเลือก create service account

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 195335.png" alt=""><figcaption><p>create service account</p></figcaption></figure>

เราจะได้ Access Key กับ Secret Key มาให้ทำการ save ไว้ก่อน

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 195752.png" alt=""><figcaption><p>minio service account key</p></figcaption></figure>

หลังจากนั้นเราจะไปสร้าง connection ไว้ให้ airflow เชื่อมต่อกับ ตัว minio กัน

เริ่มจากกลับไปที่ airflow  ให้เลือกไปที่ menu  Admin -> Connections ก็จะได้หน้าตาแบบนี้

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 200214.png" alt=""><figcaption><p>airflow connections page</p></figcaption></figure>

เข้ามาแล้วก็กด create connection

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 200434.png" alt=""><figcaption><p>create connection</p></figcaption></figure>

ทำการใส่ข้อมูลตามนี้

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 205231.png" alt=""><figcaption><p>connection config</p></figcaption></figure>

ตรง login และ password ให้ใส่  minio   Access Key กับ Secret Key ตามลำดับ&#x20;

เสร็จแล้ว กด save ใด้เลย

ถัดมากลับมาที่ file bit\_pipelie.py ของเรากัน ทำการปรับ code นิดหน่อยเพื่อทดสอบ connection ละหว่าง airflow กับ minio ตามนี้

```python
from airflow import DAG
from airflow.operators.dummy import DummyOperator
from airflow.operators.python import PythonOperator
from airflow.utils import timezone
from airflow.providers.amazon.aws.hooks.s3 import S3Hook
from airflow.providers.postgres.hooks.postgres import PostgresHook

import csv
import logging
from datetime import datetime

import ccxt

default_args = {
    "owner": "mumaroi",
    "start_date": timezone.datetime(2022, 7, 1),
}
with DAG(
    "btc_pipeline",
    default_args=default_args,
    schedule_interval=None,
) as dag:

    def fetch_btc(**context):
        ds = context["ds"]

        #Binance exchange
        exchange = ccxt.binance()

        #Fetch data
        dt_obj = datetime.strptime(ds, "%Y-%m-%d")
        millisec = int(dt_obj.timestamp() * 1000)
        ohlcv = exchange.fetch_ohlcv("BTC/USDT", timeframe="1h", since=millisec, limit=24)
        logging.info(f"OHLCV of BTC/USDT value. [ohlcv={ohlcv}]")

        #Create data to CSV file
        with open(f"btc-{ds}.csv", "w") as f:
            writer = csv.writer(f)
            writer.writerows(ohlcv)

        #Store CSV file in datalake
        s3_hook = S3Hook(aws_conn_id="myminio")
        s3_hook.load_file(
            f"btc-{ds}.csv",
            key=f"cryptocurrency/{ds}/btc.csv",
            bucket_name="data-lake",
            replace=True,
        )

    sampleTask = PythonOperator(
        task_id="fetch_btc",
        python_callable=fetch_btc
    )
```

เสร็จแล้วลอง run dag ดู ตรวจสอบ log ของ task ดูว่ามี error ไหม

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 210937.png" alt=""><figcaption><p>dag graph</p></figcaption></figure>

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 211001.png" alt=""><figcaption><p>task logs</p></figcaption></figure>

ถ้าไม่มีอะไรผิดพลาด เราจะได้ file data ของ ราคา btc ใน minio bucket

file ของเราจะถูกเก็บไว้ใน data-lake/cryptocurrency/2022-10-01/btc.csv

<figure><img src="../.gitbook/assets/Screenshot 2022-10-01 211536.png" alt=""><figcaption><p>btc.csv</p></figcaption></figure>

#### Postgres connection

หลังจาก สร้าง connection minio เสร็จแล้วก็มาสร้าง connection ให้กับ postgres กันต่อ

กรอกข้อมูลตามภาพข้างล่างได้เลย

<figure><img src="../.gitbook/assets/Screenshot 2022-10-02 141104.png" alt=""><figcaption></figcaption></figure>

### Step4: Task workflow

ขั้นตอนนี้เราจะมา สร้าง task workflow ใน file btc\_pipeline.py กัน

โดยเราจะทำการสร้าง task dummy ไว้ตามนี้ ก่อน&#x20;

```python
    dowload_btc_data_file = DummyOperator(
        task_id="dowload_btc_data_file",
    )

    create_btc_import_table = DummyOperator(
        task_id="create_btc_import_table",
    )

    load_btc_data_into_database = DummyOperator(
        task_id="load_btc_data_into_database",
    )

    create_btc_final_table = DummyOperator(
        task_id="create_btc_final_table",
    )

    merge_btc_into_final_table = DummyOperator(
        task_id="merge_btc_into_final_table",
    )

    clear_btc_import_table = DummyOperator(
        task_id="clear_btc_import_table",
    )

    notify = DummyOperator(
        task_id="notify",
    )
```

เสร็จแล้วเริ่มเขียน task dowload\_btc\_data\_file กัน code ก็ตามนี้เลย หน้าที่ของ dowload\_btc\_data\_file  คือ download file จาก minio มาเก็บไว้

```python
def download_btc_file(**context):
        ds = context["ds"]

        # Download file from minios3
        s3_hook = S3Hook(aws_conn_id="minio")
        file_name = s3_hook.download_file(
            key=f"cryptocurrency/{ds}/btc.csv",
            bucket_name="data-lake",
        )

        return file_name

        
dowload_btc_data_file = PythonOperator(
        task_id="download_btc_file",
        python_callable=download_btc_file
)
```

เสร็จแล้วมาเพิ่ม code ให้กับ task create\_btc\_import\_table  กันต่อ

โดย หน้าที่ของ create\_btc\_import\_table  จะทำการสร้าง table เพื่อลองรับข้อมูลที่เราจะนำมาพัก

อย่าลืม  import PostgresOperator กันด้วยเน้อ

<pre class="language-python"><code class="lang-python">from airflow.providers.postgres.operators.postgres import PostgresOperator #อย่าลืนเอา line นี้ไปไว้ข้างบนด้วยเน้อ

<strong>create_btc_import_table = PostgresOperator(
</strong>    task_id="create_btc_import_table ",
    postgres_conn_id="mypostgres",
    sql="""
        CREATE TABLE IF NOT EXISTS btc_import (
            timestamp BIGINT,
            open FLOAT,
            highest FLOAT,
            lowest FLOAT,
            closing FLOAT,
            volume FLOAT
        )
    """,
)
</code></pre>

ถัดมาก็เพิ่ม task load\_btc\_data\_into\_database โดยหน้าที่ของ load\_btc\_data\_into\_database  จะทำการ import data จาก file ที่ได้จาก task download\_btc\_file เข้าไปใน postgres

```python
def load_btc_into_database(**context):
        postgres_hook = PostgresHook(postgres_conn_id="mypostgres")
        conn = postgres_hook.get_conn()

        # Get file name from XComs
        file_name = context["ti"].xcom_pull(task_ids="dowload_btc_data_file", key="return_value")

        # Copy file to database
        postgres_hook.copy_expert(
            """
                COPY
                    btc_import
                FROM STDIN DELIMITER ',' CSV
            """,
            file_name,
        )
load_btc_data_into_database = PythonOperator(
        task_id="load_btc_data_into_database",
        python_callable=load_btc_into_database,
)
```

เสร็จแล้วก็สร้าง task create\_btc\_final\_table กันต่อ หน้าที่ของ create\_btc\_final\_table คือสร้าง  final table เพื่อไว้ใช้ในการเก็บข้อมูลที่จะเอาไปใช้วิเคราะห์ข้อมูลในภายภาคหน้า

```python
create_btc_final_table = PostgresOperator(
        task_id="create_btc_final_table",
        postgres_conn_id="mypostgres",
        sql="""
            CREATE TABLE IF NOT EXISTS btc (
                timestamp BIGINT PRIMARY KEY,
                open FLOAT,
                highest FLOAT,
                lowest FLOAT,
                closing FLOAT,
                volume FLOAT
            )
        """,
)
```

ขั้นต่อไปก็สร้าง task merge\_btc\_into\_final\_table หน้าที่ของ merge\_btc\_into\_final\_table คือการ merge ข้อมูลกันระหว่าง table btc\_import กับ table btc โดยเหตูผลที่ต้องทึแบบนี้เพราะ table btc\_import จะมีข้อมูลซ้ำเยอะไม่เหมาะสำหรับนำมาใช้งาน

สำหรับ code merge\_btc\_into\_final\_table  ก็ตามนี้เลย

```python
merge_btc_into_final_table = PostgresOperator(
        task_id="merge_btc_into_final_table",
        postgres_conn_id="mypostgres",
        sql="""
            INSERT INTO btc (
                timestamp,
                open,
                highest,
                lowest,
                closing,
                volume
            )
            SELECT
                timestamp,
                open,
                highest,
                lowest,
                closing,
                volume
            FROM
                btc_import
            ON CONFLICT (timestamp)
            DO UPDATE SET
                open = EXCLUDED.open,
                highest = EXCLUDED.highest,
                lowest = EXCLUDED.lowest,
                closing = EXCLUDED.closing,
                volume = EXCLUDED.volume
        """,
)
```

เสร็จแล้วเราจะมาสร้าง task สำหรับ clear table btc\_import กัน เหตุผลที่ต้อง clear เพราะข้อเมื่อเรา import ข้อมูลเข้า final table แล้วก็ไม่จำเป็นต้องเก็บ data ใน table btc\_import ไว้แล้ว

```python
clear_btc_import_table = PostgresOperator(
        task_id="clear_btc_import_table",
        postgres_conn_id="mypostgres",
        sql="""
            DELETE FROM btc_import
        """,
)
```

และแล้วก็มาถึง task อันสุดท้ายกันแล้ว โดย task อันสุดท้าย เราจะใช้ชื่อว่า notify หน้าที่ของเจ้า notify คือ เมื่อเรา  pipeline ของเราเสร็จแล้ว ตัว task notify จะทำการส่ง email แจ้งเตือนให้เรารู้

code ของ notify ก็ตามนี้เลย

<pre class="language-python"><code class="lang-python">from airflow.operators.email import EmailOperator #อย่าลืนเอา line นี้ไปไว้ข้างบนด้วยเน้อ

<strong>notify = EmailOperator(
</strong>        task_id="notify",
        to=["mmaroi@mailhog.io"],
        subject="Loaded data into database successfully on {{ ds }}",
        html_content="Your pipeline has loaded data into database successfully",
)
</code></pre>

เสร็จแล้วมากำหนด task workflow กัน

กำหนดตามนี้เลย

```python
fetch_btc_data >> dowload_btc_data_file >> create_btc_import_table >> load_btc_data_into_database 
load_btc_data_into_database >> create_btc_final_table >> merge_btc_into_final_table 
merge_btc_into_final_table >> clear_btc_import_table >> notify
```

หลังจากนั้น กำหนด default\_args ของ dag ใหม่กันหน่อย

```python
default_args = {
    "owner": "mumaroi",
    "email": ["mmaroi@mailhog.io"], #email สำหรับ ส่ง noti
    "start_date": timezone.datetime(2022, 7, 1), #เวลาที่เริ่นทำงาน
    "retries": 3, #ถ้า fail จะ run อีก 3 ครั้งก่อนจะหยุดทำงาน
    "retry_delay": timedelta(minutes=3), #ถ้า fail จะรอ 3 min เพื่อทำงานอีกครั้ง
}
```

ปรับ dag option กันเป็นดับสุดท้าย

```python
with DAG(
    "btc_pipeline",
    default_args=default_args,
    schedule_interval="@daily",#dag จะทำงานทุกๆวัน
) 
```

final dag file เราจะมีหน้าตาประมาณนี้

```python
from airflow import DAG
from airflow.operators.dummy import DummyOperator
from airflow.operators.python import PythonOperator
from airflow.utils import timezone
from airflow.providers.amazon.aws.hooks.s3 import S3Hook
from airflow.providers.postgres.hooks.postgres import PostgresHook
from airflow.providers.postgres.operators.postgres import PostgresOperator
from airflow.operators.email import EmailOperator

from datetime import timedelta

import csv
import logging
from datetime import datetime

import ccxt

default_args = {
    "owner": "mumaroi",
    "email": ["mmaroi@mailhog.io"],
    "start_date": timezone.datetime(2022, 7, 1),
    "retries": 3,
    "retry_delay": timedelta(minutes=3),
}

with DAG(
    "btc_pipeline",
    default_args=default_args,
    schedule_interval="@daily",
) as dag:

    def fetch_btc(**context):
        ds = context["ds"]

        #Binance exchange
        exchange = ccxt.binance()

        #Fetch data
        dt_obj = datetime.strptime(ds, "%Y-%m-%d")
        millisec = int(dt_obj.timestamp() * 1000)
        ohlcv = exchange.fetch_ohlcv("BTC/USDT", timeframe="1h", since=millisec, limit=24)
        logging.info(f"OHLCV of BTC/USDT value. [ohlcv={ohlcv}]")

        #Create data to CSV file
        with open(f"btc-{ds}.csv", "w") as f:
            writer = csv.writer(f)
            writer.writerows(ohlcv)

        #Store CSV file in datalake
        s3_hook = S3Hook(aws_conn_id="myminio")
        s3_hook.load_file(
            f"btc-{ds}.csv",
            key=f"cryptocurrency/{ds}/btc.csv",
            bucket_name="data-lake",
            replace=True,
        )

    def download_btc_file(**context):
        ds = context["ds"]

        # Download file from minios3
        s3_hook = S3Hook(aws_conn_id="myminio")
        file_name = s3_hook.download_file(
            key=f"cryptocurrency/{ds}/btc.csv",
            bucket_name="data-lake",
        )

        return file_name

    def load_btc_into_database(**context):
        postgres_hook = PostgresHook(postgres_conn_id="mypostgres")
        conn = postgres_hook.get_conn()

        # Get file name from XComs
        file_name = context["ti"].xcom_pull(task_ids="dowload_btc_data_file", key="return_value")

        # Copy file to database
        postgres_hook.copy_expert(
            """
                COPY
                    btc_import
                FROM STDIN DELIMITER ',' CSV
            """,
            file_name,
        )

    fetch_btc_data = PythonOperator(
        task_id="fetch_btc_data",
        python_callable=fetch_btc
    )

    dowload_btc_data_file = PythonOperator(
        task_id="dowload_btc_data_file",
        python_callable=download_btc_file
    )

    create_btc_import_table = PostgresOperator(
        task_id="create_btc_import_table",
        postgres_conn_id="mypostgres",
        sql="""
            CREATE TABLE IF NOT EXISTS btc_import (
                timestamp BIGINT,
                open FLOAT,
                highest FLOAT,
                lowest FLOAT,
                closing FLOAT,
                volume FLOAT
            )
        """,
    )

    load_btc_data_into_database = PythonOperator(
        task_id="load_btc_data_into_database",
        python_callable=load_btc_into_database,
    )

    create_btc_final_table = PostgresOperator(
        task_id="create_btc_final_table",
        postgres_conn_id="mypostgres",
        sql="""
            CREATE TABLE IF NOT EXISTS btc (
                timestamp BIGINT PRIMARY KEY,
                open FLOAT,
                highest FLOAT,
                lowest FLOAT,
                closing FLOAT,
                volume FLOAT
            )
        """,
    )

    merge_btc_into_final_table = PostgresOperator(
        task_id="merge_btc_into_final_table",
        postgres_conn_id="mypostgres",
        sql="""
            INSERT INTO btc (
                timestamp,
                open,
                highest,
                lowest,
                closing,
                volume
            )
            SELECT
                timestamp,
                open,
                highest,
                lowest,
                closing,
                volume
            FROM
                btc_import
            ON CONFLICT (timestamp)
            DO UPDATE SET
                open = EXCLUDED.open,
                highest = EXCLUDED.highest,
                lowest = EXCLUDED.lowest,
                closing = EXCLUDED.closing,
                volume = EXCLUDED.volume
        """,
    )

    clear_btc_import_table = PostgresOperator(
        task_id="clear_btc_import_table",
        postgres_conn_id="mypostgres",
        sql="""
            DELETE FROM btc_import
        """,
    )

    notify = EmailOperator(
        task_id="notify",
        to=["mmaroi@mailhog.io"],
        subject="Loaded data into database successfully on {{ ds }}",
        html_content="Your pipeline has loaded data into database successfully",
    )

fetch_btc_data >> dowload_btc_data_file >> create_btc_import_table >> load_btc_data_into_database 
load_btc_data_into_database >> create_btc_final_table >> merge_btc_into_final_table 
merge_btc_into_final_table >> clear_btc_import_table >> notify
```

> github: [https://github.com/MumAroi/sample\_de\_project](https://github.com/MumAroi/sample\_de\_project)

เป็นอันจบ Mini lab ETL กันแล้วเย็\~\~\~\~





