---
title: L2 - Data Link Layer
published: 2026-07-11T00:00:00
description: 'OSI 2계층의 역할과 핵심 개념을 정리합니다.'
image: '/assets/img/Network/L2/wireshark.png'
tags: [Network, L2, Ethernet, MAC, ARP, VLAN, STP]
category: 'Network'
draft: false
---

## 1. Data Link Layer

Data Link Layer는 OSI 7계층에서 2계층에 해당한다.  

데이터링크 계층은 같은 네트워크 내에서 `MAC 주소`를 이용하여 실제 장비까지 프레임을 전달하는 역할을 한다.  

이 계층에서 데이터는 `Frame` 단위로 다뤄진다.

**[주요 역할]** 

- MAC 주소 기반 통신
- Frame 단위 캡슐화
- 같은 LAN 내부에서의 데이터 전달
- 오류 검출을 위한 FCS 사용
- 스위치를 통한 프레임 전달

:::tip
Data Link Layer는 같은 네트워크 구간 안에서 MAC 주소를 기반으로 Frame을 전달하는 계층이다.
:::

---

## 2. Ethernet

Ethernet은 가장 널리 사용되는 Data Link Layer 기술이다.  

일반적인 유선 LAN 환경에서는 대부분 Ethernet을 기반으로 통신이 이루어진다.

Ethernet에서는 데이터를 `Ethernet Frame` 형태로 만들어 전송한다.  
이 Frame 안에는 송/수신자 MAC 주소, 상위 프로토콜 정보 등이 포함된다.  

정리하자면 이더넷은 같은 네트워크 안에서 Frame을 어떤 형식으로 만들고 전달할지 정한 규칙이다.  

---

## 3. Ethernet Frame

Ethernet Frame은 L2에서 사용하는 데이터 전달 단위이다.

```text
[Destination MAC][Source MAC][EtherType][Payload][FCS]
```

| Field | Description |
| :--: | :-- |
| Destination MAC | Frame을 받을 장비의 MAC 주소 |
| Source MAC | Frame을 보낸 장비의 MAC 주소 |
| EtherType | 상위 계층 프로토콜 종류 |
| Payload | 실제 전달할 데이터 |
| FCS | Frame 오류 검출 값 |

<figure style="text-align: center;">
  <img src="/assets/img/Network/L2/ethernet.png" alt="Ethernet II Frame" style="display: block; margin: 0 auto;">
  <figcaption>Wireshark에서 확인한 Ethernet II Frame</figcaption>
</figure>

위 사진에서는 Ethernet II Frame 안에 아래와 같은 정보가 포함되어 있다.

```text
Source MAC      : 00:15:5d:00:06:06
Destination MAC : 00:15:5d:88:22:18
Type            : IPv4 (0x0800)
```

`Type` 값이 `IPv4 (0x0800)` 이므로 Ethernet Frame의 Payload에는 IPv4 Packet이 들어있다는 것을 알 수 있다.

:::tip
Wireshark에서 NIC가 처리한 이후의 데이터를 캡처하기 때문에 트레일러나 일부 정보는 보이지 않는 경우가 있다.
:::

---

## 4. MAC Address

MAC Address는 네트워크 인터페이스를 식별하기 위한 L2 주소이다.  

IP 주소가 논리적인 주소라면 MAC 주소는 같은 네트워크 안에서 실제 Frame 전달에 사용되는 주소이다.  

```text
Source MAC      : 00:15:5d:00:06:06
Destination MAC : 00:15:5d:88:22:18
```

스위치는 Frame의 Source MAC 주소를 보고 MAC Address Table을 학습한다.  
이후 Destination MAC 주소를 기준으로 어떤 포트로 Frame을 전달할지 결정한다.

---

## 5. ARP

`ARP(Address Resolution Protocol)`는 IP 주소를 MAC 주소로 변환하기 위한 프로토콜이다.

이전 글에서는 ARP를 L3와 관련된 프로토콜로 분류했지만 실제 역할은 IP 주소를 L2의 MAC 주소로 연결 해주는것이다.

L3에서는 목적지를 IP 주소로 표현하지만 같은 LAN 안에서 실제로 Frame을 보내기 위해서는 목적지 MAC 주소가 필요하다.

ARP 동작 흐름

1. 송신자가 목적지 IP에 해당하는 MAC 주소를 모른다.
2. ARP Request를 브로드캐스트로 전송한다.
3. 해당 IP를 가진 장비가 ARP Reply로 자신의 MAC 주소를 응답한다.
4. 송신자는 응답받은 MAC 주소를 Ethernet Frame의 Destination MAC으로 사용한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L2/arp.png" alt="ARP Request and Reply" style="display: block; margin: 0 auto;">
  <figcaption>ARP Request와 Reply 흐름</figcaption>
</figure>

위 캡처에서는 다음과 같은 ARP 통신을 확인할 수 있다.

```text
Who has 172.30.253.4? Tell 172.30.240.1
172.30.253.4 is at 00:15:5d:00:06:06
```

`172.30.253.4` IP를 가진 장비의 MAC 주소가 `00:15:5d:00:06:06` 임을 응답받은 것이다.

ARP는 같은 네트워크 안에서 IP 기반 통신을 실제 Ethernet Frame 전달로 연결해주는 역할을 한다.

---

## 6. VLAN

VLAN(Virtual LAN)은 하나의 물리적 네트워크를 여러 개의 논리적 네트워크로 분리하는 기술이다.

같은 스위치에 연결되어 있더라도 VLAN이 다르면 서로 다른 네트워크에 있는 것처럼 동작한다.
<figure style="text-align: center;">
  <img src="/assets/img/Network/L2/vlan.png" alt="vlan" style="display: block; margin: 0 auto;">
  <figcaption>VLAN 구성</figcaption>
</figure>

**[VLAN 사용 목적]**

- 브로드캐스트 도메인 분리
- 네트워크 관리 단순화
- 부서 또는 서비스 단위의 네트워크 분리
- 불필요한 트래픽 확산 방지

예를 들어 하나의 스위치에 개발팀, 관리팀, 서버망이 함께 연결되어 있어도 VLAN을 나누면 서로 다른 네트워크처럼 분리할 수 있다.

VLAN이 나뉜 네트워크끼리 통신하려면 L3 장비를 통한 라우팅이 필요하다.

---

## 7. STP

`STP(Spanning Tree Protocol)`는 스위치 네트워크에서 루프를 방지하기 위한 프로토콜이다.

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/L2/stpon.png" alt="stpon">
    <figcaption>STP ON</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/L2/stpoff.png" alt="stpoff">
    <figcaption>STP OFF</figcaption>
  </figure>
</div>

스위치를 여러 경로로 연결하면 장애 발생 시 다른 경로를 사용할 수 있지만 동시에 L2 루프가 발생할 수 있다.

STP는 스위치끼리 BPDU를 교환하여 루프 없는 경로를 계산하고 하나의 포트를 `Blocking` 상태로 전환해 루프를 방지한다.

실제로 확인해보니 STP가 활성화된 상태에서 **FA0/0 포트가 BLK** 상태가 되었으며 Event List에서 **ICMP & STP** 패킷이 함께 전송되는 것을 확인할 수 있었다.

반대로 STP를 비활성화하면 `Blocking` 포트가 없어져 L2 루프가 발생하고 프레임이 네트워크 내부에서 반복 전달되면서 ICMP 통신도 목적지 PC까지 정상적으로 완료되지 않았다.

**[STP 사용 목적]**

- 스위치 간 루프 방지
- 브로드캐스트 폭주 방지
- 이중화 경로 유지
- 장애 발생 시 대체 경로 사용

---