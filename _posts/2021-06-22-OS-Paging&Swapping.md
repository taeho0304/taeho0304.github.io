---
layout: post
title: 페이징과 스와핑
description: >
  [공룡책 정리] 주메모리의 관리: Chapter 9. Main Memory (Part 2)

tags: [OS]
---

## 페이징과 스와핑

#### 페이징이란?

프로세스가 적재되는 물리 주소 공간이 연속적이지 않아도 적재를 허용하는 메모리 관리 기법이다. 연속 메모리 할당 기법에서 발생하는 외부 단편화와 그에따른 압축 작업을 피할 수 있다.

#### 기본 방법

#### 페이지 테이블 만들기

테이블 페이지의 하드웨어 구현은 여러가지 방법으로 될 수 있다.

먼저 페이지의 크기가 작은 경우 **CPU 내부**에 페이지 테이블을 만들 수 있다. 이 경우 여러개의 레지스터를 이용해 테이블을 만들어 주소 변환 속도가 매우 빠르다. 하지만 테이블의 **크기가 매우 제한**된다는 단점이 있어 거의 사용되지 않는 방법이다.

크기 제한을 해결하기 위해 대부분의 컴퓨터는 페이지 테이블을 **주 메모리**에 저장하고 페이지 테이블 기준 레지스터 ( PTBR, Page-Table Base Register )로 하여금 페이지 테이블을 가리키는 방법을 사용한다. 하지만 CPU는 프로세스의 주소에 접근하기 위해서 메모리에 위치한 페이지 테이블에 한 번, 실제 주소에 접근하는데 한 번으로 메모리에 총 2번 접근해야 한다. 따라서 **메모리 접근 속도가 두 배로 느려진다**는 단점이 있다.

위의 메모리 접근 속도 문제를 해결하기 위해 **TLB (Translation Look-aside Buffer)**라고 불리는 특수한 소형 하드웨어 캐시를 사용한다. TLB 내의 각 항목은 **key(페이지 번호)**와 **value(프레임 번호)**의 두 부분으로 구성된다.

![](https://taeho0304.github.io/assets/img/OS/Paging/tlb.PNG)

접근하려는 메모리의 페이지 번호가 TLB에서 발견(TLB hit)되는 비율 **적중률(hit ratio)**이라고 부른다.

예를 들어 메모리 접근 시간을 10ns라고 가정하고 TLB를 이용한 실제 메모리 접근 시간을 한 번 계산해보자.

- 80% hit ratio : 0.80 x 10 + 0.20 x 20 = 12ns
- 99% hit ratio : 0.99 x 10 + 0.01 x 20 = 10.1ns

#### 보호

페이지 테이블의 각 엔트리에는 **유효/무효**라는 비트가 있다. 이 비트가 유효로 설정되면 그 페이지는 프로세스의 합법적인 페이지이고, 무효로 설정되면 프로세스의 논리 주소 공간에 속하지 않는다다. 운영체제는 이 비트를 이용해서 그 페이지에 대한 접근을 허용하거나 또는 허용하지 않을 수 있다.

![](https://taeho0304.github.io/assets/img/OS/Paging/protection.PNG)

위 프로그램은 10,468의 주소만을 사용할 수 있다. 페이지의 크기가 2KB 일 때 위와 같이 1,2,3,4,5의 주소는 페이지 테이블을 통해서 정상적으로 매핑되지만, 6,7을 매핑하려고 시도하면 비트가 무효로 설정된 것을 발견해 트랩을 발생시킨다.

#### 공유페이지

![](https://taeho0304.github.io/assets/img/OS/Paging/protection.PNG)

페이징 기법을 사용하면 코드를 쉽게 공유할 수 있다. 실행 중 절대로 변하지 않는 **재진입 가능 코드(reentrant code)**두 개나 그 이상의 processor들이 동시에 같은 코드를 수행할 수 있다. 같은 코드를 실행시키지만 읽기만 하기 때문에 동기화나 데드락같은 문제가 발생하지 않는다.