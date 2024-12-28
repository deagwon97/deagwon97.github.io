---
layout: post
thumbnail: f8a0e261-5685-4b2a-8432-7d42efe0af93
title: "[Go] _test.go는  import 할 수 없다. error (no required module provides package)"
createdAt: 2023-07-26 10:59:51.735000
updatedAt: 2023-07-26 10:59:51.735000
category: "Go"
---



<img alt="image" src="/images/f8a0e261-5685-4b2a-8432-7d42efe0af93"/>


요즘 테스트 코드를 열심히 작성하고 있다.

그런데 테스트 코드 사이에서 코드의 중복이 발생하는 경우가 종종 발생하였다.

이를 태면,

사용자가 컨텐츠를 만드는 서비스를 생각해보자.

사용자를 생성하는 api는 

1. 더미 사용자 데이터 생성
2. **사용자 생성 api 호출**
2. 반환값 조사

와 같이 테스트 할 수 있고,

컨텐스 생성 api는 

1. 더미 사용자 테이터 생성
2. 사용자 생성
3. 더미 컨텐츠 데이터 생성
4. **컨텐츠 데이터 생성 api 호출**
5. 반환값 조사

와 같이 테스트 한다.


여기서 두 테스트 코드 모두 "더미 사용자 데이터 생성" 이라는 동작을 수행하고, 이는 완전히 동일한 동작이다.

나는 사용자를 관리하는 account의 test package에서 더미 사용자 데이터를 생성하고 이를 content의 test packge에서 import하여 사용하려고 하였다.

하지만 내가 사용하는 vscode의 린터는 다음과 같은 에러를 반환하였고,
``````
could not import test_project/account/test (no required module provides package "test_project/account/test"
``````
혹시나 하여 go run test . 로 실행을 해도, build 오류가 발생하여 go에서는 _test.go 파일을 다르게 관리하는지 알아봤다.

go의 git에 나와 같은 이유로 issue를 제기한 사람이 있었다.

> proposal: testing: allow importing API from _test.go functions in other tests - https://github.com/golang/go/issues/39565



결론부터 이야기하면

이 **issue는 거부**당했다.

 test코드가 다른 코드에의해 참조를 받는 상황을 막기 위해 go에서 자체적으로 이를 막은 기능을 추가한 것으로 보인다.
 
 test코드의 일부를 외부에서 참조하고 싶으면, **common package**로 따로 빼서 관리하라고 한다.
 
 
 내 상황에서는 코드의 중복이 크지 않다고 판단하여, "더미 사용자 테이터 생성"하는 함수는 사용자 테스트 패키지와 컨텐츠 패키지 둘 다 만들어 사용하였다.

 앞으로 코드 중복이 심해지면, 더미 데이터를 생성하는 함수들은 다른 packege로 빼서 관리해야겠다.

 

