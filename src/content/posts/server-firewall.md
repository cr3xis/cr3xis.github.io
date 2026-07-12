---
title: '서버 방화벽 설정하기'
published: 2026-04-20
description: 'Oracle Cloud와 Ubuntu 환경에서 외부 접속을 위한 방화벽 설정을 정리합니다.'
image: '/assets/img/server/02/1.png'
tags:
  - 'oracle-cloud'
  - 'instance'
  - 'ubuntu'
  - 'vcn'
  - 'ssh'
  - 'reserved-public-ip'
category: 'Server'
draft: false
---


서버를 생성하고 나서 잊지 않기 위해 이 글을 작성한다.    
분명히 웹 서버도 실행했고 포트도 열었다고 생각했는데 외부에서는 접속이 되지 않는 상황이다.

이때 대부분 서버 설정을 의심하게 되지만 실제로는 방화벽에서 막히는 경우가 많다.  
대부분 클라우드는 네트워크 단에서 한 번 접근을 제어하고 그 안에 있는 VM 역시 자체 방화벽을 가지고 있다.  

결국 HTTP(80), HTTPS(443) 통신을 정상적으로 사용하려면 클라우드 네트워크와 VM 내부 두 영역을 모두 설정해야 한다.  

오라클 클라우드 네트워크 설정을 먼저 열어주고 그 다음 VM 내부에서 실제 포트를 허용해주면 된다.

---

## 오라클 클라우드 네트워크 방화벽 설정

외부 트래픽은 먼저 VCN을 통과한다.  
따라서 인스턴스에 도달하기 전에 VCN에서 해당 포트를 허용해줘야 한다.
<figure style="text-align: center;">
  <img src="/assets/img/server/02/1.png" alt="Virtual Cloud Network 화면" style="display: block; margin: 0 auto;">
  <figcaption>Virtual Cloud Network 화면</figcaption>
</figure>


Networking 메뉴에서 Virtual Cloud Networks로 이동한 뒤 현재 인스턴스가 연결된 VCN을 선택한다.

VCN 내부로 들어가면 보안 관련 항목이 보인다.

<figure style="text-align: center;">
  <img src="/assets/img/server/02/2.png" alt="Security 메뉴" style="display: block; margin: 0 auto;">
  <figcaption>Security 메뉴</figcaption>
</figure>

Security 항목으로 이동하면 Security Lists가 있다.  
이 리스트가 실제 인바운드와 아웃바운드 트래픽을 제어하는 역할을 한다.

<figure style="text-align: center;">
  <img src="/assets/img/server/02/3.png" alt="Security Lists" style="display: block; margin: 0 auto;">
  <figcaption>Security Lists</figcaption>
</figure>

현재 인스턴스가 연결된 Security List를 선택한 뒤 Security Rules 탭으로 들어간다.

<figure style="text-align: center;">
  <img src="/assets/img/server/02/4.png" alt="Ingress Rules 설정 화면" style="display: block; margin: 0 auto;">
  <figcaption>Ingress Rules 설정 화면</figcaption>
</figure>

Add Ingress Rules를 선택하고 다음과 같이 설정해준다.

- Source: 0.0.0.0/0  
- IP Protocol: TCP  
- Destination Port Range: 80  
- 동일한 방식으로 443 추가  

이 설정은 외부에서 들어오는 HTTP, HTTPS 요청을 VCN 단계에서 허용하는 과정이다.

여기까지 마치면 네트워크 레벨에서는 해당 포트가 열려 있다.  
이제 VM 내부로 들어가 실제 리눅스 방화벽을 설정해야 한다.

---

## VM(Ubuntu) 내부 방화벽 설정

오라클 네트워크에서 허용했더라도 VM 내부에서 막고 있다면 여전히 접속은 되지 않는다.

Ubuntu에서는 보통 iptables가 기본 방화벽 역할을 한다.  
현재 INPUT 체인 상태를 먼저 확인한다.

```bash
sudo iptables -L INPUT --line-numbers -n -v
```

- L : 현재 규칙 출력
- n : IP주소와 포트를 숫자로 출력
- v : 패킷과 바이트 처리 출력

현재 규칙과 기본 정책을 확인하는 과정이다.  
보안을 위해 허용되지 않은 포트는 기본 정책을 DROP으로 두는 경우가 많다.  

```bash
sudo iptables -P INPUT DROP
```

- P : Policy 설정 

이 설정은 명시적으로 허용한 포트만 통과시키겠다는 의미다.  

```bash
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

이제 필요한 포트를 추가해주면 된다.  

- A : 체인에 규칙 추가
- p : 어떤 프로토콜인지
- dport : 어떤 포트
- j ACCEPT : 조건이 맞다면 허용


```bash
sudo iptables -L INPUT --line-numbers -n -v
```

그리고 실제 서비스가 리슨 중인지 확인한다.

```bash
ss -tln
```

여기서 80이나 443이 LISTEN 상태로 보이지 않는다면 웹 서버가 실행 중인지도 함께 점검해야 한다.

---

## 잘못된 규칙 삭제

규칙을 잘못 추가했다면 번호를 확인한 뒤 삭제할 수 있다.

```bash
sudo iptables -D INPUT <번호>
```

---

## 재부팅 이후 유지

iptables 설정은 재부팅 시 사라지기 때문에 현재 상태를 저장해두어야 한다.

```bash
sudo netfilter-persistent save
```

이 명령을 실행하면 현재 규칙이 `/etc/iptables/rules.v4` 파일에 저장된다.

---

## 전체 흐름

외부 사용자가 접속할 때의 흐름은 다음과 같다.

클라이언트 → 오라클 VCN Security Rule → VM iptables → 웹 서버 → 애플리케이션

이 중 하나라도 막혀 있으면 접속은 실패한다.  
그래서 방화벽 설정은 항상 네트워크와 서버 내부를 함께 확인해야 한다.
