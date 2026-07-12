---
title: 'Oracle Cloud 인스턴스 만들기'
published: 2026-04-10
description: 'Oracle Cloud에서 Ubuntu 인스턴스와 고정 공인 IP를 구성하는 과정을 정리합니다.'
image: '/assets/img/server/01/1.png'
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


## 1. 인스턴스 생성
<figure style="text-align: center;">
  <img src="/assets/img/server/01/1.png" alt="Create a VM instance" style="display: block; margin: 0 auto;">
  <figcaption>Create a VM instance</figcaption>
</figure>


Oracle Cloud에서 인스턴스를 생성한다.  

인스턴스는 결국 하나의 가상 머신이고 이 안에 우리가 사용할 Ubuntu 서버가 올라간다고 생각하면 이해가 쉽다.

---

## 2. Basic 설정

<figure style="text-align: center;">
  <img src="/assets/img/server/01/2.png" alt="Basic information" style="display: block; margin: 0 auto;">
  <figcaption>Basic information</figcaption>
</figure>

생성할 인스턴스의 이름을 정해주면 되고 나머지는 기본값으로 설정한다.  

Change Image를 통해 원하는 OS를 선택하면 되는데 나는 Ubuntu 22.04 로 선택했다.  

---

## 3. Networking 설정

<figure style="text-align: center;">
  <img src="/assets/img/server/01/3.png" alt="Networking" style="display: block; margin: 0 auto;">
  <figcaption>Networking</figcaption>
</figure>

여기서는 VNIC과 Subnet를 설정하는 부분인데 인스턴스가 어떤 네트워크 안에 속할지를 결정하는 설정이고  
하나의 VCN 안에 인스턴스가 배치된다고 이해하면 충분하다.  

SSH 관련 설정도 이 단계에서 함께 설정한다.

<figure style="text-align: center;">
  <img src="/assets/img/server/01/4.png" alt="SSH" style="display: block; margin: 0 auto;">
  <figcaption>SSH</figcaption>
</figure>

public / private 키를 다운로드 받고 잘 보관해야 한다.   
나는 사용자 폴더 내 .ssh 에 저장했다.  

---

## 4. 인스턴스 생성 완료

<figure style="text-align: center;">
  <img src="/assets/img/server/01/5.png" alt="Instance" style="display: block; margin: 0 auto;">
  <figcaption>Instance</figcaption>
</figure>

생성이 완료되면 인스턴스가 Running 상태로 올라오며 서버 자체는 준비가 된거다.   

처음엔 별도로 설정하지 않아도 공인 IP가 인스턴스 자체에 붙어있는 줄 알았는데 그게 아니라  
별도로 설정을 해줘야 되는 구조인듯 하다.  

그래서 생성이 끝났다고 바로 외부에서 접속이 가능한 것은 아니다.

---

## 5. Reserved Public IP 연결

<figure style="text-align: center;">
  <img src="/assets/img/server/01/6.png" alt="Reserved Public IPs" style="display: block; margin: 0 auto;">
  <figcaption>Reserved Public IPs</figcaption>
</figure>

공인 IP가 자동으로 할당되지 않았다면 Networking 메뉴에서 Reserved Public IPs로 이동한다.

<figure style="text-align: center;">
  <img src="/assets/img/server/01/7.png" alt="Reserved Public IPs" style="display: block; margin: 0 auto;">
  <figcaption>Reserved Public IPs</figcaption>
</figure>

여기서 Reserved Public IP를 생성 한 뒤 인스턴스의 VNIC으로 지정해 연결하면 된다.  

프리티어 기준으로는 최대 2개까지 발급이 가능하다.  
이 방식의 가장 큰 장점은 인스턴스를 중지했다가 다시 시작해도 IP가 유지된다는 점이다.  

---

## 6. 인스턴스에 공인 IP 연결

<figure style="text-align: center;">
  <img src="/assets/img/server/01/8.png" alt="Attached VNICs" style="display: block; margin: 0 auto;">
  <figcaption>Attached VNICs</figcaption>
</figure>

이제 공인 IP를 실제 VNIC에 연결해준다.  
Compute에서 해당 인스턴스로 들어간 뒤 Networking - Attached VNICs를 선택한다.  
해당 네트워크를 누르게 되면 아래와 같이 뜬다.  

<figure style="text-align: center;">
  <img src="/assets/img/server/01/9.png" alt="IP administration" style="display: block; margin: 0 auto;">
  <figcaption>IP administration</figcaption>
</figure>

IP administration 탭으로 들어가면 현재 Private IP 정보가 표시된다.  
연결하고자 하는 네트워크 우측의 점 3개 옵션에 - Edit 을 클릭한다.  

<figure style="text-align: center;">
  <img src="/assets/img/server/01/10.png" alt="Edit Private IP Address" style="display: block; margin: 0 auto;">
  <figcaption>Edit Private IP Address</figcaption>
</figure>

No public IP를 Reserved public IP로 변경한다.  
Reserved IP Address 항목에서 이전에 생성해둔 공인 IP를 선택한 뒤 Update를 누르면 연결이 완료된다.  
이제 외부와 통신 가능한 상태가 된다.

---

## 7. 정상적으로 연결되었는지 확인

<figure style="text-align: center;">
  <img src="/assets/img/server/01/11.png" alt="인스턴스 연결 상태 확인" style="display: block; margin: 0 auto;">
  <figcaption>인스턴스 연결 상태 확인</figcaption>
</figure>

Instances 화면에서 해당 인스턴스를 확인해보면 Public IP가 표시된다.  
정상적으로 IP가 붙어 있다면 이제 SSH를 통해 외부에서 접속을 시도할 수 있다.

---