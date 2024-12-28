---
layout: post
thumbnail: 2e11df41-2b4b-4def-9ad7-579bd5ce456b
title: "[네트워크] 통신을 중계하는 proxy, reverse proxy, gateway"
createdAt: 2023-07-26 10:45:52.660000
updatedAt: 2023-07-26 10:45:52.660000
category: "네트워크"
---



서버와 서버, 혹은 서버와 클라이언트 사이에서 정보를 교환할 때 이를 중계하는 무언가가 존재한다. 이에 대해 알아보자.

### Inbound Traffic vs Outbound Traffic

<img alt="image" src="/images/2e11df41-2b4b-4def-9ad7-579bd5ce456b"/>

- Inbound Traffic : 인터넷에서 서버로 들어오는 네트워크 요청
- Outbound Traffic : 클라이언트가 인터넷으로 보내는 네트워크 요청

## Proxy server
Proxy Server는 **Client**와 **Resource를 제공하는 Server**사이를 **중계**하는 서버를 말한다.

<img alt="image" src="/images/92ccf219-fd1e-441c-a9ce-815f9c0b229a"/>

## Forward Proxy

<img alt="image" src="/images/807b7d50-e4fc-475e-8b21-000015c06f3b"/>

- Outbound Traffic에서 외부 네트워크 와 Client 사이에 붙어 요청을 중계하는 방식을 Forward Proxy라고 한다.
- 즉 Client의 Out-bound 에 관여한다.

## Reverse Proxy


<img alt="image" src="/images/83ec1a53-ac6c-4d28-8a7f-1398dddac764"/>

- Resource 를 제공하는 Server로 들어오는 요청 중 외부 네트워크와 Server 사이에 존재하며 요청을 중계하는 방식을 Reverse Proxy라고 한다.
- 즉 Server의 In-bound 에 관여한다.

## Gateway

**Gateway** 는 하나의 네트워크에서 다른 네트워크로 데이터가 전달될 수 있도록 하는 하드웨어 혹은 소프트웨어이다. 변경되지 않은 정보를 전달하는 프록시 서버를 Gateway 혹은 Tunneling Server라고 한다.

## Router

 Proxy server와 Router는 데이터를 중계한다는 공통점이 있지만, 둘은 서로 다른 계층에서 동작한다. 우선 proxy server는 4 layer(transper 계층) 혹은 그 이상(대부분 application 계층)에서 동작한다. 하지만 Router는 3 layer(network 계층)에서 동작하며 네트워트 사이의 packet 교환을 담당한다.

## Reference

- [https://www.imperva.com/learn/performance/reverse-proxy/](https://www.imperva.com/learn/performance/reverse-proxy/)
- [https://superuser.com/questions/813416/difference-between-proxying-and-routing](https://superuser.com/questions/813416/difference-between-proxying-and-routing)
- [https://en.wikipedia.org/wiki/Router_(computing)](https://en.wikipedia.org/wiki/Router_(computing))
- [https://en.wikipedia.org/wiki/Gateway_(telecommunications)](https://en.wikipedia.org/wiki/Gateway_(telecommunications))
- [https://en.wikipedia.org/wiki/Proxy_server](https://en.wikipedia.org/wiki/Proxy_server)
