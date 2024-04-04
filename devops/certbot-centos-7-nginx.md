# Certbot CentOS 7 Nginx

บทความนี้จะมาสอนติดตั้ง Certbot บน CentOS 7 กับ nginx

## Step 1:  Install Nginx

ก่อนการติดตั้ง อะไรก็มา update dependency os กันก่อน

```
yum -y update
```

จากนั้นมาเริ่มติดตั้ง nginx กัน

```
yum install -y nginx
```

ต่อไปก็ทำการสร้าง config file domain ของ nginx กัน

```
vi /etc/nginx/conf.d/example.com.conf
```

ใส่ config ประมาณนี้

```
server {
	listen      80;
	server_name example.com;
	root        /var/www/html/mydirectory/operation/public;
        index index.php index.html index.htm;
	
	proxy_read_timeout 600;
   	proxy_connect_timeout 600;
   	proxy_send_timeout 600;

	charset utf-8;
	gzip on;
	gzip_types text/css application/javascript text/javascript application/x-javascript 	image/svg+xml text/plain text/xsd text/xsl text/xml image/x-icon;
	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}
	
	location ~ \.php {
        include /etc/nginx/fastcgi_params;
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	}

	location ~ /\.ht {
		deny all;
	}
}
```

แล้วก็สั่ง start nginx service  กันเลย

```
systemctl start nginx
systemctl enable nginx
```

## Step 2: Installing Snap

ทำการติดตั้ง snap ด้วย command ตามข้างล่างนี้

```
yum install snapd
```

หลังจากติดตั้งสำเร็จ ก็ทำการ  เปิด socket ให้กับ snap

```
systemctl enable --now snapd.socket
```

ถัดไปก็ทำการสร้าง symbolic ให้กับ snap

```
 ln -s /var/lib/snapd/snap /snap
```

ตรวจสอบให้มั่นใจว่า snap ของเราเป็น version ล่าสุด

```
 snap install core;
```

## Step 3:  Install Certbot

ขั้นตอนนี้จะเป็นการใช้ snap ติดตั้ง certbot&#x20;

```
snap install --classic certbot
```

## Step 4: Install Cerbot Certificate

ขั้นตอนนี้จะเป็นการติดตั้ง certificate จาก certbot ให้กับ nginx

```
certbot --nginx
```

หลังจากนั้น เลือก domain ที่เราต้องการให้เรียบร้อย

## Step 5: Automatic Renewal

ทำการตั้งค่า certbot ให้ renew certificate อัตโนมัติ

```
 certbot renew --dry-run
```

ที่ website ของเราก็จะเป็น https แล้ว

> [https://yourwebsite.com/](https://yourwebsite.com/)
