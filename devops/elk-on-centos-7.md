# ELK On CentOS 7

วันนี้จะมาลอง ติดตั้ง Elasticsearch stack บน CentOS 7 กัน

## Step 1: Install Java  and OpenJDK

ก่อนจะทำอะไรก็ update dependency centos กันก่อน

```
yum update
```

ต่อไปก็ติดตั้ง java openjdk กัน

```
yum -y install java-openjdk-devel java-openjdk
```

## Step 2: Add ELK repository

ทำการเพิ่ม elk repository ตาม command ข้างล่าง

```
vi /etc/yum.repos.d/elasticsearch.repo
```

copy code ข้างนี้ไปใส่ได้เลย

```
[elasticsearch-8.x]
name=Elasticsearch repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

## Step 3: Install and Configure Elasticsearch

ทำการติดตั้ง ตัว elasticsearch กัน

```
yum -y install elasticsearch
```

ทำการ reset password ให้กับ  elasticsearch

```
/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic -i
```

ทำการ ตั้งค่า JVM  memory limits ของ elasticsearch

```
vi /etc/elasticsearch/jvm.options
```

เปลี่ยน setting memory limits จาก

```
-Xms1g
-Xmx1g
```

เป็น

```
-Xms256m
-Xmx512m
```

ทำการ Configure Elasticsearch ต่ออีกนิดหน่อย

```
vi /etc/elasticsearch/elasticsearch.yml
```

แก้ไขค่าจาก

```
# Set the bind address to a specific IP (IPv4 or IPv6):
#
#network.host: 192.168.0.1
#
# Set a custom port for HTTP:
#
#http.port: 9200
```

เปลี่ยนเป็น

```
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: localhost
#
# Set a custom port for HTTP:
#
http.port: 9200
```

ทำการ start service elasticsearch&#x20;

```
systemctl start elasticsearch
systemctl enable elasticsearch
```

ทดลอง curl elasticsearch ดุว่าทำงามได้หรือป่าว

```
curl --cacert /etc/elasticsearch/certs/http_ca.crt -u elastic https://localhost:9200
```

ก็จะได้ผลลัพธ์ดังนี้

```
{
  "name" : "cent7.novalocal",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "5GFmuAkwQ4Sxrrrg4G-b6A",
  "version" : {
    "number" : "8.2.0",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "b174af62e8dd9f4ac4d25875e9381ffe2b9282c5",
    "build_date" : "2022-04-20T10:35:10.180408517Z",
    "build_snapshot" : false,
    "lucene_version" : "9.1.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## Step 4: Install and Configure Kibana <a href="#mce_33" id="mce_33"></a>

มาติดตั้ง kibana กันต่อ

```
yum -y install kibana
```

หลังจาก install เสร็จแล้ว  ก็ทำการ configure Kibana อีกนิดหน่อย

```
vi /etc/kibana/kibana.yml
```

แก้ config ตาม code ข้างล่างได้เลย

```
server.host: "0.0.0.0"
server.name: "kibana.example.com"
elasticsearch.hosts: ["http://localhost:9200"]
```

ทำการ start servervice kibana กันเลย

```
systemctl start kibana
systemctl enable kibana
```

หลังจากนั้นทำการ allow port ให้กับ kibana&#x20;

```
firewall-cmd --add-port=5601/tcp --permanent
firewall-cmd --reload
```

> ตรงนี้ถ้าใครลง nginx ไว้สามารถทำ  proxy pass แทนการ allow port ได้ตามนี้เลย &#x20;
>
> ```
> vi /etc/nginx/conf.d/example.com.conf
> ```
>
> nginx config code
>
> ```
> server {
>     listen 80;
>
>     server_name example.com www.example.com;
>
>     location / {
>         proxy_pass http://localhost:5601;
>         proxy_http_version 1.1;
>         proxy_set_header Upgrade $http_upgrade;
>         proxy_set_header Connection 'upgrade';
>         proxy_set_header Host $host;
>         proxy_cache_bypass $http_upgrade;
>     }
> }
> ```
>
>

หลังจากนั้นลอง เข้า kibana ผ่าน domain กัน

```
 http://ip-address:5601
 or use nginx proxy pass
 http://ip-address.com
```

ถัดไปทำการ generate Kibana token

```
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

และทำการ verification code

```
/usr/share/kibana/bin/kibana-verification-code
```

## Step 5: Install Logstash <a href="#mce_34" id="mce_34"></a>

ติดตั้ง Logstash ด้วย command ตามนี้

```
yum -y install logstash
```

สามารถแก้ไขการตั้งค่าของ logstash ได้ที่

```
vi  /etc/logstash/conf.d/
```

ทำการ start service logstash

```
systemctl start logstash
systemctl enable logstash
```

## Step 6: Install Filebeat <a href="#ftoc-heading-17" id="ftoc-heading-17"></a>

ติดตั้ง Filebeat ด้วย command ตามนี้

```
yum -y install filebeat
```

ทำการเพิ่ม   system module ของ local system logs

```
 filebeat modules enable system
```

สั่ง run filebeat ด้วยคำสั่ง

```
filebeat setup
systemctl start filebeat
systemctl enable filebeat
```

The end.
