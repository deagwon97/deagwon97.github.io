---
layout: post
thumbnail: 8e65180e-4680-4c24-b700-7600c72f28e3
title: "[Go]Dependency Injection"
createdAt: 2023-07-26 10:53:51.266000
updatedAt: 2023-07-26 10:53:51.266000
category: "클린 아키텍쳐"
---
# Dependency Injection

Dependency Injection 의존성 주입, 의존관계 주입 이란 무엇일까?

## Dependency란?

 A의 변화가 B에 영향을 준다면 B는 A에 의존한다.


<img alt="image" src="/images/87eb45f2-bc53-4d05-8279-55de36a1689c"/>

 A가 변할 경우 B는 영향을 받는다. 즉, B는 A에 의존한다. 이런 경우 B를 생성하고 실행하기 위해서는 A가 있어야한다.

A, B 보다 직관 적인 예시를 살펴보자. 편지와 우체국의 관계는 살펴보면 우체국은 편지가 있어야 존재할 수 있다. 즉 편지에 의존한다.코드를 살펴보자.


<img alt="image" src="/images/8e65180e-4680-4c24-b700-7600c72f28e3"/>

``````go
// Letter 
type Letter struct {
	Message string
}
func (letter *Letter) PrintLetter() {
	fmt.Println(letter.Message)
}

// Letter 생성자
func NewLetter(message string) {
  return &Letter{Message: message}
}
``````

``````go
// Post
type Post struct {
	letter Letter
}
type (post *Post) PrintPost() {
  post.letter.PrintLetter()
}

// Post 생성자
func NewPost struct {
  letter = NewLetter("hihihi")
  return &Post{letter: letter}
}

// main
func main() {
	post := NewPost()
	post.PrintPost()
}

``````

위의 코드는 Post 객체가 생성되는 순간에 생성자 내부에서 NewLetter를 호출하여 Letter 객체를 생성한다. 

이러한 구조에서는 Letter객체와 Post객체가 강하게 결합하여 여러가지 말썽을 일으킨다.

## Dependency Injection 적용하기

Dependency Injection은 이러한 문제를 일부해소해 주는데, 코드를 통해서 어떻게 DI를 적용할 수 있는지 알아보자.

``````go
// Letter 
type Letter struct {
	Message string
}
func (letter *Letter) PrintLetter() {
	fmt.Println(letter.Message)
}

// Letter 생성자
func NewLetter(message string) {
  return &Letter{Message: message}
}

// Post
type Post struct {
	letter Letter
}
type (post *Post) PrintPost() {
  post.letter.PrintLetter()
}

// Post 생성자
// Post 객체를 생성하는 시점에 letter객체를 생성하지 않는다.
// letter 객체는 외부에서 주입(Injection)한다.
func NewPost(letter *Letter) {
  //letter = NewLetter("hihihi")
  return &Post{letter: letter}
}

func main() {
	letter = NewLetter("hihihi")
	post := NewPost(letter)
	post.PrintPost()
}

``````

위와 같은 방법으로 DI를 구현 하였지만, 크게 와닫지 않는다. 아직도 letter가 변경될 경우, Post의 생성자를 수정해 주어야하고, 두 클래스가 강하게 결합되어 있다는 인상을 준다.

## + 다형성

Go의 다형성을 통해서 효과적인 DI를 구현해 보자.

원래 Letter는 구조체, 메소드, 생성자로 구성되어 있었다.

``````go
// Letter 
type Letter struct {
	Message string
}
// Letter Method
func (letter *Letter) PrintLetter() {
	fmt.Println(letter.Message)
}
// Letter 생성자
func NewLetter(message string) {
  return &Letter{Message: message}
}
``````

이러한 구조를 interface를 활용하여 다음과 같이 PrintLetter의 형식을 공유하는 LoveLetter와 LawyerLetter 클래스를 만들 수 있다.

``````go
// Letter 
type Letter interface {
  PrintLetter()
}

// LoveLetter 
type LoveLetter struct {
	Message string
}
func (l *LoveLetter) PrintLetter() {
	fmt.Println(l.Message + " I love you.")
}
func NewLoveLetter(message string) {
  return &LoveLetter{Message: message}
}

// LawyerLetter 
type LawyerLetter struct {
	Message string
}
func (y *LawyerLetter) PrintLetter() {
	fmt.Println(y.Message + " You are fire")
}
func NewLawyerLetter(message string) {
  return &LawyerLetter{Message: message}
}
``````

 LoveLetter와 LawyerLetter는 모두 Letter를 타입으로 갖는다.  즉, 다음과 같은 방법으로 생성할 수 있다.

``````go
var loveLetter, lawyerLetter Letter
loveLetter = NewLoveLetter("hello!");
lawyerLetter = NewLawyerLetter("hi!");
``````

이제 다시 위의 DI예제에 이를 적용해보자.

``````go
// Letter 
type Letter interface {
  PrintLetter()
}

// LoveLetter 
type LoveLetter struct {
	Message string
}
func (l *LoveLetter) PrintLetter() {
	fmt.Println(l.Message + " I love you.")
}
func NewLoveLetter(message string) {
  return &LoveLetter{Message: message}
}

// LawyerLetter 
type LawyerLetter struct {
	Message string
}
func (y *LawyerLetter) PrintLetter() {
	fmt.Println(y.Message + " You are fire")
}
func NewLawyerLetter(message string) {
  return &LawyerLetter{Message: message}
}

// Post
type Post struct {
	letter Letter
}
type (post *Post) PrintPost() {
  post.letter.PrintLetter()
}

// Post 생성자
// Post 객체를 생성하는 시점에 letter객체를 생성하지 않는다.
// letter 객체는 외부에서 주입(Injection)한다.
func NewPost(letter *Letter) {
  return &Post{letter: letter}
}

func main() {

	// letter 객체들 생성
	var loveLetter, lawyerLetter Letter
	loveLetter = NewLoveLetter("hello!");
	lawyerLetter = NewLawyerLetter("hi!");

	// post 객체 생성
	lovePost := NewPost(loveLetter)
	lovePost.PrintPost()

	// post 객체 생성
	lawyerPost := NewPost(lawyerLetter)
	lawyerPost.PrintPost()

}

``````

 이제 DI의 장점이 뚜렷해 진다.  우리는 loveLetter 객체를 넣어서 lovePost객체를 생성 하였고, lawyerLetter 객체를 넣어서 lawyerPost객체를 생성하였다. 

 여기서 주목할 점은,  Post가 Letter에 의존하는 상황에서  Post 클래스를 수정하지도 않으면서, 자유롭게 2개의 서로 다른 클래스를 주입 했다는 사실이다!

참고로, Post는 자신이 의존하고 있는 Letter객체를 생성하는 책임을 외부(main함수)로 넘겼는데, 이를 제어의 역전(Inversion of Control, IoC)라고 부른다. 실제로 여러 웹 프레임워크에서 이 개념을 채용하는 것을 쉽게 찾아 볼 수 있다.

아무튼 우리는 DI를 통해서 강하게 결합 되었던 두 클래스를 분리 하였고, 유연성을 확보하였다. 이를 통해서 다음과 같은 이점을 얻었다.

- 두 객체 사이의 관심사 분리
- 결합도 낮춤
- 유닛 테스트 용이
- 재 사용성 향상

## Reference

- [https://velog.io/@wlsdud2194/what-is-di](https://velog.io/@wlsdud2194/what-is-di)
- [https://blog.advenoh.pe.kr/go/Go에서의-다형성-Polymorphism/](https://blog.advenoh.pe.kr/go/Go%EC%97%90%EC%84%9C%EC%9D%98-%EB%8B%A4%ED%98%95%EC%84%B1-Polymorphism/)
- [https://mangkyu.tistory.com/125](https://mangkyu.tistory.com/125)
- [https://mangkyu.tistory.com/150](https://mangkyu.tistory.com/150)
