---
title: OSI 7 Layer & TCP/IP 4 Layer
published: 2026-07-12
description: 'OSI 7계층과 TCP/IP 4계층의 구조, 각 계층의 역할 및 대표 프로토콜을 정리합니다.'
image: '/assets/img/Network/Osi7/osi7tcpip4.png'
tags: [Network, OSI, TCP/IP, Layer, PDU]
category: 'Network'
draft: false
---

<figure style="text-align: center;">
  <img src="/assets/img/Network/Osi7/osi7tcpip4.png" alt="osi" style="display: block; margin: 0 auto;">
  <figcaption>OSI 7 Layer & TCP/IP 4 Layer</figcaption>
</figure>

:::tip
OSI 7계층은 이론적 모델이며, TCP/IP 4계층은 실제 인터넷에서 사용되는 실용적인 모델이다.
:::

## 1. OSI 7계층이란 ? 

OSI(Open Systems Interconnection) 7계층은 네트워크 통신 과정을 7개의 계층으로 나눠 표준화 한 참조 모델이다.  

각 계층은 서로 다른 역할을 수행하며, 하위 계층의 서비스를 이용하여 데이터를 전달한다.  

| 계층 | 이름 | 역할 | 대표 프로토콜 |
| :--: | :--: | :--: | :--: |
| L7 | **Application** (응용) | 사용자에게 네트워크 서비스 제공 | HTTP, FTP, SMTP, DHCP, DNS |
| L6 | **Presentation** (표현) | 데이터 형식 변환, 암호화, 압축 | TLS/SSL |
| L5 | **Session** (세션) | 세션 생성, 유지, 종료 | NetBIOS, RPC |
| L4 | **Transport** (전송) | 신뢰성 있는 데이터 전송 | TCP, UDP |
| L3 | **Network** (네트워크) | IP 주소 지정 및 라우팅 | IP, ICMP, ARP |
| L2 | **Data Link** (데이터 링크) | MAC 주소 기반 프레임 전송 | Ethernet, PPP, HDLC |
| L1 | **Physical** (물리) | 전기/광 신호 전송 | - |

--- 

## 2. TCP/IP 4계층이란 ?

TCP/IP 모델은 실제 인터넷에서 사용하는 네트워크 모델로 OSI 7계층을 4개의 계층으로 단순화한 구조이다.  

|         계층        | 역할             | 대표 프로토콜                                               |
| :---------------: | :------------- | :---------------------------------------------------- |
|    **Application**    | 사용자 서비스 제공     | HTTP, HTTPS, FTP, SMTP, POP3, DNS, DHCP, SNMP, Telnet |
|     **Transport**     | 종단 간 데이터 전송    | TCP, UDP                                              |
|      **Internet**     | IP 주소 지정 및 라우팅 | IP, ICMP, ARP, RARP                                   |
| **Network Interface** | 네트워크 접근        | Ethernet, PPP, HDLC                                   |


---

## 캡슐화 & 역캡슐화

### 캡슐화(Encapsulation)

- 송신 측에서 상위 계층의 데이터가 하위 계층으로 전달되면서 각 계층의 헤더 또는 트레일러가 추가되는 과정이다.

### 역캡슐화(Decapsulation)

- 수신 측에서 하위 계층으로부터 상위 계층으로 전달되면서 각 계층의 헤더 또는 트레일러를 제거하여 원래의 데이터를 복원하는 과정이다.

--- 

## 데이터 전송 과정

1. 송신 측에서 데이터가 계층별로 캡슐화 된다.
2. 물리 계층에서 비트 신호로 변환되어 전송 매체를 통해 수신 측으로 전달된다.
3. 수신 측에서 계층별로 역캡슐화되어 원래 데이터가 복원된다.


> 전송 계층에서는 TCP 또는 UDP 헤더가 추가되며 데이터 링크 계층에서는 헤더와 트레일러가 추가된다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/Osi7/capsulation.png" alt="capsul" style="display: block; margin: 0 auto;">
  <figcaption>캡슐화 & 역캡슐화 과정</figcaption>
</figure>

:::note[헤더에 포함되는 주요 정보] 
- 전송 계층: 송/수신 포트 번호
- 네트워크 계층: 송/수신 IP 주소
- 데이터 링크 계층: 송/수신 MAC 주소, FCS(트레일러)
:::

---

## PDU (Protocol Data Unit)

> 데이터는 계층을 거치면서 헤더가 추가되고 계층마다 다른 이름으로 불린다.

| 계층          | PDU                          |
| :-----------: | :----------------------------: |
| **Application** | Data                         |
| **Transport**   | Segment(TCP) / Datagram(UDP) |
| **Network**     | Packet                       |
| **Data Link**  | Frame                        |
| **Physical**    | Bit                          |


---