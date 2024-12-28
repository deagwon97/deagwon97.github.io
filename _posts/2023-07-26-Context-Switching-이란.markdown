---
layout: post
thumbnail: 64d4dfdd-a3c1-4040-a3e2-fea5348a96cf
title: "Context Switching 이란?"
createdAt: 2023-07-26 10:32:15.934000
updatedAt: 2023-07-26 10:32:15.934000
category: "운영체제/컴퓨터구조"
---

## 1. Process & Process Control Block

 Process란 하나 혹은 그 이상의 Thread로 실행되는 컴퓨터 프로그램의 instance이다. Process는 자신에 관한 정보를 하나의 데이터 구조에 저장하여 관리하고 이를 Process Control Block이라고 부른다.


## 2. Thread & Thread Control Block
### 1) Tread란?

> _쓰레드란 프로그램 내에서 다른 코드와 독립되어 실행될 수 있는 최소 단위의 instruction Sequence이다.
(A thread is a sequence of such instructions within a program that can be executed independently of other code.)
  -wikipedia_
 
 이해를 돕기위해 부연하자면 운영체제 입장에서 작업의 최소 단위는 Process이고, CPU 입장에서 작업의 최소단위는 Thread이다.
 
### 2) TCB란?
하나의 Thread를 관리하는데 필요한 정보를 담고 있는 구조체이다. 프로세스의 상태를 관리하는 PCB보다 적은 양의 정보가 담겨있다. thread사이의 context switching & process 사이의 context switching을 할 때 CPU scheduling을 하는 최소단위이다. 다음과 같은 정보를 포함한다.

- Thread Identifier: 쓰레드를 구분하는 유일한 식별자
- Stack pointer: 쓰테드 별로 고유한 Stack의 pointer
- Program counter: 현재 instruction의 주소
- 쓰레드의 상태 (running, ready, waiting, start, done)
- Thread's register values
- 쓰레드가 소속된 processor의 PCB주소
 
## 3. Context Switching
 
 ### 1) context란?
 >프로그래밍에서 Context는 (동작, 작업들의 집합)을 (정의, 관리, 실행)하도록 하는 (최소한의 상태, 재료, 속성)을 포함하는 (객체, 구조체, 정보)이다.
 [What exactly is context in programming?](https://www.quora.com/What-exactly-is-context-in-programming?no_redirect=1)
 
 
 Process의 경우 현재 프로세스가 중단 되었을 때, 중단된 시점 부터 다시 프로세스를 실행하기 위한 정보를 Context라고 부른다. 이러한 Process의 Context 정보는 PCB(Proccess Control Block)이라는 구조체에 저장된다.
 
 ### 2) scheduling이란?
 
 > 스케쥴링은 "자원"에 "작업"을 할당하는 행위이다.
 -wikipedia
  
"**자원**"은 프로세서(Processor), 네트워크 연결, 외부 장치 등을 의미하고, "**작업**"은 thread, process 혹은 data flows를 의미한다.
 스케쥴링은 **scheduler라는 프로세스**에 의해 수행되며, 모든 자원이 골고루 사용될 수 있게 디자인된다. 

### 3) Context Switching이란?

 > 작업의 주체가 현재 Context를 잠시 중단하고 다른 Context를 실행하는 것을 Context Switching이라 한다.
 
 
 CPU의 코어가 1개(자원이 1개)라면 동시에 단 하나의 프로세스만 실행이 가능하다. CPU scheduling을 통해서 하나의 CPU를 여러 작업들이 공유할 수 있게 cpu 시간을 나누어 작업을 수행한다. 
 
 이때, 프로세서가 지금까지 실행되던 프로세스(A)를 중지하고 다른 프로세스(B)의 PCB정보를 바탕으로 프로세스(B)를 실행하는 것을 Process Context Switching이라고 한다.
 
 동일한 프로세스 속에서 하나의 쓰레드(a)를 중지하고 다른 쓰레드(b)의 TCB정보를 바탕으로 쓰레드 (b)를 실행하는 것을 Thread Context Switching이라고 한다.
 
 참고로, Process는 하나이상의 Thread로 동작하기 때문에 Context Switching의 최소 단위는 TCB이다.
 
 ### 4) process context switching vs thread context switching
 
 프로세스는 하나 이상의 쓰레드를 포함한다. 이 쓰레드들은 고유한 Stack영역의 메모리와 고유한 registers를 할당 받으며 Heap영역의 메모리에서 선언된 데이터는 서로 공유한다.
 동일한 프로세스 속에서 thread context switching이 발생할 경우 processor는 stack영역의 주소와 registers 주소를 포함한 thread의 context 정보만을 변경하면 된다.
 하지만 process context switching이 발생할 경우 processor는 thread의 context뿐만 아니라 process의 context까지 모두 변경해야 한다. 
 

<img alt="image" src="/images/64d4dfdd-a3c1-4040-a3e2-fea5348a96cf"/>


## Refrence
- https://velog.io/@cchloe2311/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-Context-Switching
- https://m.blog.naver.com/adamdoha/222019884898
- Operating System Concepts – 9th Edition
- https://en.wikipedia.org/wiki/Thread_control_block
- https://en.wikipedia.org/wiki/Thread_(computing)
- https://devlog-wjdrbs96.tistory.com/262
- https://teraphonia.tistory.com/802
