---
layout: post
thumbnail: ""
title: "[개발 상식] status vs state"
createdAt: 2023-07-25 18:33:09.889000
updatedAt: 2023-07-25 18:33:09.889000
category: "기타"
---
State와 Status는 프로그래밍에서 자주 등장하는 단어지만, 둘 다 ‘상태’라고 번역되기 때문에 혼동하기 쉽다. 간단하게 그 차이를 알아보자.

### Status: 일련의 과정 중 결과로서의 상태

 Status는 상태보다는 상황으로 번역하는 것이 이해하기 쉽다. 대표적으로 ``http status`` 라는 단어를 예로 들어보자. http request를 보내면 서버는 응답으로 ``http status code`` 를 반환한다. 이는 서버에서 일련의 과정을 마치고 요청의 결과의 ‘상황’을 반환한 것이다. 이 상황은 ‘성공’, ‘인증되지 않음’, ‘서버오류’ 등일 수 있고, 이 중 하나를 선택해야 한다. 또한 요청의 결과이기 때문에 ‘서버오류’인 status가 ‘성공’으로 바뀌지 않는다.

### State: 특정 시점의 상태

  State가 한국말의 ‘상태’와 더 가깝다. 말 그대로 특정 시점의 상태를 의미한다.  어떤 프로세스는 ``Ready state`` 에 있다가 운영체제에 의해서 ``Run state`` 로 전환된다.  ``Run state`` 는 상황에 따라 ``Wait state``, ``Termination state`` 로 전환될 수 있고, 다시 ``Ready state``로 돌아갈 수도 있다. 

주로 상태 다이어그램에서 말하는 상태는 ‘state’를 의미한다.

- React life cycle state diagram
- Process State Diagram

### Reference

- [https://www.quora.com/What-is-the-difference-between-state-and-status](https://www.quora.com/What-is-the-difference-between-state-and-status)
- [https://w.cublr.com/programming/different-between-state-status/#:~:text=State는 현재 진행 상태,에 사용한다고 보면 되겠다](https://w.cublr.com/programming/different-between-state-status/#:~:text=State%EB%8A%94%20%ED%98%84%EC%9E%AC%20%EC%A7%84%ED%96%89%20%EC%83%81%ED%83%9C,%EC%97%90%20%EC%82%AC%EC%9A%A9%ED%95%9C%EB%8B%A4%EA%B3%A0%20%EB%B3%B4%EB%A9%B4%20%EB%90%98%EA%B2%A0%EB%8B%A4).
