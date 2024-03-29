---
layout: post
title: DeadLocks ( 은행원 알고리즘, 자원할당 그래프 ) [OS]
description: >
  [공룡책 정리] 데드락의 이해: Chapter 8. Deadlocks (Part 2)
  참고 <br>
  - https://www.inflearn.com/course/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-%EA%B3%B5%EB%A3%A1%EC%B1%85-%EC%A0%84%EA%B3%B5%EA%B0%95%EC%9D%98/lecture/65284?tab=curriculum
tags: [OS]
---

## 은행원 알고리즘

## 은행원 알고리즘
  
### 은행원 알고리즘 이란?

 자원의 할당 여부를 결정하기 전에 미리 결정된 모든 자원의 최대 가능한 할당량을 교착상태에 빠질 가능성이 있는지 조사해 '안전상태(safe state)'와 '불안전상태(unsafe state)'로 나눈다. 안전상태를 유지할 수 있는 요구만을 수락하고 불안전 상태를 초래할 사용자의 요구는 나중에 만족될 수 있을 때까지 계속 거절하는 알고리즘이다.

 * 이 알고르즘을 은행에 적용하면 고객들이 현금을 찾으러 와도 일정한 순서에 의해 고객의 모든 요청을 다 들어줄 수 있기 때문에 **은행원 알고리즘** 이라 불린다.

### 은행원 알고리즘 자료구조

은행원 알고리즘을 구현하려면 몇가지의 자료구조가 필요하다. n이 프로세스의 수이고, m이 자원의 종류라고 하자.

1. Available : 각 종류 별로 가용한 자원의 개수를 나타내는 크기가 m 벡터이다.
Available[j] = k 라면 현재  j 번째 자원을 k개 사용할 수 있다는 뜻이다.

2. Max : 각 프로세스가 최대로 필요로 하는 자원의 개수를 나타내는 크기가 n x m 인 행렬이다.
Max[i,j] = k라면 프로세스 Pi 가 j번째 자원을 k개까지 요청할 수 있다는 뜻이다.

3. Allocation : 각 프로세스에게 현재 나가있는 자원의 개수를 나타내는 크기가 n x m 인 행렬이다.
Allocation[i,j] = k 라면 현재 Pi가 j 번째 자원을 k개 사용하고 있다는 뜻이다.

4. Need : 각 프로세스가 향후 요청할 수 있는 자원의 개수( 남아있는 자원 )를 나타내는 크기가 n x m 인 행렬이다.
Need[i,j] = Max[i,j] - Allocation[i,j] 관계가 있음을 알 수 있다.


### 안정성 검사 알고리즘

시스템이 안전한지 검사하는 알고리즘은 다음과 같다.

1. Work 와 Finish는 크기가 m과 n인 vector이다.

2. Finish[i] == false, Need<sub>i</sub> $\leq$ Work 두 조건을 만족시키는 i 값을 찾는다.
그러한 i 값을 찾을 수 없다면 step 4로 간다.

3. Work = Work + Allocation<sub>i</sub>, Finsh[i] = true 작업을 실행 후 2번으로 돌아간다.

4. 모든 i 값에 의해 Finish[i] == true 이면 이 시스템은 항상 안정 상태에 있다.

이 알고리즘으로 안전 여부를 알아내기 위해서는 m x n<sup>2</sup> 개의 연산이 필요하다.


### 자원 요청 알고리즘

자원 요청이 안전하게 들어줄 수 있는지를 검사하는 알고리즘은 다음과 같다.
 
Request<sub>i</sub>는 프로세스 P<sub>i</sub>의 요청 vector라고 하자.
Request<sub>i</sub>[j] == k 라면 P<sub>i</sub>가 R<sub>i</sub>를 k개까지 요청하고 있음을 뜻한다. P<sub>i</sub>가 자원을 요청하게 되면 아래와 같은 조치가 취해진다.

1. 만일 Request<sub>i</sub> $\leq$ Need<sub>i</sub>이면 2번으로 이동
   아니면 시스템에 있는 개수보다 더 많이 요청했으므로 오류처리한다.

2. Request<sub>i</sub> $\leq$ Available<sub>i</sub>이면 3번으로 이동
   아니면 요청한 자원이 당장은 없으므로 P<sub>i</sub>는 기다려야 한다.

3. 마치 시스템이 P<sub>i</sub>에게 자원을 할당해준 것처럼 시스템 상태정보를 아래처럼 바꾸어 본다.

![](https://taeho0304.github.io/assets/img/OS/DeadLock/ResourceRequest.png)

만일 이렇게 바뀐 상태가 위의 안정성 검사 알고리즘을 거쳐 안전하다면 P<sub>i</sub>에게 반영된 정보대로 자원을 할당해주지만 불안전하다면 위의 자원 할당상태를 원상태로 복구시키고 P<sub>i</sub>는 Request<sub>i</sub>가 복구될 때까지 기다려야 한다.

### 단점?

1. 할당할 수 있는 자원의 수가 일정해야 한다.

2. 사용자 수가 일정해야 한다.

3. 항상 불안전 상태를 방지해야 하므로 자원 이용도가 낮다.

4. 최대 자원 요구량을 미리 알아야 힌다.

5. 프로세스들은 유한한 시간 안에 자원을 반납해야 한다.

오버헤드가 크고 프로세스가 시작할 때 프로세스가 가지고 있어야 할 자원의 최대 개수를 미리 알아야 하기 때문에 현대 시스템에서는 잘 사용되지 않는다.