---
layout: post
title: CPU 스케줄링 (Part 2)
description: >
  [공룡책 정리] CPU 스케줄링: Chapter 5. CPU Scheduling (Part 1)
tags: [OS]
---

## CPU 스케줄링: Chapter 5. CPU Scheduling (Part 2)

## CPU 스케줄링 알고리즘

#### 선입 선처리 스케줄링(First-Come, First-Served Scheduling)

비선점 스케줄링 방식으로 CPU를 먼제 요청하는 프로세스를 우선적으로 할당하는 방식이다. FIFO 큐를 사용해 쉽게 관리할 수 있다. 가장 간단한 CPU 스케줄링 기법이지만 현재에는 거의 사용되지 않는 스케줄링 기법이다.

| Process | Burst time | Trunaround time | Waiting time |
| :------ | :--------- | :-------------- | :----------- |
| P1      | 24         | 24              | 0            |
| P2      | 3          | 27              | 24           |
| P3      | 3          | 30              | 27           |

![](https://taeho0304.github.io/assets/img/OS/fcfs.PNG)

촏 대기시간 : ( 0 + 24 + 27 ) = 51
평균 대기시간 : 51 / 3 =17

모든 다른 프로세스들이 하나의 긴 프로세스가 CPU를 양도하기를 기다리는 것을 호위 효과(convoy effect)라고 한다. 이 효과는 짧은 프로세스들이 먼저 처리되도록 허용될 때보다 CPU와 장치 이용률이 저하되는 결과를 낳는다.

#### 최단 작업 우선 스케줄링 (Shortest-Job-First Scheduling)

비선점 스케줄링 방식으로 CPU burst 길이가 짧은 순서대로 할당하는 방식이다.

| Process | Burst time | Trunaround time | Waiting time |
| :------ | :--------- | :-------------- | :----------- |
| P1      | 6          | 9               | 3            |
| P2      | 8          | 24              | 16           |
| P3      | 7          | 16              | 9            |
| P4      | 3          | 3               | 0            |

![](https://taeho0304.github.io/assets/img/OS/sjf.PNG)

촏 대기시간 : ( 3 + 16 + 9 + 0 ) = 28
평균 대기시간 : 28 / 4 =7
