---
layout: post
title: 트랜스포트 계층 [ 혼잡제어 ]
description: >
  [컴퓨터 네트워킹 하향식 접근 정리] 프랜스포트 계층: 혼잡제어  <br>
  참고 : http://www.kocw.net/home/search/kemView.do?kemId=1312397

tags: [Network]
---

## 트랜스포트 계층 [ 혼잡제어]

### TCP 혼잡제어

혼잡제어를 하기 위해서는 현재 네트워크가 현재 혼잡상태인지 아닌지의 판단이 필요하다. 이 판단은 데이터 세그먼트를 보내고 돌려받는 ACK로 판단한다. 세그먼트롤 보냈는데 ACK가 잘 온다면 현재 네트워크는 혼잡상태가 아니라는 것을 의미한다.

### TCP 송신자가 트래픽을 보내는 전송률 제어 방법

TCP 연결의 양 끝 각 호스트들은 수신 버퍼, 송신버퍼 그리고 몇가지의 변수(LastByteRead, rwnd 등)로 구성된다. 송신측에서 동작하는 TCP 혼잡제어 매커니즘은 추가적인 변수인 **혼합 윈도우(congestion window)**를 기록한다. cwnd(congestion window size, 지금 네트워크가 받아들일 수 있는 데이터 양)로 표시되는 혼잡 윈도우는 TCP 송신자가 네트워크로 트래픽을 전송할 수 있는 비율을 제한하도록 한다. 대략적으로 매 왕복시간(RTT)의 시작 때, 송신자는 앞에서 cwnd바이트만큼의 데이터를 전송할 수 있고, RTT가 끝나는 시점에 데이터에 대한 확인응답을 수신한다. 그러므로 송신자의 송신율은 대략 cwnd/RTT 바이트/초이다. cwnd의 값을 조절하여, 송신자는 링크에 데이터를 전송하는 비율을 조절할 수 있다.

![](https://taeho0304.github.io/assets/img/NW/transport/congestioncontrol/cwnd_control.PNG)

Cwnd의 조작은 다음과 같이 이루어진다. 세그먼트를 보내고 그에 대한 피드백(ACK)을 잘 받는다면 그 세그먼트 사이즈만큼 cwnd를 늘려준다. 보통 세그먼트 사이즈는 MSS(Maxium Segment Size)단위로 전송한다. 이렇게 cwnd를 늘려나가는 방법을 additive increase 라고 한다.

cwnd를 계속해서 늘려가다보면 어느순간 네트워크에 문제가 발생한다. 네트워크에 문제가 생겨 ACK가 돌아오지 않는다면 cwnd를 절반으로 줄여준 뒤 다시 additive increase를 시작한다. 이 cnwd를 절반으로 줄여주는 작업을 multiplicative decrease 라고 한다.

### Slow Start & Congestion Avoidance & Fast Recovery

![](https://taeho0304.github.io/assets/img/NW/transport/congestioncontrol/slowstart.PNG)

처음 TCP연결이 시작될 때 cwnd의 값은 일반적으로 조심스럽게 1MMS로 시작한다. 이를 **Slow Start**라 한다. 하지만 1MMS로 시작하면 cwnd가 증가하는데 시간이 오래 걸리기 때문에 cwnd를 1, 2, 4, 8 ... 과 같이 2배식 증가 시켜준다. 하지만 이렇게 계속해서 2배씩 증가시키면 금방 네트워크 혼잡이 발생하기 때문에 일정 지점부터는 additive increase 방식으로 cwnd를 증가시킨다. 이 지점을 ssthresh(slow start treshold)라 하고 이러한 방식을 **Congestion Avoidance**라 한다.

![](https://taeho0304.github.io/assets/img/NW/transport/congestioncontrol/congestionavoidance.PNG)

혼잡 회피에서 패킷 loss 가 발생했을때의 동작은 TCP의 버전과 무슨 유형의 패킷손실(3 duplicate ACK, timeout)이 발생했는지에 따라 달라진다. TCP Tahoe라 불리는 초기 TCP 버전은 timeout나 duplicate ACK가 발생했을때를 구분하지 않고 무조건 손실이 발생하면 무조건 혼잡 윈도우를 1MSS로 줄이고, slow start 단계로 들어간다. 새로운 TCP 버전인 TCP Reno는 3 duplicate ACK일 때와 timeout일 때를 구분하여 동작한다. timeout이 발생했을 때는 Taeho와 같이 cwnd를 1MMS로 시작해 slow start 단계를 진행하지만 3 duplicate ACK일때는 cwnd는 cwnd/2로 값이 변경되고 additive increase로 cwnd를 증가시킨다.

![](https://taeho0304.github.io/assets/img/NW/transport/congestioncontrol/congestioncontrol.PNG)
