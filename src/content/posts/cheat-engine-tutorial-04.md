---
title: '[Cheat Engine] Tutorial: Step 4'
published: 2026-01-14
description: 'Cheat Engine Tutorial Step 4의 Float와 Double 값 검색 과정을 정리합니다.'
image: '/assets/img/CheatEngine/Step4/1.png'
tags:
  - 'Reversing'
  - 'CheatEngine'
  - 'Tutorial'
category: 'Cheat Engine'
draft: false
---


Step 4에서는 **자료형(Float / Double)** 에 따라 값을 스캔하는 방법을 직접 경험하게 된다.  
체력(Health)은 Float, 탄약(Ammo)은 Double 형태로 저장되어 있기 때문에  
이전 단계들처럼 단순히 4 Bytes로만 검색하면 제대로 된 주소가 나오지 않는다.  
자료형을 올바르게 지정하는 것이 이 단계의 핵심이다.

---

## 1. 체력 값 검색 — Float 형태

튜토리얼 창에서 체력 값을 확인한 뒤, CE에서 Value Type을 Float로 설정한다.  
아래 화면은 Float 타입으로 Health 값을 검색한 모습이다.
<figure>
  <img src="/assets/img/CheatEngine/Step4/1.png" alt="체력주소 찾기">
  <figcaption>체력주소 찾기</figcaption>
</figure>


검색 결과가 적절히 좁혀졌다면 Hit me를 눌러 값이 함께 변하는지 확인하며 실제 주소를 특정한다.

---

## 2. 탄약 값 검색 — Double 형태

탄약 값은 Double 형태로 저장되어 있다.  
따라서 Value Type을 Double로 설정하고 같은 방식으로 검색한다.

<figure>
  <img src="/assets/img/CheatEngine/Step4/2.png" alt="탄약주소 찾기">
  <figcaption>탄약주소 찾기</figcaption>
</figure>

체력 때와 마찬가지로 Fire 버튼을 눌러 값이 변경되는지 확인하면서 올바른 주소가 맞는지 확인하면 된다.

---

## 3. 찾은 값을 5000으로 변경하기

Health와 Ammo 두 값 모두 찾았다면 CE 테이블에 추가한 뒤  
Value를 더블 클릭하여 5000으로 수정한다.

<figure>
  <img src="/assets/img/CheatEngine/Step4/3.png" alt="각 주소의 벨류를 5000으로 변경">
  <figcaption>각 주소의 벨류를 5000으로 변경</figcaption>
</figure>

두 값이 모두 5000 이상으로 설정되면 Step 4의 조건이 충족되어 Next 버튼이 활성화된다.

---

## 마무리

Step 4는 값의 자료형을 올바르게 이해하는 것이 가장 중요한 단계다.  
Float, Double 같은 자료형이 실제 메모리에서 어떻게 저장되고  
왜 정확한 검색이 필요한지를 직접 경험할 수 있다.  
