---
layout: post
thumbnail: 45e16066-84a5-4bf2-8327-0a3754b6a806
title: "[Go] missing Location in call to Date 에러"
createdAt: 2023-07-26 10:47:40.513000
updatedAt: 2023-07-26 10:47:40.513000
category: "Go"
---
### missing Location in call to Date

나는 다음과 같이 2 stage로 도커파일을 구성하여, 개발 - 배포 컨테이너를 분리하였다.
`````` Dockerfile
FROM golang:latest AS builder

RUN apt-get update -y &&\
    apt-get upgrade -y

ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux

WORKDIR /root/workdir/src

COPY ./src /root/workdir/src

RUN go mod download && go mod tidy

RUN go build -o main ./

WORKDIR /dist

RUN cp /root/workdir/src/main .

FROM scratch

COPY --from=builder /dist/main .

ENTRYPOINT ["/main"]
``````

개발환경에서는 아무런 문제가 없었는데 
배포하고 난 후 timezone과 관련된 에러가 발생했다.


<img alt="image" src="/images/45e16066-84a5-4bf2-8327-0a3754b6a806"/>

``````
missing Location in call to Date
``````

찾아보니 builder 이미지에는 들어있는 timezone 정보가 scratch 이미지에는 없어서 생기는 오류였다.

`````` Dockerfile
FROM scratch

COPY --from=builder /dist/main .

# $GOROOT : /usr/local/go 
COPY --from=builder /usr/local/go/lib/time/zoneinfo.zip /

ENV ZONEINFO=/zoneinfo.zip

ENV TZ=Asia/Seoul

ENTRYPOINT ["/main"]
``````

위와 같이 GOROOT 경로에서 zeoninfo.zip 파일을 scratch로 옮겨준 후, 환경변수에 추가하여 해결하였다.

scratch 이미지는 정말 "빈" 이미지 라는 것을 기억해 두자.


### Reference
- https://medium.com/@mhcbinder/using-local-time-in-a-golang-docker-container-built-from-scratch-2900af02fbaf
- https://blog.frec.kr/golang/troubleshooting-0/
