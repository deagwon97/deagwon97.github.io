---
layout: post
thumbnail: 65378da4-f1c2-414d-aafd-c53499656c08
title: "[Go] Receiver"
createdAt: 2023-07-26 11:25:03.348000
updatedAt: 2023-07-26 11:25:03.348000
category: "Go"
---
## Go receiver


<img alt="image" src="/images/65378da4-f1c2-414d-aafd-c53499656c08"/>


 ``Go``의 함수가 특정 객체에 의존한다면 이 함수를 ``Receiver``라고 부른다. 이때, 객체의 값에 의존하면, ``Value Receiver``라고 부르고, 객체의 포인터에 의존하면 ``Pointer Receiver``라고 부른다. ``Go``는 구조체 객체에 의존하는 리시버 함수를 이용하여 클래스를 정의할 수 있다.

``````go
// Student 구조체
type Student struct {
	Name  string
	Class int
}

// Student의 Value Receiver
func (s Student) printStudent() {
	fmt.Println(s.Name)
	fmt.Println(s.Class)
}

// Student의 Pointer Receiver
func (s *Student) printStudentPointer() {
	fmt.Println(s.Name)
	fmt.Println(s.Class)
}
``````

## Value Receiver

 ``value receiver``는 함수가 호출되는 시점에 ``Student`` 객체를 복사하여 함수 내부에 새로 생성한다.

``````go
// Student의 Value Receiver
func (s Student) printStudent() {
	fmt.Println(s.Name)
	fmt.Println(s.Class)
}
``````

 따라서 함수가 호출되기 직전의 객체 값을 그대로 참조할 수 있지만, 객체의 값을 수정할 수는 없다. 아래 예시는 ``Value Receiver`` 는 두 가지 사실을 보여 준다.

1. 함수가 호출되는 순간 s 객체를 복사한다.
2. 함수 내부에 복사된 변수를 수정해도 외부의 값은 변하지 않는다.

``````go
// Student 구조체
type Student struct {
	Name  string
	Class int
}

// Student의 Value Receiver
func (s Student) printStudent() {
	fmt.Println(s.Name)
	fmt.Println(s.Class)
}

func (s Student) setStudentName(name string) {
	s.Name = name
}

func main() {
	// s 생성
	s := Student{"bdg", 1}
	s.printStudent()
		// [출력]
		// bdg
		// 1

	// 외부에서 s 의 Name 수정
	s.Name = "BDG"
	s.printStudent()
		// [출력]
		// BDG
		// 1

	//내부에서 s를 수정
	s.setStudentName("kks")
	s.printStudent()
		// [출력]
		// BDG   // s.Name이 "kks"로 바뀌지 않음
		// 1
}
``````

## Pointer Receiver

  ``pointer receiver``는 이미 생성된 Student에 직접 접근하여 값을 얻고, 수정한다. 

``````go
// Student의 Pointer Receiver
func (s *Student) printStudentPointer() {
	fmt.Println(s.Name)
	fmt.Println(s.Class)
}
``````

## Reference

- [https://go.dev/tour/methods/4](https://go.dev/tour/methods/4)
