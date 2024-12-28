---
layout: post
thumbnail: ""
title: "[kubernetes] Persistent Volumes + Deployment와 StatefulSet의 차이점"
createdAt: 2023-07-26 11:26:26.117000
updatedAt: 2023-07-26 11:26:26.117000
category: "쿠버네티스"
---
쿠버네티스는 State-less 앱을 배포할 때 Deployment를 사용하고, state-full 앱을 배포할 때 StatefulSet을 사용할 것을 권장한다.  하지만 Deployment 또한 StatefulSet처럼 persistent volume claim(pvc)을 통해서 persistent volume(PV)에 접근할 수 있고, 앱의 State를 디스크에 저장할 수 있다. 여러 블러그에서 maria db application을 배포할 때, deployment와 PV를 연결하여 배포하는 예제를 쉽게 찾을 수 있다. 

 그렇다면 Deployment에서도 pod들의 상태를 유지할 방법이 존재하는데, 굳이 StatefulSet을 사용하는 이유는 무엇일까? 아래 블로그 및 공식문서를 종합해보면  다음과 같은 답이 나온다.

[https://akomljen.com/kubernetes-persistent-volumes-with-deployment-and-statefulset/](https://akomljen.com/kubernetes-persistent-volumes-with-deployment-and-statefulset/)

 StatefuleSet의 내부 pod들은 각자 역할이 다르고, 그 pod들을 따로 관리한다. 이를 통해서 **어플리케션 자체의 State를 보장하면서 앱의 생성, 배포, 스케일링기능을 지원한다.** Deployment는 앱이 Stateless하다는 전제 아래에서 앱을 관리(생성, 배포, 스케일링)하기 때문에, PV 연결 되었어도 앱을 생성하거나 스케일링하는 과정에서 오류가 발생할 수 있다.

### StatefulSet의 특징

- 내부 pod 마다 각각 pvc, pv를 생성하여 관리
    - 내부 pod 마다 고유한 pvc를 갖도록 설정이 가능하며, 스케일 확장에 용이하다.
    - Deployment + pvc + pv 로 앱을 배포할 때는 확장이 어렵다. 이미 정해진 pod에 볼륨이 묶여있는 상태에서 pod의 수를 증가시키면, 이미 해당 볼륨이 사용되고 있다는 에러가 발생한다. ([https://akomljen.com/kubernetes-persistent-volumes-with-deployment-and-statefulset/](https://akomljen.com/kubernetes-persistent-volumes-with-deployment-and-statefulset/))
- 내부 pod 마다 개별적으로 네트워크 식별 및 접근
    - StatefulSet의 내부 pod은 구분된다. headless service를 통해서 원하는 pod만 식별하여 접근하는 기능을 제공한다.
- 내부 pod들의 배치 순서 존재
    - pod들간의 의존관계가 존재하는 경우, 주어진 순서로 pod들을 실행할 수 있다.

### Reference

- [https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/)
- [https://bcho.tistory.com/1310](https://bcho.tistory.com/1310)
- [https://akomljen.com/kubernetes-persistent-volumes-with-deployment-and-statefulset/](https://akomljen.com/kubernetes-persistent-volumes-with-deployment-and-statefulset/)
