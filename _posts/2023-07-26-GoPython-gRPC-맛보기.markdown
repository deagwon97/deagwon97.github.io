---
layout: post
thumbnail: 5e8c092c-39ed-460c-ae2a-d5f87f2d5885
title: "[Go][Python] gRPC 맛보기"
createdAt: 2023-07-26 11:01:53.653000
updatedAt: 2023-07-26 11:01:53.653000
category: "백앤드"
---
# gRPC란?


<img alt="image" src="/images/be601982-c404-4b27-921b-2a1d263cd0f1"/>

> gRPC란 구글에서 개발한 **RPC**(remote procedure call) 시스템이다. 

gRPC는 데이터 전송을 위해서 **HTTP/2**를 사용하고, **IDL**(interface description language)로 **Protocal Buffer**를 사용한다. 또한 인증, 양방향 스트리밍, blocking, nonblocking 바인딩 등 다양한 기능을 제공한다.

## RPC(Remote procedure call)
> 분산 시스템에서, 하나의 프로그램이 실행하는 도중 **다른 주소공간**(다른 프로세스, 다른 컴퓨터..)의 **프로시저**(서브루틴)을 실행하는 것을 **RPC**라고 부른다.

RPC 는 **request–response protocol**로 동작한다. 원래 프로그램은 다른 주소공간으로 reqeust를 보내고, 이 요청을 받은 서버는 프로시저를 실행하여 그 결과를 다시 원래 프로그램으로 보낸다(response).

이러한 RPC의 구현체로 
- Google의 **gRPC**
- Facebook의 **Thrift**
- Twitter의 **Finalge**

등 이 있다.


## HTTP/2
 HTTTP/1.1의 단점 중** 프로토콜의 성능에 초첨**을 맞춰 개선한 전송 프로토콜이다. 기존의 HTTP/2는 다음과 같은 개선사항을 갖는다.

* **Multiplexed Streams**
  * **하나의 Connection**에서 동시에 여러 개의 메시지를 주고 받는 기술이다. **순서에 상관없이** Stream으로 응답을 받을 수 있다.
* **Stream Prioritization**
  * **리소스 간의 의존관계**에 따른 우선순위를 설정하여 리소스 로드한다.
* **Server Push**
  * 서버가 클라이언트가 **요청하지 않은 리소스**를 클라이언트에게 보내는 기능이다.
* **Header Compression**
  * Header Table과 Huffman Encoding 기법을 통해 **헤더의 중복을 줄이고, 압축**하는 기능이다.

## protocol buffers
> protocol buffers는 구글에서 공개한 cross-platform 포맷으로, IDL의 한 종류이다. 구조화된 데이터를 언어, 플랫폼에 중립적으로 serializing(데이터를 저장될 수 있는 형태로 변환하는 것)하는 방법을 제한다.

### 간단한 .proto 작성 예시

```protobuf
syntax = "proto3"; // proto3, proto2 중에 사용할 버전 명시

package proto_test; // package 이름 명시

service RpcService { // RPC service 이름 정의(CamelCase with an initial capital)
  rpc 함수명 (입력Message) returns (출력Message);
}

message 입력Message {
  string name = 1;
}

message 출력Message {
  string code = 1;
  string name = 2;
  string symbol = 3;
}
```
 
### IDL(Interface Definition Language)
> IDL은 서로 같거나 다른 소프트웨어 컴포넌트들 사이의 데이터 구조와 인터페이스를 묘사하기 위한 명세 언어이다.

 IDL은 어느 한 언어에 국한되지 않는 언어중립적인 방법으로 인터페이스를 표현함으로써, 같은 언어를 사용하지 않는 소프트웨어 컴포넌트 사이의 통신을 가능하게 한다.
 

## gRPC 구성

<img alt="image" src="/images/5e8c092c-39ed-460c-ae2a-d5f87f2d5885"/>

gRPC는 gRPC 서버와, gRPC클라이언트(stub)로 구성된다.
* gRPC 서버
service의 함수들이 서버의 언어로 정의되어 있어, stub에서 요청이 들어오면 이를 수행한다.

* gRPC클라이언트(stub)
정의된 proto파일에 맞는 변환 함수를 가지고 있다. service에 있는 함수를 호출하려면, stub의 언어로 생성된 데이터를 serializing하여 gRPC서버로 전송하고, 그 결과값을 다시 역 serializing한다.


## gRPC compile
 언어별로 gRPC compiler를 통해 .proto파일을 각 언어 맞는 파일로 변환할 수 있다. 우선 .proto 파일을 작성한 후, 공통된 .proto파일을 통해서 server와 client에서 각각 compile한다.

### 1) .proto 파일 작성

```
syntax = "proto3";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

### 2) server compile
```console
> protoc --proto_path=<.proto파일의 dir경로> \
		 --go_out=plugins=grpc:<정의한 package이름>  \
        .proto파일
```
### 3) client compile

```console
> python -m grpc_tools.protoc \
			--proto_path <.proto파일의 dir경로> \
			--python_out=<출력 .py파일 경로> \
            --grpc_python_out=<출력 .py파일 경로> \
	        <.proto파일경로>
```

## gRPC life cycle

### Uray(단방향) RPC
가장 단순한 RPC이다. Cilent가 요청을 보내고, Server가 그 요청을 받으면 결과 값을 반환하는 구조이다.
### Server streaming RPC
Uray와 비슷하지만, Server가 응답으로 streaming message를 보낸다는 차이가 있다. 서버는 streaming message가 모두 전송될때, 서버의 status혹은 다른 meta data를 추가로 전송한다.
### Client streaming RPC
Uray와 비슷하지만, Cilent가 요청으로 streaming message를 보낸다는 차이가 있다. 서버는 streaming message가 모두 받으면 single response(필수x)를 보낸다.
### Bidirectional streaming RPC
 Client가 특정 함수를 호출하여 Sercer가 client metadata, method name, deadline을 받으면 시작된다. Server와 Client가 서로 동시에 스트리밍 데이터를 주고 받을 수 있고, 이 두 스트림은 독립적이다.


## Example
- https://github.com/deagwon97/grpc-go-python

## Reference
* https://en.wikipedia.org/wiki/GRPC
* https://brownbears.tistory.com/512
* https://chacha95.github.io/2020-06-15-gRPC1/
* https://en.wikipedia.org/wiki/Remote_procedure_call
* https://seokbeomkim.github.io/posts/http1-http2/
* https://yoongrammer.tistory.com/14
* https://developers.google.com/protocol-buffers/docs/overview
* https://www.grpc.io/docs/what-is-grpc/introduction/
