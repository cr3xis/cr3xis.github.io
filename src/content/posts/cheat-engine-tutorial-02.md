---
title: '[Cheat Engine] Tutorial: Step 2'
published: 2026-01-10
description: 'Cheat Engine Tutorial Step 2의 Exact Value Scan과 값 수정 과정을 정리합니다.'
image: '/assets/img/CheatEngine/Step2/2.png'
tags:
  - 'Reversing'
  - 'CheatEngine'
  - 'Tutorial'
category: 'Cheat Engine'
draft: false
---


Step 2는 Cheat Engine의 가장 기본적인 기능인 **(Exact Value Scan)**을 활용하여  
게임 내부 값을 직접 수정해보는 단계다.  
초보자라도 따라가기 쉬운 구조이지만, 실제로는 메모리 구조를 이해하는 데 중요한 기반이 된다.

이번 단계에서는 **Health 값(초기 100)**을 찾아 **1000으로 변경**하면 다음 단계로 넘어갈 수 있다.

---

## 1. 초기 값 스캔

튜토리얼 창에서 Health가 **100**으로 표시되어 있다.  
따라서 CE에서 가장 먼저 해야 할 일은 이 값을 메모리에서 그대로 찾아오는 것이다.

아래 화면은 CE에서 Value를 100으로 입력하고 First Scan을 수행한 모습이다.
<figure>
  <img src="/assets/img/CheatEngine/Step2/2.png" alt="값 100으로 초기 스캔">
  <figcaption>값 100으로 초기 스캔</figcaption>
</figure>


스캔 결과로 많은 주소가 검색된다.  
이는 “100이라는 숫자”가 메모리 곳곳에서 사용되기 때문이고,  
우리가 원하는 Health 값만 특정하기 위해선 값이 변하는 순간을 활용해야 한다.

---

## 2. 값이 변한 뒤 다시 스캔하기

튜토리얼 창에서 Hit me 버튼을 클릭하면 Health가 감소한다.  
예를 들어 95가 되었다면 CE에서도 검색 값을 95로 바꾸고 Next Scan을 수행한다.

<figure>
  <img src="/assets/img/CheatEngine/Step2/3.png" alt="&quot;Hit me&quot; 클릭 후 다음 스캔">
  <figcaption>&quot;Hit me&quot; 클릭 후 다음 스캔</figcaption>
</figure>

이 과정을 반복하면 입력 값이 변한 주소만 남게 되므로 최종적으로 한두 개의 주소만 남게 된다.

---

## 3. 주소를 테이블로 추가하기

남은 주소가 실제 Health 주소인지 확인하기 위해 CE 테이블에 추가한다.

<figure>
  <img src="/assets/img/CheatEngine/Step2/4.png" alt="주소를 주소 목록에 추가">
  <figcaption>주소를 주소 목록에 추가</figcaption>
</figure>

테이블에 추가된 값을 확인하면서 Hit me를 눌러 값이 연동되는지 살펴보면 실제 Health 주소인지 바로 알 수 있다.

---

## 4. 값을 직접 수정해보기

테이블에 추가한 값은 이제 자유롭게 수정할 수 있다.  
Value를 더블 클릭하여 1000으로 바꾸면 튜토리얼 창에도 바로 반영된다.

<figure>
  <img src="/assets/img/CheatEngine/Step2/5.png" alt="값을 1000으로 변경">
  <figcaption>값을 1000으로 변경</figcaption>
</figure>

Health 값이 1000으로 설정되면 Step 2의 조건이 충족되어 Next 버튼이 활성화된다.

---

## 마무리

Step 2는 단순한 값 변경 작업처럼 보이지만  
메모리 값이 어떤 방식으로 검색되고 좁혀지는지 직접 경험할 수 있는 기본 단계다.  
이 흐름을 이해하면 이후 단계에서 포인터 분석이나 코드 추적을 할 때 훨씬 수월하게 접근할 수 있다.
