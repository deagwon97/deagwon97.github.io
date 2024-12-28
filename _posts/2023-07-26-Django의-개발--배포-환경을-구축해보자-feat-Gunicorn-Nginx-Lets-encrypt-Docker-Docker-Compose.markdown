---
layout: post
thumbnail: ecd42fef-1bd1-4423-9135-c77e73eeea90
title: "Django의 개발 / 배포 환경을 구축해보자. (feat, Gunicorn, Nginx, Let's encrypt, Docker, Docker Compose)"
createdAt: 2023-07-26 10:26:56.594000
updatedAt: 2023-07-26 10:26:56.594000
category: "백앤드"
---

<img alt="image" src="/images/ecd42fef-1bd1-4423-9135-c77e73eeea90"/>

## I. 개요

 이 포스트에서는 Django의 개발 및 배포환경을 구축하는 방법에 대해 소개한다. 우리가 개발을 하다보면 다음과 같은 문제로 스트레스를 받는다. 

- 윈도우의 개발환경과 리눅스의 배포환경이 달라서 에러가 발생하거나
- Https를 연결하는 작업에 비용을 지불하거나
- Let's encrypt를 적용하면서 많은 시간을 쓰거나
- 설마 디버거 안 써..?

docker와 docke-compose를 통해서 이러한 문제를 하나씩 해결해보자. 

### docker란?

> *도커는 container라고 불리는 고립된 환경에서 하나의 어플리케이션을 패키징하는 능력을 제공한다. 이러한 어플리케이션의 컨테이너화는 하나의 host에서 여러개의 컨테이너를 동시에 병렬적으로 실행할 수 있게 도와준다.  - docker docs*
> 

### docker-compose란?

> *Compose란 여러개의 도커 컨테이너를 "정의"하고 "실행"하는 도구이다. - docker docs*
> 

  Docker를 통해서 Proxy서버 컨테이너, Django 서버 컨테이너, Let's encrypt 컨테이너를 분리하고 이를 하나의 compose파일에 정의해서 ``docker-compose -f docker-compose.prod.yml up`` 라는 명령만으로 배포를 위해 필요한 작업들인 https 붙이는 작업, Gunicorn을 세팅하고 데몬화 하는 작업을 모두 자동화하고자 한다.

 여기에 더해서 개발 전용 compose파일을 정의해서  ``docker-compose -f docker-compose.dev.yml up`` 이라는 명령어를 실행하고 몇가지 vscode 세팅을 한다면 배포환경과 매우 유사한 개발 전용 컨테이너까지 만들겠다.

## I. 폴더 구조

다음과 같이 api 폴더를 구성한다.

``````bash
api
│  .gitignore
│  .proxy-companion.prod.env# 배포용에서 사용되는 nginx-proxy container 환경변수
│  backend.dev.env# 개발용 컨테이너 환경 변수
│  backend.prod.env# 배포용 컨테이너 환경 변수
│  docker-compose.dev.yml# 개발용 docker-compose 파일
│  docker-compose.prod.yml# 배포용 docker-compose 파일
│  Dockerfile
│  README.md
│
├─dev-vscode
│      launch.json# Dajngo 디버거를 정의
│      settings.json# pylint, default python path등 vscode 세팅 정의
│
├─nginx // nginx-proxy 컨테이너
│  │  custom.conf# nginx-proxy 컨테이너의 nginx  conf.d 파일
│  │  Dockerfile# custom.conf와 vhost.d 를 컨테이너 내부로 COPY
│  │
│  └─vhost.d
│          default# nginx-proxy 컨테이너의 nginx default 파일
│
└─src# Django의 소스 코드 
	 # docker ignore를 통해서 분리할 수도 있지만 이런식으로 이미지속에 들어가는 소스 코드는 따로 분리하는 것을 선호한다.
    │  manage.py
    │  requirements.txt
    │
    ├─accounts
    │  │  admin.py
    │  │  apps.py
    │  │  mangers.py
    │  │  models.py
    │  │  permissions.py
    │  │  serializers.py
    │  │  urls.py
    │  │  views.py
    │  │  __init__.py
    │  │
    │  └─migrations
    │          0001_initial.py
    │          __init__.py
    │
    ├─post
    │  │  admin.py
    │  │  apps.py
    │  │  models.py
    │  │  tests.py
    │  │  views.py
    │  │  __init__.py
    │  │
    │  └─migrations
    │          __init__.py
    │
    └─project# settings.py가 들어있는 django의 메인 폴더
            asgi.py
            routing.py
            settings.py
            urls.py
            wsgi.py
            __init__.py
``````

## II. 개발 환경 세팅

### 1. 코드

- docker-compose.dev.yml 파일 정의

``````yml
version: '3.8'

services:
  backend-dev:
    container_name: backend-dev
    build:
      context: ./
      dockerfile: ./Dockerfile
    volumes:
      - ./src:/workdir/src# 개발중에는 소스코드를 외부와 마운트해서 사용한다.
      - ./dev-vscode:/workdir/.vscode# 미리 정의해둔 settings.json과 launch.json
    command: bash -c "cd /workdir/src && python -m pip install -r requirements.txt"
    ports:
      - 8000:8000
    env_file:
      - ./backend.dev.env# 개발에서 쓰이는 환경변수
    stdin_open: true# docker run -i
    tty: true# docker run -t
    entrypoint: ['/bin/bash', '-c']# /bin/bash를 띄워서 커널이 죽지 않도록 설정.
``````

- backend.dev.env 파일 정의

``````bash
SITE_ID=1
DEBUG=1
HTTPS=0
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1] 0.0.0.0
VIRTUAL_PORT=8000
SECRET_KEY=<django에서 생성하는 secret>
``````

- [settings.py](http://settings.py) 수정

``````python
import os 

SECRET_KEY = os.environ.get("SECRET_KEY")

# site id
SITE_ID = int(os.environ.get("SITE_ID", default=1))

# debug option
DEBUG = int(os.environ.get("DEBUG", default=0))

# allowed host
ALLOWED_HOSTS = os.environ.get("DJANGO_ALLOWED_HOSTS", default="*").split(" ")

# state
STATE = os.environ.get("STATE")

# for https
if int(os.environ.get("HTTPS")) == 1:
    SECURE_PROXY_SSL_HEADER = ("HTTP_X_FORWARDED_PROTO", "https")
    USE_X_FORWARDED_HOST = True
    SECURE_SSL_REDIRECT = True# http로 들어오면 https 로 redirect
``````

- dev-vscode/launch.json 생성
    
    print함수 등을 통해서 코드 중간 중간 변수값을 확인할 수 있지만 vscode의 훌륭한 debuger를 사용한다면 더 편하게 디버깅을 수행할 수 있다. 처음 세팅이 귀찮더라도 한 번 설정하면 그 다음 작업이 매우 편해지기 때문에 디버거 설정을 추가한다.
    
    - 컨테이너 외부에서 접근하기 때문에 0.0.0.0:8000 옵션을 추가해야함.

``````json
//launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Django",
            "type": "python",
            "request": "launch",
            "program": "NULL/backend/manage.py",
            "args": [
                "runserver", "0.0.0.0:8000"
            ],
            "django": true
        }
    ]
}
//settings.json
{
    "python.defaultInterpreterPath": "/usr/local/bin/python",
    "python.pythonPath": "/usr/local/bin/python",
    "python.linting.pylintEnabled": false,
    "python.linting.enabled": true
}
``````

- Dcokerfile

``````bash
# pull official base image
FROM python:3.8-slim-buster

#set environment variables
# I don't want to generate pcy files
ENV PYTHONDONTWRITEBYTECODE 1
# ignore buffering
ENV PYTHONUNBUFFERED 1
# set encoding
ENV PPYTHONENCODING utf-8

#set work directory
WORKDIR /workdir

# for mysql, django의 DB로 msyql을 사용하지 않는다면 굳이 필요없다.
RUN apt-get update
RUN apt-get install python3-dev default-libmysqlclient-dev gcc  -y

# 배포할 경우 프로젝트 코드를 모두 도커 이미지에 넣는다.
COPY ./src /workdir/src

# python dependencies
RUN pip install --upgrade pip
RUN pip install django
RUN pip install gunicorn
RUN pip install -r /workdir/src/requirements.txt
``````

### 2. docker-compose up

``````bash
docker-compose -f .\docker-compose.dev.yml up -d --build

# -f .\docker-compose.dev.yml : .\docker-compose.dev.yml파일 실행
# -d: 백그라운드에서 실행
# --build: 실행전에 이미지 빌드
``````

### 3. 다시 시작할 때

1. root 폴더에서 터미널 실행 후 docker-compose up

``````bash
docker-compose -f .\docker-compose.dev.yml up -d --build
``````

1. Attach Visual Studio Code

<img alt="image" src="/images/d8b954cd-cc89-4462-9807-e7480cd0a82b"/>

1. /workdir 경로의 폴더를 open


<img alt="image" src="/images/cdea7926-d077-4e67-8247-d0bda13239de"/>

1. 디버거 실행

<img alt="image" src="/images/039e4024-9e82-490d-9cee-a5f35a5c043f"/>

1. 개발!

## II. 배포 환경 세팅

### 1. 코드

- docker-compose.prod.yml 파일 정의

``````yml
version: '3.8'

services:
  backend-prod:
    container_name: backend-prod
    restart: always
    build:
      context: ./
      dockerfile: ./Dockerfile
     # workers는 무조건 2개 이상 설정하는 것을 권장한다.
    command: bash -c "cd ./src && gunicorn --workers=3 --bind 0.0.0.0:8000 --preload project.wsgi:application"
    expose:
      - 8000
    env_file:
      - ./backend.prod.env

  nginx-proxy:# proxy 컨테이너
    container_name: nginx-proxy
    restart: always
    build: ./nginx
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./backend:/workdir/backend
      - ./nginx/custom.conf:/etc/nginx/conf.d/custom.conf
      - certs:/etc/nginx/certs
      - html:/usr/share/nginx/html
      - vhost:/etc/nginx/vhost.d
      - /var/run/docker.sock:/tmp/docker.sock:ro
    depends_on:
      - backend-prod
    labels:
      - com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy

# letsencrypt 인증서를 발급 및 관리하는 컨테이너
# 사전에 도메인을 발급받아 연결하는 작업이 필요하다.
  letsencrypt:
    image: nginxproxy/acme-companion
    container_name: letsencrypt
    depends_on:
      - nginx-proxy
    volumes:
	# letsencrypt는 DinD(Docker In Docker)기술이 사용된다.
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - certs:/etc/nginx/certs
      - html:/usr/share/nginx/html
      - vhost:/etc/nginx/vhost.d
      - acme:/etc/acme.sh
    env_file:
      - .proxy-companion.prod.env
    
  redis-chat:
    image: redis
    restart: always
    container_name: redis-chat
    ports:
      - 6379:6379
    command: redis-server
    healthcheck:
      test: 'redis-cli -h 127.0.0.1 ping'
      interval: 3s
      timeout: 1s
      retries: 5

volumes:
  certs:
  html:
  vhost:
  acme:
  
``````
- letsencrypt 컨테이너
    - letsencrypt는 TLS 인증서를 발급해주는 비영리기관으로 https에 사용되는 TLS인증서가 필요하다면 letsencrypt는 매우 합리적인 방법이다.
    - 다른 유로 인증기관이 보증하는 것처럼 사이트와 사이트 이용자간에 이동하는 데이터 변조나 데이터 가로채기에 있어 안전함을 보장한다.
    - TLS 인증 자체가 그 사이트가 안전함을 보장하지 않는다.
    - letsencrypt는 DinD(Docker In Docker)기술이 사용된다. letsencrypt 컨테이너 속에서 다른 컨테이너의 환경변수를 확인하고 LETSENCRYPT_HOST의 값에 따라 인증서를 결정한다.
    - nginx-proxy서버 또한 DinD을 통해 다른 컨테이너의 환경변수를 확인하고 VIRTUAL_HOST에 따라 외부 요청을 포워딩하는 proxy 서버의 역할을 수행한다.

- backend.prod.env 파일 정의

``````bash
SITE_ID=2
DEBUG=0
HTTPS=0
DJANGO_ALLOWED_HOSTS=https://prod.domin.com host서버이름2 host서버이름3
VIRTUAL_HOST=prod.domin.com
VIRTUAL_PORT=8000
SECRET_KEY=django-insecure-bdmgpgpdmgpgpdmgpa이것은시크릿키askdljflk
LETSENCRYPT_HOST=prod.domin.com
``````

## 2. 배포

세팅이 완료되었다면 다음 한 줄로 인증서 작업, 데몬화, 배포를 모두 자동화 할 수 있다.

``````bash
docker-compose -f .\docker-compose.prod.yml up -d --build
``````

만약 프론트(뷰, 리액트, 장고 스태틱)을 배포하고 싶다면 docker-compose.prod.yml 파일에 프론트 서버용 컨테이너 추가하면된다.
