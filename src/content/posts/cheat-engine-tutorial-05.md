---
title: '[Cheat Engine] Tutorial: Step 5'
published: 2026-01-16
description: 'Cheat Engine Tutorial Step 5의 코드 추적과 NOP 패치 과정을 정리합니다.'
image: '/assets/img/CheatEngine/Step5/1.png'
tags:
  - 'Reversing'
  - 'CheatEngine'
  - 'Tutorial'
category: 'Cheat Engine'
draft: false
---


Step 5에서는 **값이 실행할 때마다 달라지는 주소를 직접 수정하는 대신**,  
그 주소에 값을 쓰는 **코드 자체를 차단하는 방식**으로 문제를 해결하게 된다.  
주소가 매번 달라지기 때문에 단순히 값을 검색해서 바꾸는 방식으로는 해결할 수 없다.  
따라서 이 단계에서는 코드가 어떤 주소에 값을 쓰는 순간을 포착하고, 그 코드를 NOP 처리하는 과정이 핵심이다.

---

## 1. 초기 값 검색

튜토리얼 창에서 보이는 값은 100으로 시작한다.  
CE에서 100을 입력하여 첫 스캔을 진행한다.
<figure>
  <img src="/assets/img/CheatEngine/Step5/1.png" alt="초기 값 100 검색">
  <figcaption>초기 값 100 검색</figcaption>
</figure>


검색 결과는 여러 개가 나오지만, 이후 값이 바뀔 때 후보가 빠르게 좁혀진다.

---

## 2. 값 변경 후 다시 검색하기

튜토리얼 창에서 **Change value** 버튼을 클릭하면 값이 155로 변한다.  
CE에서도 검색 값을 155로 바꾸고 *Next Scan*을 실행해 실제 값이 있는 주소만 남긴다.

<figure>
  <img src="/assets/img/CheatEngine/Step5/2.png" alt="&quot;Change value&quot; 클릭 후 값 155로 검색">
  <figcaption>&quot;Change value&quot; 클릭 후 값 155로 검색</figcaption>
</figure>

여러 번 반복하게 되면 최종적으로 하나의 주소만 남게 된다.

---

## 3. 해당 주소에 어떤 코드가 접근하는지 확인하기

남은 주소를 테이블에 추가한 뒤  
오른쪽 클릭하여 **Find out what writes to this address**를 선택한다.

<figure>
  <img src="/assets/img/CheatEngine/Step5/3.png" alt="&quot;Find out what writes to this address&quot; 선택">
  <figcaption>&quot;Find out what writes to this address&quot; 선택</figcaption>
</figure>

이 시점에서는 아직 아무 코드도 나타나지 않는다.  
이제 값을 변경하여 해당 주소에 쓰기를 수행하는 순간을 포착해야 한다.

---

## 4. Change value를 눌러 코드 실행을 포착하기

튜토리얼에서 다시 **Change value** 버튼을 누르면 CE의 추적 창에  
해당 주소에 값을 쓰는 코드가 기록된다.

<figure>
  <img src="/assets/img/CheatEngine/Step5/4.png" alt="&quot;Change value&quot; 클릭 후 나타난 코드">
  <figcaption>&quot;Change value&quot; 클릭 후 나타난 코드</figcaption>
</figure>

이 명령어가 바로 값이 변경되는 원인이며, 이 코드를 차단하면 값이 더 이상 바뀌지 않게 된다.

---

## 5. 코드를 NOP으로 교체하기

이제 나타난 명령어를 선택하고 **Replace**를 눌러 NOP으로 교체한다.  
NOP은 아무 동작도 하지 않는 명령어이기 때문에 코드가 실행되더라도  
실제 주소에 값이 쓰이지 않게 된다.

<figure>
  <img src="/assets/img/CheatEngine/Step5/5.png" alt="코드를 NOP으로 교체">
  <figcaption>코드를 NOP으로 교체</figcaption>
</figure>

이제 Change value 버튼을 다시 눌러 보면 값이 더 이상 바뀌지 않고 그대로 유지되는 것을 확인할 수 있다.  
이 상태가 되면 Step 5의 조건이 충족되며 Next 버튼이 활성화된다.

---

## 마무리

Step 5는 단순한 값 변경이 아니라, 어떤 코드가 값을 수정하는지 추적하고 그 동작을 차단하는  
실전적인 CE 활용 단계다.  
게임 해킹이나 메모리 분석에서도 자주 사용하는 기법이기 때문에  
이 과정을 정확히 이해해두면 이후 단계를 훨씬 수월하게 진행할 수 있다.
