---
layout: post
thumbnail: 4f7fb055-37f3-4263-ae37-3d6a7925d4a7
title: "[Kubernetes] 서비스를 외부에 노출하는 다양한 방법들. (NodePort, LoadBalancer, 그리고 Ingress)"
createdAt: 2023-07-26 11:23:35.430000
updatedAt: 2023-07-26 11:23:35.430000
category: "쿠버네티스"
---
### Pod이란?

 쿠버네티스에서 생성하고 관리할 수 있는 배포가능한 가장 작은 컴퓨팅 단위를 Pod이라고 한다. Pod은 하나 이상의 컨테이너로 구성되어 있으며 스토리지, 네트워크를 공유하고, 해당 컨테이너를 구동하는 방식에 대한 명세를 갖는다. 마치 docker-compose 를 통해 생성된 컨테이너들이 네트워크와 스토리지 볼륨을 공유하는 것과 유사하다.

### Service란?

 Pod은 Deployment에서 정의된 클러스터 목표 상태와 일치하도록 생성되고 삭제된다. Pod는 고유한 IP 주소를 갖고 있고, 클러스터 내부에서는 해당 pod에 접근할 수 있다. 하지만,  동일한 Delployment에 의해 생성된 Pod이라고 하더라도 Pod들은 서로 IP주소가 다르다. 만약 Deployment에 의해 생성된 특정 Pod들에 요청을 보내고 싶다면, 이 IP 주소들을 그룹화 해야 한다.

 Service는 셀렉터를 통해서 Pod들을 구분하는 추상적인 개념이다. 정의된 Service 를 통해서 특정 pod들을 구분하여 요청을 전달할 수 있다.


<img alt="image" src="/images/4f7fb055-37f3-4263-ae37-3d6a7925d4a7"/>


 이 Service 또한 고유한 P를 갖고있어 클러스터 내부에서는 특정 Service로 요청을 보낼 수 있다. 하지만 ClusterIP는 클러스터 내부에서만 유효할 뿐, 외부에서 요청을 보낼 수 없다. 외부에서 서비스로 요청을 보내기 위해서는 다른 장치들이 필요하다. 


<img alt="image" src="/images/e603782f-fe65-4789-b80e-641566eac9f2"/>


쿠버네티스에서는 NodePort와 LoadBalance, Ingress Controller를 통해서 외부 요청을 내부의 Service로 연결할 수 있다. 하나씩 살펴보자.

# NodePort

NodePort 서비스는 서비스에 외부 트래픽을 직접 보내는 가장 원시적인 방법이다. 모든 Node에 특정 포트를 열어두고, 이 포트로 보내지는 모든 트래픽을 서비스로 포워딩한다.

- NodePort : 모든 노드에서 서비스로 라우팅할 포트번호(주로 30000 ~ 32767 대역의 포트번호를 사용한다.)
- Port : 서비스의 포트번호
- TargetPort: 서비스가 가리키는 pod의 포트번호


<img alt="image" src="/images/995a758e-0c7c-4638-9b85-29e305019163"/>



(쿠버네티스가 동작하는 노드들의 특정 포트가 서비스로 연결된다는 점을 표현하면서 약간의 왜곡이 생겼다. 실제로는 노드들 속에서 pod, service 등이 클러스터의 모든 자원이 존재한다.)

# LoadBalancer

- 서비스를 인터넷에 노출하는 가장 일반적인 방법이다.
- LoadBalancer로 노출하고자 하는 서비스마다 자체의 IP 주소를 갖게 되고, LoadBalancer를 해당 서비스에 연결하는 구조이다.
- LoadBalancer는 클라우드 공급자가 제공해야한다.

서비스를 생성할 때, 타입을 LoadBalancer으로 지정해야한다. 외부 로드 밸런서가 라우팅되는 NodePort와 ClusterIP 서비스가 자동으로 생성되며, 외부의 로드 밸런서가 클러스터 내부 서비스로 접근할 수 있게 된다.

``````
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 192.0.2.127
``````


<img alt="image" src="/images/40e548f5-142f-466e-8254-3b8225e06b29"/>



# Ingress

위에서 소개했던, NodePort 혹은 LoadBalancer등의 방식은 기능이 상당히 제한적이다. Ingress는 이를 개선하여 더 많은 기능을 담고 있다.

> Ingress는 외부로부터 들어오는 요청에 대한 LoadBalancing, TLS/SSL 인증 처리, 도메인 기반 가상 호스팅, HTTP 경로의 라우팅 등을 정의한 규칙(API 오브젝트)이다.
> 

Ingress라는 규칙에 따라 Ingress Controller가 실행되며 ingress controller에는 다양한 종류가 있다.

## Ingress Controller

- Ingress Controller는 Ingress를 구현하기 위해 실행되는 응용 프로그램이다.

<img alt="image" src="/images/321666b0-3ae8-4edf-8afd-d1cc98be395c"/>

Ingress Controller에는 Nginx-Ingress, Traefik, Kong, HAproxy 등이 있고, 다음과 같은 차이점을 갖는다.


|                 | NGINX                                                                           | Kong                                                           |
| --------------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| 지원하는 프로토콜       | http/https, http2, grpc, tcp/udp                                                | http/https, http2, grpc, tcp (l4)                              |
| 구축 기반           | nginx/nginx plus                                                                | nginx                                                          |
| 라우팅 로직          | host, path, header, method, query param (all with regex expect host)            | host, path, method, header \*                                  |
| 범위              | Cluster or specified namespaces                                                 | Specified namespace                                            |
| 로드 벨런싱 알고리즘     | round-robin, least-conn, ip-hash, hash, random, least-time\*, sticky sessions\* | weighted-round-robin, sticky sessions                          |
| 인증 프로토콜         | Basic, Client cert, external Basic, external OAuth                              | Basic, HMAC, Key, LDAP, OAuth 2.0, PASETO, OpenID Connect \*\* |
| GUI 지원          | Yes \* \*\*                                                                     | Yes \* \*\*                                                    |
| Request Tracing | Yes                                                                             | Yes                                                            |
| 24/7 기술 지원      | Yes \*                                                                          | Yes \*                                                         |
*: 유료
**: 모듈 설치 필요

([https://kubevious.io/blog/post/comparing-top-ingress-controllers-for-kubernetes](https://kubevious.io/blog/post/comparing-top-ingress-controllers-for-kubernetes)에 올라온 표를 옮겨왔다)




# Reference

- [https://kubernetes.io/ko/docs/concepts/services-networking/service/#loadbalancer](https://kubernetes.io/ko/docs/concepts/services-networking/service/#loadbalancer)
- [https://kubevious.io/blog/post/comparing-top-ingress-controllers-for-kubernetes](https://kubevious.io/blog/post/comparing-top-ingress-controllers-for-kubernetes)
- [https://twofootdog.tistory.com/23](https://twofootdog.tistory.com/23)
- [https://blog.leocat.kr/notes/2019/08/22/translation-kubernetes-nodeport-vs-loadbalancer-vs-ingress](https://blog.leocat.kr/notes/2019/08/22/translation-kubernetes-nodeport-vs-loadbalancer-vs-ingress)
