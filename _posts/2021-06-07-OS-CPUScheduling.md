---
layout: post
title: CPU 스케줄링 (Part 1)
description: >
  [공룡책 정리] CPU 스케줄링: Chapter 5. CPU Scheduling (Part 1)
tags: [OS]
---

## CPU 스케줄링: Chapter 5. CPU Scheduling (Part 1)

## CPU 스케줄링 기본 개념

#### CPU 스케줄링이란?

컴퓨터 자원을 효율적으로 관리하기 위해 프로세스들 사이에서 CPU 할당을 위한 우선순위를 관리하는 일이다.
다중 프로그래밍에서 어떤 프로세스가 대기해야 할 경우, 운영체제는 CPU를 그 프로세스로부터 회수해 다른 프로세스에 할당한다.

#### CPU-입출력 버스트 사이클(CPU-I/O Burst Cycle)

프로세스 실행은 CPU 명령을 실행하는 CPU burst time(주로 running 상태)과 I/O를 요청하고 기다리는 I.O busrt time(주로 wating, ready 상태)의 사이클로 구성된다.
입출력 중심의 프로그램은 전형적으로 짧은 CPU 버스트를 많이 가진다. CPU 지향 프로그램은 다수의 긴 CPU 버스트를 가질 수 있다. 이러한 분포는 적절한 CPU 스케줄링 알고리즘을 선택하는 데 매우 중요하다.

#### 선점 스케줄링(Preemptive Scheduling)

선점 스케줄링은 운영체제가 강제로 프로세스의 사용권을 통제하는 방식이고, 비선점 스케줄링은 프로세스가 스스로 다음 프로세스에게 자리를 넘겨주는 방식이다. 즉, 선점 스케줄링 방식에서는 CPU에 프로세스가 할당되어 있을 때도 운영체제가 개입해 다른 프로세스에게 CPU를 할당할 수 있지만, 비선점 스케줄링 방식에서는 할당된 프로세스가 자발적으로 나오기 전까지 운영체제가 개입하지 않는다.

CPU 스케줄링 결정은 다음의 네 가지 상황에서 발생할 수 있다.

1. 프로세스가 running 상태에서 waiting 상태로 전환될 때
2. 프로세스가 running 상태에서 ready 상태로 전환될 때
3. 프로세스가 waiting 상태에서 ready 상태로 전환될 때
4. 프로세스가 종료할 때

1번 입출력 요청이 발생해 자발적으로 waiting 상태로 가거나, 4번 프로세스가 끝나 main함수를 return 하거나 exit를 호출해 terminate 상태로 가는 경우는 비선점 방식을 사용한다.
2번과 3번의 경우 선점과 비선점 방식을 선택할 수 있지만, 현대 OS의 경우 대부분 선택 폭이 넓고 효율을 높일 수 있는 선점 스케줄링 방식을 사용한다고 한다.

#### 디스패처(Dispatcher)

문맥교환을 해주는 모듈을 디스패처라고 한다. (프로세스에게 CPU를 넘겨주는 역할) - 스케줄러가 변경할 프로세스를 선택하면 디스패처가 실제로 변경시키는 역할을 수행한다.

디스패처가 하나의 프로세스를 정지하고 다른 프로세스의 수행을 시작하는 데까지 소요되는 시간을 디스패치 지연(dispatch latency)라고 한다.
디스패처는 모든 프로세스의 문맥 교환 시 호출되므로, 디스패치 지연 시간은 가능한 짧아야 한다.

#### 스케줄링 기준(Scheduling Criteria)

1. CPU 이용률 : CPU가 놀지 않고 최대한 바쁜 상태를 유지하는 것이 목표
2. 처리량(throught) : 단위 시간당 완료된 프로세스의 개수를 최대로 하는 것이 목표
3. 총처리 시간(turnaround time) : 프로세스를 실행하는데 소요된 총 시간을 줄이는 것이 목표
4. 대기시간(waiting time) : 프로세스가 ready 큐에서 대기하는 시간을 최소화하는 것이 목표
5. 응답시간(response time) : 응답시간을 최소화하는 것이 목표
