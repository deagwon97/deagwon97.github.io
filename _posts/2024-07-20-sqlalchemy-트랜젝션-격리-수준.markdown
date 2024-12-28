---
layout: post
thumbnail: ""
title: "[sqlalchemy]트랜젝션 격리 수준"
createdAt: 2024-07-20 08:28:01.750000
updatedAt: 2024-07-20 08:28:01.750000
category: "백앤드"
---
# 트랜잭션 격리 수준과 데이터베이스 조회 문제 해결

sqlalchemy를 활용하여 개발하던 중 <b>트랜잭션 격리 수준</b>으로 인해 예상치 못한 문제를 겪었습니다. 이 글에서는 트랜잭션 격리 수준이 무엇인지, 왜 중요한지, 그리고 이를 어떻게 설정하고 관리할 수 있는지에 대해 설명하겠습니다.

## 1. 서론
### 1.1. 문제 상황
 데이터베이스의 데이터를 생성한 후, 다른 sqlalchemy 세션에서 동일한 데이터를 조회할 때, 생성된 데이터를 조회할 수 없는 상황이 발생했습니다.

### 1.2. 문제의 원인 추측
sqlalchemy의 세션을 종료하고 새롭게 세션을 만들면, 정상적으로 조회가 된다는 사실을 통해서 트랜젝션 격리 수준으로 인한 문제라고 생각했습니다.

## 2. 트랜잭션 격리 수준의 이해
### 2.1. 트랜잭션과 격리 수준이란?
트랜잭션은 데이터베이스의 논리적 작업 단위이며, 격리 수준은 트랜잭션이 다른 트랜잭션의 영향을 받지 않도록 하는 정도를 정의합니다. 격리 수준은 데이터의 무결성과 일관성을 유지하기 위해 중요합니다.

read, write, commit, flush, close


### 2.2. 주요 트랜잭션 격리 수준

#### a) `READ UNCOMMITTED`
가장 낮은 격리 수준으로, 한 트랜잭션이 다른 트랜잭션의 미완료 데이터를 읽을 수 있습니다.

#### a) `READ COMMITTED`
 다른 트랜잭션의 완료된 데이터만 읽을 수 있습니다.
#### a) `REPEATABLE READ`: 같은 데이터를 여러 번 읽을 때 항상 같은 값을 읽도록 보장합니다.
#### a) `SERIALIZABLE`: 가장 높은 격리 수준으로, 트랜잭션이 직렬화된 것처럼 실행됩니다.


### 2.3. 동시성 문제
- `Dirty Read`: 미완료 데이터를 읽는 경우
- `Non-repeatable Read`: 동일 데이터를 여러 번 읽을 때 다른 값을 얻는 경우
- `Phantom Read`: 동일 쿼리를 여러 번 실행할 때 레코드 수가 달라지는 경우

## 3. 문제의 원인: 트랜잭션 격리 수준
### 3.1. 문제 발생 시나리오
데이터 수정 트랜잭션이 커밋된 후, 다른 세션에서 동일 데이터를 조회할 때 변경 사항이 반영되지 않는 문제를 겪었습니다.

### 3.2. 트랜잭션 격리 수준의 영향
문제는 데이터베이스의 트랜잭션 격리 수준이 READ COMMITTED로 설정되어 있었기 때문에 발생했습니다. 이는 한 트랜잭션이 커밋한 후에 다른 트랜잭션이 데이터를 읽을 수 있음을 의미합니다.

## 4. 문제 해결 방법
### 4.1. 격리 수준 조정
트랜잭션 격리 수준을 조정하여 문제를 해결할 수 있습니다. 애플리케이션의 요구 사항에 따라 적절한 격리 수준을 선택합니다.

예제 코드: READ COMMITTED로 설정
python
코드 복사
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine

engine = create_engine('your_database_url', isolation_level='READ COMMITTED')
SessionLocal = sessionmaker(bind=engine)
4.2. 세션 새로 고침
데이터 조회 시 세션을 새로 고침하여 최신 데이터를 반영하도록 합니다.

예제 코드: 세션 새로 고침
python
코드 복사
class MyDatabaseClass:
    def __init__(self):
        self.db = SessionLocal()

    def get_aimodel_by_name(self, name):
        self.db.expire_all() # 모든 객체를 만료시켜 강제로 새로 고침
        query = self.db.query(schema.Aimodel).filter(schema.Aimodel.name == name)
        aimodel = query.first()
        return aimodel
5. 결론
트랜잭션 격리 수준은 데이터베이스의 동시성 제어와 데이터 일관성 유지에 매우 중요한 역할을 합니다. 격리 수준을 적절히 설정하고 관리하면 데이터베이스의 예상치 못한 문제를 방지할 수 있습니다. 이 글이 트랜잭션 격리 수준을 이해하고 문제를 해결하는 데 도움이 되길 바랍니다.
