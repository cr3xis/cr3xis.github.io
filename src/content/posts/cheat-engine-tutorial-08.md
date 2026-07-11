---
title: '[Cheat Engine] Tutorial: Step 8'
published: 2026-01-22
description: 'Cheat Engine Tutorial Step 8의 다중 포인터 추적 과정을 정리합니다.'
image: '/assets/img/CheatEngine/Step8/1.png'
tags:
  - 'Reversing'
  - 'CheatEngine'
  - 'Tutorial'
category: 'Cheat Engine'
draft: false
---


Step 8에서는 지금까지 배웠던 포인터 개념을 한 번 더 확장하여  
**여러 단계로 연결된 다중 포인터 구조를 직접 추적하는 과정**을 다루게 된다.  
이 단계의 목표는 최종 값을 가리키는 베이스 포인터(Base Pointer)를 찾아 CE 테이블에 올바르게 구성하는 것이다.

포인터 체인이 여러 단계로 이어져 있기 때문에  
각 단계의 포인터를 하나씩 직접 추적해 가며 최종 주소를 찾아내는 과정이 핵심이다.

---

## 1. 초기 값 검색 및 테이블로 내리기

튜토리얼 창에서 보이는 값(예: 3883)을 기준으로 Exact Value 스캔을 진행하여 값을 찾는다.
<figure>
  <img src="/assets/img/CheatEngine/Step8/1.png" alt="초기 값 3883 검색 후 테이블로 내리기">
  <figcaption>초기 값 3883 검색 후 테이블로 내리기</figcaption>
</figure>


값이 정확히 확인되면 테이블로 내리고 이제 이 값에 접근하는 포인터를 추적하는 단계로 넘어간다.

---

## 2. 1단계 포인터 확인

테이블에서 해당 값을 오른쪽 클릭해 **Find out what accesses this address**를 선택하면  
이 값에 접근하는 첫 번째 포인터를 확인할 수 있다.

<figure>
  <img src="/assets/img/CheatEngine/Step8/2.png" alt="&quot;Find out what accesses this address&quot;로 1단계 포인터 확인">
  <figcaption>&quot;Find out what accesses this address&quot;로 1단계 포인터 확인</figcaption>
</figure>

해당 명령어에서 사용되는 레지스터를 통해 실제 1단계 포인터 주소를 추적할 수 있다.

---

## 3. 1단계 포인터 주소 검색 후 테이블로 내리기

포인터가 가리키는 주소를 메모리에서 찾아 Hex 옵션을 사용하여 그대로 검색한다.

<figure>
  <img src="/assets/img/CheatEngine/Step8/3.png" alt="1단계 포인터 주소 검색 후 테이블로 내리기">
  <figcaption>1단계 포인터 주소 검색 후 테이블로 내리기</figcaption>
</figure>

찾은 포인터 주소 역시 테이블로 내린 뒤 다음 단계 포인터를 추적한다.

---

## 4. 2단계 포인터 추적

1단계 포인터에서 다시 **Find out what accesses this address**를 수행한다.

<figure>
  <img src="/assets/img/CheatEngine/Step8/4.png" alt="2단계 포인터 추적">
  <figcaption>2단계 포인터 추적</figcaption>
</figure>

이제 두 번째 포인터가 어떤 주소를 가리키는지 확인할 수 있다.

---

## 5. 2단계 포인터 주소 검색 및 테이블 등록

추적된 주소를 다시 Hex 검색하여 2단계 포인터 주소를 찾고 테이블에 추가한다.

<figure>
  <img src="/assets/img/CheatEngine/Step8/5.png" alt="2단계 포인터 주소 검색 후 테이블로 내리기">
  <figcaption>2단계 포인터 주소 검색 후 테이블로 내리기</figcaption>
</figure>

---

## 6. 3단계 포인터 추적

2단계 포인터에 대해 다시 Find out what accesses this address 기능을 사용하여 3단계 포인터를 확인한다.

<figure>
  <img src="/assets/img/CheatEngine/Step8/6.png" alt="3단계 포인터 추적">
  <figcaption>3단계 포인터 추적</figcaption>
</figure>

---

## 7. 3단계 포인터 주소 검색 및 테이블 추가

Hex 검색을 통해 3단계 포인터 주소를 찾고 테이블에 추가한다.

<figure>
  <img src="/assets/img/CheatEngine/Step8/7.png" alt="3단계 포인터 주소 검색 후 테이블로 내리기">
  <figcaption>3단계 포인터 주소 검색 후 테이블로 내리기</figcaption>
</figure>

---

## 8. 4단계 포인터 추적

3단계 포인터에서 다시 포인터 접근 정보를 확인하여 4단계 포인터를 찾는다.

<figure>
  <img src="/assets/img/CheatEngine/Step8/8.png" alt="4단계 포인터 추적">
  <figcaption>4단계 포인터 추적</figcaption>
</figure>

여기까지 추적했다면 거의 베이스 포인터에 도달한 상태다.

---

## 9. 베이스 포인터 주소 검색 및 테이블 추가

Hex 검색을 통해 베이스 포인터 주소를 찾고 테이블에 추가한다.

<figure>
  <img src="/assets/img/CheatEngine/Step8/9.png" alt="베이스 주소 검색 후 테이블로 내리기">
  <figcaption>베이스 주소 검색 후 테이블로 내리기</figcaption>
</figure>

이제 전체 포인터 체인 구성이 가능하다.

---

## 10. 다중 포인터 경로 구성

지금까지 찾은 포인터들을 순서대로 연결하여 CE 테이블에 포인터 경로를 구성한다.

<figure>
  <img src="/assets/img/CheatEngine/Step8/10.png" alt="다중 포인터 경로 구성">
  <figcaption>다중 포인터 경로 구성</figcaption>
</figure>

각 단계의 오프셋을 정확히 입력하면  어떤 상황에서도 항상 올바른 값에 접근할 수 있는 완전한 포인터 체인이 완성된다.

---

## 11. 값 변경 및 고정

마지막으로 값에 접근하여 원하는 값(예: 5000)으로 변경하고 Active 체크로 고정한다.

<figure>
  <img src="/assets/img/CheatEngine/Step8/11.png" alt="값을 5000으로 변경하고 고정">
  <figcaption>값을 5000으로 변경하고 고정</figcaption>
</figure>

이제 튜토리얼에서 Change pointer 버튼을 눌러도 포인터 체인이 자동으로 값을 추적하므로 값은 유지되고  
Next 버튼이 활성화된다.

---

## 마무리

Step 8은 여러 단계로 이어진 포인터 체인을 직접 따라가며  
실제 프로그램이 어떤 구조로 값을 관리하는지 이해할 수 있는 중요한 단계다.  
이 과정을 익히면 복잡한 게임 구조에서도 베이스 포인터와 오프셋 조합을 빠르게 파악할 수 있게 된다.
