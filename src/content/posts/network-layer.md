---
title: L3 - Network Layer
published: 2026-07-14T00:00:00
description: 'OSI 3계층의 역할과 핵심 개념을 정리합니다.'
image: '/assets/img/Network/L3/ipv4.png'
tags: [Network, L3, IPv4, IPv6, Routing, ICMP, NAT, Subnet, CIDR, Classful, routing, EGP, BGP]
category: 'Network'
draft: false
---

## 1. Network Layer

Network Layer는 OSI 7계층에서 3계층에 해당한다.  

L3는 `IP 주소`를 기반으로 서로 다른 네트워크 간 Packet을 전달한다.

이 계층에서 데이터는 `Packet` 단위로 다뤄진다.

**[주요 역할]**

- IP 주소 기반 통신
- 서로 다른 네트워크 간 Packet 전달
- 목적지 네트워크 판단
- 라우팅 경로 결정
- 오류 및 진단 메시지 처리

:::tip
Network Layer는 목적지 IP 주소를 기준으로 Packet이 어느 네트워크로 가야 하는지 결정하는 계층이다.
:::

---

## 2. IPv4

`IPv4(Internet Protocol version 4)`는 가장 널리 사용되는 L3 주소 체계이다.

IPv4 주소는 32bit로 구성되며 사람이 읽기 쉽도록 8bit 단위의 10진수로 표현한다.


<figure style="text-align: center;">
  <img src="/assets/img/Network/L3/ipv4.png" alt="ipv4" style="display: block; margin: 0 auto;">
  <figcaption>IPv4</figcaption>
</figure>


IPv4 주소는 크게 `Network ID`와 `Host ID`로 나눌 수 있다.

- **Network ID** : 장비가 속한 네트워크를 식별
- **Host ID** : 해당 네트워크 안의 장비를 식별

---

## 3. IPv6

`IPv6(Internet Protocol version 6)`는 IPv4 주소 부족 문제를 해결하기 위해 만들어진 IP 주소 체계이다.

IPv6 주소는 128bit로 구성되며 16bit씩 8개의 그룹으로 나누어 16진수와 콜론(`:`)을 사용해 표현한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L3/ipv6.png" alt="ipv6" style="display: block; margin: 0 auto;">
  <figcaption>IPv6</figcaption>
</figure>

IPv4가 약 43억 개의 주소를 제공하는 반면 IPv6는 거의 무한에 가까운 주소 공간을 제공한다.

또한 NAT 의존도를 줄이고 주소 자동 설정(SLAAC)과 같은 기능을 지원하여 대규모 네트워크 환경에 적합하다.

|   구분  |                IPv4               |               IPv6              |
| :---: | :-------------------------------: | :-----------------------------: |
| 주소 길이 |          32bit (8bit × 4)         |        128bit (16bit × 8)       |
| 표현 방식 |            10진수 (`0~9`)           |       16진수 (`0~9`, `A~F`)       |
| 주소 예시 |           `192.168.10.5`          |          `2001:db8::1`          |
| 전송 방식 | Unicast<br>Multicast<br>Broadcast | Unicast<br>Multicast<br>Anycast |

> IPv6는 Broadcast를 지원하지 않으며 필요한 경우 Multicast와 Anycast를 사용한다.


---

## 4. Subnet Mask

Subnet Mask는 IP 주소에서 Network ID와 Host ID를 구분하기 위해 사용한다.

예를 들어 아래와 같은 주소가 있다고 가정한다.

```text
IP Address  : 192.168.10.5
Subnet Mask : 255.255.255.0
```

이 경우 `192.168.10.0`은 네트워크 주소가 되고 `5`는 해당 네트워크 안의 호스트를 의미한다.

```text
Network Address : 192.168.10.0
Host Address    : 192.168.10.5
Broadcast       : 192.168.10.255
```

---

## 5. CIDR

`CIDR(Classless Inter-Domain Routing)`은 Subnet Mask를 `/prefix` 형태로 표현하는 방식이다.

```text
192.168.10.0/24
```

위 표현에서 `/24`는 앞의 24bit가 네트워크 영역이라는 뜻이다.

| CIDR | Subnet Mask | 사용 가능한 Host 수 |
| :--: | :--: | :--: |
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /30 | 255.255.255.252 | 2 |

CIDR은 네트워크를 더 작게 나누거나 여러 네트워크를 하나로 묶어 표현할 때 사용한다.

---

## 6. Classful Addressing

Classful Addressing은  IPv4 주소 체계에서 IP 주소를 A, B, C, D, E 클래스로 나누어 사용하는 방식이다.

각 클래스는 기본 Subnet Mask가 정해져 있으며 네트워크 규모에 따라 구분된다.

| Class | Range | Default Subnet Mask | Usage |
| :--: | :--: | :--: | :-- |
| A | 1.0.0.0 ~ 126.255.255.255 | 255.0.0.0 | 대규모 네트워크 |
| B | 128.0.0.0 ~ 191.255.255.255 | 255.255.0.0 | 중간 규모 네트워크 |
| C | 192.0.0.0 ~ 223.255.255.255 | 255.255.255.0 | 소규모 네트워크 |
| D | 224.0.0.0 ~ 239.255.255.255 | - | Multicast |
| E | 240.0.0.0 ~ 255.255.255.255 | - | Reserved |

---

## 7. Network Address

Network Address는 특정 네트워크 자체를 나타내는 주소이다.

Host 영역의 bit가 모두 `0`인 주소가 Network Address가 된다.

```text
IP Address      : 192.168.10.5/24
Network Address : 192.168.10.0
```

Network Address는 네트워크를 식별하기 위한 주소이므로 일반 호스트에 직접 할당하지 않는다.

---

## 8. Broadcast Address

IPv4에서 Broadcast Address는 같은 네트워크 안의 모든 호스트에게 데이터를 보낼 때 사용하는 주소이다.

Host 영역의 bit가 모두 `1`인 주소가 Broadcast Address가 된다.

```text
IP Address        : 192.168.10.5/24
Broadcast Address : 192.168.10.255
```

Broadcast Address 역시 네트워크 전체를 대상으로 하는 주소이므로 일반 호스트에 직접 할당하지 않는다.

---

## 9. Default Gateway

Default Gateway는 내 네트워크 밖으로 나갈 때 Packet을 전달하는 기본 경로이다.

같은 네트워크 안의 장비끼리는 L2 통신으로 직접 전달할 수 있지만 다른 네트워크로 가려면 라우터를 거쳐야 한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L3/gateway.png" alt="gateway" style="display: block; margin: 0 auto;">
  <figcaption>Gateway</figcaption>
</figure>

이때 사용하는 라우터의 IP 주소가 Default Gateway이다.

---

## 10. Routing

Routing은 목적지 IP 주소를 보고 Packet을 어느 경로로 보낼지 결정하는 과정이다.

라우터는 목적지 IP가 어느 네트워크에 속하는지 확인한 뒤 다음 장비(Next Hop) 또는 출력 인터페이스를 선택한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L3/routing.png" alt="routing" style="display: block; margin: 0 auto;">
  <figcaption>Routing</figcaption>
</figure>

1. Packet의 Destination IP를 확인한다.
2. Routing Table에서 가장 적합한 경로를 찾는다.
3. Next Hop 또는 출력 인터페이스를 결정한다.
4. 해당 경로로 Packet을 전달한다.
5. 다음 라우터가 동일한 과정 반복
6. 목적지 네트워크에 도착하면 최종 호스트에게 전달

---

## 11. Routing Table

Routing Table은 목적지 네트워크별로 Packet을 어디로 보낼지 정리한 표이다.

라우터나 운영체제는 Routing Table을 보고 Packet의 다음 경로를 결정한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L3/routingtable.png" alt="routingtable" style="display: block; margin: 0 auto;">
  <figcaption>Routing table</figcaption>
</figure>

| 항목 | 의미 |
| :--: | :-- |
| Destination | 목적지 네트워크 |
| Gateway | 다음으로 전달할 장비 |
| Interface | Packet을 내보낼 인터페이스 |
| Metric | 경로의 우선순위 | 
---

## 12. Static Routing

Static Routing은 관리자가 직접 경로를 설정하는 방식이다.

네트워크 구조가 단순하고 변경이 적은 환경에서는 정적 라우팅을 사용하기 쉽다.

```text
ip route 10.10.20.0 255.255.255.0 10.10.10.2
```

| 구분 | 내용 |
| :--: | :-- |
| 장점 | 구조가 단순하고 예측하기 쉽다 |
| 단점 | 네트워크 변경 시 관리자가 직접 수정해야 한다 |

---

## 13. Dynamic Routing

Dynamic Routing은 라우터끼리 라우팅 정보를 교환하여 최적의 경로를 자동으로 학습하는 방식이다.

네트워크 규모가 커지거나 경로 변경이 자주 발생하는 환경에서는 사용된다.

**[대표적인 Dynamic Routing Protocol]**

```text
Routing
├── Static Routing
└── Dynamic Routing
    ├── IGP (Interior Gateway Protocol) // 동일 AS 내부
    │   ├── RIP
    │   ├── OSPF
    │   ├── IS-IS
    │   └── EIGRP (Cisco)
    │
    └── EGP (Exterior Gateway Protocol) // 서로 다른 AS 간
        └── BGP
```

---

## 14. RIP

`RIP(Routing Information Protocol)`은 거리 벡터 기반 라우팅 프로토콜이다.

RIP은 목적지까지의 `Hop Count`를 기준으로 경로를 선택한다.

**[특징]**

- Bellman-Ford 알고리즘 사용
- Hop Count를 Metric로 사용하며 최대 15 Hop까지 지원
- 큰 네트워크보단 소규모 네트워크 환경에 적합

---

## 15. OSPF

`OSPF(Open Shortest Path First)`는 링크 상태 기반 라우팅 프로토콜이다.

OSPF는 네트워크의 링크 상태 정보를 교환하고 이를 기반으로 최단 경로를 계산한다.

**[특징]**

- Dijkstra 알고리즘(SPF) 사용
- Link State 기반
- Cost(Metric)를 기준으로 최적 경로 계산
- Area 단위로 네트워크를 나눌 수 있음

---

## 16. BGP

`BGP(Border Gateway Protocol)`는 `AS(Autonomous System)` 간 라우팅에 사용되는 프로토콜이다.

인터넷은 여러 AS가 연결된 구조이며 BGP는 AS 사이에서 라우팅 정보를 교환하여 최적의 경로를 결정한다.

**[특징]**

- AS 간 라우팅
- 인터넷 백본에서 사용
- 경로 속성(Path Attribute)을 기반으로 경로 선택

---

## 17. ICMP

`ICMP(Internet Control Message Protocol)`는 IP 통신 중 발생하는 오류나 진단 정보를 전달하기 위한 프로토콜이다.

대표적으로 `ping` 명령에서 사용하는 Echo Request와 Echo Reply가 ICMP를 사용한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L3/icmp.png" alt="ICMP" style="display: block; margin: 0 auto;">
  <figcaption>ICMP</figcaption>
</figure>

**[사용 예시]**
- Echo Request / Echo Reply
- Destination Unreachable
- Time Exceeded
---

## 18. NAT

`NAT(Network Address Translation)`는 IP 주소를 변환하는 기술이다.

일반적으로 내부 사설 IP를 외부 공인 IP로 변환할 때 사용한다.

```text
192.168.10.5:50000 -> 203.0.113.10:40001
```

NAT를 사용하면 여러 내부 장비가 하나의 공인 IP를 공유하여 외부 네트워크와 통신할 수 있다.

**[주요 목적]**

- 사설 IP를 공인 IP로 변환하여 외부 네트워크와 통신할 수 있게 한다.
- 하나의 공인 IP를 여러 내부 장비가 공유하여 공인 IP를 절약한다.
- 외부에서는 공인 IP만 보이므로 내부 네트워크 구조를 숨길 수 있다.
- IP와 Port를 함께 `변환(PAT)`하여 여러 내부 호스트가 동시에 외부와 통신할 수 있도록 한다.


---