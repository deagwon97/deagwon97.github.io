---
layout: post
thumbnail: db7864d9-3b90-4b4d-94d8-2c5fc7c8be5f
title: "[네트워크] ARP(Address Resolution Protocol)"
createdAt: 2023-07-26 11:17:42.084000
updatedAt: 2023-07-26 11:17:42.084000
category: "네트워크"
---
> ARP는 논리적 주소(IP주소)를 물리적 주소(MAC주소)로 변환하는 프로토콜이다.
> 

ARP는 다음과 같은 상황에서 사용된다.

- 호스트 A가 호스트 B에게 데이터를 전달하려고 한다.
- 호스트 A는 호스트 B의 IP 주소를 알고 있지만 MAC 주소는 모른다.
- Routing Table을 확인해보니, 호스트 B가 자신의 네트워크에 속해 있다는 사실을 알아 냈다.
- 이제 ARP를 통해서 호스트 B의 MAC 주소를 알아낸다.


<img alt="image" src="/images/db7864d9-3b90-4b4d-94d8-2c5fc7c8be5f"/>

## ARP 동작

### 1. ARP Request

호스트A는 네트워크 전체에 IP 주소에 해당하는 MAC 주소를 물어보는 요청을 보낸다. (Broadcast)

### 2. ARP Reply

 본인의 IP주소에 대한 MAC 주소를 요청받은 호스트 B는 자신의 MAC 주소를 담아 응답을 보낸다.

### 3. ARP Cache 업데이트

호스트 A는 이 ARP의 정보를 담고 있는 저장소(ARP Cache)에 IP-MAC 주소 쌍을 저장하고, 일정시간이 지나면 이를 삭제한다.

이런 과정을 거쳐서 호스트 B의 MAC 주소를 알아내면, 호스트 A는 이더넷 패킷을 만들어 스위치로 전달한다. 스위치는 이더넷 패킷 속 MAC 주소를 확인하여 호스트 B에게 정보를 전달한다.

## RARP(Reverce ARP)

반대로 MAC 주소로 부터 IP 주소를 변환할 수도 있다. 이런 작업을 위해서는 RARP 서버를 필요로 하며, ARP 와 유사한 방식으로 진행된다.

### 1. RARP Request

호스트A 는 자신의 IP도 모르고, RARP 서버가 어디있는지도 모른다. RARP 패킷에 자신의 MAC 주소를 담아서 자신과 연결된 네트워크 전체에 RARP Request를 브로드 케스팅으로 보낸다. 

### 2. RARP Reply

RARP 서버가 이 요청을 받으면 호스트 A의 IP 주소를 담아 RARP 응답을 전송한다.

## ARP 스푸핑(ARP Spoofing)

같은 네트워크에 속해있는 서로다른 호스트가 통신하기 위해서는 ARP를 통해서 IP 주소에 대한 MAC 주소를 알아내야한다. 

하지만 이런 ARP Request는 네트워크 내에 위치한 모든 호스트에게 보내기 때문에 보안상 위험이 존재한다. 대표적인 방법으로 ARP 스푸핑을 통해서 공격자는 다른 두 호스트 사이에 주고 받는 데이터를 가로챌 수 있다.

1. ARP를 통해서 B의 IP 주소에 해당하는 MAC 주소를 찾고자 한다.
2. 이때, 만약 공격자 호스트가 네트워크 내부의 모든 호스트 들에게 자신의 B의 IP가 자신의 MAC 주소라는 ARP 응답을 보내게 되면, 호스트 A는 B의 IP에 대한 MAC 주소를 공격자 호스트의 MAC 주소로 착각하게 된다.
3. 그럼 호스트 A는 B가 아닌 공격자에게 데이터를 보내버린다.


<img alt="image" src="/images/186b8e7e-94cb-40a6-9bbe-e250614678f0"/>


<img alt="image" src="/images/b27d96ea-3443-4960-b2fc-dfad08eb3333"/>


<img alt="image" src="/images/5b92a244-0273-4cc7-a4c0-df87b2b6a33b"/>



## Reference

- [https://reakwon.tistory.com/139?category=300675](https://reakwon.tistory.com/139?category=300675)
- [https://aws-hyoh.tistory.com/entry/ARP-쉽게-이해하기](https://aws-hyoh.tistory.com/entry/ARP-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)
