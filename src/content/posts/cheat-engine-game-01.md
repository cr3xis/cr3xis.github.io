---
title: '[Cheat Engine] Games: Step 1'
published: 2026-02-01
description: 'Cheat Engine Games Step 1에서 숨겨진 값을 검색하고 수정하는 과정을 정리합니다.'
image: '/assets/img/CheatEngine/Games1/1.png'
tags:
  - 'Reversing'
  - 'CheatEngine'
  - 'Tutorial'
  - 'Games'
category: 'Cheat Engine'
draft: false
---


Games Step 1은 기존 튜토리얼보다 훨씬 실제 게임에 가까운 환경에서  
값을 탐색하고 원하는 방식으로 조작하는 경험을 제공한다.  
특히 이 단계는 값이 단순하지 않게 숨겨져 있기 때문에,  
기존에 배운 **정확한 값 검색**, **값 변화 기반 검색**, **Unknown Initial Value** 개념을  
실전적으로 어떻게 활용해야 하는지를 자연스럽게 익히게 된다.

---

## 1. 게임 시작과 구조 파악

게임을 실행하면 단순한 형태의 슈팅 화면이 나타난다.  
플레이어는 총을 발사할 수 있고, 발사할 때마다 총알 수가 줄어들게 되어 있다.  
우리가 해야 할 일은 **총알 값이 저장된 주소를 찾아 수정하는 것**이다.
<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games1/1.png" alt="게임 실행 시 화면" style="display: block; margin: 0 auto;">
  <figcaption>게임 실행 시 화면</figcaption>
</figure>


처음 총알 값은 5로 표시된다.  
보통 튜토리얼 단계에서는 이런 단순한 값은 바로 Exact Value Scan으로 찾을 수 있지만,  
Games 단계는 다르다.  
겉으로 보이는 값과 메모리에 저장된 값이 일치하지 않을 수 있다.

---

## 2. 일반적인 Exact Value Scan 시도

처음에는 5를 입력해 검색해 보고, 총을 한 발 쏜 뒤 4로 바뀐 값을 기준으로 Next Scan을 시도한다.

<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games1/2.png" alt="총알 값 검색 (5 → 4) 후 Found: 0" style="display: block; margin: 0 auto;">
  <figcaption>총알 값 검색 (5 → 4) 후 Found: 0</figcaption>
</figure>

단 한 개의 주소도 검색되지 않는다.  
값이 보이는 그대로 저장되어 있지 않기 때문에 단순 스캔 방식으로는 접근할 수 없다는 의미다.  
이런 상황에서 떠올려야 할 방법이 바로 **Unknown Initial Value**다.

---

## 3. Unknown Initial Value로 총알 값 찾기

정확한 값이 무엇인지 모르기 때문에  
Scan Type을 Unknown Initial Value로 설정한 뒤 첫 스캔을 진행한다.  
이 상태에서 총을 발사하여 값이 감소하는 순간을 기준으로 **Decreased Value** 스캔을 반복한다.

<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games1/3.png" alt="Unknown Initial Value 스캔으로 찾은 총알 값 주소" style="display: block; margin: 0 auto;">
  <figcaption>Unknown Initial Value 스캔으로 찾은 총알 값 주소</figcaption>
</figure>

몇 번 반복하면 총을 발사할 때마다 값이 함께 변하는 주소 하나만 남게 된다.  
이 주소가 실제 총알 값을 관리하는 메모리 영역이다.

여기까지 도달하면 총알 값을 원하는 값으로 직접 수정하거나 유지할 수 있다.

---

## 4. 총알 값 Freeze로 고정시키기

찾아낸 주소를 테이블에 추가한 뒤 값을 수정하고 **Freeze(고정)** 기능을 활성화하면  
총을 계속 쏴도 값이 줄어들지 않게 된다.

<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games1/4.png" alt="총알 값 Freeze 후" style="display: block; margin: 0 auto;">
  <figcaption>총알 값 Freeze 후</figcaption>
</figure>

이제 플레이어는 무한한 탄약을 가진 상태로 게임을 진행할 수 있다.

---

## 마무리

Games Step 1은 단순한 값 스캔으로 해결되지 않는 상황에서 어떤 접근 방식을 사용해야 하는지 알려주는 단계다.  
겉으로 보이는 숫자가 항상 실제 메모리에 저장된 값과 일치하지 않기 때문에 Unknown Initial Value와  
값 변화 기반 스캔은 게임 메모리 분석에서 매우 중요한 기초 기술이다.

이 단계를 이해하면 이후 Games 시리즈에서 등장하는 더 복잡한 값 구조나 포인터 기반 변수들도 훨씬 자연스럽게 분석할 수 있다.
