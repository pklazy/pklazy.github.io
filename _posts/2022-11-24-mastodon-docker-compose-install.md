---
layout: post
title: docker-compose로 mastodon 4.0.2 설치 기록
category: 메모
tags: linux mastodon docker
date: 2022-11-24 22:14:00 +0900
---

설치 환경

- 하드웨어: Raspberry Pi 4
- OS: Raspberry Pi OS Lite (64-bit)

미리 준비할것

- 사용할 도메인
- 계정 인증 및 알림 관련 메일 송신을 위한 SMTP 서버
  - 여기서는 간단하게 gmail SMTP 사용, 비밀번호는 google 계정 비밀번호가 아닌 "앱 비밀번호"에서 메일용으로 생성한 비밀번호

docker 설치

```shell
$ sudo apt remove docker docker-engine docker.io containerd runc
$ sudo apt install ca-certificates curl gnupg lsb-release
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg 
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt update
$ sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

편의를 위해 docker 그룹에 유저 추가

```shell
$ sudo gpasswd -a $USER docker
```

그룹 변경적용을 위해 재 로그인

mastodon docker-compose 파일 다운로드

```shell
$ mkdir mastodon
$ cd mastodon
$ wget https://raw.githubusercontent.com/mastodon/mastodon/v4.0.2/docker-compose.yml
```

빌드하지 않을거기 때문에 "build: ."를 주석 처리

mastodon 버전을 지정하기 위해 image 값을 "tootsuite/mastodon:v4.0.2"로 수정

```shell
$ sed -i 's/build: \./# \0/g' docker-compose.yml
$ sed -i 's/image: tootsuite\/mastodon/\0:v4.0.2/g' docker-compose.yml
```

setup 실행을 위해 필요한 파일 임시 생성

```shell
$ touch .env.production
```

setup을 실행.

사용할 도메인 이름을 넣고 혼자 사용하기에 single mode로 설정, smtp는 gmail을 사용.

```shell
$ docker compose run --rm web bundle exec rake mastodon:setup
Your instance is identified by its domain name. Changing it afterward will break things.
Domain name: YOUR_DOMAIN

Single user mode disables registrations and redirects the landing page to your public profile.
Do you want to enable single user mode? yes

Are you using Docker to run Mastodon? Yes

PostgreSQL host: db
PostgreSQL port: 5432
Name of PostgreSQL database: postgres
Name of PostgreSQL user: postgres
Password of PostgreSQL user:
Database configuration works! 🎆

Redis host: redis
Redis port: 6379
Redis password:
Redis configuration works! 🎆

Do you want to store uploaded files on the cloud? No

Do you want to send e-mails from localhost? No
SMTP server: smtp.gmail.com
SMTP port: 587
SMTP username: YOUR_GMAIL_SMTP_USERNAME
SMTP password: YOUR_GMAIL_SMTP_PASSWORD
SMTP authentication: plain
SMTP OpenSSL verify mode: none
Enable STARTTLS: auto
E-mail address to send e-mails "from": Mastodon <YOUR_GMAIL@gmail.com>
Send a test e-mail with this configuration right now? Yes
Send test e-mail to: YOUR_TEST_MAIL@gmail.com

This configuration will be written to .env.production
Save configuration? Yes
Below is your configuration, save it to an .env.production file outside Docker:

# Generated with mastodon:setup on 2022-XX-XX XX:XX:XX UTC

# Some variables in this file will be interpreted differently whether you are
# using docker-compose or not.

LOCAL_DOMAIN=YOUR_DOMAIN
SINGLE_USER_MODE=true
SECRET_KEY_BASE=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
OTP_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
VAPID_PRIVATE_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
VAPID_PUBLIC_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
DB_HOST=db
DB_PORT=5432
DB_NAME=postgres
DB_USER=postgres
DB_PASS=
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587
SMTP_LOGIN=YOUR_GMAIL_SMTP_USERNAME
SMTP_PASSWORD=YOUR_GMAIL_SMTP_PASSWORD
SMTP_AUTH_METHOD=plain
SMTP_OPENSSL_VERIFY_MODE=none
SMTP_ENABLE_STARTTLS=auto
SMTP_FROM_ADDRESS=Mastodon <YOUR_GMAIL@gmail.com>

It is also saved within this container so you can proceed with this wizard.

Now that configuration is saved, the database schema must be loaded.
If the database already exists, this will erase its contents.
Prepare the database now? Yes
Running `RAILS_ENV=production rails db:setup` ...


Database 'postgres' already exists
Done!

All done! You can now power on the Mastodon server 🐘

Do you want to create an admin user straight away? no

```

실행하여 나온 내용을 .env.production 에 붙여넣기

.env.production 내용

```conf
# Generated with mastodon:setup on 2022-XX-XX XX:XX:XX UTC

# Some variables in this file will be interpreted differently whether you are
# using docker-compose or not.

LOCAL_DOMAIN=YOUR_DOMAIN
SINGLE_USER_MODE=true
SECRET_KEY_BASE=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
OTP_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
VAPID_PRIVATE_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
VAPID_PUBLIC_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
DB_HOST=db
DB_PORT=5432
DB_NAME=postgres
DB_USER=postgres
DB_PASS=
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587
SMTP_LOGIN=YOUR_GMAIL_SMTP_USERNAME
SMTP_PASSWORD=YOUR_GMAIL_SMTP_PASSWORD
SMTP_AUTH_METHOD=plain
SMTP_OPENSSL_VERIFY_MODE=none
SMTP_ENABLE_STARTTLS=auto
SMTP_FROM_ADDRESS=Mastodon <YOUR_GMAIL@gmail.com>
```

<https://pgtune.leopard.in.ua> 참고해 postgres14/postgresql.conf 수정

pubic/system 폴더 권한 변경

```shell
$ sudo chown -R 991:991 public/system
```

mastodon 실행

```shell
$ docker compose up -d
```

nginx, certbot 설치

```shell
$ sudo apt install nginx certbot python3-certbot-nginx
```

mastodon nginx 설정 파일을 다운로드하여 수정

- example.com을 사용할 도메인으로 수정
- ssl_certificate 주석 제거
- ssl_certificate_key 주석 제거
- root public 위치 변경
- docker로 동작하므로 `try_files $uri =404;` 를 `try_files $uri @proxy;`로 수정

```shell
$ sudo wget https://raw.githubusercontent.com/mastodon/mastodon/v4.0.2/dist/nginx.conf -O /etc/nginx/sites-available/mastodon
$ sudo sed -i 's/example.com/YOUR_DOMAIN/g' /etc/nginx/sites-available/mastodon
$ sudo sed -i 's/# ssl_certificate/ssl_certificate/g' /etc/nginx/sites-available/mastodon
$ sudo sed -i 's@root .*@root '"$PWD"'\/public;@g' /etc/nginx/sites-available/mastodon
$ sudo sed -i 's/try_files $uri =404;/try_files $uri @proxy;/g' /etc/nginx/sites-available/mastodon
```

도메인 인증서 발급

```shell
$ sudo certbot certonly --nginx -d YOUR_DOMAIN
```

설정 파일을 활성화 후 ngnix 재실행

```shell
$ sudo ln -s /etc/nginx/sites-available/mastodon /etc/nginx/sites-enabled/mastodon
$ sudo systemctl restart nginx
```

<!-- 
관리자 계정 생성

```shell
$ docker compose exec web ./bin/tootctl accounts create ADMIN_USERNAME --email ADMIN@EMAIL --confirmed
```
-->

---

참고

- <https://docs.docker.com/engine/install/debian/>
- <https://docs.joinmastodon.org/admin/install/>
- <https://gist.github.com/TrillCyborg/84939cd4013ace9960031b803a0590c4>
- <https://docs.joinmastodon.org/admin/tootctl/>
