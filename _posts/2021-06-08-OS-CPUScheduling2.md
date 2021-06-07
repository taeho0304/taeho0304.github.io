---
layout: post
title: CPU 스케줄링 (Part 2)
description: >
  [공룡책 정리] CPU 스케줄링: Chapter 5. CPU Scheduling (Part 1)
tags: [OS]
---

## CPU 스케줄링: Chapter 5. CPU Scheduling (Part 2)

## CPU 스케줄링 알고리즘

### 선입 선처리 스케줄링(First-Come, First-Served Scheduling)

비선점 스케줄링 방식으로 CPU를 먼제 요청하는 프로세스를 우선적으로 할당하는 방식이다. FIFO 큐를 사용해 쉽게 관리할 수 있다. 가장 간단한 CPU 스케줄링 기법이지만 현재에는 거의 사용되지 않는 스케줄링 기법이다.

| Process | Burst time | Trunaround time | Waiting time |
| :------ | :--------- | :-------------- | :----------- |
| P1      | 24         | 24              | 0            |
| P2      | 3          | 27              | 24           |
| P3      | 3          | 30              | 27           |

![](https://taeho0304.github.io/assets/img/OS/fcfs.PNG)

총 대기시간 : ( 0 + 24 + 27 ) = 51
평균 대기시간 : 51 / 3 =17

모든 다른 프로세스들이 하나의 긴 프로세스가 CPU를 양도하기를 기다리는 것을 호위 효과(convoy effect)라고 한다. 이 효과는 짧은 프로세스들이 먼저 처리되도록 허용될 때보다 CPU와 장치 이용률이 저하되는 결과를 낳는다.

### 최단 작업 우선 스케줄링 (Shortest-Job-First Scheduling)

CPU burst 길이가 짧은 순서대로 할당하는 방식이다. 최적의 알고리즘이지만 다음 CPU burst 길이를 알 수 있는 방법이 없어 근사적인 값을 활용해 구현한다.(앞서 처리한 프로세스들의 기록을 보고 추측한다.)

SJF 알고리즘은 선점형이거나 또는 비선점형일 수 있다. 앞의 프로세스가 실행되는 동안 새로운 프로세스가 Ready Queue에 도착하면 선택이 발생한다. 새로운 프로세스가 현재 실행되고 있는 프로세스의 남은 시간보다도 더 짧은 CPU burst를 가질 수 있다. 선점형 SJF 알고리즘은 현재 실행하는 프로세스를 선점할 것이고 반면에 비선점형 SJF 알고리즘은 현재 실행하고 있는 프로세스가 자신의 CPU 버스트를 끝내도록 허용한다. 선점형 SJF 알고리즘은 **최소 잔여 시간 우선 스케줄링(shortest remaining time first)**이라고 불립니다.

비선점형일 경우 CPU burst 시간이 큰 프로세스는 계속 뒤로 밀려나는 기아(Starvation)가 발생한다.

| Process | Burst time | Trunaround time | Waiting time |
| :------ | :--------- | :-------------- | :----------- |
| P1      | 6          | 9               | 3            |
| P2      | 8          | 24              | 16           |
| P3      | 7          | 16              | 9            |
| P4      | 3          | 3               | 0            |

![](https://taeho0304.github.io/assets/img/OS/sjf.PNG)

총 대기시간 : ( 3 + 16 + 9 + 0 ) = 28
평균 대기시간 : 28 / 4 =7

### 최소 잔여 시간 우선 스케줄링(shortest remaining time first)

프로세스의 남은 수행 시간이 짧은 순서에 따라 프로세서에 할당하는 방식으로, 수행 중 다른 프로세스보다 남은 수행 시간이 적어지면 운영체제가 개입해 자리를 바꾸는 선점 스케줄링 방식이다.

SJF에서 발생하는 기아 문제를 해결할 수 있다.

| Process | Arrival time | Burst time | Trunaround time | Waiting time  |
| :------ | :----------- | :--------- | :-------------- | :------------ |
| P1      | 0            | 8          | 17              | 9 ( 10 - 1 )  |
| P2      | 1            | 4          | 4               | 0 ( 1 - 1 )   |
| P3      | 2            | 9          | 26              | 15 ( 17 - 2 ) |
| P4      | 3            | 5          | 7               | 2 ( 5 - 3 )   |

![](https://taeho0304.github.io/assets/img/OS/srtf.PNG)

총 대기시간 : ( 9 + 0 + 15 + 2 ) = 26
평균 대기시간 : 26 / 4 =6.5 ( \* 똑같은 데이터로 SJF 스케줄링 수행 시 7.75)

### 라운드 로빈 스케줄링( Round - Roubin Scheduling )
