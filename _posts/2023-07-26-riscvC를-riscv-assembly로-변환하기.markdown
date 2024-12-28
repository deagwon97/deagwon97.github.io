---
layout: post
thumbnail: 9b8973dc-1414-45c6-8669-1cbb9df1e431
title: "[risc-v]C를 risc-v assembly로 변환하기"
createdAt: 2023-07-26 11:03:35.754000
updatedAt: 2023-07-26 11:03:35.754000
category: "운영체제/컴퓨터구조"
---
## Assembly Language란?
> 어셈블리어(assembly language) 또는 어셈블러 언어(assembler language)는 기계어와 일대일 대응이 되는 컴퓨터 프로그래밍의 저급 언어이다. (wiki)

어샘블리어를 소개하기 전에 개발자가 C언어로 작성한 소스코드가 실제로 어떻게 실행되는지 알아보자. C언어 프로그램은 다음과 같은 순서로 작성되고 실행된다.

1. 소스코드 작성(C언어)
개발자는 C언어 문법에 맞게 프로그램을 작성한다.
``````c
int s = 0; 
for (i=0; i<10; i++) {
    if (A[i]==0) continue;
    s += A[i];
}
``````
*  하지만, CPU는 이 C언어 코드를 직접 이해할 수 없다. 따라서 컴파일러라는 도구를 통해 CPU가 이해할 수 있는 언어로 컴파일한다.

2. 컴파일
컴파일러가 컴파일하고자 하는 CPU의 아키텍처에 맞는 기계어를 생성한다.
``````c
0x00000000		0x000009B3
0x00000004		0x00000A33
0x00000008		0x10000A97
0x0000000C		0xFF8A8A93
0x00000010		0x00A00E13
0x00000014		0x03CA5063
0x00000018		0x002A1513
0x0000001C		0x01550533
0x00000020		0x00052483
0x00000024		0x00048463
0x00000028		0x009989B3
0x0000002C		0x001A0A13
0x00000030		0xFE0002E3
``````

3. 컴파일된 기계어를 CPU가 실행
CPU는 생성된 기계어를 순차적으로 읽으면서 프로그램을 실행한다.
CPU는 register, ALU, MUX등으로 구성되어 있는데, 들어오는 기계어 instruction set이 아래 그림처럼 순차적으로 fetch, decode, execute 되면서 프로그램이 동작한다.(이 구조는 다음 포스트에서 자세히 다루겠다.)

<img alt="image" src="/images/9b8973dc-1414-45c6-8669-1cbb9df1e431"/>

### 그렇다면 여기서 어셈블리어란 무엇일까?
> 어셈블리어는 기계어와 일대일 대응이 되는 low level 프로그래밍 언어이다.

따라서 컴파일러가 컴파일한 기계어는 어셈블리어와 일대일로 대응한다. 만약 개발자가 어셈블리어를 통해 프로그램을 작성할 경우, 프로그램의 동작을 instruction set 수준에서 완벽하게 제어할 수 있다.

``````asc
0x00000000		0x000009B3		add x19 x0 x0
0x00000004		0x00000A33		add x20 x0 x0
0x00000008		0x10000A97		auipc x21 65536
0x0000000C		0xFF8A8A93		addi x21 x21 -8
0x00000010		0x00A00E13		addi x28 x0 10
0x00000014		0x03CA5063		bge x20 x28 32
0x00000018		0x002A1513		slli x10 x20 2
0x0000001C		0x01550533		add x10 x10 x21
0x00000020		0x00052483		lw x9 0(x10)
0x00000024		0x00048463		beq x9 x0 8
0x00000028		0x009989B3		add x19 x19 x9
0x0000002C		0x001A0A13		addi x20 x20 1
0x00000030		0xFE0002E3		beq x0 x0 -28
``````

### C를 RISC-V 어셈블리로 변환하기
위의 c코드의 동작을 설명하면 다음과 같다.

1. s라는 변수를 선언하고, 값을 0으로 초기화.
``````c
int s = 0;
``````
2. i라는 변수 선언, 0으로 초기화, i가 10보다 크거나 같으면 반복문 탈출, 반복문이 끝날 때, i값은 +1.
``````c
for (i=0; i<10; i++) { 
``````
3. 배열 A의 i번 째 값이 0이라면 continue.
``````c
    if (A[i]==0) continue;
``````
4. s값에 배열 A의 i번 째 값을 더함.
``````c
    s += A[i];
``````
``````c
}
``````

이제 s라는 변수는 x19, i는 x20, A의 주소는 x21이라는 레지스터에 각각 저장되어 있다 가정하고, 다음과 같이 어셈블리 코드를 작성할 수 있다.

``````
        add x19 x0 x0    // s = 0
        add x20 x0 x0    // i = 0
        addi x28 x0 10   // x28 = 10
LOOP:   bge x28 x20 EXIT // if 10 >= i; GOTO EXIT
        slli x10 x20 2   // x10 = i * 4
        add x10 x10 x21  // x10 = &A[0] + i*8
        lw x9 0(x10)     // x9 = A[i]
        beq x9 x0 L1     // if A[i] == 0; GOTO L1
        add x19 x19 x9   // s = s + A[i]
L1  :   addi x20 x20 1   // i = i + 1
        beq x0 x0 LOOP   // GOTO LOOP
EXIT:
``````

우선 s와 i를 나타네는 레지스터를 0(x0)로 초기화 하였다. bge는 상수를 비교하지 않고, register에 저장된 값을 비교하기 때문에, x28이라는 임시 레지스터를 만들어 숫자 10을 저장하였다. 3~4번 째 줄과, 뒤에서 3~2번 째 줄을 통해서 ``for (i=0; i<10; i++)``가 구현된다. 
32bit riscv를 사용한다고 할 때, A\[i]의 값은 다음과 같은 방식으로 load한다.

1. i의 값 * 4 => x10에 저장
	32bit riscv는 4byte가 1 워드이다. 따라서 배열의 index에 4를 곱해야 index가 의미하는 배열의 간격을 구할 수 있다.
2. A의 시작 주소 +  x10  => x10에 저장
	배열의 시작 주소에 간격을 더하여 실제 값이 저장된 주소값을 구한다.
3. x10에 저장된 메모리상을 주소의 값을 통해 메모리에 접근해 값을 불러와 x9레지스터에 저장
``````
slli x10 x20 2   // x10 = i * 4
add x10 x10 x21  // x10 = &A[0] + i*8
lw x9 0(x10)     // x9 = A[i]
``````

### 시뮬레이션
Vscode의 RISC-V Venus Simulator Extention을 활용하여 작성한 assembly 코드를 시뮬레이션할 수 있다. (Venus simulator는 32bit riscv을 지원하기 때문에 load word를 사용하였다.)

<img alt="image" src="/images/833675e6-0ea5-4d6e-b105-c465174ab7e6"/>

위의 코드를 실행하면, 

<img alt="image" src="/images/9bb89ed6-c238-4e37-b6a6-9b47b7a0d057"/>

Loop를 반복하면서 x9, x19  레지스터에 적절한 값이 저장되고, loop가 예상한 순간 종료되는 모습을 확인할 수 있다.

### Reference
- https://ko.wikipedia.org/wiki/%EC%96%B4%EC%85%88%EB%B8%94%EB%A6%AC%EC%96%B4
- Computer Organization and Design RISC-V edition
- 컴퓨터 구조 수업자료, 이영민 교수님, 서울시립대학교

