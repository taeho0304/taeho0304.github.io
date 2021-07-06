---
layout: post
title: 트랜스포트 계층 [ 혼잡 원인 ]
description: >
  [컴퓨터 네트워킹 하향식 접근 정리] 프랜스포트 계층: 혼잡 원인 <br>
  참고 : http://www.kocw.net/home/search/kemView.do?kemId=1312397

tags: [Network]
---

## 트랜스포트 계층 [ 혼잡 원인]

### 혼잡의 원인과 비용

- 혼잡 : network이 처리할 수 있는 양보다 더 많은 데이터가 들어왔을 때 생기는 현상
  1. 패킷유실
  2. 지연

#### 시나리오 1 : 2개의 송신자와 무한 버퍼를 갖는 하나의 라우터

![](https://taeho0304.github.io/assets/img/NW/transport/congestioncontrol/scenario1.PNG)

위 그림에서 라우터가 하나 있는데, 이 라우터에 있는 큐는 무제한이고 재전송이 없다고 가정한다. 그리고 라우터의 bandwidth는 R이라 가정해보자.송신자가 보내는 모든 데이터는 유한한 지연으로 수신자에서 수신된다. 송신자가 보내는 λin(전송률)을 쭉 증가시키면 받는 속도 λout또한 계속해서 증가한다. 그러나 λin이 R/2 이상일 때, 처리량은 단지 R/2이다. 이러한 제한은 두 연결 사이에서 링크 용량 공유의 결과이다.

R/2의 연결당 처리량을 얻는 것은 목적지에 패킷을 전달하는 데 링크를 최대로 활용하므로 좋은 현상이다. 그러나 전송률이 R/2에 근접했을 때, 평균 지연은 점점 커져 R/2를 초과할 때 출발지와 목적지 사이의 평균 지연이 무제한이 된다. **즉 패킷 도착률이 링크용량에 근접함에 따라 큐잉 지연이 커진다.**

![](https://taeho0304.github.io/assets/img/NW/transport/congestioncontrol/scenario1_delay.PNG)

#### 시나리오 2 : 2개의 송신자, 유한 버퍼를 가진 하나의 라우터

![](https://taeho0304.github.io/assets/img/NW/transport/congestioncontrol/scenario2.PNG)

시나리오 2에서는 좀 더 현실적으로 큐의 크기가 제한되고 재전송이 있다 (timeout이 존재). 즉 라우터 버퍼가 꽉 차서 패킷이 손실되거나 패킷이 크게 지연되어 송신측의 타이머가 타임아웃될 때 재전송이 일어난다. 재전송이 존재하므로 Application에서 전송하는 데이터 λin 보다 실제로 TCP가 보내는 λ'in이 더 크다. **즉 버퍼 오버플로우니 지연으로 인한 제전송은 라우터가 패킷의 불필요한 복사본들을 전송하는데 링크 대역폭을 사용하는 원인이 된다.**

#### 시나리오 3 : 4개의 송신자와 유한 버퍼를 가진 라우터, 그리고 멀티홉 경로

![](https://taeho0304.github.io/assets/img/NW/transport/congestioncontrol/scenario3.PNG)

조금 더 현실적으로 각각 모든 호스트가 라우터 2개를 거쳐 목적지로 가는 상황이다. 위의 상황에서 보내는 속도 λin을 증가시키면 처음에는 네트워크가 상태가 좋아 λout 또한 증가한다. 하지만 특정 시점부터 보내는 양을 늘리면 오히라 받는양이 줄어드는 경우가 발생한다. 트래픽이 많아지면 각 라우터 퍼버에는 빈 공간이 없어지게 된다. 그런 상황이 발생하면 **패킷이 송신지에서 두 번째 라우터에 도달할 때마다 버려지게 되고 앞의 upstream 라우터에서 사용된 전송량은 헛된 것이 된다.** 이러한 현상은 트래픽이 크면 클수록 더욱 심해진다.

![](https://taeho0304.github.io/assets/img/NW/transport/congestioncontrol/scenario3_performance.PNG)

### TCP 혼잡제어

#### TCP 송신자가 연결로 트래픽을 보내는 전송률을 어떻게 제한하는가?

TCP 연결의 양 끝 각 호스트들은 수신 버퍼, 송신버퍼 그리고 몇가지의 변수(LastByteRead, rwnd 등)로 구성된다. 송신측에서 동작하는 TCP 혼잡제어 매커니즘은 추가적인 변수인 **혼합 윈도우(congestion window)**를 기록한다. cwnd(congestion window size)로 표시되는 혼잡 윈도우는 TCP 송신자가 네트워크로 트래픽을 전송할 수 있는 비율을 제한하도록 한다. 대략적으로 매 왕복시간(RTT)의 시작 때, 송신자는 앞에서 언급한 제약 조건에 따라 cwnd바이트만큼의 데이터를 전송할 수 있고, RTT가 끝나는 시점에 데이터에 대한 확인응답을 수신한다. 그러므로 송신자의 송신율은 대략 cwnd/RTT 바이트/초이다. cwnd의 값을 조절하여, 송신자는 링크에 데이터를 전송하는 비율을 조절할 수 있다.
