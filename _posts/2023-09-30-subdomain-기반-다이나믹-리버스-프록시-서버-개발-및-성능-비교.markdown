---
layout: post
thumbnail: d7d08003-93df-4fb4-b599-5d23625769b9
title: "subdomain 기반 다이나믹 리버스 프록시 서버 개발 및 성능 비교"
createdAt: 2023-09-30 12:04:05.774000
updatedAt: 2023-09-30 12:04:05.774000
category: "네트워크"
---

subdomain 기반으로 목적지 서버를 식별하여 사용자를 중계해주는 다이나믹 리버스 프록시 서버를 개발했습니다.  처음 개발할 때는 `nodejs` 로 중계 서버를 만들었습니다. 하지만 생각보다 성능이 높지 않았고, `golang` 를 통해서 중계 서버를 만든다면 성능상 어떤 차이가 있을지 확인해보고 싶어 이 실험을 계획했습니다. 

# 배경 설명

- 리버스 프록시 서버란?
    
    리버스 프록시 서버는 클라이언트와 웹 서버(또는 여러 웹 서버) 사이에 위치하여 클라이언트의 요청을 대신 받아 웹 서버에 전달하고, 웹 서버의 응답을 다시 클라이언트에게 전달하는 중개자의 역할을 합니다.
    
- 다이나믹 리버스 프록시 서버란?
    
    일반적인 용어는 아닙니다. 보통 Nginx, Traefik 과 같은 웹서버들은 정적인 설정을 지원합니다.  저는 클라이언트의 reqeust http header에서 subdomain을 파싱하여 목적지를 결정하는 중계 서버를 만들었고, 여기서는 이를 다이나믹 리버스 프록시 서버라고 부르겠습니다.
    

가장 단순한 동작은 아래와 같이 Client가 리버스 프록시 서버(중계 서버)로 요청을 보내면, 중계 서버는 요청의 header를 보고 적절한 서버로 데이터을 전달하는 것입니다. 이후 서버로부터 응답이 돌아오면 클라이언트에게 그대로 응답을 전달합니다.

<img alt="image" src="/images/5db38129-4a48-4d9e-8fce-9fc1cff28f1a"/>

실제로 https 요청을 보내면, 중계 서버는 client-중계서버 사이의 TCP connection을 생성하고, 중계서버-server 사이에도 TCP connection을 연결합니다. 두 connection이 모두 유효하다면 서버와 클라이언트는 모든 TCP packet을 주고받을 수 있습니다.

<img alt="image" src="/images/b647d01a-20a7-4f47-a5fc-dc54d58d93cd"/>

이번에는 클라이언트가 여러 개인 상황을 가정해 보겠습니다. 실제로 많은 웹 페이지들이 여러 개의  port를 통해 서버에 비동기로 데이터를 요청합니다. 이때 중계 서버는 (클라이언트-중계서버)를 연결하는 웹소켓, (중계서버-서버)을 연결하는 웹소켓을 한쌍으로 하여 각 소켓에서 들어오는 패킷을 적절히 중계해야합니다. 

<img alt="image" src="/images/62aa5f73-45e5-4dc5-8b94-8d9dc9bd15c9"/>

또한 서로 다른 Client가 다른 서버로  연결될 수도 있습니다.

<img alt="image" src="/images/e5f894e4-7928-4432-8fb4-4bb9afb367f5"/>

위 상황들을 종합하면 아래와 같은 그림이 나옵니다. 서로 다른 클라이언트들이 각기 다른 서버로 연결되면서, 중계 서버는 클라이언트와 서버를 식별하여, 일치하는 곳으로 데이터를 전달해야합니다.

<img alt="image" src="/images/d7d08003-93df-4fb4-b599-5d23625769b9"/>

# 실험 환경

실험은 고성능 CPU(Intel Xeon Gold 6240R)에서 진행했습니다. 메모리 또한 756GB로 매우 충분해서 하드웨어 제약을 받지 않고 실험할 수 있었습니다. 중계 서버 및 실험 코드는 하나의 도커 컨테이너에서 진행했습니다. 따라서 실제 네트워크 트래픽을 고려하지는 않았습니다. 각 중계 서버가 서브도메인을 해석하여 TCP 연결을 형성하고 데이터를 주고 받는데 걸리는 시간을 측정했습니다. 

- OS: Centos7
- CPU: Intel Xeon Gold 6240R 2.4GHz 48 cores
- RAM: 756GB

# 실험 코드

실험에 사용된 코드는 아래 github에 올라가 있고, dockerfile 및 devcontainer.json 파일을 가지고 있어, vscode를 통해서 환경을 손쉽게 구축하고 실험을 재현할 수 있습니다.

- https://github.com/deagwon97/subdomain-tcp-proxy

프로젝트는 총 3개의 파트로 나뉩니다. 

- Dynamic Reverse Proxy Server 구현 - Go
    - /subdomain-tcp-proxy/proxy-go
    - go로 동작하는 중계 서버입니다.
- Dynamic Reverse Proxy Server 구현 - Node.js
    - /subdomain-tcp-proxy/proxy-nodejs
    - node.js로 동작하는 중계 서버입니다. typescript로 작성되었고, webpack으로 빌드했습니다.
- proxy-test
    - /subdomain-tcp-proxy/proxy-test
    - 실험을 위한 코드입니다. go와 bash 스크립트로 작성했습니다. 테스트를 위해서 만든 서버는 간단한 웹소켓 서버입니다. 요청이 들어오면 들어온 요청을 그대로 반환합니다.

## 테스트 코드

 아래는 테스트코드의 로직을 보여주는 수도 코드입니다. 먼저 중계 서버를 실행하고, 주어진 수 만큼 목적지 서버를 go routine으로 실행합니다. 서버가 실행될 때까지 기다렸다가, NUM_OF_SERVER * NUM_OF_CLIENT 만큼 클라이언트를 만들어 각 클라이언트 마다 NUM_REPEAT 회 데이터를 전송합니다. 모든 데이터를 보내고 받는데 까지 걸리는 시간을 측정해서, 각 중계 서버마다 성능을 비교했습니다. 

```go
중계_서버_실행()

for i := range NUM_OF_SERVER {
    app_host := "0.0.0.0:app포트"
    암호화된_문자열 = 암호화("app-host")
    /etc/hosts 파일에 "0.0.0.0   암호화된_문자열.service.com:중계서버포트" 문장 추가
    (비동기 동작) {
       서버 "app-host" 실행
    }
}
    
모든_서버가_실행될_때까지_대기()

시작시간 := now()

for  i := range NUM_OF_SERVER {
    for j:= range NUM_OF_CLIENT {
        (비동기 동작){
            클라이언트 실행
            for k := range NUM_REPEAT{
                중계 서버로 DATA_SIZE 만큼의 데이터 전송
            }
        }
        클라이언트가 모든 응답을 받을 때 까지 대기
    }
}

끝시간 := now()

출력(종료시간 - 시작시간)
```

# 실험 결과

가장 먼저 연결가능한 최대 서버 수를 측정했습니다. 서버당 클라이언트 수는 1개, 데이터 사이즈는 TCP 패킷의 최소 크기인 20 Byte 로 고정하고 서버수를 500개 부터 500개씩 증가 시키면서 7000개까지 실험했습니다.

| NUM_OF_CLIENT | NUM_OF_REPEAT | DATA_SIZE(Byte) |
| --- | --- | --- |
| 1 | 1 | 20 |

### Go

- 500 ~ 5000 구간 : 모든 데이터 교환 성공
- 5000 ~ 5500 구간: 일부 데이터 손실
- 6000 ~ 구간: 중계 서버 에서 오류 발생

### Node.js

- 500 ~ 4000 구간: 모든 데이터 교환 성공
- 4000 ~ 구간: 중계 서버에서 오류 발생

서버 수가 증가할 때, Go가 Node.js에 비해서 더 안정적인 성능을 보여주고 있습니다.  

## SERVER 수에 따른 데이터 전송 시간(초)

가장 먼저 서버당 클라이언트 수, 반복 수, 데이터 크기를 다음과 같이 고정하고 서버 수가 증가할 때 중계 서버의 성능을 비교했습니다. 서버당 클라이언트 수를 10개로 고정하고 전체 커넥션의 오류가 발생하지 않는 구간인 0 ~ 3000 개 구간에서 실험을 진행했습니다. 예를 들어 클라이언트 10개 서버 275개는 총 2750개의 connection을 형성합니다.

| NUM_OF_CLIENT | NUM_OF_REPEAT | DATA_SIZE(Byte) |
| --- | --- | --- |
| 10 | 1000 | 1024 |

아래 보이는 바와 같이, Go가 Node에 비해서 전반적으로 빠르게 데이터를 처리합니다.


<img alt="image" src="/images/820eee52-83b9-4a90-b93b-55ab2ae9d79d"/>

| NUM_OF_SERVER | GO | NODEJS |
| --- | --- | --- |
| 50 | 1.814738 | 3.191617 |
| 75 | 2.645334 | 3.505688 |
| 100 | 3.541519 | 3.379056 |
| 125 | 3.936755 | 4.525397 |
| 150 | 5.129159 | 6.366753 |
| 175 | 5.92851 | 7.470977 |
| 200 | 6.180126 | 8.649865 |
| 225 | 6.911448 | 8.355023 |
| 250 | 7.897144 | 9.623627 |
| 275 | 7.972524 | 7.972524 |

## CLIENT 수에 따른 데이터 전송 시간(초)

두번째는 다음과 같이 서버 수, 반복 수, 데이터 크기를 고정하고 클라이언트 수가 증가할 때, 성능 비교입니다.

| NUM_OF_SERVER | NUM_OF_REPEAT | DATA_SIZE(Byte) |
| --- | --- | --- |
| 10 | 1000 | 1024 |

서버 수를 늘렸을 때와 동일하게, Go가 Node.js 보다 근소하게 높은 성능을 보입니다. 특히, 클라이언트 수가 250개 ~ 275개 구간에서는 Go가 두배 이상 빠른 처리속도를 보여주고 있습니다.


<img alt="image" src="/images/da233158-9f08-438e-a1f7-48fa3a7d172f"/>



NUM_OF_CLIENT|GO|NODEJS
| --- | --- | --- |
50	|1.792352	|2.651158
75	|2.414255	|3.357894
100	|3.011296	|3.624805
125	|4.343298	|3.347146
150	|4.332794	|7.49678
175	|5.40624	|5.063743
200	|5.51915	|7.94199
225	|6.48191	|7.948503
250	|8.255354	|16.593525
275	|8.656713	|16.408483



# 결론

 Goroutines는 가벼운 스레드와 같은 동시성 모델을 사용합니다. 많은 수의 고루틴을 동시에 실행할 수 있어 동시성 처리에 매우 효율적이라고 알려져있습니다. 또한 I/O 바운드 작업뿐만 아니라 CPU 바운드 작업에서도 우수한 성능을 보이기 때문에 docker, kubernetes, traefik, prometheus 등 다양한 프로그램들을 만들 때 사용됩니다. 

하지만, 단일 스레드에서 동작 함에도 불구하고 이벤트 루프를 사용하여 비동기 I/O를 처리하는 Node.js가 Go에 비해서 유사한 성능을 보입니다. 이는 Go에서 connection을 연결할 때 마다 Goroutine을 생성하는데,  Goroutines을 생성할 때 발생하는 overhead가 영향을 주는 것으로 추정됩니다. 

전체적인 성능에서 큰 차이가 없지만, 안정성 측면에서 더 우수한 성능을 보이는 go dynamic reverse proxy 서버를 중계 서버로 사용하기로 결정했습니다.

# Reference

- https://go.dev/doc/
- https://go.dev/doc/effective_go#goroutines
- https://pkg.go.dev/io
- https://www.npmjs.com/package/websocket
- [https://ko.wikipedia.org/wiki/리버스_프록시](https://ko.wikipedia.org/wiki/%EB%A6%AC%EB%B2%84%EC%8A%A4_%ED%94%84%EB%A1%9D%EC%8B%9C)
- Forouzan, Behrouz A. *Data Communications and Networking.* Publisher, year.



