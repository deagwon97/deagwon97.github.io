---
layout: post
thumbnail: 7b961ede-3090-4e24-9550-e4cc020b1afc
title: "[Go] Embedding"
createdAt: 2023-07-26 11:25:45.315000
updatedAt: 2023-07-26 11:25:45.315000
category: "Go"
---
## 상속과 컴포지션

<img alt="image" src="/images/7b961ede-3090-4e24-9550-e4cc020b1afc"/>


 이미 정의한 타입(무언가)을 재활용하여 새로운 타입(무언가)을 만드는 방법으로 **상속**과 **컴포지션**이 있다. 많은 언어에서 이 두 가지를 지원한다. 

- **상속**: 하위 클래스가 상위 클래스의 특성을 재정의, (IS-A) 관계
- **컴포지션**: 하위 클래스가 상위 클래스를 포함, (HAS-A) 관계

 많은 언어가 이미 정의된 타입을 참고하여 새로운 타입을 정의할 때, 상속을 주로 사용하였다. 하지만 이러한 상속은 몇 가지 문제점을 가지고 있다.

- Open Close 원칙을 위반할 수 있다.
    - 객체지향프로그래밍은 기존의 코드를 클래스를 수정하지 말고, 기능을 추가하는 방식으로 프로그래밍하는 것을 지향한다. 하지만, 상속은 오버라이딩을 통해서 부모 클래스의 함수를 “수정”할 수 있으며, 이는 개방폐쇄 원칙을 위반한다.
- 캡슐화를 위반할 수 있다.
    - 자식 클래스가 부모 클래스를 수정하는 경우, 부모 클래스의 내부 동작에 접근하여 동작을 수정하는 경우가 생긴다. 이는 캡슐화를 위반한다.

 컴포지션은 원래 클래스를 수정할 수 없기 때문에 개방폐쇄 원칙을 강제한다. 또한  원래 클래스가 캡슐화를 잘 지켜서 만들었다면, 이를 포함하는 새로운 클래스는 원래 클래스의 내부동작에 간섭하지 못함으로 캡슐화 또한 지켜진다. 많은 개발자들이 상속의 문제점에 공감하였고, 따라서 Go는 상속기능을 지원하지 않고, 오로지 컴포지션(Go Embedding) 기능만을 지원하도록 설계되었다. 

``````go
type Base struct {
	Name  string
	Value int
}

type EmbedsBase struct {
	Base
	Other string
}
``````

## 타입 임베딩 vs 포인터 임베딩

임베딩을 할 때 두 가지 방법이 존재한다. 하나는 타입을 임베딩하는 방식이고, 다른 하나는 타입의 포인터를 임베딩하는 방식이다.

### 타입 임베딩

EmbedsBase 는 Base의 Name과 Value를 모두 가지는 새로운 타입이다.

``````go
type Base struct {
	Name  string
	Value int
}

type EmbedsBase struct {
	Base
	Other string
}
``````

### 포인터 임베딩

EmbedsPointerToBase 는 Base의 포인터를 변수로 갖는 새로운 타입이다.

``````go
type Base struct {
	Name  string
	Value int
}

type EmbedsPointerToBase struct {
	*Base
	Other string
}
``````

## 비교

- [https://stackoverflow.com/questions/28501976/embedding-in-go-with-pointer-or-with-value](https://stackoverflow.com/questions/28501976/embedding-in-go-with-pointer-or-with-value)

 위의 질문을 보면 타입 임베딩과 포인터 임베딩에 대한 여러 개발자의 의견을 엿볼 수 있다. 정리해 보면, 두 방식은 객체 생성 및 전달에서 차이가 있으며, 객체의 내부 변수에 어떻게 접근하고 싶은가에 따라 선택적으로 사용하면 된다고 한다.

 Base의 메서드가 Base 의 값을 제어하고, 이때 이 Base를 임베딩한 EmbedsBase 객체를 값으로 넘기는 특수한 함수를 가정하면, 이 함수 내부에서 Base를 조작하는 것은 외부에 영향을 주지 못한다.

``````go
type Base struct {
	Name  string
	Value int
}

func (b Base) PrintBase() {
	fmt.Println(b.Name, b.Value)
}

type EmbedsBase struct {
	Base
	Other string
}

func ControlEmbedsBaseValue(value *EmbedsBase) {
	value.Name = "other name"
	value.PrintBase()
}

func main() {
	b := Base{"name", 2}
	eb := EmbedsBase{b, "other"}
	ControlEmbedsBaseValue(&eb) // other name 2 출력
	eb.PrintBase()              // name 2       출력
}
``````

하지만, 포인터를 임베딩한 EmbedsPointerToBase 을 사용하면, 값을 넘기더라도 ControlEmbedsBaseValue함수로 EmbedsPointerToBase의 Name 변수를 제어할 수 있다.

``````go
type Base struct {
	Name  string
	Value int
}

func (b Base) PrintBase() {
	fmt.Println(b.Name, b.Value)
}

type EmbedsPointerToBase struct {
  *Base
	Other string
}

func ControlEmbedsBaseValue(value EmbedsPointerToBase) {
	value.Name = "other name"
	value.PrintBase()
}

func main() {
	b := Base{"name", 2}
	eb := EmbedsBase{b, "other"}
	ControlEmbedsBaseValue(&eb) // other name 2 출력
	eb.PrintBase()              // other name 2 출력
}
``````

장황하게 예시를 들었지만, Go의 함수 동작을 이해하면 당연한 동작이다.

 Go는 함수를 실행하는 순간에 외부에서 전달받은 변수를 복사하여 함수 안에 새롭게 생성한다. 만약 전달받은 값이 포인터이고, 이 포인터를 통해서 변수를 조작했다면, 외부 변수에 영향을 줄 것이다. 반대로 전달받은 값이 단순한 변수라면, 이 변수를 조작 하더라도 외부 변수에 영향을 줄 수 없다. 이를 이해하고 상황에 따라 선택적으로 사용하면 된다.

## Reference

- [https://stackoverflow.com/questions/28501976/embedding-in-go-with-pointer-or-with-value](https://stackoverflow.com/questions/28501976/embedding-in-go-with-pointer-or-with-value)
- [https://incheol-jung.gitbook.io/docs/q-and-a/architecture/undefined-2](https://incheol-jung.gitbook.io/docs/q-and-a/architecture/undefined-2)
- [https://smjeon.dev/etc/composite-extends/](https://smjeon.dev/etc/composite-extends/)
