---
layout: post
thumbnail: 86568c0b-a153-4dfc-87c8-f07d801e0da9
title: "JWT (Json Web Token)와 비대칭키"
createdAt: 2023-07-26 10:57:37.288000
updatedAt: 2023-07-26 10:57:37.288000
category: "보안"
---
## 1. JWT란?

 JSON object를 사용하여 정보 전달을 암호화 하는 방법의 일종으로 공개된 표준이다. JWT는 편리하고(compact) 자가수용적인 (self-contained) 방식으로 정보를 안전성 있게 전달한다. 

### 자가 수용적 (Self-Contained)?

JWT 는 필요한 모든 정보를 JWT 스스로 가지고 있다. JWT 시스템에서 발급된 토큰은 “**토큰에 대한 기본정보**”, “**전달 할 정보**” 그리고 “**토큰의 유효성을 검증할 정보**”를 포함한다.


## 2. JWT의 구성

### 1) Header

알고리즘 형식 (RSA / SHA256 등)을 정의한다.

- **alg : 서명 암호화 알고리즘(ex: HMAC SHA256, RSA)**
- **typ : 토큰 유형**

### 2) Payload

토큰에서 사용할 정보의 조각들인 **Claim** 이 담겨있다. (실제 JWT 를 통해서 알 수 있는 데이터) 즉, 서버와 클라이언트가 주고받는 시스템에서 실제로 **사용될 정보에 대한 내용**이다.

### 3) Signature

토큰이 유효함을 나타내는 정보이다.  signature는 사용자가 조작할 수 없다.

## 3. JWT의 동작 방식


<img alt="image" src="/images/a73f54d8-3468-4f83-8d27-bee30c93379d"/>

### 1) JWT 생성

JWT는 Header와 Payload를 인코딩하여 합친 후, 특정 알고리즘을 통해서 암호화하여 signature를 생성한다.

> encoding(Header) + encoding(Payload) → **encryption →** signature
> 

``````jsx
header = {
	"alg":"HMACSHA256",
	"typ":"JWT"
}

payload = {
	"user_id":1,
	"name":"bdg",
	"class":"2"
	...
}

signature = HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)

JWT = base64UrlEncode(header).base64UrlEncode(payload).signatue
``````

### 2) JWT 소비

사용자는 base64 decodeing을 통해서 header와 payload를 쉽게 얻을 수 있다. 하지만, secret은 서버만 알고 있기 때문에 header와 payload를 수정한 후 signature를 다시 만들 수 없다. 사용자는 서버로부터 발급 받은 JWT를 그대로 소비하기만 하고, 수정없이 다시 서버로 전달한다.

### 3) 유효성 검증

서버는 사용자가 다시 보내온 JWT의  header와 payload로 signature를 다시 생성하여 사용자가 보낸 signature와 비교한다. 이 두 signature가 일치할 경우, 사용자가 JWT를 수정하지 않았음이 증명된다.

payload가 수정되지 않았음이 증명되었기 때문에, 사용자는 payload에 들어있는 사용자 id를 신뢰할 수 있고, 이를 통해서 사용자를 식별할 수 있다.

## 4. 비대칭 키를 활용한 JWT

비대칭 키를 활용할 경우, **인가**(**Athorization**)을 제 3의 어플리케이션이 수행할 수 있다.

### 1) 비대칭 키 암호화란?

 한 쌍의 키가 존재하고, 상대방의 키로 암호화한 값을 자신의 키로 복호화 할 수 있는 방식을 비대칭 키 암호화 방식이라고 부른다.

예를 들어 A, B 라는 2개의 키가 존재한다고 가정해 보자

우선 "**중요한 값**"을 A를 통해서 암호화 하겠다. 

> A("**중요한 값**") ⇒ ”**암호화 된 값**”

이 ”**암호화 된 값**”은 B를 통해서 복호화한다.

> B(”**암호화 된 값**") ⇒ "**중요한 값**"

여기서 특이한 점은 **A로는 암호화된 값을 복호화 할 수 없다.** 오로지 자신이 암호화한 값은 다른 키로 만 복호화가 가능하다.

> A(”**암호화 된 값**") ⇒ “**중요한 값” — (X)**

### 2) 비대칭키 적용한 JWT 인증/인가 방식

 위의 “JWT의 동작 방식”에서는 HMACSHA256 알고리즘과 secret 이라는 하나의 키값을 통해서 암호화 하였다. 또한 특별히 복호화의 과정을 거치지 않았다. 하지만 이럴 경우, secret을 모르는 제3의 서비스는 JWT가 유효한지 그렇지 않은지 판단할 수 없다. 

여기에 **비대킹 키**를 적용해 보자. 한쌍의 비대칭키(A: 공개키, B: 비밀키)를 사용하겠다. 우선 비밀키(A)는 서버만 가지고 있고, 절대 공개하지 않는다. 이와 반면에 공개키(A)는 공개되어 있어서, 누구든 쉽게 이 키를 구할 수 있다.


<img alt="image" src="/images/86568c0b-a153-4dfc-87c8-f07d801e0da9"/>


1. **JWT 생성**
    
    Signature를 생성하는 과정에서 비밀 키(B)를 통해서 signatrue를 생성한다. 이 signature는 오로지 비밀 키(B)로만 만들 수 있고, 공개 키(A)로만 해독 할 수 있다.
    
2. **JWT 소비**
    
    사용자는 서버로부터 전달 받은 base64 decoding으로 JWT의 header와 payload를 쉽게 복원할 수 있다. 그리고 sinature도 공개 키(A)를 통해서 복화할 수 있다.(사실 사용자가 signature를 복호화할 필요는 없다.)
    
3. **제 3의 서비스**
    
    사용자가 JWT를 서버가 아니라 제 3의 서비스에게 전달한다고 해보자. 이 제3의 서비스는 사용자의 ID를 식별해야하고, 이것이 위조되지 않았는지 판단해야한다.
    
    제 3의 서비스는 원래 서버로부터 공개키를 받아왔다.
    
    이 공개 키(A)를 사용하면 사용자가 보낸 JWT의 signature를 복호화할 수 있다. 복호화한 결과가, JWT 속 header, payload와 일치 한다면, 이 JWT가 위조 되지 않았음이 증명된다. 이를 통해 인증 서버 없이 **인가**(Athorization)를 수행할 수 있다.
    
    제 3의 서버는 공개 키(A) 만으로 사용자의 JWT의 위조 여부를 판별한다.
    

### 3) 비대칭키 방식의 JWT 활용

 주로 **Micro Service Architecture**에서 이를 활용할 수 있다. 처음 id와 Password를 받아서 JWT를 발급해주는 **인증서버**를 분리하고, **각각의 다른 어플리케이션**들은 공개 키만 가지고 사용자가 제출한 JWT가 유효한지 그렇지 않은지 판단할 수 있다. 

### Reference

- [https://jwt.io/introduction](https://jwt.io/introduction)
- [http://www.opennaru.com/opennaru-blog/jwt-json-web-token-with-microservice/](http://www.opennaru.com/opennaru-blog/jwt-json-web-token-with-microservice/)
- [https://velopert.com/2389](https://velopert.com/2389)
