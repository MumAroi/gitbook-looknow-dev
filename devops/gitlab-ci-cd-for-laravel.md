# Gitlab CI-CD For Laravel

วันนี้เราจะมาลองทำ ci-cd ของ larava ด้วย gitlab ci-cd  บน centos 8 กัน

## Setting Centos8

เริ่มต้นด้วยการ update os package กันก่อน

```
yum update
```

ต่อไปก็ทำการ install  remi and epel repository

```
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

ถัดไปก็ install nginx ให้เรียบร้อย

```
dnf install -y nginx
systemctl start nginx
systemctl enable nginx
```

ขั้นตอนนี้ ถ้าใครไม่ได้ใช้ redis ก็ข้ามไปเลย

```
dnf install -y redis
systemctl start redis 
systemctl enable redis
```

ติดตั้ง firewalld

```
dnf install -y firewalld
systemctl start firewalld
systemctl enable firewalld
```

ทำการเปิด allow http https service

```
firewall-cmd --add-service=http
firewall-cmd --add-service=https
firewall-cmd --runtime-to-permanent
```

ทำการ allow port 80

```
firewall-cmd --permanent --add-port=80/tcp
```

ขั้นตอนนี้ถ้าไม่ได้ลง redis ก็ข้ามไปเลย (allow port 6379 redis/tcp)

```
firewall-cmd --new-zone=redis --permanent
firewall-cmd --zone=redis --add-port=6379/tcp --permanent
```

ต่อไปก็ reload firewall configuration สะ

```
firewall-cmd --reload
```

เริ่มติดตั้ง package สำคัญๆ&#x20;

```
dnf install -y  wget
dnf install -y unzip
dnf install -y policycoreutils-python
dnf install -y yum-utils
dnf install -y git
```

ติดตั้ง php 8.1

```
dnf module reset php
dnf module install php:remi-8.1
```

ติดตั้ง php modules ที่จำเป็น

```
dnf install -y php php-fpm php-common php-opcache php-pecl-apcu php-cli php-pear php-pdo php-mysqlnd php81-php-pecl-redis5 php-gd php-mbstring php-mcrypt php-xml php-zip
```

หลังจากนั้นทำการ start service ของ  php-fpm

```
systemctl start php-fpm
systemctl enable php-fpm
systemctl status php-fpm
```

ติดตั้ง composer&#x20;

```
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer
```

ตั้งค่า user และ group ใน php-fpm config ให้เรียบร้อย

```
vi /etc/php-fpm.d/www.conf
- listen = 127.0.0.1:9000
- users = nginx
- group = nginx
```

ทำการสร้าง directory ที่ชื่อ mydirectory

```
cd /var/www/html
mkdir mydirectory
```

สร้าง user deployer  มาให้ gitlab deployment ใช้งาน

<pre><code>useradd -d /home/deployer deployer
mkdir -p /home/deployer/.ssh
chown -R deployer:deployer /home/deployer/
chmod 700 /home/deployer/.ssh
chmod 644 /home/deployerr/.ssh/authorized_keys
usermod -aG nginx deployer
<strong>ssh-keygen -t rsa -b 2048 -C "deployer-gitlab" (ต้อง login เป็น deployer ก่อนแล้วค่อย run command อันนี้)
</strong>cat /home/deployer/.ssh/id_ed25519.pub >> /home/deployer/.ssh/authorized_keys
setfacl -R -m u:deployer:rwx /var/www/html/mydirectory
</code></pre>

ทำการ create nginx file

```
vi /etc/nginx/conf.d/mydomain.conf
```

```
server {
	listen      80;
	server_name mydomain.me;
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

restart all service

```
systemctl restart redis
systemctl restart php-fpm
systemctl restart nginx
```

get private ssh key เพื่อนำไปใช้กับ gitlab ci-cd

```
cat /home/deployer/.ssh/id_ed25519
```

ทำการเพิ่ม  private ssh key ให้ gitlab ci-cd

> Project > Settings > CI/CD&#x20;

> SSH\_PRIVATE\_KEY

![](../.gitbook/assets/variables\_page.png)

get public ssh key  เพื่อนำไปใช้กับ gitlab ci-cd

```
cat /home/deployer/.ssh/id_ed25519.pub
```

เพิ่ม  ssh public  key ของ Deploy ให้กับ gitlab&#x20;

> Project > Settings > Repository > Deploy keys

![](../.gitbook/assets/deploy\_keys\_page.png)

ทดสอป clone project ด้วย user deployer

```
cd /var/www/html/mydirectory
git clone git@gitlab.example.com:<USERNAME>/myproject.git
```

ทำการติดตั้ง dependency ของ project ให้เรียบร้อย

```
cd /var/www/html/mydirectory/myproject 
composer install
```

ทำการย้าย .env กับ directory strorage ออกมาไว้ข้างนอก

```
cp /var/www/html/mydirectory/myproject/.env /var/www/html/mydirectory/.env
cp -r /var/www/html/mydirectory/myproject/storage /var/www/html/mydirectory/storage
```

ทำการตั้งค่า permissions ให้กับ mydirectory&#x20;

```
chown -R nginx:nginx /var/www/html/mydirectory
chmod -R 775 /var/www/html/mydirectory
```

ทำการ set SELinux context of files ให้กับ directory mydirectory

```
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/mydirectory(/.*)?'
```

ต่อไปก็ทำการ reset the SELinux security context ให้กับ mydirectory directory

```
restorecon -Rv '/var/www/html/mydirectory/'
```

หลังจากย้าย fire .env กับ storage directory เสร็จแล้ว ก็ทำการลบ  project ที่ clone มาเมื่อนข้างต้นทิ้ง

```
rm -rf /var/www/html/mydirectory/myproject
```

## Setting Envoy

สร้าง fire  Envoy.blade.php ใน  project  ตาม code ข้างล่าง

```
@servers(['web' => 'deployer@{yourIP}'])

@task('list', ['on' => 'web'])
    ls -l
@endtask

@setup
    $repository = 'git clone git@gitlab.example.com:<USERNAME>/myproject.git';
    $releases_dir = '/var/www/html/mydirectory/releases';
    $app_dir = '/var/www/html/mydirectory';
    $release = date('YmdHis');
    $new_release_dir = $releases_dir .'/'. $release;
@endsetup

@story('deploy')
    clone_repository
    update_symlinks
    run_composer
    cache_reset
@endstory

@task('clone_repository')
    echo 'Cloning repository'
    [ -d {{ $releases_dir }} ] || mkdir {{ $releases_dir }}
    git clone --depth 1 {{ $repository }} {{ $new_release_dir }}
    cd {{ $new_release_dir }}
    git reset --hard {{ $commit }}
@endtask

@task('update_symlinks')
    echo 'create storage link to {{ $app_dir }}/storage ...'
    [ -d {{ $app_dir }}/storage ] || mkdir {{ $app_dir }}/storage
    rm -rf {{ $new_release_dir }}/storage
    ln -nfs {{ $app_dir }}/storage {{ $new_release_dir }}/storage

    echo 'Linking .env file'
    ln -nfs {{ $app_dir }}/.env {{ $new_release_dir }}/.env

    echo 'Linking current release'
    ln -nfs {{ $new_release_dir }} {{ $app_dir }}/operation
@endtask

@task('run_composer')
    echo "Starting deployment ({{ $release }})"
    cd {{ $new_release_dir }}
    composer install --prefer-dist --no-scripts -q -o
@endtask

@task('cache_reset')
    echo "Cache reset"
    cd {{ $new_release_dir }}
    php artisan opcache:clear
@endtask



```

## Setting DockerFile

สร้าง DockerFile ตาม code ข้างล่าง

```
FROM php:8.1-fpm-alpine

# Add docker-php-extension-installer script
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

# Install dependencies
RUN apk update && apk add --no-cache \
    bash \
    curl \
    freetype-dev \
    g++ \
    gcc \
    git \
    icu-dev \
    icu-libs \
    libc-dev \
    libzip-dev \
    make \
    mysql-client \
    # nodejs \
    # npm \
    oniguruma-dev \
    # yarn \
    # redis \
    openssh-client \
    postgresql-libs \
    rsync \
    zlib-dev

# Install php extensions
RUN chmod +x /usr/local/bin/install-php-extensions && \
    install-php-extensions \
    @composer \
    redis-stable \
    imagick-stable \
    xdebug-stable \
    bcmath \
    calendar \
    exif \
    gd \
    intl \
    pdo_mysql \
    # pdo_pgsql \
    pcntl \
    soap \
    zip \
    opcache \
    mbstring \
    iconv

# fix work iconv library with alphine
RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/community/ --allow-untrusted gnu-libiconv
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

RUN rm -rf /tmp/* /var/cache/apk/*

EXPOSE 900
```

ทำการ build and push image ไปไว้ที่ gitlab registry

```
docker login registry.gitlab.com
docker build -t registry.gitlab.com/<USERNAME>/myproject/php-fpm8:latest .
docker push registry.gitlab.com/<USERNAME>/myproject/php-fpm8:latest
```

## Setting Gitlab-ci

สร้าง file .gitlab-ci.yml ตาม code ข้างล่าง

```
image: registry.gitlab.com/<USERNAME>/myproject/php-fpm8:latest

services:
  - mysql:latest

variables:
  MYSQL_DATABASE: homestead
  MYSQL_ROOT_PASSWORD: secret
  DB_HOST: mysql
  DB_USERNAME: root

# This folder is cached between builds
# https://docs.gitlab.com/ee/ci/yaml/index.html#cache
cache:
  paths:
    - vendor/
  #  - node_modules/

# This is a basic example for a gem or script which doesn't use
# services such as redis or postgres
before_script:
  - composer install
    
test:
  script:
    # run laravel tests
    - php vendor/bin/phpunit --coverage-text --colors=never
    # run frontend tests
    # if you have any task for testing frontend
    # set it in your package.json script
    # comment this out if you don't have a frontend test
    #- npm test

deploy:
  script:
    - 'which ssh-agent || ( apk update -y )'
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SSH_PRIVATE_KEY")
    - mkdir -p ~/.ssh
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
    - php vendor/bin/envoy run deploy --commit="$CI_COMMIT_SHA"
  environment:
    name: production
    url: http://192.168.1.1
  when: manual
  only:
    - main
```

## Deployment

หลังจาก commit code ขึ้น gitlab ไปแล้ว ให้ไปที่ menu

> CI/CD -> Pipelines

ก็จะเจอกับ หน้าต่างแบบ นี้&#x20;

<figure><img src="../.gitbook/assets/Screen Shot 2565-09-05 at 13.20.42.png" alt=""><figcaption><p>(ภาพตัวอย่าง)</p></figcaption></figure>

สามารถกดเข้าไปดู  stages ได้ ด้วยการกดปุ่ม skipped ใน column status&#x20;

<figure><img src="../.gitbook/assets/Screen Shot 2565-09-05 at 13.26.58.png" alt=""><figcaption><p>ภาพตัวอย่าง ในที่ทำไว้แค่ 1 stage</p></figcaption></figure>

หลังจากนั้น สามารถกด ปุ่มเพื่อสั่งให้ stage ทำงานได้ เพื่อทำการ deploy code

The end.

> example project: [https://github.com/MumAroi/laravel-cicd-gitlab-setting](https://github.com/MumAroi/laravel-cicd-gitlab-setting)
