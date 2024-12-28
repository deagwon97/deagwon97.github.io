---
layout: post
thumbnail: d4b2680e-3267-423c-871c-1a562d25cda7
title: "프로그램에서 오류와 예외를 다루는 좋은 방법 with golang"
createdAt: 2023-08-21 14:38:40.128000
updatedAt: 2023-08-21 14:38:40.128000
category: "클린 코드"
---
프로그램은 여러 가지 원인으로 인해서 의도하지 않은 방향으로 동작하거나, 종료될 수 있습니다. 이러한 문제를 넓게는 Error 라고 부르며 Error(오류), Exception(예외) 등으로 구분해서 부르기도 합니다. 먼저 각 용어의 의미를 정리하고, 오류와 예외를 처리하기 위한 조건에 대해서 알아보겠습니다. 이후 Golang에서 오류/예외를 처리하는 좋은 방법에 대해 소개합니다.

### Exception(예외)

예외란 프로그램의 실행 중에 발생한 **예상치 못한 상황이지만 프로그램의 실행 흐름을 제어하여 처리할 수 있는 문제**를 의미합니다. 외부 API의 반환 값이 달라지거나 데이터베이스의 연결이 잠시 끊어지는 경우가 대표적인 Exception의 예시입니다. Exception은 런타임 중에 발생하기 때문에 예측이 어렵고 처리하기가 까다롭지만, 적절히 처리하면 프로그램이 비정상적으로 종료되는 것을 막을 수 있습니다. 이러한 처리를 Exception Handling이라고 부릅니다.

### Error(오류)

 좁은 의미에서는 **프로그램으로 해결할 수 없는 잘못된 상황**을 의미합니다. 문법이 틀렸거나, 하드웨어적인 고장이 발생했거나, 논리적으로 잘못된 코드를 작성하는 등 다양한 원인에 의해서 발생합니다. 보통 컴파일 타임 에러와 런타임 에러로 구분됩니다. 컴파일 타임 에러는 소스 코드를 컴파일하는 동안 발생하는 문제를 의미하며, 프로그램이 실행되기 전에 발생합니다. 런타임 에러는 프로그램이 실행되는 동안 발생하는 문제로, 프로그램 실행 중에 예상치 못한 동작이 발생하는 경우를 말합니다. 

 하지만 넓은 의미에서 Error는 Exception을 포함하는 의미로 사용됩니다. 프로그램에서 발생하는 모든 문제를 지칭하는 단어로 사용되기도 합니다.

# Exception, Error 처리

## **Exception** Handling (예외 처리)

**예외 처리의 가장 큰 목적은 실행 중인 프로그램의 비정상적인 종료를 막고, 의도한 동작을 지속해서 수행하는 것** 입니다. 적절한 Exception Handling을 위해서는 다음 조건을 만족해야 합니다.

### 복원

예외가 발생하면, 프로그램은 비정상적으로 동작하거나 종료될 수 있습니다. 이러한 예외를 감지하고, 정상 상태로 되돌리는 작업이 필요합니다. 많은 언어에서 ``try-catch`` 같은 문법을 지원하며, 문제가 발생했을 때 특정 동작을 수행하는 방식으로 예외를 처리합니다. 또한 golang에서는 ``defer``, ``panic``, ``recover`` 를 활용하여 이와 유사하게 문제를 해결합니다.

### 리소스 해제

예외가 발생할 때 자원을 적절히 해제하고 정리하는 코드를 작성해야 합니다. 예를 들어 외부와 통신하는 모듈을 사용한다면 예외가 발생할 때, 연결을 해제하고 연결과 관련된 객체를 제거하는 작업이 필요합니다.

### 로그

프로그래머가 아무리 꼼꼼히 코드를 작성하더라도 모든 예외를 다 예상할 수는 없습니다. 개발 단계의 프로그램은 이런 예외가 더 많이 발생합니다. 런타임 중 어디에서 에러가 발생했는지 알 수 있는 가장 좋은 방법은 로그를 남기는 것 입니다. 로그를 통해 어디서 어떤 문제가 발생했는지 발견하고 추후에 같은 문제가 발생하지 않도록 처리해야 합니다. 

### 예외 클래스 혹은 타입

예외를 단순히 문자열로 전달하게 되면, 프로젝트가 커지면서 유지보수에 문제가 발생합니다. 예외 클래스나 타입을 정의하여 예외를 분류하고 수준에 따른 로깅과 처리가 이뤄져야 합니다.

### Exception Propagation (예외 전파)

모든 상황에서 예외를 발생하는 것은 예외 처리 로직을 분산시켜 코드의 가독성을 떨어트리고 유지보수를 어렵게 합니다. 필요한 경우에만 예외를 처리하고, 나머지는 호출자에게 예외를 전파시켜 이러한 문제를 해결해야 합니다.

## Error Handling (에러 처리)

Error Handling또한 Exception Handling과 유사한 조건을 만족해야 좋은 코드를 만들 수 있습니다. 하지만, Error는 프로그램의 복구보다는 문제를 식별하고 사용자에게 적절한 피드백을 전달하는 것이 목적입니다.

# Exception, Error의 전파

예외 전파(Exception Propagation)와 에러 전파(Error Propagation)는 비슷한 개념으로 오류나 예외를 상위 수준의 호출자에게 전달하는 것을 의미합니다.

## Exception Propagation

예외 전파(Exception Propagation)는 예외가 발생한 함수 또는 메서드 내부에서 예외를 처리하지 않고 호출자(caller)로 예외를 전달하는 과정을 의미합니다. 이로써 예외 처리를 더 중앙 집중화하거나, 더 높은 수준의 예외 처리 코드에서 처리할 수 있도록 할 수 있습니다.

예외 전파를 사용하면 코드를 더 모듈화하고 유연하게 만들 수 있습니다. 여러 함수나 메서드가 하나의 큰 작업을 수행할 때, 예외 처리를 전적으로 해당 작업의 최상위 호출자에게 위임하는 것이 도움이 될 수 있습니다. 예를 들어, 다음과 같은 함수가 있다고 가정해봅시다.

``````java
public void processData() {
    try {
        readDataFromFile();
        processAndSaveData();
    } catch (IOException e) {
        // 예외 처리
    }
}
``````

위 함수에서 ``readDataFromFile()``과 ``processAndSaveData()`` 메서드에서 예외가 발생하는 경우, 이 예외를 ``processData()`` 함수 내부에서 처리하고 있습니다. 그러나 이 예외들이 ``readDataFromFile()``과 ``processAndSaveData()`` 메서드 자체에서 해결될 수 없는 문제라면, 이 예외를 둘 중 하나가 아닌 상위 호출자에게 전달하는 것이 바람직합니다.

``````java
public void processData() throws IOException {
    readDataFromFile();
    processAndSaveData();
}
``````

## Error Propagation

 함수에서 에러가 발생하면 함수 호출자 또한 에러의 영향을 받습니다. 따라서 함수의 호출자에게 에러를 전파해 호출자도 에러를 인지하고 처리하는 구조로 코드를 작성해야 합니다. 

단순히 에러를 전파만 할 경우, 에러가 최초로 발생한 지점을 찾지 못할 수도 있습니다. 따라서 에러를 전파할 때는, 에러가 발생한 지점의 정보를 포함하여 상위로 전달할 필요가 있습니다. 또한 에러도 클래스나 타입을 정의하여 분류하고 수준에 따른 로깅과 처리가 함께 이뤄져야합니다.

# Go의 Error와 Exception 처리하기

지금까지의 내용을 바탕으로 Go 언어에서 Error와 Exception을 처리하는 방법을 알아보겠습니다. 사실 Go에는 Exception이라는 단어를 직접 사용하지 않습니다. 문제 상황을 Error Interface로 처리하며, Exception이 발생하는 경우에는 defer와 recover를 사용하여 처리합니다. 

## Go의 defer recover

``defer`` 는 함수가 종료되는 시점에 수행돼야하는 동작들을 stack에 담고 순차적으로 꺼내어 실행합니다. 마치 다은 언어의 try-catch-final 에서 final에 해당한다고 볼 수 있습니다. ``defer`` 를 통해서 Exception이 발생하더라도 연결을 종료하거나 메모리 자원을 해제할 수 있습니다.

``````go
package main

import "fmt"

func main() {
    defer fmt.Println("This will be executed last.")
    defer fmt.Println("This will be executed second.")
    fmt.Println("This will be executed first.")
}
``````

 ``panic`` 은 심각한 오류 상황에서 프로그램을 중단시키는데, ``recover`` 는 ``panic``이 발생했을 때 그 상황을 처리하고 프로그램을 원래 상태로 되돌리기 위해 사용됩니다. ``recover`` 함수는 ``defer`` 문 내에서만 사용될 수 있으며, 패닉이 발생한 함수의 호출 스택을 타고 올라가면서 가장 가까운 ``defer`` 문에서 호출됩니다. 패닉이 발생하지 않으면 ``recover`` 함수는 ``nil``을 반환합니다.

``````go
import "fmt"

func main() {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("Recovered from:", err)
        }
    }()

    panic("Something went wrong!")
}
``````

# Go의 Error 전파

Go 에서는 Exception과 Error를 모두 ``error`` interface로 전파하는 패턴을 사용합니다. 

``````go
func ReadFile() error {
	_, err := os.Open("nonexistent.txt")
	if err != nil {
		fmt.Println("file not exist!!!")
	}
	return err
}

func main(){
	err := ReadFile()
	if err != nil {
		fmt.Println("dealing with error")
	}
}
``````

이러한 패턴을 통해서 error를 전달하는 방법에는 여러가지 논의가 있었고, GoCon spring conference에서 구체적인 방법이 제시되었습니다.

- **Don’t just check errors, handle them gracefully**
- https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

 이제 윗글을 바탕으로 Custom Error Type과 Opaque Error 가 어떻게 다르고, 왜 Opaque Error를 써야 하는지 설명하겠습니다. 또한 Error를 전파할 때 어떤 점을 고려해야 하는지 구체적인 예시를 통해서 알아보겠습니다. 

## Custom Error Type vs Opaque Error

### 문제상황

 호출당한 함수를 callee, 호출한 함수를 caller라고 부르겠습니다. 이제 callee에서 에러가 발생하면 그 에러를 caller에게 반환하는 상황을 가정해 보겠습니다. 여기서 caller는 callee가 반환하는 에러에 따라 다른 동작을 수행해야 합니다. 예를들면, callee가 반환하는 에러가 network 연결과 관련된 에러라면, caller는 network를 복구하는 동작을 수행 해야하고, 다른 종류의 에러라면 그에 맞는 동작을 수행해야 합니다. **이처럼 호출자가 피호출자의 에러 종류에 따라 동작을 결정하는 상황이 종종 발생합니다.** Custom Error Type과 Opaque Error는 위 상황을 해결하는 방법을 제시합니다. 

### Custom Error Type

 **에러를 구분**하는 대표적인 방법은 에러 타입을 커스텀 하는 것입니다. 아래와 같이 ``NetworkError`` 타입을 만들어 사용하면 caller가 callee의 에러의 타입을 보고 에러의 종류를 구분할 수 있습니다.

``````go
//------------------------------------------------
// project/error/error.go

type NetworkError struct {
        Msg            string
}

// NetworkError가 golang 내장 error를 임베딩하기 위해서는
// Error() string 함수를 구현해야합니다.
// Error() string 함수를 구현하는 type은 error와 호환됩니다.
func (ne *NetworkError) Error() string { 
        return ne.Msg
}

func (ne *NetworkError) IsNetworkError() bool {
	fmt.Println("callee impelemented caller's NetworkError")
	return true
}

//------------------------------------------------
// project/callee/callee.go

import e "project/error"

func Callee() error {
		err := doSomething()
		if err != nil {
			return &e.NetworkError{"Something happened"}
		}
		err = doOtherThing()
		if err != nil {
			return err
		}
}

//------------------------------------------------
// project/caller/caller.go

import (
	e "project/error"
)

func Caller(callee func() error) (err error) {
		err := callee()
		// Callee가 반환한 error가 
		// NetworkError 메서드를 가지고 있는지 확인하고
		// 가지고 있다면 recoverConnection()함수를 실행합니다.
		// 따라서 Caller는 "project/error" 모듈에 의존하게 됩니다.
		if netErr, ok := err.(e.IsNetworkError); ok {
				recoverConnection()
				return
		}
		if err != nil {
				return  err
		}
}
//------------------------------------------------
// project/main.go

import 	ce "project/callee"

func main() {
	err := Caller(ce.Callee)
	if err != nil {
		...
}
``````

 만약 에러가 ``NetworkError`` 타입이라면 ``recoverConnection()`` 함수를 실행하고 그렇지 않다면 에러를 무시하는 방식으로 동작합니다.

하지만 위 구조는 Caller와 Callee 모두 ``NetworkError``에 의존하게 됩니다. 이러한 타입 의존성은 불안정한 API를 만들기 때문에 바람직하지 않습니다. 특히 다른 caller가 NetworkError에 의존하기 시작하면 서로 다른 caller의 결합도가 증가하면서 코드가 지저분해질 수 있습니다.


<img alt="image" src="/images/d4b2680e-3267-423c-871c-1a562d25cda7"/>

### Opaque Errors (불투명 에러)

GoCon Spring Conference에서는 이런 의존성을 없애고 **Opaque Error** (불투명 에러)를 사용할 것을 제안했습니다.

**Opaque Error**에서 caller는 호출한 함수가 출력한 error 타입에 의존하지 않습니다. caller가 검사하고자 하는 행위의 인터페이스는 caller와 같은 모듈에서 선언되며, caller는 callee의 error가 이 행위를 구현하는지 보고 error를 assertion합니다. 구체적인 예시를 통해서 살펴보겠습니다.

``````go
//------------------------------------------------
// project/callee/callee.go

// custom error는 callee에서 선언됩니다.
type NetworkError struct {
	Msg string
}

func (e *NetworkError) Error() string {
	return e.Msg
}

func (e *NetworkError) IsNetworkError() bool {
	fmt.Println("callee impelemented caller's Temporary()")
	return true
}

func doSomething() error {
	err := fmt.Errorf("error occurred")
	return err
}

func Callee() (err error) {
	err = doSomething()
	if err != nil {
		err = NetworkError{Msg: err.Error()}
		return
	}
	return
}
//------------------------------------------------
// project/caller/caller.go

// networkError의 interface는 caller 모듈에 선언됩니다.
type networkErrorInterface interface {
	IsNetworkError() bool
}

func Caller(callee func() error) (err error) {
	err = callee()
	// caller는 자신이 선언한 networkErrorInterface의 동작을
	// callee가 반환한 에러가 구현하고 있는지 확인합니다.
	if ne, ok := err.(networkErrorInterface); ok {
		// 만약 구현하고 있다면 recoverConnection()를 실행합니다.
		recoverConnection()
		return 
	}
	if err != nil {
		return err
	}
	return
}

//------------------------------------------------
// project/main.go

import 	ce "project/callee"

func main() {
	err := Caller(ce.Callee)
	if err != nil {
		...
}
``````

callee의 에러가 networkErrorInterface를 구현하고 있는지는 caller의 관심사입니다. 따라서 caller는 이 관심사를 관리할 책임을 지고 내부에서 networkErrorInterface를 정의합니다. 이후 callee가 반환하는 에러의 networkErrorInterface 구현 여부를 따져 에러를 구분합니다.


<img alt="image" src="/images/fb462ec3-cc84-4973-b6d9-173939c31acd"/>


이제 caller는 자신의 관심사를 다른 모듈에 의존하지 않고 직접 관리하게 됩니다. callee가 이 networkErrorInterface를 구현한 에러를 반환한다면 caller는 관심 있는 에러가 발생한 것으로 판단하고 동작을 수행합니다.

## **Annotating errors**

두 번째 이슈는 에러에 주석을 다는 것입니다. 중첩된 함수 속에서 발생하는 에러를 상위 함수로 전파하다 보면, 어디서 어떤 에러가 발생했는지 context에 관한 정보가 사라집니다. golang에서는 내장 패키지인 ``[github.com/pkg/error](https://godoc.org/github.com/pkg/errors)`` 와 ``runtime``를 통해서 이 문제를 해결할 수 있습니다.

``````go
func WrapError(err error, message string) error {
	// context 정보를 추출
	pc, filename, line, ok := runtime.Caller(1)
	details := runtime.FuncForPC(pc)
	if ok && details != nil {
    message = fmt.Sprintf(``{ "file" : "%s", "line" : "%d", "func" : "%s" , "message":"%v" } \n``, 
						filename, line, deatils.Name(), message)
		// 메시지와 에러를 묶어서 다시 에러 객체를 생성
		err = errors.Wrap(err, message)
   }
	return err
}
``````

위와 같이 ``WrapError`` 함수를 만들어 사용하면, 에러가 전파될 때마다 각 함수들의 에러 발생 지점을 에러에 포함할 수 있게 됩니다. 또한 프로그래머가 원하는 메시지도 에러에 담아서 전달 할 수 있게 됩니다. 아래는 ``WrapError`` 함수를 사용한 예시입니다.

``````go

func callee() error {
	err := doSomething()
	if err != nil {
		err := WrapError(err, "callee error message")
		return err
	}
	return err
}

func caller(callee func() error) error {
	err := callee()
	err != nil {
		err := WrapError(err, "caller error message")
		return err
	}
	return err
}

var logger *log.Logger

func main() error {
	logger = log.New(os.Stderr, "Error: ", log.LstdFlags)
	err := caller(callee)
	err != nil {
		Logger.Print(err.Error())
	}
}

``````

## opaque error와 WrapError

그렇다면 opaque error와 WrapError를 함께 사용하여 error를 처리해보겠습니다.

``````go

//------------------------------------------------
// project/error/error.go
package error

import (
	"fmt"
	"runtime"
	"github.com/pkg/errors"
)

func WrapError(err error, message string) error {
	// context 정보를 추출
	pc, filename, line, ok := runtime.Caller(1)
	details := runtime.FuncForPC(pc)
	if ok && details != nil {
		message = fmt.Sprintf(``{"file": "%s", "line": %d, "func": "%s" , "message": "%v"}``,
			filename, line, details.Name(), message)
		// 메시지와 에러를 묶어서 다시 에러 객체를 생성
		err = errors.Wrap(err, message)
	}
	return err
}

//------------------------------------------------
// project/callee/callee.go
package callee

type NetworkError struct {
	Msg string
}

func (e *NetworkError) Error() string {
	return e.Msg
}

func (e *NetworkError) IsNetworkError() bool {
	fmt.Println("callee impelemented caller's Temporary()")
	return true
}

func doSomething() error {
	err := fmt.Errorf("error occurred")
	return err
}

func Callee() (err error) {
	err = doSomething()
	if err != nil {
		err = NetworkError{Msg: err.Error()}
		return
	}
	return
}
//------------------------------------------------
// project/caller/caller.go
package caller

import (
	"log"
	"os"
	e "project/error"
)
type networkErrorInterface interface {
	IsNetworkError() bool
}

func Caller(callee func() error) (err error) {
	err = callee
	// caller는 자신이 선언한 networkErrorInterface의 동작을
	// callee가 반환한 에러가 구현하고 있는지 확인합니다.
	if ne, ok := err.(networkErrorInterface); ok {
		// 만약 구현하고 있다면 recoverConnection()를 실행합니다.
		err = e.WrapError(err, "this is just networkError error!!")
		recoverConnection()
		return 
	}
	if err != nil {
		err = e.WrapError(err, "critical error!!")
		return err
	}
	return
}

// myproject/main.go
package main

import (
	"log"
	"os"
	"project/callee"
	"project/caller"
	e "project/error"
)

var logger *log.Logger

func main() {
	logger = log.New(os.Stdout, ``[Error]``, log.LstdFlags)
	err := Caller(callee.Callee)
	if err != nil {
		err = e.WrapError(err, "main error message")
		logger.Print(err)
	}
}
``````

 위 패턴을 참고하면 코드에 불필요한 의존성을 없애고, 디버깅에 용이한 에러/예외 처리 코드를 작성할 수 있을 것입니다.

## Referense

- https://www.scaler.com/topics/java/error-vs-exception-in-java/
- https://80000coding.oopy.io/0fbbee1b-bf6e-4164-bd9e-005d841e3052
- https://ocwokocw.tistory.com/250
- https://hermeslog.tistory.com/72
- https://radlohead.gitbook.io/typescript-deep-dive/type-system/exceptions

