---
layout: post
thumbnail: 3e2d1625-afaa-46a0-a491-5ac098cb3ed5
title: "Multi Task, Multi Processor"
createdAt: 2023-07-26 10:36:05.239000
updatedAt: 2023-07-26 10:36:05.239000
category: "운영체제/컴퓨터구조"
---
## I. "작업"과 "자원"의 의미
컴퓨터는 자원으로 작업을 수행하는 장치이다. 자세히 알아보자.


### 1. 작업이란?
> _Task란 "실행" 혹은 "일"의 단위이다. Linux Kernel의 Resource Shareing관점에서 Tasks는 Process, Thread으로 간주될 것이다.
-wikipedia_

 Process, Thread 또한 정의된 "작업"이다. (나는 처음 공부할 때, Process와 Thread가 일을 하는 자원이라고 오해하여 많이 헤맸던 경험이 있다.)

### 2. 자원이란?
Computer의 자원은 Computer가 동작하기 위해 필요한 모든 물리적 장치를 의미한다. 하지만 이번 포스트에서는 이 자원중에서도 Processor에 대해 다루겠다.
> Processor (Processing Unit)은 메모리 혹은 다른 외부 데이터 자원에 대한 작업을 수행하는 논리 회로를 의미한다.
-wikipedia

### 3. 그래서..?

 정리해보면 **컴퓨터는 "Process & Thread"라는 작업을 생성하고 CPU라는 자원이 이 작업을 수행하는 장치이다.**

**자원**의 개수에 따라
  - Single Processor System
  - Multi Processor System
  
이 분리되고

**작업**의 개수가 하나 이상인 경우
- multi Programing
- multi Tasking
- multi processing
- multi Threading

등이 존재한다.

 만약 여러분이 최근에 나온 컴퓨터로 Chrome탭을 여러개 실행하고 있다면,
여러분은 Multi processor System에서 Multi Processing을 수행하고 있으며, 동시에 각 Process마다 Multi Threading이 수행되고 있다.

## II. Multi task, Multi processor

### 1. Single Core, Single Process, Single Thread

<img alt="image" src="/images/5cf4beab-a158-461e-bdba-25d80edfff20"/>

가장 단순한 경우이다. **1개의 작업을 1개의 자원이 수행**한다. Scheduling같은 작업이 필요하지 않다.

### 2. Single Core, Multi Process, Single Thread

<img alt="image" src="/images/bdf403f8-1a48-4614-80bc-64efa082eb32"/>

- CPU의 Core가 1개이고 이 Core는 동시에 1개의 Thread만 실행이 가능하다.
- 각 프로세스마다 실행 시간을 할당(Scheduling)하고 번갈아가면서 수행(Context Switching)해야 한다.
- Scheduling을 통해 **CPU Protection**을 수행한다.
- Process는 다른 I/O, Memory의 자원을 커널의 도움 없이 접근할 수 없다. (**Memory/IO Protection**)
- Process 수준의 Context Switching이 발생하기 때문에 **Switch overhead**가 높다. (PCB의 정보를 모두 저장 Load해야한다.)

### 3. Single Core, Multi Process, Multi Thread


<img alt="image" src="/images/8ee519c4-03d7-41ed-8b8b-0a548f8abf55"/>

- CPU의 Core가 1개이고 이 Core는 동시에 1개의 Thread만 실행이 가능하다.
- 각 쓰레드마다 실행 시간을 할당(Scheduling)하고 번갈아가면서 수행(Context Switching)해야 한다.
- Scheduling을 통해 **CPU Protection**을 수행한다.
- **동일한 Process** 안에서 Context Switching이 발생하는 경우
  - 일부 자원을 공유하기 때문에 **Switch overhead가 낮다**. (TCB의 정보만 변경한다.)
  - 다른 Thread의 일부 I/O, Memory의 자원에 접근할 수 있다. (**Memory/IO Protection이 지켜지지 않는다**.)
  - 새로운 Thread를 생성하는 비용은 새로운 Process를 생성하는 비용에 비해 저렴하다. (**low thread creation**)

### 4. Multi Core, Multi Process, Multi Thread

<img alt="image" src="/images/2ac5e857-9b37-4697-a4e4-e38a3cde87b7"/>

- CPU의 Core가 여러개이고, 각각의 Core는 동시에 1개의 Thread만 실행이 가능하다.
- 각 쓰레드마다 실행 시간을 할당(Scheduling)하고 번갈아가면서 수행(Context Switching)해야 한다.
- Scheduling과 다른 core의 실행을 통해 **CPU Protection**을 수행한다.
- **동일한 Process** 안에서 Context Switching이 발생하는 경우
  - 일부 자원을 공유하기 때문에 **Switch overhead가 낮다**. (TCB의 정보만 변경한다.)
  - 다른 Thread의 일부 I/O, Memory의 자원에 접근할 수 있다. (**Memory/IO Protection이 지켜지지 않는다**.)
  - 새로운 Thread를 생성하는 비용은 새로운 Process를 생성하는 비용에 비해 저렴하다. (**low thread creation**)
 
### 5. Multi Core, Multi Process, Multi Thread And Hyper Threading

 CPU내부에서는 부동소수점 처리 코어인 FPU와 정수 연산 코어인 ALU가 존재한다. 대부분의 경우 FPU가 덜 사용된다. 이를 활용하여 가상 쓰레드를 하나 더 만들어 하나의 코어가 하나 이상의 쓰레드를 동시에 실행할 수 있도록하는 기술을 하이퍼 쓰레딩이라고 한다.(명확하지 않다. 더 정리가 필요하다.) 
 
 하이퍼 쓰레딩 기술이 하나의 코어를 완전히 2개의 코어로 만들지는 못하지만 개념적으로 하나의 코어가 여러개의 쓰레드를 수행할 수 있다.


<img alt="image" src="/images/3e2d1625-afaa-46a0-a491-5ac098cb3ed5"/>

- FPU와 ALU가 경합하는 상황이 발생할 경우 성능이 저하될 수 있다.
- 동일한 Core 내에서 Thread의 Context Switchng을 할 경우, 매우 낮은 비용의 Switch overhead가 발생한다.


## Reference
- https://www.quora.com/How-does-multithreading-work-in-a-single-core-computer
- https://stackoverflow.com/questions/16116952/can-multithreading-be-implemented-on-a-single-processor-system
- https://en.wikipedia.org/wiki/Task_(computing)
- https://levelup.gitconnected.com/how-web-browsers-use-processes-and-threads-9f8f8fa23371
- https://coolenjoy.net/bbs/27/56356


_(위 이미지들의 출처는 [Anthony D. Joseph and John Canny CS162 ©UCB Fall 2013]이다.)_
