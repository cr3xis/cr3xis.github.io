---
title: '도메인 연결과 HTTPS 구성 흐름 정리'
published: 2026-05-01
description: '도메인 연결부터 Cloudflare, Nginx, HTTPS 인증서 구성까지의 흐름을 정리합니다.'
image: '/assets/img/server/03/01.png'
tags:
  - 'cloudflare'
  - 'ssl'
  - 'tls'
  - 'certbot'
  - 'letsencrypt'
  - 'nginx'
category: 'Server'
draft: false
---


서버를 외부에 공개하는 것은 단순히 포트를 여는 작업과는 다르다.  
도메인을 연결하고 HTTPS를 적용하고 요청이 어떤 경로를 통해 전달되는지 이해해야 전체 구조가 보인다.

현재 구성 흐름은 다음과 같다.

사용자 -> Cloudflare -> Nginx 오리진 서버

핵심은 사용자가 오리진 서버에 직접 접근하지 않는 구조를 만드는 것이다.

---

## 1. 도메인 구매 및 Cloudflare 등록

나는 Cloudflare에서 도메인을 구매했기 때문에 별도로 네임서버를 변경하는 과정은 없었다.

도메인을 Cloudflare에서 관리하게 되면 모든 DNS 요청은 Cloudflare를 통해 처리된다.  
이 시점부터 외부 요청은 Cloudflare를 거쳐 오리진 서버로 전달되는 구조가 된다.

---

## 2. DNS A 레코드 추가 및 Proxy 활성화

Cloudflare DNS 설정에서 A 레코드를 추가했다.
<figure>
  <img src="/assets/img/server/03/01.png" alt="DNS 설정">
  <figcaption>DNS 설정</figcaption>
</figure>


- Type : A  
- Name : example.com  
- IPv4 address : Public IP  
- Proxy status :  Proxied  

Proxy를 활성화하면 DNS 조회 시 오리진 서버 IP가 아니라 Cloudflare IP가 반환된다.

즉 사용자 -> Cloudflare -> 오리진 서버 흐름이 만들어진다.  

정상적으로 추가가 되면 아래와 같은 형태로 레코드가 등록된다.

<figure>
  <img src="/assets/img/server/03/02.png" alt="DNS 레코드 등록 상태">
  <figcaption>DNS 레코드 등록 상태</figcaption>
</figure>

이 단계에서 직접 서버를 외부에 노출하는 구조는 벗어난 상태다.

---

## 3. 오리진 서버에서 Let's Encrypt 인증서 발급

Cloudflare에서 HTTPS를 사용하더라도 오리진 서버 자체도 HTTPS를 사용해야 한다.

먼저 VM에 nginx를 설치하고 실행한다.

```bash
sudo apt install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

nginx가 정상적으로 실행되고 80 포트에서 응답 중인 상태여야 인증서 발급이 정상적으로 진행된다.

이후 certbot을 설치한다.

```bash
sudo apt install certbot python3-certbot-nginx
```

설치가 완료되면 인증서를 발급한다.

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Cloudflare 프록시가 활성화된 상태에서는 인증 과정에서 문제가 발생할 수 있다.  
이 경우 프록시를 잠시 비활성화한 뒤 인증서를 발급하고 다시 활성화하는 방식으로 처리할 수 있다.

정상적으로 발급이 완료되면 `/etc/letsencrypt/live` 경로에 인증서가 생성된다.

Certbot은 Let's Encrypt CA를 통해 인증서를 발급하고 nginx 설정에 자동으로 반영한다.

자동 갱신 동작을 확인하고 싶다면 다음 명령어를 사용한다.

```bash
sudo certbot renew --dry-run
```

이 과정을 통해 오리진 서버도 공식 CA 인증서를 사용하는 상태가 된다.

---

## 4. Cloudflare SSL/TLS 

Cloudflare 대시보드에서 SSL/TLS를 Full Strict로 설정한다.

<figure>
  <img src="/assets/img/server/03/03.png" alt="Cloudflare SSL/TLS">
  <figcaption>Cloudflare SSL/TLS</figcaption>
</figure>

Full Strict 모드는 다음과 같은 의미를 가진다.

- 사용자와 Cloudflare 구간은 HTTPS 암호화  
- Cloudflare와 오리진 구간도 HTTPS 암호화  
- 오리진 인증서가 신뢰 가능한 CA 인증서인지 검증  

즉 Cloudflare가 오리진 서버의 인증서를 실제로 검증하는 구조다.

---

## 현재 구성의 의미

외부 사용자는 HTTPS로 암호화된 연결을 통해 서비스에 접근한다.  
Cloudflare Proxy 구조가 적용되어 있으며 오리진 서버는 공식 CA 인증서를 사용한다.  
Cloudflare와 nginx 구간 역시 암호화와 검증이 이루어진다.

단순히 HTTPS를 적용한 것이 아니라 검증 가능한 TLS 구조를 갖춘 상태다.

다만 오리진 IP 직접 접근 제한이나 추가 방화벽 설정은 별도의 하드닝 단계로 남아 있다.

---

## 정리

Cloudflare Proxy와 Full Strict 모드 그리고 Let's Encrypt 인증서를 통해  
외부 사용자는 암호화된 HTTPS 환경에서 서비스에 접근하고  
오리진 서버 역시 신뢰 가능한 인증서를 기반으로 검증되는 구조를 갖추게 된다.
