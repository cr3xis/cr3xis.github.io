---
title: L7 - Application Layer
published: 2026-07-16T00:00:00
description: 'OSI 7계층의 역할과 핵심 개념을 정리합니다.'
image: '/assets/img/Network/L7/clienttoserver.png'
tags: [ Network,L7,HTTP,HTTPS,DNS,DHCP,FTP,SSH,MAIL ]
category: 'Network'
draft: false
---

## 1. Application Layer란?

Application Layer는 OSI 7계층에서 7계층에 해당한다.

사용자와 가장 가까운 계층이며 애플리케이션이 네트워크 기능을 사용할 수 있도록 통신 규칙을 제공한다.

L7은 서비스별 데이터 형식과 요청 방식 그리고 응답 방식을 정의한다.

**[주요 역할]**

- 웹 페이지 요청과 응답
- 도메인 이름 해석
- IP 주소 자동 할당
- 파일 전송
- 원격 접속
- 이메일 송수신

---

## 2. Client - Server 구조

Client는 서비스를 요청하는 쪽이다.

Server는 요청을 처리하고 결과를 응답하는 쪽이다.

웹에서는 브라우저가 Client 역할을 하고 웹 서버가 Server 역할을 한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L7/clienttoserver.png" alt="clienttoserver" style="display: block; margin: 0 auto;">
  <figcaption>Client - Server</figcaption>
</figure>

Client - Server 구조는 L7 프로토콜을 이해할 때 가장 기본이 되는 흐름이다.

---

## 3. 대표 프로토콜

L7 프로토콜은 사용자가 실제로 사용하는 서비스와 직접 연결된다.

| Protocol | Purpose | Common Port |
| --- | --- | --- |
| HTTP | 웹 문서 요청과 응답 | TCP 80 |
| HTTPS | TLS로 보호되는 웹 통신 | TCP 443 |
| DNS | 도메인 이름을 IP 주소로 변환 | UDP/TCP 53 |
| DHCP | IP 주소와 네트워크 설정 자동 할당 | UDP 67/68 |
| FTP | 파일 전송 | TCP 20/21 |
| SSH | 보안 원격 접속 | TCP 22 |
| SMTP | 메일 전송 | TCP 25/587/465 |
| POP3 | 메일 수신 | TCP 110/995 |
| IMAP | 메일 동기화 | TCP 143/993 |

---

## 4. HTTP

`HTTP(HyperText Transfer Protocol)`는 웹에서 리소스를 요청하고 응답받기 위한 프로토콜이다.

브라우저는 HTTP Request를 보내고 서버는 HTTP Response를 돌려준다.

HTTP 자체는 상태를 저장하지 않는다.  
그래서 로그인 상태 같은 정보는 Cookie나 Session 같은 별도 방식으로 유지한다.


<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/L7/httprequest.png" alt="request">
    <figcaption>HTTP Request</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/L7/httpresponse.png" alt="response">
    <figcaption>HTTP Response</figcaption>
  </figure>
</div>


- **Method** : 클라이언트가 서버에 요청하는 방식 (GET, POST, PUT, DELETE ...)
- **URI** : 요청 대상 리소스의 위치
- **HTTP Version** : 사용 중인 HTTP 프로토콜 버전 (HTTP/1.1, HTTP/2 ...)
- **Status Code & Message** : 서버의 요청 처리 결과 (200 OK, 404 Not Found, 500 Internal Server Error ...)
- **Header** : 요청 및 응답에 대한 부가 정보
- **Body** : 실제 전송되는 데이터

---

## 5. HTTPS

`HTTPS(Hypertext Transfer Protocol Secure)`는 HTTP를 `TLS(Transport Layer Security)`로 보호하는 방식이다.

HTTP의 요청과 응답 구조는 그대로 유지하면서 전송 구간을 암호화하여 데이터를 안전하게 보호한다.

HTTPS를 사용하면 중간에서 패킷을 캡처하더라도 HTTP Request와 Response 내용을 직접 확인할 수 없으며  
Wireshark에서는 `TLS Application Data` 형태로 표시된다.

또한 서버 인증서를 검증하여 접속한 서버가 신뢰할 수 있는 대상인지 확인한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L7/https.png" alt="https" style="display: block; margin: 0 auto;">
  <figcaption>HTTPS (Wireshark)</figcaption>
</figure>

- **14 ~ 18** : TCP 3-Way Handshake
- **19, 37** : TLS Handshake (보안 연결 설정)
- **39 ~** : HTTPS 통신 시작 (HTTP Request/Response가 TLS로 암호화되어 `Application Data`로 전송)

---

## 6. DNS

`DNS(Domain Name System)`는 사람이 읽기 쉬운 도메인 이름(예: google.com)을  
컴퓨터가 통신에 사용하는 IP 주소(예: 142.250.198.163)로 변환해 주는 시스템이다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L7/dns.png" alt="dns" style="display: block; margin: 0 auto;">
  <figcaption>DNS (Wireshark)</figcaption>
</figure>

```text
C:\Windows\System32>nslookup google.co.kr
Server:  dns.google
Address: 8.8.8.8

Non-authoritative answer:
Name:    google.co.kr
Addresses:  2404:6800:4005:826::2003
          142.250.198.163
```

| Record | Meaning |
| --- | --- |
| A | 도메인 이름을 IPv4 주소에 매핑 |
| AAAA | 도메인 이름을 IPv6 주소에 매핑 |
| CNAME | 다른 도메인 이름을 가리키는 별칭 |
| MX | 메일을 수신하는 메일 서버를 지정(SMTP 전달 대상) |
| NS | 해당 도메인의 DNS 정보를 관리하는 네임 서버를 지정 |

---

## 7. DHCP

`DHCP(Dynamic Host Configuration Protocol)`는 네트워크에 접속한 장비에게 IP 주소와 네트워크 설정을 자동으로 할당하는 프로토콜이다.

사용자는 직접 IP를 설정하지 않아도 DHCP 서버를 통해 필요한 정보를 자동으로 받아 네트워크를 사용할 수 있다.

DHCP를 통해 일반적으로 다음 정보를 할당받는다.

- IP Address / Subnet Mask / Default Gateway / DNS Server

```text
Discover → Offer → Request → ACK
```

<figure style="text-align: center;">
  <img src="/assets/img/Network/L7/dhcp.png" alt="dhcp" style="display: block; margin: 0 auto;">
  <figcaption>DHCP (Wireshark)</figcaption>
</figure>


위 과정을 **DORA**라고 부른다.

- **Discover** : 클라이언트가 DHCP 서버를 찾으며 IP 주소를 요청한다.
- **Offer** : DHCP 서버가 사용 가능한 IP 주소와 네트워크 설정(IP, Subnet Mask, Gateway, DNS 등)을 제안한다.
- **Request** : 클라이언트가 제안받은 IP 주소를 사용하겠다고 요청한다.
- **ACK** : DHCP 서버가 요청을 승인하고 IP 주소와 네트워크 설정을 최종 할당한다.

DHCP가 완료되면 클라이언트는 할당받은 IP 주소와 네트워크 설정을 이용하여 같은 네트워크 및 외부 인터넷과 통신할 수 있다.

---

## 8. FTP

`FTP(File Transfer Protocol)`는 Client와 Server 사이에서 파일을 업로드하거나 다운로드할 때 사용하는 프로토콜이다.

FTP는 명령을 전달하는 **Control Connection**과 실제 파일을 전송하는 **Data Connection**을 별도로 사용한다.

> https://medium.com/@ananthakrish16ks/ftp-control-and-data-connections-f74ceb9eae95

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/L7/ftp.png" alt="FTP Control Connection">
    <figcaption>FTP Control Connection</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/Network/L7/ftp2.png" alt="FTP Data Connection">
    <figcaption>FTP Data Connection</figcaption>
  </figure>
</div>

| Connection | Role |
| --- | --- |
| Control | 로그인, 파일 목록 조회(LIST), 다운로드(RETR) 등 명령 전달 |
| Data | 실제 파일 및 디렉터리 목록 전송 |


기본 FTP는 암호화를 제공하지 않기 떄문에 계정 정보와 파일 내용이 평문으로 전송될 수 있어 현재는 **SFTP** 또는 **FTPS**를 사용하는 경우가 많다.

---

## 9. SSH

SSH(Secure Shell)는 원격 서버에 안전하게 접속하여 명령을 실행하기 위한 프로토콜이다.

기본 포트는 TCP 22번을 사용한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L7/ssh.png" alt="ssh" style="display: block; margin: 0 auto;">
  <figcaption>SSH Remote Login</figcaption>
</figure>

[주요 기능]

- 원격 로그인
- 원격 명령 실행
- 암호화된 터널링
- 키 기반 인증

SSH는 모든 통신을 암호화하여 평문 원격 접속(Telnet)의 보안 문제를 해결한다.

---

## 10. SMTP / POP3 / IMAP

이메일은 하나의 프로토콜만으로 동작하지 않는다.

메일을 보내는 역할과 메일을 받는 역할이 서로 분리되어 있다.

| Protocol | Role |
| --- | --- |
| SMTP | 메일을 보내거나 다른 메일 서버로 전달 |
| POP3 | 서버의 메일을 클라이언트로 내려받음 |
| IMAP | 서버에 메일을 유지한 채 여러 장치와 동기화 |

SMTP는 메일을 발신하거나 메일 서버 간 메일을 전달할 때 사용된다.

POP3는 메일을 클라이언트로 다운로드하는 방식이며 일반적으로 서버의 메일을 내려받아 사용하는 구조이다.

IMAP은 메일을 서버에 그대로 유지하므로 PC, 스마트폰 등 여러 장치에서 동일한 메일함을 동기화하여 사용할 수 있다.

---

## 11. 실제 웹 접속 과정

사용자가 브라우저에 URL을 입력하면 여러 프로토콜이 순차적으로 동작하여 웹 페이지를 화면에 표시한다.

<figure style="text-align: center;">
  <img src="/assets/img/Network/L7/naver.png" alt="naver" style="display: block; margin: 0 auto;">
  <figcaption>Web Access Flow (Wireshark)</figcaption>
</figure>

Wireshark 캡처에서는 다음과 같은 순서로 웹 접속이 이루어진다.

- **DNS Query / Response** : 도메인 이름을 IP 주소로 변환한다.
- **TCP 3-Way Handshake** : 서버와 TCP 연결을 생성한다.
- **TLS Handshake** : 서버 인증서를 검증하고 암호화 통신을 위한 세션 키를 교환한다.
- **Application Data** : HTTP Request와 HTTP Response가 TLS로 암호화되어 전송된다.


위 사진 흐름을 순서대로 풀어서 설명하면 아래처럼 표현할수 있다.  

1. 사용자가 브라우저에 URL을 입력한다.
2. DNS를 통해 도메인 이름을 IP 주소로 변환한다.
3. 서버와 TCP 3-Way Handshake를 수행하여 연결을 생성한다.
4. HTTPS라면 TLS Handshake를 수행하여 서버 인증서를 검증하고 암호화 통신을 준비한다.
5. 브라우저가 HTTP Request를 전송한다.
6. 서버가 HTTP Response를 반환한다.
7. 브라우저가 HTML, CSS, JavaScript를 해석하여 화면을 렌더링한다.


이처럼 하나의 웹 페이지를 열기 위해서는 DNS, TCP, TLS, HTTP 등 여러 프로토콜이 순차적으로 동작한다.

---