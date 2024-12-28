---
layout: post
thumbnail: 9f3cf486-00a9-4650-b25f-c199aba13389
title: "Race Condition"
createdAt: 2023-07-26 10:38:06.506000
updatedAt: 2023-07-26 10:38:06.506000
category: "운영체제/컴퓨터구조"
---
 Mutli Task를 수행하다보면 동일한 자원에 서로다른 task가 접근하는 상황이 생길 수 있다. 이때 Race Condition 혹은 Dead Lock과 같은 문제가 발생하기도 한다.
 잘알려진 자료구조인 Linked List를 통해서 Race Condition과 Dead Lock에 대해 알아보자.
 
## Race Condition
 > _Race Condition(경쟁 상태)이란 공유 자원에 대해 여러 개의 프로세스가 동시에 접근을 시도할 때 접근의 타이밍이나 순서 등이 결과값에 영향을 줄 수 있는 상태를 말한다.
 wikipedia_
 
### Linked List
 Linked List란 하나의 노드가 값과 다음 노드의 주소를 가지고 있는 자료구조이다. 

<img alt="image" src="/images/455302c4-30ec-4ecd-9d79-99008711acb3"/>

 Linked List의 중간에 Node B를 추가하기 위해서는
 
 1. Node B를 생성하고 Head Node의 다음 노드 주소를 Node B로 변경.
 2. Node B의 다음 노드 주소를 Node A로 변경.
 
과 같은 작업이 필요하다.

<img alt="image" src="/images/21d576e0-2803-4941-9507-0410ff76999e"/>

 
### Linked List - MultiThread


<img alt="image" src="/images/dda55b3a-be2d-4c43-9f7b-bff6a02cf9ef"/>
 
 이렇게 Linked List 중간에 노드를 추가하는 작업을 2개의 Thread를 통해서 수행한다고 가정해 보자. (Thread_1는 Node B를 Thread_2는 Node C를 추가) 이때 Thread_1과 Thead_2는 동일한 메모리자원에 번갈아 가면서 동작을 수행한다. 
 
 **Thread_1**의 입장에서
 (1) List Head의 "다음 노드주소"칸에 Node B의 주소를 입력
 (2) Node B의 주소칸에 Node A의 주소값를 입력
 
 **Thread_2**의 입장에서는
 (a) List Head의 "다음 노드 주소"칸에 Node C의 주소를 입력
 (b) Node C의 주소칸에 Node A의 주소값을 입력

 이때, 동작(1)과 동작(a)는 둘다 **_List Head의 "다음 노드 주소칸"_** 이라는 자원을 사용한다. 여기서 동작(1), 동작(a)가 실행되는 순서에 따라 다른 결과가 나온다.
 
 동작(a)가 먼저 실행되서 Head의 다음 노드 주소칸에 Node C의 주소가 들어가고,

<img alt="image" src="/images/2b8da6a7-ae2d-4149-962f-9438c17c54e2"/>

  동작(1)이 나중에 수행되면서  Head의 다음 노드 주소칸에 Node B가 들어가면 Node C를 가리키는 Node가 사라진다.


<img alt="image" src="/images/9f3cf486-00a9-4650-b25f-c199aba13389"/>
 
 이처럼 공유 자원에 대해 여러 개의 프로세스가 동시에 접근을 시도할 때 접근의 타이밍이나 순서 등이 결과값에 영향을 줄 수 있는 상태를 **Race Condition**이라고 부른다.
 
### Atomic Operations

만약 동작(1-2)와 동작(a-b)를 각각 묶어서 둘 사이에 다른 동작을 수행하지 못하도록 했다면 어떻게 됐을까?

<img alt="image" src="/images/b48f0ae3-0d8a-438c-83ea-8459053d7af9"/>

Node B가 완전히 추가되고 Node C를 추가하면 위와 같이 모든 모드들이 정상적으로 추가될 수 있다.
이처럼 특정 작업이 수행되는 동안 다른 작업이 수행되지 못하도록 막을 경우 Race Condition을 해결할 수 있다. 이러한 작업을 Atomic Operation이라고 부른다.

다음 포스트에서 Race Condition을 방지할 수 있는 방법에 대해서 다루겠다.

### Reference 
- System Programing 수업의 강의자료를 발췌하여 정리하였다.
- Seong Jong Choi chois@uos.ac.kr Multimedia Lab. Dept. of Electrical and Computer Eng. University of Seoul Seoul, Korea
