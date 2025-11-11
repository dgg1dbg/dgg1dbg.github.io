---
title: 'Database Concurrency Models'
date: 2025-11-11 12:18:00 +0900
categories: ['DB', 'Concurrency']
tags: [db, concurrency, transaction]
---


## 0. Concurrency of Database System
데이터베이스 시스템은 여러 사용자(**transactions**)가 동시에 접근하더라도 **정확성(correctness)** 과 **고가용성(high availability)** 을 유지해야 한다. 하지만 여러 트랜잭션이 동시에 실행될 때, 즉 _concurrent access_ 상황에서는 데이터의 일관성이 깨질 위험이 있다. 이를 방지하기 위해 **병행 제어(concurrency control)** 가 필요하다.

## 1. Transaction
트랜잭션은 데이터베이스에서 하나의 논리적인 작업 단위를 의미한다.  
트랜잭션은 다음 네 가지 연산으로 구성되는데, 
- **READ** — 데이터 읽기
- **WRITE** — 데이터 쓰기
- **ABORT** — 트랜잭션 취소
- **COMMIT** — 트랜잭션 확정  
이 연산들 만이 데이터베이스의 상태를 변경할 수 있는 유일한 단위이다.

#### ACID 속성
트랜잭션은 다음 네 가지 속성인 **ACID**를 만족해야한다.

| 속성                    | 설명                                  |
| --------------------- | ----------------------------------- |
| **Atomicity (원자성)**   | 트랜잭션의 모든 작업이 수행되거나, 전혀 수행되지 않아야 한다. |
| **Consistency (일관성)** | 트랜잭션 전후로 데이터베이스는 항상 일관된 상태여야 한다.    |
| **Isolation (고립성)**   | 동시에 실행되는 트랜잭션이 서로에게 영향을 주지 않아야 한다.  |
| **Durability (지속성)**  | 커밋된 결과는 시스템 장애 후에도 보존되어야 한다.        |

#### Example
은행 계좌 이체를 예로 들어보자.  `T1`은 고객 A의 계좌에서 100원을 출금해 고객 B의 계좌로 입금하는 트랜잭션이다.
```
T1:  A 계좌에서 100 출금 → B 계좌에 100 입금
```
이를 유저의 연산으로 표현하면 다음과 같다.
```
T1:
    READ(A)
    A := A - 100
    WRITE(A)
    READ(B)
    B := B + 100
    WRITE(B)
    COMMIT
```
그리고 데이터베이스에서 실제로 수행하는 연산은,
```
READ(A) -> WRITE(B) -> READ(B) -> WRITE(A)
```
가 된다.

## 2. Schedules
지금까지는 하나의 트랜잭션이 실행되는 경우만 살펴보았는데, 우리의 목표인 *concurrent access* 상황을 분석하기 위해서는 몇가지 추가적인 개념 정리가 필요하다.  
**스케줄(Schedule)** 이란, 여러 트랜잭션이 동시에 실행될 때, 그들의 연산이 섞인 순서를 의미한다.
예를 들어  
```
T1: R(A), W(A), R(B), W(B)
T2: R(A), R(B)
```
이 두 트랜잭션이 병행 실행되면 다음과 같이 스케줄 될 수 있다.  
```
r1(A) w1(A) r2(A) r2(B) r1(B) w1(B)
```
데이터베이스는 이 스케줄에 정의된 순서에 따라 연산을 수행하게 된다. 즉  
1. `T1` reads and writes `A`
2. `T2` reads `A`
3. `T1` reads and writes `B`
4. `T2` reads `B`

#### Conflicting Operations
한편 위와 같이 여러 트랜잭션이 동시에 같은 데이터 항목에 접근할때,  
- 두 연산이 **서로 다른 트랜잭션에서**, **같은 데이터 항목에 접근**하고,  
- **하나 이상이 WRITE**일 경우 이 둘은 충돌(conflict)한다고 한다.  
위의 스케줄에서 서로 충돌하는 연산 조합은 아래와 같다.  
- `w1(A)` ↔ `r2(A)`
- `w1(A)` ↔ `w2(A)`
- `r1(A)` ↔ `w2(A)`

#### Equivalent Schedule
두 스케줄이 동등하려면 다음을 만족해야 한다.
- 각 트랜잭션 내부의 연산 순서가 같다
- 충돌하는 연산들의 트랜잭션 간 순서가 같다

```
S1: r1(A) w1(A) r2(A) r1(B) w1(B) r2(B)
S2: r1(A) w1(A) r1(B) w1(B) r2(A) r2(B)
```
S1과 S2는 동등한 스케줄이다. 충돌 연산의 순서가 동일하기 때문이다.
```
S3: r2(A) r2(B) r1(A) w1(A) r1(B) w1(B)
```
S3는 S1, S2와 동등하지 않다.

#### Dependency Graph
직렬 가능성을 판별하기 위해 **의존 그래프(Precedence Graph)** 를 사용한다.
- 각 트랜잭션을 노드로 두고,
- Ti의 연산이 Tj의 연산과 충돌하며 Ti가 먼저 수행되었다면, **Ti → Tj** 간선을 추가한다.
```
Schedule: r1(A) w1(A) r2(A) r1(B) w1(B) r2(B)
Conflicts: w1(A) → r2(A), w1(B) → r2(B)
Dependency Graph: T1 → T2
```

## 3. Concurrency Control
데이터베이스에 여러 사용자, 즉 여러개의 트랜잭션이 동시에 요청되어 실행될때 시스템에서 요구하는 정확성(Correctness)를 만족시키면서도 시스템의 가용성(Availability)를 높이는 **스케줄**을 생성하는 것이라고 할 수 있겠다.  
그러면 어떤게 Correct한 스케줄일까?  
가장 직관적으로는, 트랜잭션의 속성 중 *Isolation*을 만족해야 할 것 같다. 그러기 위해서는 가장 간단하게 여러개의 트랜잭션이 수행되더라도, 그 트랜잭션들이 순차적으로 수행되면 되지 않을까?  
즉,  
```
T1: R(A), W(A), R(B), W(B)
T2: R(A), R(B)

Schedule: R1(A), W1(A), R1(B), W1(B), R2(A), R2(B)
or
		R2(A), W2(A), R1(B), W1(B), R1(A), R1(B)
```
이런식으로 스케줄하는 것이다. 이런 스케줄을 **Serial Schedule**이라고 부른다.  
#### Serializable(SR) Schedule
그러나 Serial Schedule로는 가용성이 너무 떨어지기에, Serial Schedule과 동등한(equivalent) Schedule을 고려해볼 수 있고, 이것을 직렬 가능 스케줄이라고 부른다.

예를 들어
```
r1(A) w1(A) r2(A) r1(B) w1(B) r2(B)
```
는 Serializable Schedule이다.  
여기서 중요한점은, 모든 Serializable Schedule은 Dependency Graph를 그렸을때 **Acyclic**하다는 것이다. 이 특징을 통해 어떤 스케줄이 Serializable 한지 아닌지 쉽게 확인할 수 있다.


## 4. Anomalies
그러나 Serializable Schedule은 트랜잭션의 **상태(commit/abort)** 를 고려하지 않기 때문에 여전히 이상 현상(anomalies)이 발생할 수 있다.  
대표적인 이상현상으로는,  

|충돌 유형|설명|
|---|---|
|**RW (unrepeatable read)**|같은 트랜잭션이 같은 데이터를 두 번 읽었는데 값이 바뀌는 경우|
|**WR (dirty read)**|커밋되지 않은 데이터를 다른 트랜잭션이 읽는 경우|
|**WW (uncommitted write)**|커밋되지 않은 데이터를 덮어쓰는 경우|

가 있다. 예를 들어 아래와 같이 스케줄링 되었을때,
```
r1(A) w2(A) r1(A)
```
RW conflict로 트랜잭션 1이 두 번째로 read를 했을때 값이 바뀐다.  
Serializable Schedule에서는 일어나지 않는다.
```
w1(A) r2(A)
```
이런경우는 WR conflict로, 트랜잭션 2가 read를 한 후 트랜잭션 1이 abort하게 된다면 dirty read 문제가 생긴다.
```
w1(A) w2(A)
```
마지막으로는 WW conflict로, 트랜잭션 1이 이후에 abort하고 복구하려고 할때, 이미 트랜잭션 2가 덮어 쓴 상태이기 때문에 복구가 어려워진다.  

따라서 위와 같은 이상현상을 해결하기 위해, Serializable Schedule에 추가적인 Schedule을 정의할 필요가 있다.  

#### Recoverable (RC) Schedule
- 정의:  
    트랜잭션 Tj가 Ti로부터 데이터를 읽었다면, **Tj는 반드시 Ti가 commit된 이후에 commit** 되어야 한다.  
    즉, **dirty read**(아직 commit되지 않은 데이터를 읽는 행위)는 허용되지만, commit 순서를 통해 문제를 방지한다.
- 특징:
    - dirty read는 가능하다.
    - 하지만 cascading abort(연쇄 abort)가 발생할 수 있다.
    - Ti가 abort될 경우, Ti의 데이터를 읽은 Tj도 abort되어야 하기 때문이다.
- 예시:
    `r1(A) w1(A) r2(A) c1 c2`
    위의 스케줄에서 `T2`는 `T1`이 수정한 A를 읽었다.  
    하지만 `T1`이 먼저 commit되기 때문에, recoverable하다.  
    만약 `T2`가 `T1`보다 먼저 commit했다면 recoverable하지 않은 스케줄이 된다.


#### Avoids Cascading Abort (ACA) Schedule
- 정의:  
    트랜잭션 Tj가 Ti로부터 데이터를 읽을 때, 그 데이터는 **commit된 데이터**여야 한다.  
    즉, dirty read 자체를 원천적으로 금지한다.
- 특징:
    - dirty read를 허용하지 않는다.
    - cascading abort를 피할 수 있다.
    - 단, 여전히 다른 anomaly(예: non-repeatable read)는 발생할 수 있다.
- 예시:
    `r1(A) w1(A) c1 r2(A) c2`
    `T2`는 `T1`이 commit한 이후에 데이터를 읽기 때문에, dirty read가 발생하지 않는다.  
    따라서 cascading abort가 일어나지 않는다.

#### Strict (ST) Schedule
- 정의:  
    dirty read와 **uncommitted 데이터의 overwrite** 모두 허용하지 않는다.  
    즉, **Tj가 Ti의 데이터를 읽거나 쓸 때**, Ti는 이미 commit 또는 abort 상태여야 한다.
- 특징:
    - dirty read 금지
    - dirty write 금지 (아직 commit되지 않은 데이터에 대한 write)
    - 시스템이 crash되더라도 복구가 단순하다
    - 그러나 **unrepeatable read**(같은 데이터를 두 번 읽을 때 값이 바뀌는 문제)는 막지 못한다.
- 예시:
    `r1(A) w1(A) c1 r2(A) w2(A) c2`
    `T1`이 A를 수정하고 commit한 뒤에야 `T2`가 A를 읽거나 쓸 수 있다.  
    따라서 dirty read와 dirty write 모두 발생하지 않는다.

#### 정리

| **Schedule Types** | **Unrepeatable read (RW)** | **Dirty read (WR)** | **Overwriting uncommitted writes (WW)** |
| ------------------ | -------------------------- | ------------------- | --------------------------------------- |
| **SR**             | ✓                          |                     |                                         |
| **RC**             | ✓                          | ✓                   |                                         |
| **ACA**            | ✓                          |                     |                                         |
| **ST**             | ✓                          |                     | ✓                                       |
| **Serial**         | ✓                          | ✓                   | ✓                                       |

각 Schedule이 방지하는 이상현상은 위의 테이블에서 확인할 수 있다.