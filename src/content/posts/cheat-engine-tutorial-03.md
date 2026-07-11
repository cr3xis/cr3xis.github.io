---
title: '[Cheat Engine] Tutorial: Step 3'
published: 2026-01-12
description: 'Cheat Engine Tutorial Step 3의 Unknown Initial Value 검색 방법을 정리합니다.'
image: '/assets/img/CheatEngine/Step3/1.png'
tags:
  - 'Reversing'
  - 'CheatEngine'
  - 'Tutorial'
category: 'Cheat Engine'
draft: false
---


Step 3에서는 **초기 값을 모르는 상태에서 값이 어떻게 변하는지를 기반으로 실제 주소를 찾아내는 과정**을 경험하게 된다.  
이전 단계와는 달리 Health 값이 0~500 사이의 임의의 값으로 설정되어 있기 때문에,  
정확한 값을 입력해서 스캔할 수 없다는 점이 이 단계의 핵심이라고 볼 수 있다.

따라서 이번 단계에서 가장 중요한 개념은  
**Unknown initial value**와 **값의 변화에 따른 필터링**이다.

---

## 1. 초기 스캔 — Unknown initial value 사용

튜토리얼 창에 표시된 Health 값은 숫자로 정확히 주어지지 않는다.  
따라서 CE에서도 처음에는 어떤 값인지 알 수 없다.  
이럴 때 사용하는 옵션이 바로 *Unknown initial value*이다.

아래 화면은 Unknown initial value로 전체 메모리를 첫 스캔한 모습이다.
<figure>
  <img src="/assets/img/CheatEngine/Step3/1.png" alt="Unknown initial value로 초기 스캔">
  <figcaption>Unknown initial value로 초기 스캔</figcaption>
</figure>



검색 결과는 매우 많은 주소가 나오는데,  
지금은 그중 어떤 것이 Health인지 알 수 없으므로  
값의 변화를 통해 후보를 줄여야 한다.

---

## 2. 값이 감소한 뒤 다시 스캔하기

튜토리얼 창에서 Hit me를 누르면 Health가 감소한다.  
이때 CE에서 Scan Type을 *Decreased value*로 변경하고 Next Scan을 수행하면  
Health처럼 실제로 값이 감소한 주소만 남게 된다.


<figure>
  <img src="/assets/img/CheatEngine/Step3/2.png" alt="Hit me 클릭 후 Decreased value 스캔">
  <figcaption>Hit me 클릭 후 Decreased value 스캔</figcaption>
</figure>

이 과정을 여러 번 반복하게 되면 Health 값처럼 일정하게 줄어드는 주소들만 남기 때문에 최종적으로 몇 개의 후보만 남게 된다.

---

## 3. 남은 주소 확인 후 테이블로 내리기

필터링된 주소 목록에서 값이 일정 범위(0~500)에 있는 주소를 선택하여 CE 테이블에 추가한다.


<figure>
  <img src="/assets/img/CheatEngine/Step3/3.png" alt="필터링된 주소에서 체력 값 선택">
  <figcaption>필터링된 주소에서 체력 값 선택</figcaption>
</figure>

주소를 테이블에 추가한 뒤 튜토리얼 창에서 Hit me를 눌러 값이 함께 변하는지 확인하면 실제로 Health를 나타내는 값인지 쉽게 구별할 수 있다.

---

## 4. Health 값을 직접 수정하기

Health 주소가 맞다는 것을 확인했다면 이제 값을 직접 변경해볼 차례다.  
테이블에서 해당 Value를 더블 클릭하여 5000으로 수정하면 튜토리얼 창에서도 값이 즉시 반영된다.

<figure>
  <img src="/assets/img/CheatEngine/Step3/4.png" alt="체력 값을 5000으로 변경">
  <figcaption>체력 값을 5000으로 변경</figcaption>
</figure>

Health 값이 충분히 큰 값으로 설정되면  
Step 3의 조건을 충족하여 Next 버튼이 활성화된다.

---

## 마무리

Step 3는 눈에 보이는 값만을 기준으로 직접 검색하던 방식에서 벗어나,  
**값이 어떻게 변화하는지를 이용해 메모리 주소를 특정하는 방법**을 익히는 단계다.  
Unknown initial value, Decreased value 같은 옵션이 어떤 상황에서 쓰이는지를 이해하면 앞으로 더 복잡한 구조의 프로그램을 분석할 때 큰 도움이 된다.
