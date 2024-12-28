---
layout: post
thumbnail: ""
title: "Nginx, Let’s Encrypt SSL 인증서 발급"
createdAt: 2023-07-26 10:18:36.752000
updatedAt: 2023-07-26 10:18:36.752000
category: "기타"
---
### prerequired

https 인증서를 발급하기에 앞서 다음 과정이 선행되어야 합니다.

- 80번 포트 외부 접속 허용
- 443번 포트 외부 접속 허용
- 본인 소유 도메인 발급

### Nginx 설치

``````shell
# 설치
$ sudo apt-get install -y nginx nginx-common nginx-full
# 삭제
#$ sudo apt-get purge --auto-remove -y nginx nginx-common nginx-full
``````

### Setup NGINX

``````shell
# default파일 수정
$ sudo vim /etc/nginx/site-enabled/default
``````
(nginx에서는 site-abaliable을 수정한 후 symbolic link을 연결하는 것을 권장합니다.)

### well-known 접근 설정
 다음과 같이 /\.well-known/acme-challenge/으로 요청이 들어올 때 응답을 정의합니다.

- 80번 포트 접근 허용

``````bash
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        server_name {domain-name};

        location ~ /\.well-known/acme-challenge/ {
           allow all;
           root /var/www/letsencrypt;
        }
}
``````

### Nginx 실행
 파일을 저장하고, 새롭게 정의한 defualt 파일이 문제가 없는지 확인 한 후, nginx를 시작합니다.

``````bash
sudo nginx -t# 테스트
sudo service nginx start# 실행
``````

### Setup SSL
자 이제 본격적으로 인증서를 발급하겠습니다.

``````bash
# let's encrypt 설치
$ sudo apt update
$ sudo apt install certbot python3-certbot-nginx
$ sudo mkdir /var/www/letsencrypt
``````

``````bash
# 인증서 발급
$ sudo certbot certonly --webroot -w /var/www/letsencrypt -d {도메인} --agree-tos -m {이메일}
``````


 인증서가 정상적으로 발급되었다면 다음 경로에서 발급된 인증서를 확인하실 수 있습니다.
``````bash
# 생성된 인증서 확인
$ sudo ls /etc/letsencrypt/live/{도메인}
>>> README  cert.pem  chain.pem  fullchain.pem  privkey.pem
``````

### Setup Nginx

 이제 인증서경로를 연결하여 443 포트를 설정해 봅시다.

``````bash
# default파일 수정
$ sudo vim /etc/nginx/site-avaliable/default
``````

``````bash

server {
        listen 80 default_server;
        listen [::]:80 default_server;

        server_name {domain-name};

        location ~ /\.well-known/acme-challenge/ {
           allow all;
           root /var/www/letsencrypt;
        }
}

server {
        listen  443 ssl;
        server_name 도메인;

       #access_log /var/log/nginx/proxy/access.log;
       #error_log /var/log/nginx/proxy/error.log;
        
        ssl_certificate /etc/letsencrypt/live/도메인/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/도메인/privkey.pem;

        location / {

		//https 포트로 서비스할 html 경로를 설정합니다.
                //proxy_pass를 통해서 내부에 열린 다른 port로 연결 할 수도 있습니다.
                
                root /build폴더 경로/build;
                index  index.html index.htm;
                try_files $uri $uri/ /index.html;

		// or

		proxy_pass http://127.0.0.1:8000;
				
        }
}
``````

### Nginx 실행

``````bash
sudo nginx -t# 테스트
sudo service nginx restart# 실행
``````

### 로그 확인

``````bash
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
``````

### 인증서 자동 갱신

``````bash
sudo crontab -e

# 3개월 마다 인증서를 자동 갱신
15 3 * * * certbot renew --quiet --renew-hook "/etc/init.d/nginx reload"
``````











