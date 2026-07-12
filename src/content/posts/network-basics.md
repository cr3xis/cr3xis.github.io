---
title: Network Basic
published: 2026-07-02
description: 'Netowrk, LAN/WAN, Topology, Packet Switching의 기본 개념을 정리합니다.'
tags: ["Network", "LAN", "WAN", "Topology"]
category: "Network"
draft: false
---


## 1. Network란?

네트워크(Network)는 여러 장치(Device)가 서로 데이터를 주고받기 위해 연결된 구조이다.

장치 간 데이터를 안정적으로 전달하기 위해 다양한 규칙(Protocol)과 장비를 사용한다.

---

## 2. LAN과 WAN

### LAN

LAN(Local Area Network)은 제한된 공간 안에서 구성되는 네트워크이다.
<figure style="text-align: center;">
  <img src="/assets/img/Network/Basic/LAN.png" alt="LAN" style="display: block; margin: 0 auto;">
  <figcaption>LAN</figcaption>
</figure>

- 집 내부 네트워크
- 회사 내부망
- 학교 네트워크

### WAN

WAN(Wide Area Network)은 여러 지역에 있는 네트워크를 연결하는 구조이다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/Basic/WAN.png" alt="WAN" style="display: block; margin: 0 auto;">
  <figcaption>WAN</figcaption>
</figure>

대표적으로 인터넷이 WAN에 해당한다.

---

## 3. Topology

Topology는 네트워크 장치들이 어떤 형태로 연결되어 있는지를 의미한다.

**주요 형태**

- Bus : 하나의 케이블을 여러 장치가 공유한다.
<figure style="text-align: center;">
  <img src="/assets/img/Network/Basic/Bus.png" alt="LAN" style="display: block; margin: 0 auto;">
  <figcaption>LAN</figcaption>
</figure>

- Star : 중앙 장비(Switch)를 중심으로 연결한다.
<figure style="text-align: center;">
  <img src="/assets/img/Network/Basic/Star.png" alt="Star" style="display: block; margin: 0 auto;">
  <figcaption>Star</figcaption>
</figure>

- Ring : 원형으로 연결되어 순차적으로 데이터를 전달한다.
<figure style="text-align: center;">
  <img src="/assets/img/Network/Basic/Ring.png" alt="Ring" style="display: block; margin: 0 auto;">
  <figcaption>Ring</figcaption>
</figure>

- Mesh : 모든 장치가 서로 연결되어 높은 안정성을 제공한다.
<figure style="text-align: center;">
  <img src="/assets/img/Network/Basic/Mesh.png" alt="Mesh" style="display: block; margin: 0 auto;">
  <figcaption>Mesh</figcaption>
</figure>

---


## 4. Packet Switching

### 데이터그램 패킷 교환 방식

<figure style="text-align: center;">
  <img src="/assets/img/Network/Basic/Datagram.png" alt="Datagram" style="display: block; margin: 0 auto;">
  <figcaption>Datagram</figcaption>
</figure>

- 데이터를 전송하기 전에 논리적 연결이 설정되어 있지 않음 ( 비연결형 )
- 송신 측에서 전송한 순서와 수신 측에서 도착한 순서가 다를수 있음
- 패킷이 독립적으로 전송됨

### 가상회선 패킷 교환 방식

<figure style="text-align: center;">
  <img src="/assets/img/Network/Basic/Circuit.png" alt="Circuit" style="display: block; margin: 0 auto;">
  <figcaption>Circuit</figcaption>
</figure>

- 데이터를 전송하기 전에 논리적 연결이 설정됨 (연결형)
- 각 패킷에는 식별 번호가 포함되어 있으며 모든 패킷을 전송하면 가상회선이 해제됨
- 패킷들은 전송된 순서대로 도착함