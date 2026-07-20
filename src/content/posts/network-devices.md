---
title: Network Devices
published: 2026-07-17T00:00:00
description: '네트워크 장비의 역할과 실제 웹 통신 흐름을 정리합니다.'
image: '/assets/img/Network/Devices/hub.png'
tags: [ Network,Device,NIC,Switch,Router,Firewall,IDS,IPS,Proxy,Load Balancer,WAF ]
category: 'Network'
draft: false
---

## 1. Network Device란?

`Network Device`는 네트워크에서 트래픽을 전달하거나 제어하거나 보호하는 장비를 의미한다.

`L2` 장비는 MAC 주소를 기준으로 Frame을 전달하고 `L3` 장비는 IP 주소를 기준으로 Packet을 라우팅 한다. 

`L4` 장비는 Port와 Protocol을 기준으로 트래픽을 제어하고 `L7` 장비는 HTTP Request 같은 애플리케이션 데이터를 기준으로 동작한다.

| Layer | 기준 정보 | 대표 장비 |
| --- | --- | --- |
| L1 | 전기 신호 | Hub |
| L2 | MAC Address | L2 Switch |
| L3 | IP Address | Router / L3 Switch |
| L4 | IP / Port / Protocol | Firewall, Load Balancer |
| L7 | HTTP Header, URI, Body | Proxy / WAF / L7 Load Balancer |


---

## 2. NIC

`NIC(Network Interface Card)`는 PC나 서버가 네트워크에 연결되기 위한 인터페이스다.

물리적인 랜카드일 수도 있고 가상 머신이나 VPN에서 생성되는 가상 인터페이스일 수도 있다.

NIC에는 일반적으로 MAC 주소가 존재하며 OS는 NIC를 통해 Frame을 송수신한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/Devices/nic.png" alt="NIC" style="display: block; margin: 0 auto;">
  <figcaption>NIC</figcaption>
</figure>

---

## 3. Hub

`Hub`는 L1 장비이며 들어온 전기 신호를 해석하지 않고 연결된 모든 포트로 그대로 전달한다.

MAC 주소나 IP 주소를 보고 판단하지 않기 때문에 목적지와 상관없이 모든 장비가 같은 신호를 받게 된다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/Devices/hub.png" alt="hub" style="display: block; margin: 0 auto;">
  <figcaption>HUB</figcaption>
</figure>

---

## 4. L2 Switch

`L2 Switch`는 MAC 주소를 기준으로 Frame을 전달하는 장비이다. 

Switch는 Frame의 출발지 MAC 주소를 학습해서 MAC 주소 테이블을 만들고 목적지 MAC 주소를 기준으로 어느 포트로 보낼지 결정한다.

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/Devices/l2table.png" alt="table">
    <figcaption>MAC Address Table</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/Devices/l2switch.png" alt="switch">
    <figcaption>L2 Switch</figcaption>
  </figure>
</div>


| 단계 | 동작 |
| :--: | :-- |
| 1  | Frame을 수신한다. |
| 2  | 출발지 MAC 주소를 MAC Address Table에 학습한다. |
| 3  | 목적지 MAC 주소를 조회한다. |
| 4  | MAC 주소가 있으면 해당 포트로만 전달한다. |
| 5  | MAC 주소가 없으면 Flooding을 수행하고 이후 학습한다. |

---

## 5. L3 Switch

`L3 Switch`는 기존 `L2 Switch`에서 Routing 기능을 추가한 장비이다.

`L2 Switch`가 MAC 주소를 기준으로 Frame을 전달한다면 `L3 Switch`는 각 VLAN의 Gateway(SVI)를 통해 서로 다른 VLAN이나 서브넷 사이를 라우팅 할수 있다.


<figure style="text-align: center;">
  <img src="/assets/img/Network/Devices/l3switch.png" alt="switch" style="display: block; margin: 0 auto;">
  <figcaption>L3 Switch</figcaption>
</figure>

| 단계 | 동작 |
| :--: | :-- |
| 1 | Packet을 수신하고 목적지 IP 주소를 확인한다. |
| 2 | Routing Table에서 목적지 네트워크를 조회한다. |
| 3 | 목적지 VLAN을 결정한다. |
| 4 | 목적지 MAC Address를 확인한다. |
| 5 | 새로운 Ethernet Frame을 생성하여 전달한다. |

---

## 6. Router

`Router`는 서로 다른 네트워크를 연결하는 L3 장비이다.

목적지 IP 주소를 기준으로 Routing Table을 조회하여 Packet을 다음 네트워크로 전달한다.

일반적으로 내부 네트워크의 Default Gateway 역할을 하며 Internet과 같은 외부 네트워크를 연결하는 데 사용된다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/Devices/router.png" alt="router" style="display: block; margin: 0 auto;">
  <figcaption>Router</figcaption>
</figure>

| 단계 | 동작 |
| :--: | :-- |
| 1 | Packet을 수신하고 목적지 IP Address를 확인한다. |
| 2 | Routing Table에서 다음 경로를 조회한다. |
| 3 | 목적지 MAC Address를 확인한다. |
| 4 | 새로운 Ethernet Frame을 생성하여 다음 장비로 전달한다. |


### L2 & L3 장비 차이점

| 장비        | 주요 특징                          |
| --------- | ------------------------------ |
| L2 Switch | MAC Address 기반 Switching       |
| L3 Switch | LAN 환경에서 Routing 수행        |
| Router    | WAN을 포함한 네트워크 간 Routing 수행 |


---

## 7. Firewall

Firewall은 네트워크를 통과하는 트래픽을 허용하거나 차단하는 보안 장비이다.

사전에 정의된 Rule을 기준으로 여러가지 정보를 검사하여 허용 또는 차단을 결정한다.

| Rule 기준 | 예시 |
| --- | --- |
| Source IP | 192.168.0.10 |
| Destination IP | 10.0.0.20 |
| Port | 80, 443, 22 |
| Protocol | TCP, UDP, ICMP |
| Action | Allow, Deny |

Firewall은 일반적으로 네트워크 경계에서 내부와 외부 네트워크 사이의 접근을 제어하기 위해 사용된다.

WAF와 달리 Firewall은 HTTP Body나 SQL Injection, XSS와 같은 웹 공격 패턴을 분석하는 장비는 아니다.

---

## 8. IDS

`IDS(Intrusion Detection System)`는 침입 탐지 시스템이다.

네트워크를 통과하는 트래픽이나 시스템 이벤트를 모니터링하여 의심스러운 행위를 탐지하고 관리자에게 알려준다.

IDS는 일반적으로 SPAN/TAP을 통해 전달받은 트래픽의 복사본을 분석한다.

따라서 실제 통신에는 직접 관여하지 않으며 공격을 탐지하더라도 트래픽을 차단하지는 않는다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/Devices/ids.png" alt="ids" style="display: block; margin: 0 auto;">
  <figcaption>IDS Diagram</figcaption>
</figure>

---

## 9. IPS

`IPS(Intrusion Prevention System)`는 침입 방지 시스템이다.

IPS는 IDS처럼 의심스러운 트래픽을 탐지하지만 탐지 이후 공격으로 판단되면 트래픽을 차단할 수 있다.

IPS는 일반적으로 네트워크 통신 경로 중간에 Inline 형태로 배치되어 모든 트래픽을 검사한다.

정상적인 트래픽은 그대로 전달하고 악성 트래픽은 Drop, Block, Reset 등의 방식으로 서버에 도달하기 전에 차단한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/Devices/ips.png" alt="ips" style="display: block; margin: 0 auto;">
  <figcaption>IPS Diagram</figcaption>
</figure>

---

### IDS/IPS 차이점

| 구분 | IDS | IPS |
| --- | --- | --- |
| 목적 | 공격 탐지 | 공격 탐지 및 차단 |
| 배치 | SPAN/TAP(Mirroring) | Inline |
| 통신 경로 | 통과하지 않음 | 모든 트래픽이 반드시 통과 |
| 동작 | Alert 중심 | Drop, Block, Reset 가능 |

---

## 10. Proxy

`Proxy`는 Client와 Server 사이에서 요청과 응답을 대신 전달하는 중계 서버이다.

Proxy는 요청을 그대로 전달할 수도 있고 헤더를 수정하거나 캐시된 응답을 반환하는 등 다양한 기능을 수행할 수 있다.

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/Devices/forward.png" alt="forward">
    <figcaption>Forward Proxy</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/Devices/reverse.png" alt="reverse">
    <figcaption>Reverse Proxy</figcaption>
  </figure>
</div>

| Type | 위치 | 역할 |
| --- | --- | --- |
| Forward Proxy | Client 앞 | Client를 대신하여 서버에 요청을 전달 |
| Reverse Proxy | Server 앞 | Server를 대신하여 Client의 요청을 받아 내부 서버로 전달 |


`Forward Proxy`는 Client를 대신하여 외부 서버와 통신하며 익명성 제공이나 인터넷 접근 제어를 위해 사용된다.

`Reverse Proxy`는 Server를 대신하여 Client의 요청을 처리하며 서버 보호, 부하 분산, Cache, TLS 종료 등의 목적으로 사용된다.

---

## 11. Load Balancer

`Load Balancer`는 들어오는 트래픽을 여러 서버로 분산하는 장비다.

한 서버에 모든 요청이 몰리지 않도록 분산하고 장애가 발생한 서버를 제외하여 서비스의 가용성과 성능을 향상시킨다.

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/Devices/before.png" alt="before">
    <figcaption>Before</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/Devices/after.png" alt="after">
    <figcaption>After</figcaption>
  </figure>
</div>

| Type | 기준 정보 | 예시 |
| --- | --- | --- |
| L4 Load Balancer | IP / Port / Protocol | TCP 443 트래픽 분산 |
| L7 Load Balancer | HTTP Header, URI, Cookie | `/api`는 API 서버, `/images`는 이미지 서버로 전달 |

---

## 12. WAF

`WAF(Web Application Firewall)`는 웹 애플리케이션을 보호하기 위한 L7 보안 장비다.

HTTP 또는 HTTPS 요청의 내용을 분석하여 웹 공격으로 판단되는 요청을 차단한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/Devices/waf.png" alt="waf" style="display: block; margin: 0 auto;">
  <figcaption>WAF</figcaption>
</figure>

| 검사 대상 | 탐지 가능한 공격 |
| --- | --- |
| URI | Path Traversal / 비정상 URL 요청 |
| Query String | SQL Injection / XSS |
| HTTP Header | Header 변조 / 비정상 요청 |
| Cookie | Cookie 변조 |
| Request Body | SQL Injection / XSS / 악성 파일 업로드 |

---

## 참고 자료

- https://www.paloaltonetworks.co.kr/cyberpedia/what-is-an-intrusion-detection-system-ids
- https://www.cloudflare.com/ko-kr/learning/cdn/glossary/reverse-proxy/
- https://www.cloudflare.com/ko-kr/learning/performance/what-is-load-balancing/
- https://www.cloudflare.com/ko-kr/learning/ddos/glossary/web-application-firewall-waf/