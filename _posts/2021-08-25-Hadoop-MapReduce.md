---
layout: post
title: MapReduce 알고리즘 [Hadoop]
description: >
  MapReduce 알고리즘 [Hadoop]
tags: []
---


## MapReduce 알고리즘

### 병렬분산 알고리즘을 사용하는 이유
 고가의 서버들은 가격에 관점에서는 선형적으로 성능이 증가하지 않는다. 예를 들어 두배 성능의 프로세서 한개를 가진 컴퓨터의 가격이 일반적인 프로세서 한개를 가진 컴퓨터의 가격의 두 배보다 훨씬 더 비싸다.
 
 따라서, 데이터 중심( data-intensive ) 애플리케이션 분야에서는 아주 많은 값싼 서버들을 많이 이용하는 것(Scale-out)을 선호한다.

 #### MapReduce

 * MapReduce는 빅데이터를 이용한 효율적인 계산이 가능한 첫 번째 프로그래밍 모델 
 * 값싼 컴퓨터들을 모아서 클러스터를 만들고 여기에서 빅 데이터를 처리하기 위한 스케일러블(scalable) 병렬 소프트웨어의

