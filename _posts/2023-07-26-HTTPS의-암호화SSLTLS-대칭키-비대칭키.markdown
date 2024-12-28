---
layout: post
thumbnail: 5ae3bacc-46c1-4aa7-905a-71bfbd865ad2
title: "HTTPS의 암호화(SSL/TLS, 대칭키, 비대칭키)"
createdAt: 2023-07-26 10:23:32.560000
updatedAt: 2023-07-26 10:23:32.560000
category: "보안"
---
## 대칭키 암호화

- 대칭키 암호화
    - **암호화 키**와 **복호화 키**가 동일한 키를 사용하는 암호화 방법

## 비대칭키 암호화

- 비대칭키 암호화(공개키 암호화)
    - **암호화 키**와 **복호화 키**가 다른 키를 사용하는 암호화 방법
    - 한쌍의 키는 서로의 역할을 각각 수행할 수 있다.
        - 공개키로 암호화 → 개인키로 복호화
        - 개인키로 암호화 → 공개키로 복호화

## https의 암호화 방법

- 암호화키를 개인키로 하여 서버만 소유한다. (외부에서 알 수 없다.)
- 인증기관이 복호화키(공개키)를 공공에 배포한다.
- 알고리즘

<img alt="image" src="/images/5ae3bacc-46c1-4aa7-905a-71bfbd865ad2"/>
    
    1. TCP 3way handshake
        1. SYN
        2. SYN ACK
        3. ACK
        
 
    2. TLS handshake
        1. Client Hello
            1. 클라이언트가 서버로 요청
            2. Cipher Suite(사용가능한 SSL 프로토콜, 인증서 검정, 데이터 암호화 프로토콜), session id 등등을 전달
        2. Server Hello
            1. Cipher Suite를 결정하여 클라이언트에게 전달
        3. Certificate
            1. 이전에 미리 서버는 CA(인증기관)로부터 암호화된 서버 인증서를 발급받음
                1. CA의 개인키로 암호화
                2. 서버의 공개키도 서버 인증서에 포함
                3. 클라이언트는 CA의 공개키를 통해 암호화된 서버의 인증서를 복호화 하여 서버의 공개키와 인증서를 얻을 수 있다.
            2. 서버의 인증서(인증서 + 서버의 SSL 공개키)를 클라이언트에게 전달
        4. Sever Key Exchange / ServerHello + Done
            1. Sever Key Exchange
                - 서버의 SSL 공개키가 인증서에 들어있지 않는 경우
                - 서버가 키를 직접 전달
            2. ServerHello
                - 서버의 SSL 공개키가 인증서에 들어있는 경우
                - (클라이언트가 CA의 공개키를 통해서 서버의 인증서를 복호화하여 서버의 SSL 공개키를 획득)
            3. Done
                - 서버가 행동을 마쳤음을 클라이언트에게 전달
        5. Client Key Exchange
            1. 클라이언트가 대칭키를 생성하고 서버의 공개키로 이를 암호화
            2. 암호화된 대칭키를 서버에게 전달
        6. ChangeCipherSpec Finished
            1. 클라이언트와 서버가 서로에게 보내는 패킷
            2. 통신준비가 끝났음을 알림
            

※암호화 방식에 따라 대칭키가 아닌 키의 재료를 전달하기도 한다.

## reference
- https://ko.wikipedia.org/wiki/HTTPS
- https://aws-hyoh.tistory.com/39
- https://sleepyeyes.tistory.com/4
- https://100100e.tistory.com/317
