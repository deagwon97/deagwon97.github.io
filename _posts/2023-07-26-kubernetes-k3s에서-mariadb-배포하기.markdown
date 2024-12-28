---
layout: post
thumbnail: bceea9b1-6e40-476c-ba01-88cdd2d69dda
title: "[kubernetes] k3s에서 mariadb 배포하기"
createdAt: 2023-07-26 11:10:09.543000
updatedAt: 2023-07-26 11:10:09.543000
category: "쿠버네티스"
---

<img alt="image" src="/images/bceea9b1-6e40-476c-ba01-88cdd2d69dda"/>


필자는 그동안 AWS의 RDS를 주로 사용해왔다. 지금까지 진행한 개인 프로젝트들은 DB에 접근하는 작업이 많지 않았기 때문에, 부담되지 않는 가격으로 간편하게 클라우드 서비스를 이용했다.

하지만, 최근 테스트 코드를 자주 활용하면서, db에 접근하는 횟수가 크게 늘었고, RDS에 지불하는 가격이 신경 쓰이기 시작했다. 이러한 이유로 클라우드에서 독립하여 개인 데이터 베이스를 구축했다.

 이번 포스트에서는 쿠버네티스(k3s), traefik, mariadb를 조합하여 데이터베이스 서비스를 구축하는 방법에 대해 다루고, 이론적인 내용은 다음 포스트에서 자세히 다루겠다.

## 1. secret 생성하기

데이터 베이스의 패스워드와 같은 민감한 설정값들을 secret으로 생성하여 관리한다. 이렇게 만들어진 secret은 deployment를 정의할 때, 포함된다.

``````
// env.db

MYSQL_HOST=%
MYSQL_PORT=3306
MYSQL_ROOT_PASSWORD=passwordimnida
MYSQL_DATABASE=nameimnida
MYSQL_USER=usernameimnida
MYSQL_PASSWORD=newpasswordimnida
``````

``````bash

$ kubectl create secret generic mariadb-bdg --from-env-file env.db

$ rm ./env.db
``````

## 2. PersistentVolume 정의하기

 퍼시스턴트볼륨은 클러스터에서 생성된 스토리지 자원이다. 노드가 클러스터의 리소스인 것처럼 PV는 클러스터의 리소스에 속한다. 퍼시스턴트의 볼륨에 저장된 데이터는 스토리지에 저장되기 때문에, 클러스터가 종료되어도, 데이터가 유지된다.

``````yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mariadb-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
``````

## 3. PersistentVolumeClaim 정의하기

*퍼시스턴트볼륨클레임* (PVC)은 사용자의 스토리지에 대한 요청을 처리하는 자원이다. PVC는 PV 리소스를 사용하며, 우리가 만들 데이터베이스의 디플로이먼트와 연결된다.

``````yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
``````

## 4. Deployment 정의

mariadb image를 통해서 데이터 베이스를 생성하고자 한다. 미리 생성한 secret, persistace volume claim을 연결하여 하나의 데이터 베이스 디플로이멘트를 정의한다.

``````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
spec:
  selector:
    matchLabels:
      app: mariadb
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
        - image: mariadb:latest
          name: mariadb
          envFrom:
            - secretRef:
                name: mariadb-bdg
          ports:
            - containerPort: 3306
              name: mariadb
          volumeMounts:
            - name: mariadb-persistent-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mariadb-persistent-storage
          persistentVolumeClaim:
            claimName: mariadb-pv-claim
``````

## 5. Service 정의하기

deployment가 배포되었다고 해도, 아직 pod 외부에서 접근할 수 없다. pod외부에서 pod에 접근할 수 있도록 서비스를 정의한다.

``````yaml
apiVersion: v1
kind: Service
metadata:
  name: mariadb
spec:
  type: LoadBalancer
  ports:
    - protocol: TCP
      name: mariadb
      port: 3306
      targetPort: 3306
  selector:
    app: mariadb
``````

정상적으로 서비스가 배포되면 해당 팟으로 접근하는 cluster-ip가 생성된다.

``````bash
$ kubectl get services

>>
NAME     TYPE          CLUSTER-IP     EXTERNAL-IP              PORT(S)       
mariadb  LoadBalancer  10.43.232.156  172.30.1.13,172.30.1.21  3306:31955/TCP
``````

cluster-ip를 통해서 mariadb에 접근하여  데이터베이스가 잘 동작하는지 확인한다.

``````bash
$ mysql -h localhost -P 10.43.232.156 -u root -p
Enter password: ************

>> 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 14
Server version: 10.8.3-MariaDB-1:10.8.3+maria~jammy mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| your_database_name |
| sys                |
+--------------------+
6 rows in set (0.004 sec)
``````

## 6. 클러스터 외부로 배포하기

 지금까지 데이터베이스를 생성하고, 클러스터 내부에서 db에 접근하는 방법에 대해 알아보았다. 클러스터 외부에서 db에 직접 접근하는 방식은 보안상 취약점이 많기 때문에 여기까지 배포해두고, 클러스터의 노드에 따로 접근하여 개발하고 배포하는 것을 권장한다. 

 하지만, 클러스터 외부에서 데이터 베이스에 직접 접근하고자 할 수도 있다. 이를 위해서는 새로운 ``ingress router``를 생성해야 한다. 필자의 클러스터는 k3s에 내장된 ``traefik``이라는 ``ingress controller``를 사용하여 클러스터 외부의 요청을 처리한다.

 ``traefik`` 를 사용하여 일반적인 web서비스를 클러스터 외부로 배포하는 ``ingressRouter`` 설정은 매우 간단하다. 하지만 database와 같은 http, https 와 다른 프로토콜을 사용하는 서비스를 배포하기 위해서는 몇 가지 작업을 추가해야 한다.

 

### 6-1. Traefik entrypoints 추가하기

 ``traefik`` 은 기본 entry point로 80(``web``), 443(``websecure``), 1704/udp(``streaming``)포트를 지원한다.  따라서 ``web`` 혹은 ``websecure`` entry points를 배포하는  ``ingressRoutes`` 설정은 다음과 같이 작성하면 된다.

``````yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: ingressroute-my-service
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(``deagwon.com``)
    kind: Rule
    services:
    - name: bdg-front
      port: 80
	tls: {}
``````

 하지만, 앞서 언급했듯이 ``traefik``은 database에서 주로 사용하는 3306포트를 지원하지 않는다. 따라서 ``traefik``의 설정값 변경하여 3306 포트를 추가한다.

일반적인 방법으로 k3s를 설치했다면 다음 경로에 k3s의 manifests 폴더가 위치한다. 해당 경로에 traefik-config.yaml을 생성한다.

``````bash
$ sudo ls /var/lib/rancher/k3s/server/manifests/

>> 
ccm.yaml      local-storage.yaml  rolebindings.yaml    traefik.yaml
coredns.yaml  metrics-server
``````

 ``/var/lib`` 경로의 파일들은 관리자만 접근할 수 있기 때문에 관리자 계정으로 접근하거나, ``sudo`` 를 넣어주면 접근할 수 있다.

``````bash
$ sudo vi /var/lib/rancher/k3s/server/manifests/traefik-config.yaml
``````

``mariadb``의 배포를 위해서 생성하는 entry point이므로 이름을 ``mariadb``로 하였다.

``````yaml
# /var/lib/rancher/k3s/server/manifests/traefik-config.yaml
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: traefik
  namespace: kube-system
spec:
  valuesContent: |-
    additionalArguments:
    - "--entryPoints.mariadb.address=:3306"
  entryPoints:
     mariadb:
      address: ':3306'
``````

``````bash
$ sudo kubectl apply -f /var/lib/rancher/k3s/server/manifests/traefik-config.yaml
``````

이렇게 하면, ``traefik``의 ``ingressRouter``에서 ``mariadb``라는 이름의 entry point를 사용할 수 있게 된다.

### 6-2. IngressRouteTCP 생성하기

 클러스터 외부에서 ``mariadb`` entry point로 들어오는 요청을 ``mariadb`` 서비스로 라우팅하는 ingress router를 정의한다.

``IngressRoute`` 가 http 요청을 routing한다면, ``IngressRouteTCP``는 tcp 요청을 routing하는 쿠버네티스 커스텀 자원(CRD)이다. ``ingressRoute`` 에서 Host를 매치 조건으로 사용하는 반면, ``IngressRouteTCP``  는 HostSNI(Server Name Indication)를 매치 조건으로 사용한다.

``````yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRouteTCP
metadata:
  name: mariadb-ingressroute
spec:
  entryPoints:
  - mariadb
  routes:
  - match: HostSNI(``*``)
    services:
    - name: mariadb
      port: 3306
``````

이제 클러스터 외부에서도 데이터베이스에 접근할 수 있다.

``````bash
$ mysql -h localhost -P my-pulic-ip -u root -p
Enter password: ************

>> 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 14
Server version: 10.8.3-MariaDB-1:10.8.3+maria~jammy mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| your_database_name |
| sys                |
+--------------------+
6 rows in set (0.004 sec)
``````

### 6-3. 주의사항

- 3306포트가 외부에서 클러스터까지 적절히 포트포워딩돼야 한다.
- 방화벽에 3306포트가 열려있어야 한다.
- 외부 엔드 포인트를 3306 이외의 다른 포트를 두고(ex. mariadb, :12314), 내부 서비스에 포워딩하는 것이 보안상 유리하다.

## Reference

- [https://stackoverflow.com/questions/63633098/traefik-2-2-1-expose-mysql-service-in-kubernetes-cluster](https://stackoverflow.com/questions/63633098/traefik-2-2-1-expose-mysql-service-in-kubernetes-cluster)
- [https://docs.rke2.io/helm/#customizing-packaged-components-with-helmchartconfig](https://docs.rke2.io/helm/#customizing-packaged-components-with-helmchartconfig)
- [https://rancher.com/docs/k3s/latest/en/helm/](https://rancher.com/docs/k3s/latest/en/helm/)
- [https://doc.traefik.io/traefik/routing/entrypoints/](https://doc.traefik.io/traefik/routing/entrypoints/)
