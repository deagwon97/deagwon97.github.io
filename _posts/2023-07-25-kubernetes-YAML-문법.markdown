---
layout: post
thumbnail: ""
title: "[kubernetes] YAML 문법"
createdAt: 2023-07-25 18:32:23.453000
updatedAt: 2023-07-25 18:32:23.453000
category: "쿠버네티스"
---
## 1. 들여쓰기로 계층을 구분한다.

들여쓰기로 계층을 구분하며, 동일한 계층의 항목은 동일한 들여쓰기로 표현된다.

``````yaml
# yaml
spec:
  containers: 1
  containers2:
  containers3: 4
  containers4:
  - name: minio
``````

``````json
// json
{
  "spec": {
    "containers": 1,
    "containers2": null,
    "containers3": 4,
    "containers4": [
      {
        "name": "minio",
``````

## 2. 하이픈(-)으로 배열을 표현한다.

``````yaml
# yaml
spec:
  - containers: 1
  - containers2:
  -	containers3: 4
  - name: minio
``````

``````json
// json
{
  "spec": [
    {
      "containers": 1
    },
    {
      "containers2": null
    },
    {
      "containers3": 4
    },
    {
      "name": "minio",
``````

## 혼동하기 쉬운 문법

### Case 1.

``````yaml
envFrom:
  - secretRef:
    name: minio-key
``````

``````json
{
	"envFrom": [
        {
          "secretRef": null,
          "name": "minio-key"
        }
}
``````

 case1은 envFrom 리스트는 하나의 객체를 갖는다. 이 객체에는 secretRef와 name가 동일한 계층으로 존재한다.

### Case 2.

``````yaml
envFrom:
  - secretRef:
      name: minio-key
``````

``````json
"envFrom": [
        {
          "secretRef": {
            "name": "minio-key"
          }
        }
``````

 case1도 envFrom 리스트 하나의 객체를 는다. 하지만 이 객체에는 secretRef 항목 아래에 name가 존재하는 구조이다. 들여쓰기 한 칸으로 항목의 계층관계가 달라지기 때문에 주의해야하한다.
