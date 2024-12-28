---
layout: post
thumbnail: dbe10019-29a5-487d-a8b7-c32f90989511
title: "[kubernetes] commonLabels 은 모든 리소스에 동일한 레이블을 추가한다."
createdAt: 2023-07-25 18:18:16.695000
updatedAt: 2023-07-25 18:18:16.695000
category: "쿠버네티스"
---

 ## Label과 Selector란?

*label* 은 쿠버네티스 오브젝트(파드 등)에 첨부된 키와 값의 쌍이다. 오브젝트의 특성을 식별하는 데 사용되어 사용자에게 중요하지만, 코어 시스템에 직접적인 의미는 없다. 예를들어 어떤 service가 Deployment를 식별하기 위해서는 deployment에 label 존재하며, 이 label이 Service 명세에 selector 항목에 적혀있어, Service는 Deployment를 식별하게된다.
 

<img alt="image" src="/images/dbe10019-29a5-487d-a8b7-c32f90989511"/>


## 어떤 문제를 마주했나?

 백앤드와 프론트앤드 앱을 Traefik으로 배포하는 앱을 Kustomize 로 묶는 작업을 하고 있었다. Kustomize 없이 배포했을 때는 아무런 문제가 없었는데, ``kubectl apply -k .`` 커맨드로 배포하면 앱이 이상하게 동작했다. ``curl`` 을 통해서 배포된 앱에 접속해보면 1/2 확률로 ``bad gateway`` 에러가 발생했다.

## 문제의 원인은?

``kustomize build .`` 커맨드로 빌드된 프로젝트 전체 명세와 실제 명세를 비교해보니, kustomize 명세에는 backend 앱과 frontend 앱이 모두 동일하게 ``app: procject-name`` 레이블이 들어가 있었다.

``````yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: project-name
...
commonLabels:
  app: project-name
...
``````

빌드된 명세가 이렇게 바뀐 이유는 ``kustomization.yaml`` 파일에 ``commonLabels`` 항목이 들어있기 때문이다. ``commonLabels`` 는 묶여있는 kustomization 앱에 레이블을 주어진 값으로 변경한다.



<img alt="image" src="/images/0c205268-103b-40bf-a074-820062f669b5"/>


``commonLabels`` 로 인해서 frontend service와 backend service는 자신이 연결할 pods들을 식별할 수 없게 되었다. 결국 1/2 확률로 ``bad gateway`` 에러가 발생한 것이다. 

## 해결책

``commonLabels`` 를 삭제하고 앱을 다시 배포했더니 앱이 정상적으로 동작했다. 

``````bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: project-name
...
~~commonLabels:
  app: project-name~~
...
``````

### commonLabels 사용시 주의사항

``commonLabels``는 kustomization으로 묶인 모든 리소스에 레이블을 추가할 때, 사용된다. 만약 하나의 프로젝트 내부에서 backend, frontend와 같이 앱이 구분된다면, ``commonLabels`` 에 중복되는 레이블이 들어가지 않도록 주의해야한다.

## Reference

- [https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/labels/](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/labels/)
- [https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)
