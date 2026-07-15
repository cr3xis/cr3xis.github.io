---
title: L4 - Transport Layer
published: 2026-07-15T00:00:00
description: 'OSI 4계층의 역할과 핵심 개념을 정리합니다.'
image: ''
tags: [Network, L4, TCP, UDP, Port, Socket, Handshake]
category: 'Network'
draft: false
---

## 1. Transport Layer

Transport Layer는 OSI 7계층에서 4계층에 해당한다.

L4는 `Port 번호`를 기반으로 호스트 내부의 어떤 프로세스와 통신할지 구분한다.

이 계층에서 데이터는 TCP 기준으로 `Segment`, UDP 기준으로 `Datagram` 단위로 다뤄진다.

**[주요 역할]**

- 프로세스 간 통신 구분
- Port 번호 기반 데이터 전달
- TCP를 통한 신뢰성 있는 전송
- UDP를 통한 빠른 비연결형 전송
- 흐름 제어와 혼잡 제어

---

## 2. Port

Port는 하나의 호스트 안에서 어떤 프로세스가 데이터를 받을지 구분하기 위한 번호이다.

IP 주소가 목적지 호스트를 찾기 위한 값이라면 Port 번호는 그 호스트 안의 애플리케이션을 찾기 위한 값이다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L4/port.png" alt="port" style="display: block; margin: 0 auto;">
  <figcaption>Port</figcaption>
</figure>

출발지인 `49754`는 클라이언트가 임의로 생성한 포트이며 운영체제가 자동으로 할당한다.

목적지인 `443`은 서버가 HTTPS 서비스를 제공하는 고정 포트이다.

**[Port 범위]**

| Range | Name | Description |
| :--: | :--: | :-- |
| 0 ~ 1023 | Well-Known Port | 주요 서비스에서 사용 |
| 1024 ~ 49151 | Registered Port | 특정 애플리케이션에서 등록해 사용 |
| 49152 ~ 65535 | Dynamic Port | 클라이언트 임시 포트로 주로 사용 |

---

## 3. Socket

Socket은 운영체제가 제공하는 네트워크 통신의 끝점(Endpoint)이다.

프로그램은 소켓을 생성한 뒤 IP 주소, Port 번호, Protocol(TCP/UDP) 정보를 이용하여 다른 호스트와 통신한다.

```text
TCP 192.168.10.5:50000  →  93.184.216.34:80
```

위 예시에서 `192.168.10.5:50000`은 클라이언트 소켓, `93.184.216.34:80`은 서버 소켓을 의미한다.

같은 IP 주소를 사용하는 호스트라도 Port 번호가 다르면 운영체제는 서로 다른 프로세스로 데이터를 전달할 수 있다.

---

## 4. TCP

`TCP(Transmission Control Protocol)`는 연결 지향형 전송 프로토콜이다.

데이터를 보내기 전에 연결을 수립하고, 전송 중에는 순서 보장과 재전송을 통해 신뢰성을 제공한다.

**[특징]**

- 연결 지향
- 신뢰성 있는 데이터 전송
- 데이터 순서 보장
- 손실 발생 시 재전송
- 흐름 제어와 혼잡 제어 지원

TCP는 속도보다 데이터가 정확히 도착하는 것이 중요한 통신에 적합하다.

---

## 5. TCP Header

TCP Header에는 신뢰성 있는 전송을 위해 필요한 정보가 포함된다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L4/tcpheader.png" alt="tcpheader" style="display: block; margin: 0 auto;">
  <figcaption>Tcp Header</figcaption>
</figure>

| Field | Description |
| :--: | :-- |
| Source Port | 송신 프로세스의 Port 번호 |
| Destination Port | 수신 프로세스의 Port 번호 |
| Sequence Number | 데이터 순서 확인에 사용 |
| Acknowledgment Number | 다음에 받아야 할 데이터 번호 |
| Header Length | TCP Header의 길이 |
| Flags | 연결 제어 상태 표시 |
| Window Size | 수신 측이 받을 수 있는 데이터 크기 |
| Checksum | 오류 검출 |

---

## 6. TCP Flags

TCP Flags는 TCP 연결 상태와 제어 정보를 나타내는 값이다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L4/tcpflags.png" alt="tcpflags" style="display: block; margin: 0 auto;">
  <figcaption>Tcp flags</figcaption>
</figure>

| Flag | Description |
| :--: | :-- |
| SYN | 연결 요청 |
| ACK | 응답 확인 |
| FIN | 연결 종료 요청 |
| RST | 연결 강제 종료 |
| PSH | 데이터를 즉시 상위 계층으로 전달 |
| URG | 긴급 데이터 표시 |

TCP 3-Way Handshake와 4-Way Handshake는 이 Flags를 이용해 연결을 수립하고 종료한다.

---

## 7. 3-Way Handshake

3-Way Handshake는 TCP 연결을 수립하는 과정이다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L4/3wayf.png" alt="3way" style="display: block; margin: 0 auto;">
  <figcaption>https://wiki.wireshark.org/</figcaption>
</figure>


<figure style="text-align: center;">
  <img src="/assets/img/Network/L4/3way.png" alt="3way" style="display: block; margin: 0 auto;">
  <figcaption>3-way</figcaption>
</figure>

```text
Client -> Server : SYN
Server -> Client : SYN + ACK
Client -> Server : ACK
```

1. Client가 Server에게 `SYN`을 보내 연결을 요청한다.
2. Server는 `SYN + ACK`로 요청을 수락한다.
3. Client가 `ACK`를 보내면 연결이 성립된다.

이 과정을 통해 양쪽은 서로 통신 가능한 상태인지 확인하고 초기 Sequence Number를 교환한다.

---

## 8. 4-Way Handshake

4-Way Handshake는 TCP 연결을 종료하는 과정이다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L4/4wayf.png" alt="4way" style="display: block; margin: 0 auto;">
  <figcaption>https://wiki.wireshark.org/</figcaption>
</figure>

<figure style="text-align: center;">
  <img src="/assets/img/Network/L4/4way.png" alt="4way" style="display: block; margin: 0 auto;">
  <figcaption>4-way</figcaption>
</figure>

> 현재 이미지의 경우 서버가 즉시 종료 가능해 ACK와 FIN을 하나의 패킷으로 합쳐 보냈기에 3개로 보인다.

```text
Client -> Server : FIN
Server -> Client : ACK
Server -> Client : FIN
Client -> Server : ACK
```

1. 한쪽에서 `FIN`을 보내 연결 종료를 요청한다.
2. 상대는 `ACK`로 종료 요청을 확인한다.
3. 상대도 보낼 데이터가 끝나면 `FIN`을 보낸다.
4. 마지막으로 `ACK`를 보내 연결을 종료한다.

TCP는 양방향 통신이므로 각 방향의 종료를 따로 확인한다.

---

## 9. UDP

`UDP(User Datagram Protocol)`는 비연결형 전송 프로토콜이다.

TCP와 달리 연결을 수립하지 않고 데이터를 바로 전송한다.

**[특징]**

- 비연결형
- Header 구조가 단순함
- 전송 속도가 빠름
- 순서 보장 없음
- 손실 발생 시 자체 재전송 없음

UDP는 실시간성이 중요한 통신에 자주 사용된다.

---

## 10. UDP Header

UDP Header는 TCP Header보다 단순하다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L4/udp.png" alt="udp" style="display: block; margin: 0 auto;">
  <figcaption>UDP Header</figcaption>
</figure>

| Field | Description |
| :--: | :-- |
| Source Port | 송신 프로세스의 Port 번호 |
| Destination Port | 수신 프로세스의 Port 번호 |
| Length | UDP Header와 Data의 전체 길이 |
| Checksum | 오류 검출 |

UDP는 연결 관리, 순서 제어, 재전송 기능이 없기 때문에 Header가 짧고 처리 오버헤드가 낮다.

---

## 11. TCP vs UDP

TCP와 UDP는 모두 L4 프로토콜이지만 목적이 다르다.

| 구분 | TCP | UDP |
| :--: | :--: | :--: |
| 연결 방식 | 연결형 | 비연결형 |
| 신뢰성 | 높음 | 낮음 |
| 순서 보장 | 보장 | 보장하지 않음 |
| 재전송 | 지원 | 지원하지 않음 |
| 속도 | 느림 |  빠름 |
| 사용 예시 | HTTP, HTTPS, SSH, FTP | DNS, DHCP, VoIP, Streaming |

TCP는 정확성이 중요한 통신에 적합하고 UDP는 속도와 실시간성이 중요한 통신에 적합하다.

---

## 12. 대표 Port 번호

자주 사용되는 서비스는 Well-Known Port를 사용한다.

| Port | Protocol | Service |
| :--: | :--: | :-- |
| 20, 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 67, 68 | UDP | DHCP |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 143 | TCP | IMAP |
| 443 | TCP | HTTPS |
| 3389 | TCP | RDP |

Port 번호는 통신 대상 서비스나 프로세스를 구분하기 위한 값이다.

---